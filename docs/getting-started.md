# Getting Started

This guide runs a local Sparge producer and explorer for development or public-alpha evaluation.

## Prerequisites

- Node.js 20 is the container runtime and recommended local version.
- npm
- Git
- Windows PowerShell for the supplied operational scripts

For containers, install Docker Engine with Compose v2 instead of a local Node.js toolchain.

## Install

From the repository root:

```powershell
npm install
npm start
```

The explorer and API are available at `http://localhost:3051`. Check the node with:

```powershell
Invoke-RestMethod http://localhost:3051/api/status
```

The first start creates the configured chain data in `server/data`. Later starts reuse that data; restarting does not reset the chain.

## Start local block production

Administrative mining controls are disabled unless `dev.enableAdmin` or its environment override is enabled. For local testing:

```powershell
$env:DEV_ENABLE_ADMIN="true"
npm start
```

In another terminal:

```powershell
npm run mine:start
npm run mine:status
npm run mine:stop
```

Clear the override after testing:

```powershell
Remove-Item Env:DEV_ENABLE_ADMIN -ErrorAction SilentlyContinue
```

Never expose local mining controls on a public deployment.

## Create a wallet

```powershell
npm run wallet create
npm run wallet show
```

Use `npm run wallet show -- --full` only in a private terminal because it displays the private key. Continue with the [Wallet guide](wallet.md) for signed transactions and participation.

## Run with Docker

The local Compose stack publishes both application ports:

```powershell
docker compose up -d --build
docker compose ps
```

- Producer explorer: `http://localhost:3051`
- Observer explorer: `http://localhost:3052`

The services use separate persistent volumes. `docker compose down` keeps them; `docker compose down -v` permanently removes their chain data. Use the [Operator Guide](operator-guide.md#docker) before treating a container deployment as persistent infrastructure.

## Next steps

- Run a read-only node with the [Observer Node guide](observer.md).
- Review endpoint contracts in the [RPC API](rpc.md).
- Understand chain behavior in the [Protocol Guide](protocol.md).
- Use the [Operator Guide](operator-guide.md) for HTTPS, monitoring, backups, and recovery.
