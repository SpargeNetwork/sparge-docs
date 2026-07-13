# Security

Sparge is experimental public-alpha software. Treat every public deployment as internet-facing infrastructure and every protocol/economics assumption as subject to change before a stable release.

## Security model

The current chain has one official producer. Observers independently validate state but cannot provide decentralized producer liveness, censorship resistance, fork choice, or finality. Heartbeats report network health only and never influence chain state or consensus.

Wallet keys are generated and stored locally. Nodes, explorers, observer registries, logs, backups, and the Operator Dashboard must never receive or expose wallet private keys.

## Public deployment

Use the supplied production Compose and Caddy topology:

- Caddy is the only service exposed on ports 80 and 443.
- Producer and observer application ports stay on the internal network.
- Express trusts exactly one controlled Caddy hop.
- Caddy overwrites forwarding headers and applies HTTPS/security headers.
- Exact same-origin CORS replaces wildcard access.
- Caddy and the application enforce request-size limits.
- Endpoint-specific rate limits remain enabled.
- Operator and debug routes are blocked at both proxy and application layers.

Complete procedures are in [Operator Guide: HTTPS and Caddy](operator-guide.md#https-and-caddy).

## Administrative access

Mining start/stop routes are disabled by default and restricted to local requests when deliberately enabled for development. The private Operator Dashboard must remain disabled or loopback-only and be accessed through localhost, SSH tunneling, or a controlled VPN. Rate limiting is not authentication.

Do not expose `/operator`, `/api/operator/*`, state-changing `/api/mining/*`, or `/api/debug/*` to the internet.

## API and denial-of-service controls

The public API uses strict schemas, byte-limited JSON parsing, endpoint and global throttles, transaction concurrency limits, bounded pagination, bounded sync ranges, and a bounded process-local mempool. Unsupported content types and compressed bodies are rejected.

Proxy trust is false by default. Trusting arbitrary forwarded headers lets clients choose their apparent IP and bypass per-IP controls.

See the [Developer Guide](developer-guide.md) for exact request-boundary behavior.

## Privacy

Observer public listing is opt-in. Aggregate counts include private observers, but public lists omit IPs, internal node IDs, hostnames, usernames, machine metadata, and latest hashes.

Structured logs redact keys, secrets, signatures, request bodies, raw transactions, raw IPs, machine/user names, and internal observer identities. Do not put secrets in URLs because proxy logs normally include request paths and queries.

## Data safety

Use versioned, checksummed SQLite backups and regularly verify them by restoring into an isolated directory and running deterministic replay. Restore never belongs behind HTTP and must not run against an active producer.

Replay is CLI-only and read-only against its source. It provides deterministic reconstruction evidence, not formal verification, consensus proof, cryptographic audit, or independent security review.

Never solve a genesis or identity mismatch by deleting production files. Freeze mining, preserve evidence, and follow the [recovery procedure](operator-guide.md#recovery).

## Reporting a vulnerability

Report security issues privately. PGP contact details are available on request. The project aims to acknowledge receipt within 72 hours and provide a remediation timeline after triage.

The reporting scope covers core chain logic, wallet and signing flows, RPC interfaces, and Explorer behavior. Third-party dependencies and self-hosting misconfiguration are outside the project security scope unless they expose a defect in Sparge itself.

Do not include private keys, live exploit secrets, complete signatures, or sensitive production data in a public issue.
