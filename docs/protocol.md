# How Sparge Works

This page explains the current Public Alpha network. It describes the existing Sparge chain; it is not a guide for creating another chain.

## Network model

Sparge currently uses one official producer. The producer orders accepted transactions and creates blocks. Observer nodes independently download, verify, and store those blocks, but they do not produce blocks, select producers, or participate in consensus.

This design provides independent validation, not decentralized producer liveness, censorship resistance, fork choice, or finality. Observer heartbeats report network health only and cannot change chain state or consensus.

## Blocks and confirmation

Transactions enter a process-local mempool before a producer includes them in a block. A response saying `queued` or a wallet state saying Pending is not confirmation.

Each block commits to its previous block, transactions, chain identity, and deterministic state root. Explorer transaction pages use the complete 64-character transaction ID.

Pending transactions can disappear after a producer restart and may need to be submitted again. Confirmed transactions remain in chain history.

## Addresses and keys

Sparge wallets use Ed25519 keys. Public addresses begin with `spg_`. Private keys authorize transactions and must never be sent to a node, Explorer, Sponsor, or support channel.

Addresses, balances, transactions, participation records, and sponsorship relationships are public. A wallet name is only a local browser label.

## Transaction types

Users can submit:

- `transfer`: move SPRG to another address;
- `register_participant`: register a Participant and lock a Sponsor Bond;
- `heartbeat`: refresh on-chain Participant activity;
- `unregister_participant`: remove a Participant and release its bond.

The browser wallet creates and signs these transactions locally. Fees go to treasury. Exact signed fields and endpoint contracts are documented in the [Public API](rpc.md).

## Participation and rewards

Participation is an optional on-chain role. A registered Participant can receive a share of the Participant reward pool while active. Registration does not grant block-production rights.

### Sponsor and Participant

A **Sponsor** signs and pays the registration transaction and locks the Sponsor Bond. The **Participant** is the registered wallet that receives Participant rewards and maintains its own activity.

You may sponsor yourself. In that case the same wallet is Sponsor and Participant.

When sponsoring another wallet:

- the Sponsor pays registration costs and locks the bond;
- the Participant retains sole control of its private key;
- rewards belong entirely to the Participant;
- the Participant signs its own heartbeats and unregister transaction;
- the Sponsor cannot transact or unregister on behalf of the Participant.

Sponsorship is not a referral, delegation, commission, or revenue-sharing system. A Sponsor receives no commission or reward share.

### Sponsor capacity and bond

A Sponsor can have up to 10 active Sponsored Participants. Inactive records remain visible but do not consume an active slot.

The configured bond is `50,000,000` base units. With the current nine token decimals, the Explorer displays this as **0.05 SPRG**. It must not be described as 50 SPRG unless a future explicit economics migration changes the configured amount.

The bond is locked rather than burned. A successful Participant-initiated unregister returns it to the original Sponsor. Sponsor reclaim is unavailable in this protocol version. If a Participant loses its private key, its bond may remain locked indefinitely.

### Activity

A Participant stays active when qualifying on-chain activity updates its Last Seen Height within the 5,100-block activity window, approximately three days at the current target block time.

An inactive Participant remains registered but does not receive Participant rewards. A later valid transaction from that Participant can reactivate it.

Inactivity pauses rewards but does not reset Reward Maturity. Unregistering removes the registration; registering again creates a new Registered Height and starts maturity again.

### Reward Maturity

Reward Maturity gradually raises a Participant's share according to the age of the current registration:

| Age in blocks | Multiplier | Stage |
| ---: | ---: | --- |
| 0 through 5,100 | 25% | New |
| 5,101 through 10,200 | 60% | Growing |
| More than 10,200 | 100% | Mature |

Maturity is based on Registered Height, not timestamps or observer heartbeats. Before activation block 1,000, legacy full Participant rewards apply. After activation, the configured stages apply deterministically.

If an equal base share is 10 SPRG, a New Participant receives 2.5 SPRG, a Growing Participant 6 SPRG, and a Mature Participant 10 SPRG. Integer calculations round down in base units; deterministic remainder goes to treasury.

### Reward calculation

For Participant pool `P`, active Participant count `N`, and maturity multiplier `M` in basis points:

```text
baseShare = floor(P / N)
participantReward = floor(baseShare * M / 10000)
treasuryRemainder = P - sum(participantReward)
```

This calculation uses integers and conserves the complete pool.

## Block reward distribution

The current per-block allocation is:

| Destination | Share | Behavior |
| --- | ---: | --- |
| Active Participants | 15% | Equal base shares adjusted by Reward Maturity. |
| Node pool | 70% | Accrues for the configured Node Holder mechanism. |
| Treasury | 10% | Credited directly, plus fees and deterministic remainder. |
| Holder pool | 5% | Accrues for eligible holder payouts. |

Holder eligibility uses a rolling 14-day average balance and a current threshold of 1,000 SPRG. The exact window is calculated in blocks from the configured target block time.

## Current limitations

- One official producer controls ordering and block availability.
- Sponsor reclaim and Participant co-signing during sponsored registration are unavailable.
- Participation does not identify people or prevent one person from controlling multiple wallets.
- Economics and protocol behavior can still change during Public Alpha.
- The implementation has test coverage and deterministic replay tooling but is not formally verified or represented as independently audited.
