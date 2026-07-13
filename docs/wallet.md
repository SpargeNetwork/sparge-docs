# Wallet

Sparge wallets generate Ed25519 keys locally. Browser wallet keys stay in browser storage; CLI wallet keys stay in the local wallet file. Private keys are not sent to producer or observer endpoints.

## Create and inspect a CLI wallet

```powershell
npm run wallet create
npm run wallet show
```

`npm run wallet show -- --full` displays private material and should only be used in a private terminal. Back up exported wallet material separately from node backups: producer backups contain chain state, not wallet private keys.

## Send a transfer

```powershell
npm run tx send --to <spg_address> --amount 1 --fee 0.000001 --memo "optional"
```

The CLI signs locally and submits the signed transaction to the configured producer. An observer rejects `POST /api/tx` with HTTP `403` because observer mode is read-only.

## Participation transactions

The wallet supports `register_participant`, `unregister_participant`, and protocol `heartbeat` transactions in addition to transfers. These on-chain participant heartbeats are distinct from the private observer-node heartbeat used for network-health reporting.

Registration status can remain pending until the transaction is included in a block. See [Participation](protocol.md#participation) for bond, sponsor, active-window, and genesis-operator rules.

## Browser wallet

The browser wallet provides client-side key generation, transaction signing, and JSON import/export. Treat exported JSON as sensitive private-key material. Do not upload it to the explorer, operator dashboard, issue tracker, or support channel.

## Address format

An address is:

```text
spg_ + base58(sha256(publicKeyBytes)[0..20])
```

`publicKeyHex` is the raw 32-byte Ed25519 public key encoded as 64 lowercase hexadecimal characters. For canonical signing and transaction fields, see the [Protocol Guide](protocol.md#transactions).
