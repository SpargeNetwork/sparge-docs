# RPC API

The public JSON API uses the `/api` base path. Examples assume `http://localhost:3051`.

## Common behavior

- Requests and responses use JSON unless noted otherwise.
- JSON writes require `Content-Type: application/json`.
- Every API response receives an `X-Request-ID` header.
- Malformed input returns structured HTTP `400` validation errors.
- Oversized bodies return HTTP `413`; unsupported media types return HTTP `415`.
- Rate-limited requests return HTTP `429` with `Retry-After`.
- Missing blocks or transactions return HTTP `404`.

See the [Developer Guide](developer-guide.md) for validation, size limits, rate limits, and shared error behavior.

## Node and genesis

### `GET /api/status`

Returns chain identity, node mode, synchronization and health state, current height/hash, configured addresses and economics, mempool aggregates, and invariant status.

```powershell
Invoke-RestMethod http://localhost:3051/api/status
```

Observer-specific fields include `syncState`, `syncedHeight`, `producerHeight`, and `lagBlocks`. Producer health fields include `healthy`, `chainHealthy`, `storageHealthy`, `mempoolHealthy`, `invariantStatus`, and `miningPausedForSafety`.

### `GET /api/genesis`

Returns canonical genesis data for chain-identity verification.

```powershell
Invoke-RestMethod http://localhost:3051/api/genesis
```

## Balances and nonces

### `GET /api/balance/:addr`

Returns `{ address, balanceMicro }`.

### `GET /api/nonce/:addr`

Returns `{ address, nonce }`. A sender must also account for sequential pending transactions when constructing a new transaction.

## Blocks

### `GET /api/blocks?page=<n>&limit=<n>`

Returns paginated blocks with `total`, `page`, `limit`, and `blocks`. Defaults are page 1 and limit 10; the maximum limit is 50.

### `GET /api/blocks?fromHeight=<n>&limit=<n>`

Producer synchronization feed. Returns chain identity, requested and latest heights, and up to 200 blocks. The default limit is 50.

### `GET /api/blocks/state`

Returns the same current chain-state representation used by node status.

### `GET /api/block/:height`

Returns one block by non-negative integer height.

```powershell
Invoke-RestMethod http://localhost:3051/api/block/0
```

## Transactions

### `GET /api/tx/:txid`

Returns a confirmed transaction by its 64-character lowercase transaction ID.

### `POST /api/tx`

Submits a locally signed transaction to a producer. An observer returns HTTP `403`.

Accepted fields are:

```text
type, chainId, from, to, amountMicro, feeMicro, nonce,
publicKeyHex, signatureHex, sponsor, participant, memo
```

Strict payloads require unused canonical fields to be empty strings. A successful response means the transaction is queued, not confirmed:

```json
{
  "status": "queued",
  "txid": "<64 lowercase hex characters>",
  "message": "<canonical signed message>"
}
```

Use `GET /api/tx/:txid` or address history to determine confirmation. Protocol field order and transaction-type rules are in [Transactions](protocol.md#transactions).

## Addresses

### `GET /api/address/:addr`

Returns aggregate address statistics.

### `GET /api/address/:addr/txs?limit=<n>`

Returns `{ address, txs }`. The default and maximum limit are 50 and 100 respectively.

## Mempool

### `GET /api/mempool`

Returns aggregate count, byte/capacity/rejection metrics, and the current pending transaction list. Pending data is process-local and disappears on restart.

## Network

### `GET /api/network/status`

Returns producer status and count, observer aggregate counts, current and observer heights, average observer lag, average block time, last block timestamp, and mempool size. Private observers contribute to aggregate counts.

### `GET /api/network/observers`

Returns only opted-in public observers. Query parameters:

- `page`: default 1
- `limit`: default 25, maximum 100
- `status`: allowlisted observer status
- `version`: validated client version
- `country`: ISO alpha-2 country code

The endpoint returns HTTP `403` when global public listing is disabled. It never returns raw IPs, internal node IDs, hostnames, or latest hashes.

### `POST /api/network/heartbeat`

Observer-to-producer health heartbeat. It is registry-only and cannot alter chain state. Payload and privacy rules are documented in the [Developer Guide](developer-guide.md#observer-heartbeat-schema).

## Local observer settings

### `GET /api/observer/settings`

### `POST /api/observer/settings`

Read or update public-listing settings on an observer. These routes are available only in observer mode and only from a local request.

## Mining status and local controls

### `GET /api/mining/status`

Returns producer mining state when the producer mining router is mounted.

### `POST /api/mining/start`

### `POST /api/mining/stop`

These are disabled unless local development administration is explicitly enabled and are then restricted to local requests. They are blocked by the production Caddy configuration and are not public operator APIs.

## Private operator endpoint

### `GET /api/operator/status`

Provides the read-only Operator Dashboard payload only when that dashboard is explicitly enabled on a producer and the request satisfies its loopback restriction. It must not be exposed through public Caddy. See [Operator Dashboard](operator-guide.md#operator-dashboard).

## Error reference

Common structured codes include:

| HTTP | Code | Meaning |
| ---: | --- | --- |
| 400 | `VALIDATION_ERROR` | Body, parameter, or query failed schema validation. |
| 400 | `NONCE_TOO_FAR_AHEAD` | Nonce exceeds the configured future distance. |
| 403 | `OBSERVER_READ_ONLY` or text equivalent | State-changing submission reached an observer. |
| 409 | `MEMPOOL_DUPLICATE` | Transaction is pending or already confirmed. |
| 413 | `PAYLOAD_TOO_LARGE` | Request exceeds its byte limit. |
| 415 | `UNSUPPORTED_MEDIA_TYPE` | JSON endpoint received another content type. |
| 429 | `RATE_LIMITED` | Request allowance exhausted. |
| 429 | `MEMPOOL_SENDER_LIMIT` | Sender has too many pending transactions. |
| 503 | `MEMPOOL_FULL` | Producer mempool reached capacity. |

Some protocol/business validation failures currently return `{ "error": "<message>" }` rather than a code. Clients should use HTTP status plus the documented structured code when available and should not parse human-readable messages as stable contracts.
