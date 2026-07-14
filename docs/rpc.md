# Public API

The Sparge Explorer exposes a JSON API under `/api`. Use the official HTTPS API origin published by Sparge. Localhost examples in source-code documentation are development examples, not a separate public network.

## Common behavior

- JSON write requests require `Content-Type: application/json`.
- Identifiers and signed fields are validated strictly.
- List endpoints use bounded pagination or limits.
- Every response includes an `X-Request-ID` header.
- HTTP `413` means the request body is too large.
- HTTP `429` means the client must slow down and respect `Retry-After`.
- A queued transaction is not confirmed until included in a block.

## Chain

### `GET /api/status`

Returns chain identity, version information, node mode, health, height, latest hash, mempool aggregates, configured public economics, and Participant reward-ramp metadata.

Applications should verify chain ID, genesis hash, protocol version, and economics version before enabling signed transactions.

### `GET /api/genesis`

Returns the canonical public genesis document used to verify chain identity.

## Balances and nonces

### `GET /api/balance/:address`

Returns the public address and confirmed balance in integer base units.

### `GET /api/nonce/:address`

Returns the confirmed sender nonce. Transaction builders must also account for sequential transactions already pending from the same sender.

## Blocks

### `GET /api/blocks?page=<page>&limit=<limit>`

Returns paginated recent blocks. Default limit is 10 and maximum limit is 50.

### `GET /api/block/:height`

Returns one block by non-negative integer height.

### `GET /api/blocks?fromHeight=<height>&limit=<limit>`

Returns a bounded ascending block range used by observers. Maximum limit is 200.

### `GET /api/blocks/state`

Returns the current public chain-state representation used by node status.

## Transactions

### `GET /api/tx/:txid`

Returns a confirmed transaction. `txid` must be exactly 64 lowercase hexadecimal characters. Shortened display IDs are never valid API identifiers.

### `POST /api/tx`

Submits a locally signed transaction to the producer. Observer nodes reject writes with HTTP `403`.

Accepted canonical fields are:

```text
type, chainId, from, to, amountMicro, feeMicro, nonce,
publicKeyHex, signatureHex, sponsor, participant, memo
```

Supported client transaction types are `transfer`, `register_participant`, `heartbeat`, and `unregister_participant`.

A successful submission resembles:

```json
{
  "status": "queued",
  "txid": "<64 lowercase hex characters>",
  "message": "<canonical signed message>"
}
```

Poll `GET /api/tx/:txid` with the complete ID or inspect address history to determine confirmation.

## Addresses

### `GET /api/address/:address`

Returns public address aggregates, including balance/activity statistics and, when applicable, Participant and sponsorship data.

Important Participant fields include:

| Field | Meaning |
| --- | --- |
| `participant.status` | Current `active` or `inactive` eligibility. |
| `participant.sponsor` | Address that signed registration and locked the bond. |
| `participant.bondMicro` | Locked bond in integer base units. |
| `participant.registeredHeight` | Start of the current registration. |
| `participant.lastSeenHeight` | Latest qualifying on-chain activity. |
| `participant.rewardMaturityPercent` | Current effective maturity percentage. |
| `participant.rewardMaturityStage` | `New`, `Growing`, or `Mature`. |
| `participant.maturityAgeBlocks` | Current registration age in blocks. |
| `participant.blocksUntilNextMaturityStage` | Remaining blocks, or `null` at maturity. |

Sponsor aggregate fields include `sponsoredActiveCount`, `sponsoredInactiveCount`, `availableSponsorSlots`, `lockedBondMicro`, and `sponsoredParticipants`.

`reclaimableBondMicro` is `null`: Sponsor reclaim is unavailable in this protocol version.

### `GET /api/address/:address/txs?limit=<limit>`

Returns bounded address activity. Default limit is 50 and maximum limit is 100.

## Mempool

### `GET /api/mempool`

Returns public pending transaction and aggregate mempool information. The mempool is process-local; pending entries can disappear after restart.

## Network

### `GET /api/network/status`

Returns producer status, observer aggregates, chain and observer heights, average lag, block timing, and mempool size. Aggregate counts can include private observers.

### `GET /api/network/observers?page=<page>&limit=<limit>`

Returns only observers that opted into public listing. Optional filters are `status`, `version`, and two-letter `country`.

The response does not expose raw IP addresses, hostnames, internal node IDs, machine metadata, or latest block hashes. It can return HTTP `403` when public listing is disabled while aggregate counts remain available.

### `POST /api/network/heartbeat`

Used by observer software to report recent health. Heartbeats update only the observer registry and cannot alter blocks, transactions, balances, validation, mining, or consensus.

## Errors

| HTTP | Typical code | Meaning |
| ---: | --- | --- |
| 400 | `VALIDATION_ERROR` | Body, route parameter, or query is invalid. |
| 400 | `NONCE_TOO_FAR_AHEAD` | Sender nonce is beyond the accepted pending range. |
| 403 | `OBSERVER_READ_ONLY` | A write was sent to an observer. |
| 404 | varies | Block or transaction does not exist. |
| 409 | `MEMPOOL_DUPLICATE` | Transaction is pending or already confirmed. |
| 413 | `PAYLOAD_TOO_LARGE` | Request exceeded its byte limit. |
| 415 | `UNSUPPORTED_MEDIA_TYPE` | A JSON endpoint received another content type. |
| 429 | `RATE_LIMITED` | Request allowance was exhausted. |
| 429 | `MEMPOOL_SENDER_LIMIT` | Sender has too many pending transactions. |
| 503 | `MEMPOOL_FULL` | Producer mempool is at capacity. |

Error text can improve over time. Integrations should rely on HTTP status and documented machine-readable codes, not exact message wording.
