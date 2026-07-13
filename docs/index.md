# Sparge Chain

Sparge Chain is an experimental, JavaScript-native blockchain for public-alpha testing and community experimentation. It is infrastructure software, not an investment, bank, fiat-backed system, or promise of value.

The current network uses one official producer. Observer nodes independently sync and validate the chain state, but they do not produce blocks or participate in consensus. Observer sync currently uses HTTP rather than peer-to-peer discovery or gossip.

## Public alpha

Sparge is early-stage software. Expect breaking changes, migrations, occasional resets between milestones, evolving economics, and no guarantee of uptime, permanence, exchange listing, or future direction.

The project prioritizes:

- deterministic protocol behavior
- readable JavaScript implementation
- locally held wallet keys
- independently validating observer nodes
- bounded and validated public APIs
- recoverable producer operations

## Choose a guide

- [Getting Started](getting-started.md): install the software and run a local producer.
- [Wallet](wallet.md): create keys and submit signed transactions.
- [Observer Node](observer.md): run a read-only validating node and configure public listing.
- [Protocol](protocol.md): understand blocks, transactions, participation, rewards, and current consensus limitations.
- [Developer Guide](developer-guide.md): integrate safely with validation, limits, errors, and node internals.
- [RPC API](rpc.md): use the canonical HTTP endpoint reference.
- [Operator Guide](operator-guide.md): deploy, monitor, back up, replay, recover, and upgrade a producer.
- [Security](security.md): understand the public-alpha security model and report vulnerabilities.
- [Configuration](reference/configuration.md): review configuration keys and environment overrides.

## Network model

Producer nodes create blocks. Observer nodes fetch blocks from a configured producer, verify chain identity and continuity, apply the same state transitions locally, and provide a read-only explorer. Recent privacy-preserving observer heartbeats provide aggregate network-health information but never affect block validation, mining, chain state, or consensus.

Wallet keys are generated and stored locally. Private keys are never sent to node endpoints.

## Project principles

Sparge is an experiment built in the open. It does not over-promise or optimize for hype. The best ways to participate are to run an observer, test wallet and explorer flows, review the code, and report reproducible issues.
