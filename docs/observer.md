# Help Verify Sparge

An Observer Node independently downloads, validates, and stores Sparge blocks. Running one helps compare the official producer's chain state and makes network behavior more transparent.

An observer is optional. Users and application builders do not need one to create wallets, read the public API, or submit transactions.

## What an observer does

- synchronizes blocks from a configured producer
- verifies chain identity, height continuity, hashes, and state transitions
- maintains its own SQLite chain state
- serves a local read-only Explorer and API
- reports optional privacy-preserving network health

An observer does not create blocks, participate in consensus, or accept `POST /api/tx`.

## Why it matters

The current network has one official producer. Observers let independent parties:

- compare height and latest hash
- detect a chain-identity mismatch
- verify state locally
- operate a separate read endpoint for their own applications
- contribute aggregate sync-health information

Observers improve verification and transparency; they do not remove the current single-producer trust and availability model.

## Install from source

Requirements are Node.js 20, npm, Git, and enough disk space for chain growth.

```powershell
git clone https://github.com/SpargeNetwork/sparge-node.git
cd sparge-node
npm install
```

Set observer mode in `config/config.yml`:

```yaml
node:
  mode: observer
  producerUrl: "https://your-approved-sparge-endpoint.example"
```

Use a producer URL announced through an official SpargeNetwork channel. Then start:

```powershell
npm start
```

Open the local Explorer at the configured port and check `/api/status`.

## Windows desktop observer

When an official Windows installer is published, use its first-run setup to choose the producer URL and local port. Development builds can be created from source with:

```powershell
npm run dist:observer:win
```

The desktop observer stores files under `%APPDATA%\SpargeObserver\`:

- `config.json`
- `data\state.db`
- `logs\node.log`

Installers should be obtained from official SpargeNetwork release channels, not third-party download sites.

## Check synchronization

Call the observer's local `GET /api/status` and inspect:

- `nodeMode`: `observer`
- `chainId` and `genesisHash`: expected network identity
- `syncState`: `syncing`, `synced`, or `error`
- `syncedHeight` and `producerHeight`
- `lagBlocks`
- `latestHash`
- `lastSyncError`

A small temporary lag is normal. Do not use an observer as a current read source when it is in `error`, serves an unexpected chain, or exceeds your application's allowed lag.

## Privacy and public listing

Observer listing is opt-in and disabled by default:

```yaml
observer:
  publicListingEnabled: false
  publicAlias: ""
  countryCode: ""
```

When enabled, the public list can show alias, country code, version, height, lag, status, and last-seen time. It does not expose raw IP addresses, hostnames, usernames, machine metadata, internal node IDs, or latest hashes.

Private observers still contribute to aggregate network counts. Observer heartbeats are registry-only and cannot affect blocks, transactions, rewards, or consensus.

The observer stores a random persistent identity in `observer-node-id.json`. It is not derived from hardware or host identity. Keep this file when preserving the same observer identity; removing it creates a new registry identity.

## Local privacy settings

The local observer UI can update listing preferences. Local-only endpoints are also available:

- `GET /api/observer/settings`
- `POST /api/observer/settings`

Example body:

```json
{
  "publicListingEnabled": true,
  "publicAlias": "Brussels Observer",
  "countryCode": "BE"
}
```

Turning listing off takes effect on the next heartbeat while the observer continues contributing to aggregate counts.

## Updating

For a source installation:

1. Stop the observer cleanly.
2. Record its chain identity, height, and configured producer URL.
3. Pull the intended release and run `npm install` when dependencies changed.
4. Start with the same observer data directory.
5. Verify chain identity and synchronization status.

Observer state can normally be rebuilt from the producer. Preserve the old database and logs before deleting local state when investigating a mismatch.

## Troubleshooting

### Connection refused or timeout

Verify the configured producer URL, DNS, TLS, firewall, and local clock. Inside a container, `localhost` refers to that container rather than another service.

### Genesis hash mismatch

The observer is pointed at another chain or retains state from a different chain. Confirm the intended official endpoint. Preserve evidence, then reset only the observer data and synchronize again.

### Previous-hash mismatch

Local observer history diverged from the producer feed. Stop the observer, preserve its database and logs, verify chain identity, and rebuild the observer state.

### Observer remains offline in the public list

Check producer reachability, heartbeat errors, system time, and the configured interval. Public status becomes offline when no recent heartbeat arrives; it does not necessarily mean the local process has stopped.

### Transaction submission returns `403`

This is expected. Observers are read-only. Broadcast signed transactions to the official producer endpoint.
