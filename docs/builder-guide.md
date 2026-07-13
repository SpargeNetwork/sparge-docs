# User and Builder Documentation Audit

This internal record describes the builder-first redesign performed after the public-alpha architecture refactor. It is excluded from the public MkDocs build.

## Audience decision

Public documentation serves:

1. Users creating wallets, receiving and sending SPRG, and reading the Explorer.
2. Builders integrating the RPC API or creating wallets, games, payment applications, marketplaces, explorers, indexers, and SDKs.
3. Optional Observer operators helping verify chain state.

Producer operators, release engineers, and internal maintainers are not public-navigation audiences.

## Public tree

```text
docs/
  index.md
  why-sparge.md
  wallet.md
  builder-guide.md
  rpc.md
  protocol.md
  observer.md
  security.md
  faq.md
  reference/
    configuration.md
```

## Public navigation

```text
Home
Why Sparge
Wallet
Build on Sparge
RPC API
Protocol
Observer Node
Security
FAQ
Reference
  Configuration
```

## Files rewritten

| File | New public purpose |
| --- | --- |
| `index.md` | User/builder value, current model, resource links, first integration flow. |
| `wallet.md` | Create, receive, send, confirm, recover, and protect a wallet. |
| `protocol.md` | Blocks, accounts, transactions, participation, economics, state, and limitations without operations. |
| `observer.md` | Optional independent verification, privacy, installation, updates, and troubleshooting. |
| `rpc.md` | Canonical public endpoint reference with method, parameters, requests, responses, and errors. |
| `security.md` | Threat guidance for wallet users, application builders, and Observer operators. |
| `faq.md` | User, builder, and Observer questions only. |
| `reference/configuration.md` | Client identity, endpoint, polling, amount, and optional Observer configuration. |

New public files are `why-sparge.md` and `builder-guide.md`.

## Moved to internal

| Previous public file | Internal destination | Reason |
| --- | --- | --- |
| `operator-guide.md` | `internal/operator-guide.md` | Official producer deployment and recovery. |
| `developer-guide.md` | `internal/api-implementation-notes.md` | Server validation, limits, mempool, invariants, logging, and tests. |
| `reference/configuration.md` (full node version) | `internal/operator-configuration.md` | Producer, storage, economics, security, and deployment settings. |

The producer-oriented `getting-started.md` was removed after its operational content remained available in the internal Operator Guide.

## Duplication removed

- Endpoint purpose and response descriptions now live only in `rpc.md`.
- Signing and application workflows live in `builder-guide.md`, not the RPC reference.
- Address and canonical transaction rules are explained conceptually in Protocol and implemented once in Builder examples.
- Wallet safety is user-focused in Wallet and threat-focused in Security without repeating node operations.
- Observer setup and privacy are owned by the Observer guide; RPC only specifies endpoint contracts.
- Producer deployment, monitoring, logging, backup, replay, recovery, and dashboard content no longer appears publicly.
- Public configuration no longer reproduces producer configuration.

## Builder documentation gaps

The redesign found product or protocol documentation that cannot be completed accurately without new project artifacts:

1. No canonical public Explorer, Wallet, or API base URL is declared in the official repositories.
2. No published release manifest supplies the canonical genesis hash and supported client compatibility matrix.
3. No official SDK exists for JavaScript, TypeScript, mobile, or backend languages.
4. No shared cross-language test vectors cover key generation, address derivation, canonical messages, signatures, and transaction IDs.
5. No OpenAPI or machine-readable RPC schema exists.
6. No public testnet, faucet, mock server, or builder sandbox is documented.
7. No WebSocket, SSE, webhook, or subscription API exists; builders must poll.
8. Business-validation errors are not uniformly exposed as stable machine-readable codes.
9. Confirmed transaction lookup returns `404` while pending; there is no lightweight transaction-status endpoint.
10. Address history is bounded but not paginated, so complete services must index blocks.
11. There is no block-by-hash endpoint.
12. No protocol-level finality depth or recommended application confirmation policy is published.
13. No smart-contract, custom-token, NFT, event, or escrow primitive exists; application documentation must avoid implying otherwise.
14. Informal requests sometimes use `SPG`, while the configured chain symbol is `SPRG`; public branding should standardize one symbol.

These are documented as limitations rather than filled with invented URLs, guarantees, or SDK behavior.
