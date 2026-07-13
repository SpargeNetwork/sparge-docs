# Security

Sparge is public-alpha software. Security begins with understanding the current trust model: users control wallet keys, the official producer creates blocks, and Observer Nodes independently validate the resulting chain.

## For wallet users

- Keep private keys and wallet exports secret.
- Back up a wallet before receiving funds.
- Verify the complete recipient address and network before signing.
- Review amount, fee, and memo on a trusted screen.
- Treat memos as permanent public data.
- Distinguish pending from confirmed transactions.
- Never provide a private key to support staff, an Explorer, or a node operator.

Browser storage is convenient but not equivalent to an operating-system keystore or hardware-backed key. Current wallet tooling is experimental.

## For application builders

### Verify chain identity

Pin the expected `chainId`, `genesisHash`, `protocolVersion`, and `economicsVersion`. Verify them at startup and before signing. Do not trust a token symbol or hostname alone.

### Keep signing local

Do not send private keys to a Sparge endpoint. Prefer client-side, hardware-backed, or dedicated signing boundaries. If a backend must sign, isolate keys from web-facing processes and restrict what can be signed.

### Use exact arithmetic

Use decimal strings and `BigInt`. Floating-point rounding can send or credit the wrong amount.

### Treat public data as untrusted input

Addresses, aliases, memos, versions, hashes, and API error messages must be escaped before rendering. Do not create HTML with unsanitized chain data.

### Design idempotent workflows

Use transaction IDs as stable reconciliation keys. A repeated poll, process restart, or duplicate submission must not fulfill a payment or game reward twice.

### Respect API protections

Use bounded requests, backoff, jitter, and `Retry-After`. Do not bypass request-size or rate limits by distributing traffic across hosts. Cache immutable blocks and transactions.

## Endpoint trust

The official public Explorer/API URL is not currently declared in the project repositories. Use only endpoints announced through official SpargeNetwork channels.

A malicious endpoint can lie about balances, nonces, fees, or transaction confirmation and can observe queried addresses. Local signing prevents it from directly learning a private key, but it cannot make false read data trustworthy.

For higher assurance, compare independent endpoints or run an [Observer Node](observer.md).

## Confirmation and finality

Transaction admission returns `queued`, not confirmed. Confirmation means the transaction appears in a stored block. The current single-producer model has no separate decentralized finality protocol. Applications must define and communicate their own confirmation policy for public-alpha risk.

## Privacy

Sparge is a transparent chain. Addresses, amounts, transaction relationships, and memos can be indexed publicly. Reusing an address links its activity.

Do not put personal data or secrets in memos. Querying a hosted Explorer can also reveal which addresses interest the requesting IP address.

Observer listing is opt-in. Public listings omit IPs, hostnames, internal node IDs, machine metadata, and latest block hashes.

## Observer safety

- Download source and installers only from official repositories or release channels.
- Verify chain identity after installation and update.
- Do not expose local observer settings to the internet.
- Keep observer data separate from wallet key storage.
- Preserve data and logs before resetting a mismatch under investigation.
- Remember that an observer is a read endpoint, not a transaction broadcaster.

## Protocol limitations

Sparge currently has one official producer, no smart-contract sandbox, no multi-producer consensus, no subscription transport, and no formal protocol audit implied by its test suites. These limits should be included in application threat models.

## Reporting a vulnerability

Report security issues privately. PGP contact details are available on request. The project aims to acknowledge receipt within 72 hours and provide a remediation timeline after triage.

The reporting scope covers core chain logic, wallet and signing flows, RPC interfaces, and Explorer behavior. Third-party dependencies and self-hosting misconfiguration are outside the project scope unless they expose a Sparge defect.

Do not publish private keys, live exploit secrets, complete sensitive payloads, or production data in an issue.
