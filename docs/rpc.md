# RPC API

Sparge exposes a JSON-over-HTTP API under `/api`. This page is the canonical public endpoint reference. Application architecture, signing code, polling, and product workflows belong in [Build on Sparge](builder-guide.md).

The examples use:

```text
https://your-approved-sparge-endpoint.example/api
```

Replace that host with an endpoint announced through an official SpargeNetwork channel. The official repositories do not currently declare a canonical public deployment URL.

## Conventions

- JSON writes require `Content-Type: application/json`.
- Amounts, fees, balances, supply values, and nonces may be decimal strings.
- Hashes, public keys, and signatures are lowercase hexadecimal.
- Every API response includes `X-Request-ID`.
- `429` responses include `Retry-After` and `RateLimit-*` headers.
- Examples are abbreviated where an endpoint returns additional metrics or block fields.

Common error bodies:

```json
{
  "error": "VALIDATION_ERROR",
  "message": "Request validation failed",
  "details": [
    { "field": "height", "reason": "Expected a non-negative safe integer" }
  ]
}
```

```json
{
  "error": "RATE_LIMITED",
  "message": "Too many requests. Please try again later.",
  "retryAfterSeconds": 42
}
```

## Node and chain

### Get node status

`GET /api/status`

**Purpose:** Read chain identity, current tip, network parameters, account display settings, node synchronization, health, and mempool aggregates.

**Parameters:** None.

**Example request:**

```bash
curl https://your-approved-sparge-endpoint.example/api/status
```

**Example response (abbreviated):**

```json
{
  "chainId": "sparge-mainnet",
  "protocolVersion": "1.0.0",
  "economicsVersion": "1.0.0",
  "genesisHash": "f0f0755198040a562fd41b86fd7055aa4228d954d46d5381aece5376162dc5d1",
  "chain": "Sparge",
  "symbol": "SPRG",
  "decimals": 9,
  "blockTimeSeconds": 51,
  "latestHeight": "129",
  "latestHash": "3dedb244fe3247efc13e2591ec3db119842b7c2aeb04e932f723a4a098a0888b",
  "latestBlock": {
    "height": 129,
    "hash": "3dedb244fe3247efc13e2591ec3db119842b7c2aeb04e932f723a4a098a0888b",
    "timestamp": "2026-07-13T12:19:35.267Z",
    "txCount": 4
  },
  "baseFeeMicro": "7",
  "minFeeMicro": "1000",
  "nodeMode": "producer",
  "syncState": "synced",
  "healthy": true,
  "mempoolTransactionCount": 0
}
```

**Common errors:** `429 RATE_LIMITED`; `5xx` when the selected endpoint is unavailable. A `200` response can still describe an unhealthy or lagging node, so inspect health and sync fields.

### Get genesis

`GET /api/genesis`

**Purpose:** Read the canonical genesis identity and initial compatibility parameters.

**Parameters:** None.

**Example request:**

```bash
curl https://your-approved-sparge-endpoint.example/api/genesis
```

**Example response:**

```json
{
  "chainId": "sparge-mainnet",
  "chainName": "Sparge",
  "symbol": "SPRG",
  "blockTimeSeconds": 51,
  "protocolVersion": "1.0.0",
  "economicsVersion": "1.0.0",
  "genesisOperatorAddress": "spg_2jCwDGKiH9CdfhkAZWKv6fSiAacn",
  "genesisFreeBlocks": 100,
  "createdAt": "2026-01-01T00:00:00.000Z",
  "genesisHash": "f0f0755198040a562fd41b86fd7055aa4228d954d46d5381aece5376162dc5d1"
}
```

**Common errors:** `429 RATE_LIMITED`; `5xx` when genesis data cannot be served. Clients should reject an unexpected identity even when HTTP status is `200`.

## Accounts

### Get balance

`GET /api/balance/:addr`

**Purpose:** Read the current confirmed balance of one account in base units.

**Path parameters:**

- `addr`: valid `spg_` address.

**Example request:**

```bash
curl https://your-approved-sparge-endpoint.example/api/balance/spg_3D9sm1pyziUMXCkqYaJML3KJRpPd
```

**Example response:**

```json
{
  "address": "spg_3D9sm1pyziUMXCkqYaJML3KJRpPd",
  "balanceMicro": "12500000000"
}
```

**Common errors:** `400 VALIDATION_ERROR` for a malformed address; `429 RATE_LIMITED`. A valid unknown address returns a zero balance rather than `404`.

### Get confirmed nonce

`GET /api/nonce/:addr`

**Purpose:** Read the account's confirmed transaction nonce before constructing a new signed transaction.

**Path parameters:**

- `addr`: valid `spg_` address.

**Example request:**

```bash
curl https://your-approved-sparge-endpoint.example/api/nonce/spg_3D9sm1pyziUMXCkqYaJML3KJRpPd
```

**Example response:**

```json
{
  "address": "spg_3D9sm1pyziUMXCkqYaJML3KJRpPd",
  "nonce": "4"
}
```

**Common errors:** `400 VALIDATION_ERROR`; `429 RATE_LIMITED`. The response excludes pending mempool nonces; serialize submissions per sender.

### Get address summary

`GET /api/address/:addr`

**Purpose:** Read balance, nonce, history summary, holder-average eligibility, participation, and sponsorship information for an address.

**Path parameters:**

- `addr`: valid `spg_` address.

**Example request:**

```bash
curl https://your-approved-sparge-endpoint.example/api/address/spg_3D9sm1pyziUMXCkqYaJML3KJRpPd
```

**Example response:**

```json
{
  "address": "spg_3D9sm1pyziUMXCkqYaJML3KJRpPd",
  "balanceMicro": "12500000000",
  "avgBalanceMicro": "11800000000",
  "avgEligible": true,
  "nonce": "4",
  "txCount": 7,
  "firstSeenHeight": 12,
  "lastSeenHeight": 125,
  "firstSeen": "2026-07-01T10:00:00.000Z",
  "lastSeen": "2026-07-13T11:00:00.000Z",
  "participant": null,
  "sponsoredActiveCount": 0
}
```

**Common errors:** `400 VALIDATION_ERROR`; `429 RATE_LIMITED`. A valid address with no history returns zero/null summary fields.

### Get address transactions

`GET /api/address/:addr/txs`

**Purpose:** Read a bounded list of recent confirmed transactions associated with an address.

**Path parameters:**

- `addr`: valid `spg_` address.

**Query parameters:**

- `limit`: optional integer, default `50`, maximum `100`.

**Example request:**

```bash
curl "https://your-approved-sparge-endpoint.example/api/address/spg_3D9sm1pyziUMXCkqYaJML3KJRpPd/txs?limit=20"
```

**Example response:**

```json
{
  "address": "spg_3D9sm1pyziUMXCkqYaJML3KJRpPd",
  "txs": [
    {
      "txid": "8fc0e8d8d856b78a6451f92dbf88a484de83ed937f63bd22f902db3da8d20db7",
      "type": "transfer",
      "from": "spg_3D9sm1pyziUMXCkqYaJML3KJRpPd",
      "to": "spg_2jCwDGKiH9CdfhkAZWKv6fSiAacn",
      "amountMicro": "1000000000",
      "feeMicro": "1000"
    }
  ]
}
```

**Common errors:** `400 VALIDATION_ERROR` for address or out-of-range limit; `429 RATE_LIMITED`. This endpoint is not paginated and is not a complete indexer feed.

## Blocks

### List recent blocks

`GET /api/blocks`

**Purpose:** Read a newest-first paginated block list for Explorer and browsing interfaces.

**Query parameters:**

- `page`: optional integer, default `1`, minimum `1`.
- `limit`: optional integer, default `10`, maximum `50`.

Do not combine `page` with `fromHeight`.

**Example request:**

```bash
curl "https://your-approved-sparge-endpoint.example/api/blocks?page=1&limit=10"
```

**Example response (block abbreviated):**

```json
{
  "total": 130,
  "page": 1,
  "limit": 10,
  "blocks": [
    {
      "height": 129,
      "timestamp": "2026-07-13T12:19:35.267Z",
      "prevHash": "3492247c2209c0f8cacefd7b43fc44ff0738812f9c55c14df91377080e87ef7e",
      "hash": "3dedb244fe3247efc13e2591ec3db119842b7c2aeb04e932f723a4a098a0888b",
      "txCount": 4,
      "stateRoot": "7a09b6a9f19d46d02a7175575ef91384ca3a0455fe81bf5dac7b57bcdfee66e9"
    }
  ]
}
```

**Common errors:** `400 VALIDATION_ERROR`; `429 RATE_LIMITED`.

### Read blocks from a height

`GET /api/blocks?fromHeight=:height&limit=:limit`

**Purpose:** Read an ascending bounded range for Observer synchronization, explorers, and indexers.

**Query parameters:**

- `fromHeight`: required non-negative integer in range mode.
- `limit`: optional integer, default `50`, maximum `200`.

**Example request:**

```bash
curl "https://your-approved-sparge-endpoint.example/api/blocks?fromHeight=120&limit=50"
```

**Example response (blocks abbreviated):**

```json
{
  "chainId": "sparge-mainnet",
  "genesisHash": "f0f0755198040a562fd41b86fd7055aa4228d954d46d5381aece5376162dc5d1",
  "protocolVersion": "1.0.0",
  "economicsVersion": "1.0.0",
  "fromHeight": 120,
  "latestHeight": "129",
  "blocks": [
    {
      "height": 120,
      "hash": "ec97c4b13ab55ff9b02f1544be7a2f3b4a9e79534a8f1d6ca710a9021d52d67f",
      "prevHash": "b983b38f4b3309ead0ff55bdad41da16776f5bcb7934ec38ba4c2c9e3b3ec8fa"
    }
  ]
}
```

**Common errors:** `400 VALIDATION_ERROR`; `429 RATE_LIMITED`, including the additional synchronization-feed limit. Verify identity and continuity before persisting a batch.

### Get block state summary

`GET /api/blocks/state`

**Purpose:** Read the current node/chain state representation through the blocks router. It currently returns the same state object as `/api/status`.

**Parameters:** None.

**Example request:**

```bash
curl https://your-approved-sparge-endpoint.example/api/blocks/state
```

**Example response:** See [Get node status](#get-node-status).

**Common errors:** `429 RATE_LIMITED`; `5xx` when state is unavailable. Prefer `/api/status` for new general integrations.

### Get block by height

`GET /api/block/:height`

**Purpose:** Read one complete confirmed block by its non-negative height.

**Path parameters:**

- `height`: non-negative safe integer.

**Example request:**

```bash
curl https://your-approved-sparge-endpoint.example/api/block/129
```

**Example response (abbreviated):**

```json
{
  "height": 129,
  "timestamp": "2026-07-13T12:19:35.267Z",
  "prevHash": "3492247c2209c0f8cacefd7b43fc44ff0738812f9c55c14df91377080e87ef7e",
  "hash": "3dedb244fe3247efc13e2591ec3db119842b7c2aeb04e932f723a4a098a0888b",
  "chainId": "sparge-mainnet",
  "protocolVersion": "1.0.0",
  "economicsVersion": "1.0.0",
  "transactions": [],
  "txCount": 0,
  "baseFeeBaseUnits": "7",
  "stateRoot": "7a09b6a9f19d46d02a7175575ef91384ca3a0455fe81bf5dac7b57bcdfee66e9"
}
```

**Common errors:** `400 VALIDATION_ERROR`; `404` with `{ "error": "block not found" }`; `429 RATE_LIMITED`.

## Transactions

### Get confirmed transaction

`GET /api/tx/:txid`

**Purpose:** Look up one confirmed transaction by its canonical transaction ID.

**Path parameters:**

- `txid`: 64 lowercase hexadecimal characters.

**Example request:**

```bash
curl https://your-approved-sparge-endpoint.example/api/tx/8fc0e8d8d856b78a6451f92dbf88a484de83ed937f63bd22f902db3da8d20db7
```

**Example response:**

```json
{
  "id": "8fc0e8d8d856b78a6451f92dbf88a484de83ed937f63bd22f902db3da8d20db7",
  "txid": "8fc0e8d8d856b78a6451f92dbf88a484de83ed937f63bd22f902db3da8d20db7",
  "type": "transfer",
  "from": "spg_3D9sm1pyziUMXCkqYaJML3KJRpPd",
  "to": "spg_2jCwDGKiH9CdfhkAZWKv6fSiAacn",
  "amountMicro": "1000000000",
  "feeMicro": "1000",
  "nonce": "4",
  "memo": "order-1042",
  "timestamp": "2026-07-13T11:00:00.000Z"
}
```

**Common errors:** `400 VALIDATION_ERROR`; `404` with `{ "error": "tx not found" }` while pending, expired, or unknown; `429 RATE_LIMITED`.

### Broadcast signed transaction

`POST /api/tx`

**Purpose:** Submit a complete locally signed transaction to the producer mempool.

**Headers:** `Content-Type: application/json`.

**Body fields:**

| Field | Type | Notes |
| --- | --- | --- |
| `type` | string | `transfer`, `register_participant`, `unregister_participant`, or `heartbeat`. |
| `chainId` | string | Must match producer chain. |
| `from` | address | Must derive from `publicKeyHex`. |
| `to` | address/string | Required for transfer; otherwise empty. |
| `amountMicro` | integer string | Greater than zero for transfer; otherwise `"0"`. |
| `feeMicro` | integer string | Must satisfy current minimum unless a protocol exception applies. |
| `nonce` | integer string | Next expected sender nonce. |
| `publicKeyHex` | string | 64 lowercase hex characters. |
| `signatureHex` | string | 128 lowercase hex characters. |
| `txid` | string | Optional accepted field; when supplied, 64 lowercase hex characters. |
| `sponsor` | address/string | Registration sponsor or empty. |
| `participant` | address/string | Required for registration; otherwise empty. |
| `memo` | string | Optional, maximum 128 UTF-8 bytes. |

**Example request:**

```bash
curl -X POST https://your-approved-sparge-endpoint.example/api/tx \
  -H "Content-Type: application/json" \
  -d '{
    "type": "transfer",
    "chainId": "sparge-mainnet",
    "from": "spg_3D9sm1pyziUMXCkqYaJML3KJRpPd",
    "to": "spg_2jCwDGKiH9CdfhkAZWKv6fSiAacn",
    "amountMicro": "1000000000",
    "feeMicro": "1000",
    "nonce": "4",
    "publicKeyHex": "0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef",
    "signatureHex": "<128 lowercase hex characters>",
    "txid": "<64 lowercase hex characters>",
    "sponsor": "",
    "participant": "",
    "memo": "order-1042"
  }'
```

The example key/signature placeholders illustrate shape and are not a valid signed transaction. Create a real payload using [Canonical signing](builder-guide.md#canonical-signing).

**Example response:**

```json
{
  "status": "queued",
  "txid": "8fc0e8d8d856b78a6451f92dbf88a484de83ed937f63bd22f902db3da8d20db7",
  "message": "transfer|sparge-mainnet|spg_...|spg_...|1000000000|1000|4|...|||order-1042"
}
```

`queued` means accepted into temporary memory, not confirmed.

**Common errors:**

- `400 VALIDATION_ERROR`: malformed body or type-specific fields.
- `400`: invalid chain, signer, signature, fee, balance, nonce, or participant state.
- `400 NONCE_TOO_FAR_AHEAD`.
- `403`: observer is read-only.
- `409 MEMPOOL_DUPLICATE`.
- `413 PAYLOAD_TOO_LARGE`.
- `415 UNSUPPORTED_MEDIA_TYPE`.
- `429 RATE_LIMITED` or `MEMPOOL_SENDER_LIMIT`.
- `503 MEMPOOL_FULL` or mempool unavailable.

## Mempool

### Get pending transactions

`GET /api/mempool`

**Purpose:** Read current pending transactions and aggregate capacity/rejection metrics.

**Parameters:** None.

**Example request:**

```bash
curl https://your-approved-sparge-endpoint.example/api/mempool
```

**Example response:**

```json
{
  "count": 0,
  "mempoolTransactionCount": 0,
  "mempoolBytes": 0,
  "mempoolMaxTransactions": 10000,
  "mempoolMaxBytes": 52428800,
  "mempoolUtilizationPercent": 0,
  "transactions": []
}
```

**Common errors:** `429 RATE_LIMITED`; `5xx` if mempool state is unavailable. Pending transactions are temporary and can disappear on restart. Returned transaction fields, including memos and signatures, are public data.

## Network

### Get network status

`GET /api/network/status`

**Purpose:** Read official producer status plus aggregate Observer Node health, height, lag, block timing, and mempool size.

**Parameters:** None.

**Example request:**

```bash
curl https://your-approved-sparge-endpoint.example/api/network/status
```

**Example response:**

```json
{
  "producer": {
    "online": true,
    "count": 1,
    "chainId": "sparge-mainnet",
    "height": 129,
    "latestHash": "3dedb244fe3247efc13e2591ec3db119842b7c2aeb04e932f723a4a098a0888b"
  },
  "producerCount": 1,
  "activeObserverCount": 3,
  "fullySyncedObserverCount": 2,
  "syncingObserverCount": 1,
  "mismatchObserverCount": 0,
  "offlineObserverCount": 1,
  "currentHeight": 129,
  "highestObserverHeight": 129,
  "lowestObserverHeight": 126,
  "averageObserverLag": 1,
  "averageBlockTimeSeconds": 51,
  "lastBlockTimestamp": "2026-07-13T12:19:35.267Z",
  "mempoolSize": 0
}
```

**Common errors:** `429 RATE_LIMITED`; `5xx` when status cannot be calculated. Aggregate counts include private observers; mismatch and offline observers are not healthy active observers.

### List public observers

`GET /api/network/observers`

**Purpose:** Read opted-in Observer Nodes without exposing internal identity or raw IP data.

**Query parameters:**

- `page`: optional, default `1`.
- `limit`: optional, default `25`, maximum `100`.
- `status`: optional `fully_synced`, `syncing`, `mismatch`, or `offline`.
- `version`: optional exact client version.
- `country`: optional ISO 3166-1 alpha-2 country code.

**Example request:**

```bash
curl "https://your-approved-sparge-endpoint.example/api/network/observers?status=fully_synced&country=BE&page=1&limit=25"
```

**Example response:**

```json
{
  "page": 1,
  "limit": 25,
  "total": 1,
  "observers": [
    {
      "publicAlias": "Brussels Observer",
      "countryCode": "BE",
      "version": "1.0.0",
      "height": 129,
      "lag": 0,
      "lastSeen": "2026-07-13T12:19:46.824Z",
      "secondsSinceLastSeen": 24,
      "status": "fully_synced"
    }
  ]
}
```

**Common errors:** `400 VALIDATION_ERROR`; `403` when public observer listing is globally disabled; `429 RATE_LIMITED`.

### Submit observer heartbeat

`POST /api/network/heartbeat`

**Purpose:** Let an Observer Node update the privacy-preserving observer registry. This endpoint cannot modify chain state.

**Headers:** `Content-Type: application/json`.

**Body parameters:**

- `nodeId`: persistent random identifier, 8 to 128 allowed characters.
- `nodeMode`: must be `observer`.
- `version`: optional bounded version string.
- `height`: non-negative safe integer.
- `latestHash`: empty or 64 lowercase hex characters.
- `publicListingEnabled`: boolean.
- `publicAlias`: optional safe alias, maximum 40 characters.
- `countryCode`: optional ISO alpha-2 code.

**Example request:**

```bash
curl -X POST https://your-approved-sparge-endpoint.example/api/network/heartbeat \
  -H "Content-Type: application/json" \
  -d '{
    "nodeId": "obs_7KQm2x4n9Pz1",
    "nodeMode": "observer",
    "version": "1.0.0",
    "height": 129,
    "latestHash": "3dedb244fe3247efc13e2591ec3db119842b7c2aeb04e932f723a4a098a0888b",
    "publicListingEnabled": false,
    "publicAlias": "",
    "countryCode": ""
  }'
```

**Example response:**

```json
{ "ok": true }
```

**Common errors:** `400 VALIDATION_ERROR`; `413 PAYLOAD_TOO_LARGE`; `415 UNSUPPORTED_MEDIA_TYPE`; `429 RATE_LIMITED` by IP or validated node ID. Client-provided IP and hostname fields are rejected.

## Observer-local settings

These endpoints are available only on an Observer Node and only through a local request. They are not public remote application APIs.

### Get observer privacy settings

`GET /api/observer/settings`

**Purpose:** Read local public-listing preferences.

**Parameters:** None.

**Example request:**

```bash
curl http://127.0.0.1:3052/api/observer/settings
```

**Example response:**

```json
{
  "publicListingEnabled": false,
  "publicAlias": "",
  "countryCode": ""
}
```

**Common errors:** `403` for non-local access; `404` when not running in observer mode; `429 RATE_LIMITED`.

### Update observer privacy settings

`POST /api/observer/settings`

**Purpose:** Update local listing consent, alias, and country.

**Headers:** `Content-Type: application/json`.

**Body parameters:** `publicListingEnabled` boolean, `publicAlias` string, and `countryCode` string.

**Example request:**

```bash
curl -X POST http://127.0.0.1:3052/api/observer/settings \
  -H "Content-Type: application/json" \
  -d '{
    "publicListingEnabled": true,
    "publicAlias": "Brussels Observer",
    "countryCode": "BE"
  }'
```

**Example response:**

```json
{
  "ok": true,
  "settings": {
    "publicListingEnabled": true,
    "publicAlias": "Brussels Observer",
    "countryCode": "BE"
  }
}
```

**Common errors:** `400` for invalid settings or persistence failure; `403` for non-local access; `404` outside observer mode; `413`; `415`; `429`.

## Internal endpoints

Producer mining controls, invariant/debug routes, and Operator Dashboard endpoints are intentionally excluded from this public integration contract. They are for the official producer's maintainers and remain outside public navigation. Application builders must not depend on them.
