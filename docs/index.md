# Build on Sparge

Sparge is an experimental public-alpha blockchain for wallets, payments, games, explorers, marketplaces, and community-built tools. It exposes a small JSON API, uses deterministic signed transactions, and keeps wallet keys under the user's control.

You do not need to run the official producer to use Sparge or build an application. Applications read chain data from the public API, sign transactions locally, and broadcast the signed payload to the producer.

!!! warning "Public alpha"
    Sparge is experimental software, not an investment or financial guarantee. APIs, economics, chain data, and compatibility rules may change before a stable release.

## What Sparge provides

- Ed25519 accounts with locally controlled keys
- deterministic addresses and transaction identifiers
- transfers with integer-denominated amounts and optional public memos
- account balances, nonces, transaction history, blocks, and network status over HTTP
- independently validating Observer Nodes for additional transparency
- a deliberately small protocol surface that can be integrated without a large SDK

Sparge currently has one official producer. Observer Nodes independently synchronize and validate its chain state, but they do not produce blocks or provide decentralized consensus.

## Start here

| I want to... | Go to |
| --- | --- |
| Understand the project | [Why Sparge](why-sparge.md) |
| Create or use an account | [Wallet](wallet.md) |
| Build an application | [Build on Sparge](builder-guide.md) |
| Integrate an endpoint | [RPC API](rpc.md) |
| Understand chain rules | [Protocol](protocol.md) |
| Help verify the network | [Observer Node](observer.md) |
| Handle keys and data safely | [Security](security.md) |

## Developer resources

- [Sparge Node source](https://github.com/SpargeNetwork/sparge-node)
- [Documentation source](https://github.com/SpargeNetwork/sparge-docs)
- [Builder Guide](builder-guide.md)
- [RPC reference](rpc.md)
- [Integration configuration](reference/configuration.md)

The Explorer and browser Wallet are served by a Sparge node. A canonical public deployment URL is not currently declared in the project repositories; use only a URL announced through an official SpargeNetwork channel. The Explorer implementation is available in the [node repository](https://github.com/SpargeNetwork/sparge-node/tree/main/public).

## A simple application flow

1. Read `/api/status` and verify the expected chain identity.
2. Generate or import an Ed25519 key locally.
3. Derive the `spg_` address from the public key.
4. Read the account nonce and balance.
5. Build the canonical transaction message.
6. Sign locally and broadcast to `POST /api/tx`.
7. Poll the transaction endpoint until it is confirmed in a block.

Continue with [Build on Sparge](builder-guide.md#your-first-transfer-integration).
