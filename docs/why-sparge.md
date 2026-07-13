# Why Sparge

Sparge exists as a readable, practical environment for experimenting with on-chain applications without requiring every builder to operate blockchain infrastructure.

## Designed for understandable integrations

Many blockchain integrations begin with a large SDK, browser extension, contract language, indexing stack, and several network-specific abstractions. Sparge public alpha exposes a narrower model:

- accounts are derived from Ed25519 public keys
- transactions have a documented canonical message
- amounts and nonces use exact integer strings
- chain state is readable through a conventional JSON API
- confirmation can be followed by transaction ID, address, or block height

That makes the system useful for learning, prototyping, auditing application flows, and building focused ecosystem tools.

## What builders can create

Sparge can support applications that use transfers and public chain data, including:

- self-custody wallets
- payment requests and merchant reconciliation
- block and address explorers
- games that settle entry fees or rewards through ordinary transfers
- marketplaces whose listings remain off-chain and whose payments settle on-chain
- dashboards, analytics, alerts, bots, and indexing services
- language-specific client libraries and SDKs

Sparge does not currently expose a smart-contract runtime, token-creation API, NFT standard, event subscription stream, or trustless marketplace primitive. Applications needing those features must keep their own non-canonical application state and use Sparge transfers only for supported settlement flows.

## Users keep control of keys

Wallet creation and transaction signing happen locally. An application sends public keys and signatures to the API, never private keys. This supports self-custody, but it also makes secure backup, explicit signing consent, and safe key storage application responsibilities.

## Transparent verification

The current network has one official producer. That producer determines transaction ordering and creates blocks. Observer Nodes independently fetch, validate, and store the resulting chain, making it easier for third parties to compare chain state and detect disagreement.

Observers improve transparency; they do not turn the current release into decentralized consensus. Producer availability and censorship resistance remain operational properties rather than protocol guarantees.

## Public-alpha boundaries

Sparge is appropriate for experiments and integrations that can tolerate change. Builders should pin chain identity and protocol versions, avoid promises of irreversible value, and design migration paths.

Known boundaries include:

- one official producer
- HTTP polling rather than WebSocket, SSE, or subscription APIs
- no formal finality depth beyond block inclusion
- no official application SDK yet
- no smart contracts
- non-durable producer mempool
- evolving economics and compatibility before stable release

These limits are part of the public contract and should be visible in application UX.

## Project direction

The immediate ecosystem opportunity is better client tooling: typed API clients, signing libraries, mobile-safe key storage, indexers, test fixtures, payment helpers, and reusable UI components. Future SDKs should wrap the documented protocol rather than create a second, incompatible interpretation of it.

Start with the [Builder Guide](builder-guide.md) or inspect the [RPC API](rpc.md).
