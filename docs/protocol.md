# Protocol

This guide explains the public-alpha chain model. It describes what applications and users can rely on without covering producer deployment or internal implementation.

## Current network model

Sparge currently uses one official producer. It orders accepted transactions and creates blocks. Observer Nodes independently synchronize, validate, and store the chain state.

Observers make chain comparison and independent verification possible, but they do not produce blocks, vote on blocks, or provide a decentralized consensus/finality mechanism. Producer availability, ordering, and censorship resistance are operational properties in this release.

## Chain identity

A Sparge chain is identified by:

- `chainId`
- `genesisHash`
- `protocolVersion`
- `economicsVersion`

Wallets and applications should verify these values before signing or indexing. A matching name or token symbol is not sufficient proof that two endpoints serve the same chain.

## Blocks

Blocks have sequential heights beginning at genesis, a timestamp, previous-block hash, chain identity, transaction list, fee and reward data, and a state root.

The previous hash links each block to its predecessor. The state root commits to the resulting canonical state. Explorers and indexers should preserve the last processed height and hash and stop if continuity changes unexpectedly.

The configured target block interval is exposed by `/api/status`. A target is not a guarantee: applications should use actual timestamps and tolerate delayed blocks.

## Accounts

An account is identified by an address derived from a 32-byte Ed25519 public key:

```text
spg_ + base58(sha256(publicKeyBytes)[0..20])
```

Canonical account state includes a balance and nonce. Participation records and balance history add protocol-specific state for eligible accounts.

The nonce prevents replay and orders transactions from one sender. Submitted transactions must use the next nonce expected by the producer. Applications sending multiple transactions from one account should serialize them.

## Amounts and fees

Balances, transfers, fees, pools, and rewards use integer base units represented as decimal strings. The current chain exposes 9 decimals, but clients should read `decimals` from status.

Fees are paid by the sender and credited to the treasury. The current minimum is exposed as `minFeeMicro`. Applications must avoid floating-point arithmetic.

## Transactions

Client-submittable transaction types are:

- `transfer`: moves an amount from one account to another
- `register_participant`: registers a participant and may lock a sponsor bond
- `unregister_participant`: removes the sender's participation record and releases its bond
- `heartbeat`: refreshes an on-chain participant's activity

The participant heartbeat is an on-chain transaction. It is different from an Observer Node's private network-health heartbeat.

### Canonical message

Transactions sign this UTF-8 field order:

```text
type|chainId|from|to|amountMicro|feeMicro|nonce|publicKeyHex|sponsor|participant|memo
```

The transaction ID is SHA-256 of the canonical message bytes. The Ed25519 signature covers the same bytes and is excluded from the transaction ID.

Unused signed fields are empty strings. Memos are optional, public, and limited to 128 UTF-8 bytes.

### Lifecycle

1. A wallet reads chain identity, balance, fee, and nonce.
2. It constructs and signs the canonical message locally.
3. The producer validates and queues the transaction.
4. Queued transactions remain non-canonical until included in a block.
5. Block inclusion updates account and protocol state.

The mempool is temporary and is cleared by a producer restart. Queued is therefore not the same as confirmed.

## Participation

A participant is registered through an on-chain transaction. The signing `from` account is the sponsor, fee payer, nonce owner, and source of the refundable bond. The `participant` field identifies the account being registered.

The configured genesis operator has one free self-registration opportunity during the first `genesisFreeBlocks`, provided it has not already been used. All other registrations follow normal fee, bond, and sponsorship rules.

A participant is active when its `lastSeenHeight` falls within the protocol activity window. An accepted transaction sent by the participant updates that height. Unregistering removes the participant and returns the bond to its sponsor.

## Rewards and economics

The current per-block distribution is:

| Destination | Share | Rule |
| --- | ---: | --- |
| Active participants | 15% | Split equally; sent to treasury when no participant is active. |
| Node-holder pool | 70% | Accumulates for scheduled distribution. |
| Treasury | 10% | Credited directly, plus fees and integer remainders. |
| Eligible-holder pool | 5% | Accumulates for scheduled distribution. |

Pool addresses are transparent system addresses and are not ordinary spendable wallets.

### Holder eligibility

Holder rewards use a rolling 14-day average balance measured in blocks:

```text
windowBlocks = floor((14 * 24 * 60 * 60) / blockTimeSeconds)
```

An address is eligible when its average balance reaches 1,000 SPRG. Eligible addresses receive a proportional share:

```text
reward = holderPool * addressAverage / totalEligibleAverage
```

Integer remainders go to the treasury. When no address is eligible at payout, the complete holder pool goes to the treasury.

## State

Canonical state includes:

- balances and nonces
- participant and sponsor records
- reward-pool accounting
- balance history used by holder eligibility
- total supply and mint accounting
- latest chain identity, height, hash, and state root

The public API exposes selected state and derived summaries. Application-specific records such as game inventory, marketplace listings, invoices, or user profiles are not Sparge protocol state.

## Known limitations

- one official producer and no multi-producer consensus
- no protocol-level finality depth beyond inclusion in the current chain
- no smart-contract runtime
- no token or NFT creation standard
- no WebSocket, SSE, webhook, or subscription API
- temporary, non-durable mempool
- economics and compatibility may change before stable release
- no formal verification or independent security audit implied by runtime tests

Builders should communicate these limits and design migrations, retries, and reconciliation accordingly.
