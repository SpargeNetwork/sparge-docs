# Ops / Runbook

## Producer Run

- Start node + explorer: `npm start`
- Start block production: `npm run mine:start`
- Stop block production: `npm run mine:stop`
- Mining status: `npm run mine:status`

## Observer Run (source)

In `config/config.yml`:
- `node.mode: observer`
- `node.producerUrl: "http://localhost:3051"`

Start observer:
- `npm start`

## Observer Run (Windows app)

- Build installer: `npm run dist:observer:win`
- Install: `release/Sparge Observer Setup 0.1.0.exe`
- Data/log path: `%APPDATA%\\SpargeObserver\\`

## Reset Data

Producer local reset script:
- `powershell scripts/reset-chain.ps1`

Observer app reset (desktop app):
- remove `%APPDATA%\\SpargeObserver\\data`
- optionally remove `%APPDATA%\\SpargeObserver\\config.json`

## Core Config

Main config file:
- `config/config.yml`

Important keys:
- `node.mode` (`producer` or `observer`)
- `node.producerUrl`
- `chain.*`, `token.*`, `rewards.*`, `tx.*`

## Release Notes

Do not commit build artifacts:
- `release/`
- `build/`
- `electron/runtime/`
