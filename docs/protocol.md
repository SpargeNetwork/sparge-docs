# Protocol Guide

This guide describes the protocol implemented by the current public-alpha release. Values such as addresses, timing, supply, and fee parameters come from configuration and genesis data; check [Configuration](reference/configuration.md) and `GET /api/genesis` for a particular chain instance.

## Overview

Sparge is currently a single-producer chain. The official producer orders transactions and creates blocks. Observer nodes independently validate and apply those blocks, but there is no multi-producer election, fork choice, finality protocol, P2P gossip, external governance, or slashing in this release.

This makes censorship resistance and liveness operational properties of the producer rather than guarantees of decentralized consensus.

## Chain

Chain identity is bound to `chainId`, the genesis hash, `protocolVersion`, and `economicsVersion`. Nodes reject stored or synchronized state that does not match their configured identity.

Blocks form a height-ordered hash chain. Validation checks height continuity, previous hash, recomputed block hash, transaction count, protocol/economics identity, state root, and supported transaction types. Runtime invariants and deterministic replay add operational verification but do not introduce new consensus rules.

Amounts use integer micro-units. Floating-point arithmetic is not used for consensus state.

## Addresses

- An Ed25519 public key is 32 raw bytes.
- `publicKeyHex` is 64 lowercase hexadecimal characters.
- Address = `spg_` + `base58(sha256(publicKeyBytes)[0..20])`.

## Transactions

The canonical UTF-8 signing message is:

```text
type|chainId|from|to|amountMicro|feeMicro|nonce|publicKeyHex|sponsor|participant|memo?
```

Fields are separated by `|`; numeric values are canonical decimal strings. The optional memo is limited to 128 characters. The transaction ID excludes the signature:

```text
txid = sha256(utf8(canonicalMessage))
```

Client-submittable transaction types are:

- `transfer`
- `register_participant`
- `unregister_participant`
- `heartbeat`

System-generated block entries may include `participant_reward`, `holder_reward`, `node_reward`, `treasury_reward`, `node_pool_accrual`, and `holder_pool_accrual`.

Submitted transactions pass request-shape, chain ID, signature, nonce, fee, balance, duplicate, mempool, and type-specific state checks before admission. Admission is not confirmation: a pending transaction changes chain state only when included in a valid block.

## State

Canonical state includes balances, nonces, stakes, participant records, reward-pool accounting, balance history, and chain metadata. Each block commits a deterministic state root. SQLite writes block data, indexes, state, and metadata atomically.

The producer mempool is process-local and is not canonical state. A restart clears pending transactions and users may need to resubmit them.

## Participation

`register_participant` is an on-chain transaction:

- `from` is the sponsor, signer, fee payer, and nonce owner.
- `participant` is the address being registered.
- The sponsor locks the configured bond as refundable collateral.

The genesis operator has one free-registration opportunity when all conditions hold:

- `tx.from` equals `genesisOperatorAddress`
- `participant` equals `from`
- current height is below `genesisFreeBlocks`
- the opportunity has not already been used

The fee may be zero during that window. After it is used or expires, the normal bond and fee rules apply.

A participant is active when its `lastSeenHeight` is within the configured active window. Any accepted on-chain transaction sent from the participant updates that height. `unregister_participant` removes the participant and releases the bond to its sponsor.

The participant heartbeat transaction is an on-chain liveness action. It is unrelated to the off-chain observer heartbeat described in the [Observer Node guide](observer.md#heartbeats-and-network-health).

## Block rewards

The current per-block split is:

| Recipient | Share | Behavior |
| --- | ---: | --- |
| Active participants | 15% | Split equally; goes to treasury when none are active. |
| Node holders | 70% | Accrues in the node pool for scheduled payout. |
| Treasury | 10% | Credited directly, plus fees and deterministic remainders. |
| Eligible holders | 5% | Accrues in the holder pool for scheduled payout. |

All transaction fees go to the treasury. Node and holder pools accrue at transparent, unspendable system addresses configured as `nodePoolAddress` and `holderPoolAddress`.

Integer division is deterministic. Every remainder goes to the treasury so each block's distribution remains conserved.

## Holder rewards

Holder eligibility uses a rolling 14-day average balance, measured in blocks:

```text
windowBlocks = floor((14 * 24 * 60 * 60) / blockTimeSeconds)
```

At a 51-second block time this is approximately 23,717 blocks. The exact integer is derived from chain parameters.

An address is eligible when its average balance over the window is at least 1,000 SPRG. If `H` is the holder pool, `avg_i` is an eligible address's average, and `Total` is the sum of eligible averages:

```text
reward_i = H * avg_i / Total
```

Payout occurs when `height - lastPayoutHeight >= windowBlocks`. If no holder is eligible, the complete holder pool goes to the treasury. The chain stores balance-history entries at each balance change so the average can be calculated deterministically.

For a new chain, enabling this behavior from genesis provides a complete first window. On pre-existing state without earlier balance history, the first window assumes a flat balance before an address's first recorded change; a protocol/storage migration may therefore require a documented chain reset.

## Economics limitations

- Equal participant rewards resist Sybil behavior only to the extent that bond and sponsorship constraints do.
- Rolling holder eligibility can still be strategy-tested near window boundaries.
- Pool payout and timing parameters may change before a stable release.
- There is no external governance or slashing mechanism.
- A single producer provides no protocol-backed liveness or censorship guarantee.

The economics smoke suite exercises sponsor caps, free-rider rejection, participant reward behavior, and holder-window boundaries. Tests provide implementation evidence, not an economic-security proof.

## Known limitations

Sparge public alpha is not formal verification, a consensus-safety proof, or an audited decentralized network. The current block format has no separate producer-signature verification rule. Dedicated protocol-correctness coverage for signed transaction bursts and the complete participant lifecycle remains a known testing gap; runtime invariant and deterministic replay suites cover related behavior but are not a substitute for that focused suite.
