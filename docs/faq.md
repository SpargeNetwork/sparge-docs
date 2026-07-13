# Frequently Asked Questions

## Is Sparge production ready?

No. It is public-alpha, experimental software with one official producer. Breaking changes, migrations, and resets between explicitly announced milestones remain possible.

## Is Sparge an investment or guaranteed store of value?

No. The project makes no promise of value, uptime, permanence, exchange listing, or future direction.

## Does restarting reset the chain?

No. A normal restart reopens `state.db` and `genesis.json` in the configured `DATA_DIR`. A different/empty data directory or deleted Docker volume starts separate state. If a producer unexpectedly starts at genesis, stop it and investigate the path or volume before mining.

## Is `genesis.json` recreated if I remove it?

Do not remove genesis from an established producer. Automatic initialization is for an intentionally empty data directory, not recovery. Restore the matching genesis and database from a verified backup.

## What is `proposerAddress`?

It is the configured protocol address associated with block production. It is a public address, not a private key. Changing a genesis-bound address can make existing chain data incompatible.

## What is `genesisOperatorAddress`?

It identifies the address eligible for the one-time free participant registration during `genesisFreeBlocks`. Registration still requires a correctly signed on-chain transaction from that same address and remains pending until mined.

## Why is my participant registration pending?

Successful submission only queues a transaction. Mining must be active and the transaction must pass block application before registration becomes confirmed. The explorer should distinguish queued/pending state from confirmed active participation.

## Where do wallet keys live?

Browser wallet keys remain in browser storage; CLI wallet keys remain in the local wallet file. Producer backups do not back up wallet keys.

## What does an observer do?

It synchronizes blocks from a producer, verifies identity/continuity and state transitions locally, stores its own chain state, and serves a read-only explorer. It does not produce blocks or accept transaction submissions.

## Is observer listing private?

Listing is opt-in. Private observers still contribute to aggregate health, but public responses omit IPs, internal IDs, hostnames, machine/user metadata, and latest hashes.

## Why is an observer syncing or mismatched?

Ordinary lag can occur while downloading blocks. Persistent errors usually indicate an unreachable producer, different genesis, or divergent local observer data. Preserve evidence, verify the intended producer, and reset only the observer if resynchronization is required.

## How are rewards split?

See [Block rewards](protocol.md#block-rewards) and [Holder rewards](protocol.md#holder-rewards).

## How do I make a safe backup?

Use `npm run backup`, verify the archive, restore it to an isolated directory, and run deterministic replay. See [Operator Guide: Backups](operator-guide.md#backups).

## Where is the complete API reference?

See the [RPC API](rpc.md). Shared validation and rate/body-limit behavior is in the [Developer Guide](developer-guide.md).
