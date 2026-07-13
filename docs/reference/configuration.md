# Configuration Reference

The primary configuration file is `config/config.yml`. Environment variables override selected runtime and deployment values. Chain-identity and economics changes can make existing data incompatible; back up and verify before changing them.

## Precedence and startup validation

The node loads YAML configuration and applies supported environment overrides. Invalid modes, paths, body limits, rate limits, mempool bounds, and invariant values fail startup rather than being silently corrected.

Keep recovery-relevant configuration with producer backups. Never put wallet private keys, seeds, or unrelated secrets in node configuration.

## Chain

```yaml
chain:
  name: Sparge
  symbol: SPRG
  chainId: "sparge-mainnet"
  blockTimeSeconds: 51
  protocolVersion: "1.0.0"
  economicsVersion: "1.0.0"
```

- `name`, `symbol`: display identity.
- `chainId`: canonical transaction and storage identity.
- `blockTimeSeconds`: target producer interval and economics timing input.
- `protocolVersion`, `economicsVersion`: compatibility identities checked by storage, sync, backup, and replay.

Changing identity/version values can require a documented migration or fresh chain. Do not edit them on a running chain without a release plan.

## Supply and addresses

```yaml
token:
  decimals: 9
  initialSupplyTokens: "5100000000"

mining:
  proposerAddress: "spg_..."
  genesisOperatorAddress: "spg_..."
  genesisFreeBlocks: 100

rewards:
  treasuryAddress: "spg_..."
  nodePoolAddress: "spg_..."
  holderPoolAddress: "spg_..."
```

Amounts are strings where integer precision matters. `proposerAddress` receives producer-directed protocol effects. `genesisOperatorAddress` identifies the one-time free participant-registration candidate; it is not a private key. Pool addresses are transparent unspendable system addresses.

Changing any genesis-bound value on an existing data directory can trigger identity/audit failure. A removed `genesis.json` is not a supported migration mechanism; restore it from backup or initialize an intentionally empty data directory.

## Storage and gas

```yaml
storage:
  backend: sqlite
  blocksPerFile: 510

gas:
  blockLimit: 510
  targetRatioBps: 8000
  baseFeeInitialMicro: "1000"
  baseFeeChangeDenominator: 8
  minBaseFeeMicro: "0"

tx:
  minFeeMicro: "1000"
```

SQLite is the supported durable producer and observer backend. Fee and gas values are protocol-sensitive. `tx.minFeeMicro` is the route-level minimum used for submitted transactions.

## Node and synchronization

```yaml
node:
  mode: producer
  producerUrl: "http://localhost:3051"
  sync:
    batchSize: 50
    intervalMs: 2000
    timeoutMs: 5000
```

- `mode`: `producer` or `observer`; override with `NODE_MODE`.
- `producerUrl`: observer upstream; override with `PRODUCER_URL`.
- `sync.batchSize`: blocks requested per sync pass.
- `sync.intervalMs`: polling interval.
- `sync.timeoutMs`: producer request timeout.

General runtime overrides include `PORT`, `DATA_DIR`, and `CONFIG_PATH`. Use separate data directories for every node process.

## Mempool

```yaml
mempool:
  sort: fee
  maxTransactions: 10000
  maxBytes: 52428800
  maxTransactionsPerSender: 100
  transactionTtlSeconds: 3600
  maxFutureNonceGap: 100
  minimumFeeMicro: "0"
```

Count, byte, sender, TTL, and nonce-gap values must be positive safe integers; `minimumFeeMicro` is a non-negative integer string. Raising bounds increases worst-case memory and scan costs. This pool is process-local and non-durable.

## Runtime invariants

```yaml
invariants:
  enabled: true
  fastChecksEveryBlock: true
  fullAuditOnStartup: true
  fullAuditIntervalBlocks: 0
  stopMiningOnFailure: true
```

`fullAuditIntervalBlocks: 0` disables periodic full audits, not startup or fast checks. Keep safety pause enabled on a public producer unless carrying out an isolated, documented recovery operation.

## Structured logging

```yaml
logging:
  level: info
  format: pretty
  directory: logs
  fileEnabled: true
  consoleEnabled: true
  maxFileSizeBytes: 10485760
  maxFiles: 10
  redactSensitiveFields: true
  logEmptyBlocks: false
```

Levels are `debug`, `info`, `warn`, `error`, and `fatal`; formats are `pretty` and newline-delimited `json`. The directory must be relative to `DATA_DIR`; absolute and traversal paths are rejected.

Overrides: `LOG_LEVEL`, `LOG_FORMAT`, `LOG_DIRECTORY`, `LOG_FILE_ENABLED`, and `LOG_CONSOLE_ENABLED`.

## Observer registry and privacy

```yaml
observer:
  publicListingEnabled: false
  publicAlias: ""
  countryCode: ""

network:
  heartbeatIntervalSeconds: 60
  observerOfflineAfterSeconds: 180
  observerRetentionDays: 180
  publicObserverListEnabled: true
  heartbeatRateLimit:
    windowMs: 60000
    max: 20
```

Observer overrides are `OBSERVER_PUBLIC_LISTING_ENABLED`, `OBSERVER_PUBLIC_ALIAS`, and `OBSERVER_COUNTRY_CODE`. The nested legacy heartbeat limiter remains configuration-compatible; the endpoint-specific security limiter is the primary layered control.

## HTTP and CORS

```yaml
http:
  corsAllowedOrigins:
    - "http://localhost:3051"
    - "http://127.0.0.1:3051"
  rateLimit:
    windowMs: 60000
    max: 600
  txRateLimit:
    windowMs: 60000
    max: 60
  blocksSyncRateLimit:
    windowMs: 60000
    max: 120
```

Use exact CORS origins in production; override with `CORS_ALLOW_ORIGINS`. Legacy HTTP limit values remain for route compatibility, while the `security.rateLimits` groups below provide the main endpoint-specific policy.

## Body and rate limits

```yaml
security:
  trustProxy: false
  maxJsonBodyBytes: 32768
  maxTransactionBodyBytes: 16384
  maxHeartbeatBodyBytes: 4096
  maxObserverSettingsBodyBytes: 4096
  rateLimits:
    enabled: true
    global: { windowSeconds: 60, maxRequests: 300 }
    transaction: { windowSeconds: 60, maxRequestsPerIp: 10, maxConcurrentPerIp: 3 }
    heartbeat: { windowSeconds: 60, maxRequestsPerIp: 10, maxRequestsPerNodeId: 2 }
    observerSettings: { windowSeconds: 60, maxRequestsPerIp: 10 }
    addressHistory: { windowSeconds: 60, maxRequestsPerIp: 30 }
    blockAndTxLookup: { windowSeconds: 60, maxRequestsPerIp: 60 }
    publicRead: { windowSeconds: 60, maxRequestsPerIp: 120 }
    operator: { windowSeconds: 60, maxRequestsPerIp: 5 }
```

Body-limit overrides:

- `MAX_JSON_BODY_BYTES`
- `MAX_TRANSACTION_BODY_BYTES`
- `MAX_HEARTBEAT_BODY_BYTES`
- `MAX_OBSERVER_SETTINGS_BODY_BYTES`

`SECURITY_TRUST_PROXY=1` trusts exactly one proxy hop and is intended for the supplied Caddy topology. Leave proxy trust false when the node is directly reachable. Disable rate limits only in an isolated test environment.

## Local administration

```yaml
dev:
  enableAdmin: false

operatorDashboard:
  enabled: false
  bindLocalOnly: true
```

The secure behavior when the dashboard key is absent is disabled. For public production, explicitly set `operatorDashboard.enabled: false` and retain local-only binding. Environment overrides are `DEV_ENABLE_ADMIN`, `OPERATOR_DASHBOARD_ENABLED`, and `OPERATOR_DASHBOARD_LOCAL_ONLY`.

The dashboard may be temporarily enabled in a developer checkout for local evaluation. That local choice must not be copied into a public deployment.

## Docker environment

The Compose files additionally use:

- `SPARGE_PRODUCER_PORT`, `SPARGE_OBSERVER_PORT`: local published ports
- `SPARGE_DOMAIN`: public Caddy hostname

Do not put wallet keys or production secrets in `.env`, Compose YAML, or image layers.
