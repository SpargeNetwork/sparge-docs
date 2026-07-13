# Build on Sparge

This guide is for application developers. It explains how to create accounts, sign and broadcast transactions, read chain data, and design wallets, games, payment applications, explorers, marketplaces, and ecosystem tools.

The [RPC API](rpc.md) is the canonical endpoint reference. This guide focuses on workflows and design decisions rather than repeating endpoint definitions.

## Overview

A Sparge application normally has three parts:

```text
User or application
  |-- stores private keys locally
  |-- builds and signs canonical transactions
  |
  +--> Sparge JSON API
         |-- reads blocks, balances, nonces, addresses, and network status
         |-- accepts signed transactions
         |
         +--> official producer creates blocks
                |
                +--> Observer Nodes independently sync and validate
```

Applications do not send private keys to the network. Read operations can use a producer or compatible Observer Node. Signed transaction submission must use the producer because observers are read-only.

## Integration architecture

Choose an API base URL through application configuration. Do not hardcode `localhost` or silently switch between unrelated nodes.

At startup:

1. Call `GET /api/status`.
2. Verify `chainId`, `genesisHash`, `protocolVersion`, and `economicsVersion` against values your release supports.
3. Check node health and, when reading from an observer, `syncState` and `lagBlocks`.
4. Refuse signing or broadcasting when chain identity is unexpected.

See [Integration Configuration](reference/configuration.md) for a recommended client configuration model.

## Accounts and keys

Sparge accounts use Ed25519 key pairs. A wallet consists of:

- a 32-byte private key seed
- a 32-byte public key
- an address derived from the public key

The current JavaScript tooling represents both keys as lowercase hexadecimal. Private keys should be stored in an operating-system keystore, hardware-backed key store, encrypted wallet, or another application-appropriate secret store. Browser local storage is suitable only for experimental use with clearly communicated risk.

Generate a key pair in Node.js:

```javascript
import crypto from "node:crypto";

const { publicKey, privateKey } = crypto.generateKeyPairSync("ed25519");
const publicJwk = publicKey.export({ format: "jwk" });
const privateJwk = privateKey.export({ format: "jwk" });

const publicKeyHex = Buffer.from(publicJwk.x, "base64url").toString("hex");
const privateKeyHex = Buffer.from(privateJwk.d, "base64url").toString("hex");
```

Never log, transmit, commit, or include `privateKeyHex` in an error report.

## Addresses

An address is derived from the public key:

```text
spg_ + base58(sha256(publicKeyBytes)[0..20])
```

Example using the `bs58` package:

```javascript
import crypto from "node:crypto";
import bs58 from "bs58";

export function deriveAddress(publicKeyHex) {
  const publicKey = Buffer.from(publicKeyHex, "hex");
  if (publicKey.length !== 32) throw new Error("Expected a 32-byte public key");

  const payload = crypto
    .createHash("sha256")
    .update(publicKey)
    .digest()
    .subarray(0, 20);

  return `spg_${bs58.encode(payload)}`;
}
```

Always rederive the address before signing and verify it matches the transaction's `from` field.

## Amounts

SPRG uses the `decimals` value returned by `/api/status`; the current chain uses 9 decimals. Protocol and API amounts use integer strings such as `"1000000000"`, not JavaScript numbers.

Convert a user-entered decimal without floating point:

```javascript
export function tokensToBaseUnits(input, decimals) {
  if (!/^(0|[1-9]\d*)(\.\d+)?$/.test(input)) {
    throw new Error("Expected a non-negative decimal string");
  }

  const [whole, fraction = ""] = input.split(".");
  if (fraction.length > decimals) throw new Error("Too many decimal places");

  return `${whole}${fraction.padEnd(decimals, "0")}`.replace(/^0+(?=\d)/, "");
}
```

Use `BigInt` for arithmetic and strings at JSON boundaries. Never use binary floating point for balances, fees, rewards, or settlement decisions.

## Transactions

Client-submittable types are:

- `transfer`
- `register_participant`
- `unregister_participant`
- `heartbeat`

The last three relate to protocol participation. Most applications only need `transfer`.

A signed transaction contains:

```text
type, chainId, from, to, amountMicro, feeMicro, nonce,
publicKeyHex, signatureHex, txid, sponsor, participant, memo
```

Unused canonical fields must be empty strings. Do not omit or reorder signed fields in your signing implementation.

## Canonical signing

The canonical message is the UTF-8 encoding of fields joined by `|`:

```text
type|chainId|from|to|amountMicro|feeMicro|nonce|publicKeyHex|sponsor|participant|memo
```

The transaction ID is the lowercase SHA-256 hash of those message bytes. The Ed25519 signature signs the same bytes. The signature is not part of the transaction ID.

```javascript
import crypto from "node:crypto";

export function canonicalMessage(tx) {
  return [
    tx.type,
    tx.chainId,
    tx.from,
    tx.to,
    tx.amountMicro,
    tx.feeMicro,
    tx.nonce,
    tx.publicKeyHex,
    tx.sponsor,
    tx.participant,
    tx.memo
  ].join("|");
}

export function signTransaction(tx, wallet) {
  if (deriveAddress(wallet.publicKeyHex) !== tx.from) {
    throw new Error("Wallet does not match transaction sender");
  }

  const key = crypto.createPrivateKey({
    format: "jwk",
    key: {
      kty: "OKP",
      crv: "Ed25519",
      x: Buffer.from(wallet.publicKeyHex, "hex").toString("base64url"),
      d: Buffer.from(wallet.privateKeyHex, "hex").toString("base64url")
    }
  });

  const message = Buffer.from(canonicalMessage(tx), "utf8");
  return {
    ...tx,
    signatureHex: crypto.sign(null, message, key).toString("hex"),
    txid: crypto.createHash("sha256").update(message).digest("hex")
  };
}
```

Test signing code against deterministic fixtures before handling user funds. A future SDK should publish shared cross-language vectors for addresses, messages, signatures, and transaction IDs.

## Your first transfer integration

### 1. Read chain parameters

```javascript
const API = "https://your-approved-sparge-endpoint.example/api";

const status = await fetch(`${API}/status`).then((response) => response.json());
if (status.chainId !== "sparge-mainnet") throw new Error("Unexpected chain");
```

Treat the URL as configuration. Replace the example host with an endpoint announced through an official SpargeNetwork channel.

### 2. Read balance and nonce

```javascript
const [balance, nonce] = await Promise.all([
  fetch(`${API}/balance/${wallet.address}`).then((response) => response.json()),
  fetch(`${API}/nonce/${wallet.address}`).then((response) => response.json())
]);
```

`/api/nonce` returns the confirmed nonce. If your application submits several transactions from one account, serialize submissions and track accepted pending nonces locally. Do not submit parallel transactions with the same nonce.

### 3. Build and sign

```javascript
const unsigned = {
  type: "transfer",
  chainId: status.chainId,
  from: wallet.address,
  to: recipientAddress,
  amountMicro: tokensToBaseUnits("12.5", status.decimals),
  feeMicro: status.minFeeMicro,
  nonce: String(nonce.nonce),
  publicKeyHex: wallet.publicKeyHex,
  sponsor: "",
  participant: "",
  memo: "order-1042"
};

const signed = signTransaction(unsigned, wallet);
```

Before requesting signing consent, show the user the chain, recipient, exact amount, fee, and public memo.

### 4. Broadcast

```javascript
const response = await fetch(`${API}/tx`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify(signed)
});

const result = await response.json();
if (!response.ok) throw new Error(result.error || result.message || "Broadcast failed");
```

A successful response has `status: "queued"`. Queued is not confirmed.

### 5. Confirm

Poll `GET /api/tx/:txid` with bounded exponential backoff. It returns `404` until the transaction is stored in a block. Stop after an application-defined timeout and show an unknown or expired state rather than claiming failure or success.

```javascript
export async function waitForConfirmation(api, txid, attempts = 30) {
  for (let i = 0; i < attempts; i += 1) {
    const response = await fetch(`${api}/tx/${txid}`);
    if (response.ok) return response.json();
    if (response.status !== 404) throw new Error(`Confirmation lookup failed: ${response.status}`);
    await new Promise((resolve) => setTimeout(resolve, Math.min(1000 * 2 ** i, 15000)));
  }
  return null;
}
```

## Reading blockchain data

Use the [RPC API](rpc.md) for exact parameters and response fields.

Common read patterns:

- dashboard: status, network status, and recent blocks
- account screen: address summary plus address transactions
- payment confirmation: transaction by ID
- fee display: status `minFeeMicro` and `baseFeeMicro`
- explorer indexer: blocks from a persisted height cursor
- Observer comparison: status identity, height, hash, and lag

Cache immutable confirmed blocks and transactions. Refresh mutable status, mempool, balance, and observer data according to user needs and documented rate limits.

## Polling and subscriptions

Sparge currently provides HTTP polling only. There is no WebSocket, Server-Sent Events, webhook, or subscription API.

Recommended polling behavior:

- poll `/api/status` every 5 to 15 seconds for interactive applications
- stop or slow polling when a browser tab is hidden
- use exponential backoff with jitter after network errors or HTTP `429`
- honor `Retry-After`
- request block ranges from the last persisted height instead of downloading full history repeatedly
- never poll every endpoint for every user independently from a shared backend

Builders needing push notifications should operate an application backend that polls once, persists its cursor, and sends application-specific notifications to clients.

## Building a wallet

A production-quality wallet should provide:

- secure key generation and encrypted backup
- explicit network identity and endpoint selection
- address and public-key consistency checks
- exact integer amount conversion
- recipient validation and review before signing
- fee and nonce retrieval
- pending, confirmed, rejected, expired, and unknown states
- transaction history with public-memo warnings
- recovery and export without sending secrets to a server

Do not invent a different canonical serialization. Reuse shared test vectors when an official SDK becomes available.

## Building a payment application

For merchant payments:

1. Create an invoice identifier in your application database.
2. Associate it with a receiving address, a public memo, or both.
3. Broadcast only from the payer's wallet; the merchant never needs the payer's private key.
4. Match confirmed transfers by transaction ID and expected amount.
5. Make reconciliation idempotent so one block or transaction cannot be credited twice.
6. Display confirmation timeouts and public-alpha risk clearly.

Memos are public and limited to 128 UTF-8 bytes. Never place names, email addresses, secrets, authentication tokens, or sensitive order details in them.

Address history returns a bounded recent list. A payment service that cannot miss older transactions should index blocks from a persistent height cursor.

## Building a game

Sparge does not run game logic or smart contracts. Keep gameplay, randomness, anti-cheat controls, and inventory in your application. Use ordinary transfers only for flows the protocol supports.

Recommended design:

- keep a server-side ledger linking game events to transaction IDs
- wait for block confirmation before granting irreversible value
- make reward and purchase handling idempotent
- use exact base units for prices
- never ask players to paste private keys into the game
- distinguish off-chain game state from canonical Sparge balances

Do not describe off-chain game outcomes as trustless or enforced by Sparge.

## Building a marketplace

Listings, search, moderation, escrow, disputes, and asset ownership are not protocol features. A marketplace can publish listings off-chain and use Sparge transfers for payment, but any escrow or delivery guarantee remains an application responsibility.

Store transaction IDs with orders, verify recipient and amount from confirmed chain data, and design replay-safe fulfillment. Never infer an NFT or token transfer from a memo alone.

## Building an explorer or indexer

Use `GET /api/blocks?fromHeight=<height>&limit=<limit>` to process blocks incrementally.

Persist at least:

- chain ID and genesis hash
- last indexed height and block hash
- block and transaction records needed by your product
- address-to-transaction indexes

For every batch, verify chain identity, sequential heights, and `prevHash`. Stop on a mismatch instead of silently joining two histories. The current API does not provide a block-by-hash lookup or subscription stream.

The address-history endpoint is useful for a wallet UI but is not a replacement for a complete indexer because its result is bounded.

## Error handling

Handle status codes before parsing business results:

| Status | Meaning for applications | Recommended action |
| ---: | --- | --- |
| `400` | Invalid shape, signature, nonce, fee, balance, or protocol state | Show a safe specific message; rebuild state-dependent transactions. |
| `403` | Write sent to an observer or listing/settings unavailable | Use the producer for broadcast; do not retry blindly. |
| `404` | Block/transaction absent or internal route disabled | For transaction confirmation, continue bounded polling. |
| `409` | Duplicate transaction | Treat the transaction ID as already submitted and check confirmation. |
| `413` | Body too large | Fix the client payload; do not retry unchanged. |
| `415` | Wrong content type | Send JSON with `application/json`. |
| `429` | Rate or sender limit | Honor `Retry-After` and back off. |
| `503` | Mempool unavailable or full | Retry later without changing nonce unless chain state changed. |

Some business-validation errors are human-readable strings rather than stable machine codes. Branch on HTTP status and documented structured codes only; display unknown messages safely and never parse them as a protocol contract.

## Best practices

- Pin chain identity and supported versions.
- Keep private keys out of application servers whenever possible.
- Derive and verify the sender address before signing.
- Use strings and `BigInt`, never floating point.
- Serialize transactions per sender account.
- Treat queued transactions as pending.
- Use bounded retries, jitter, and `Retry-After`.
- Persist indexer cursors and verify hash continuity.
- Escape all aliases, memos, addresses, hashes, and error text before rendering.
- Avoid sensitive data in public memos and URLs.
- Make payment, game, and marketplace fulfillment idempotent.
- Show public-alpha limitations in user-facing risk decisions.

## SDK status

There is no official JavaScript, TypeScript, mobile, or backend SDK yet. Builders currently integrate with the JSON API and canonical signing rules directly.

A future SDK should include:

- typed RPC clients
- exact amount conversion
- address derivation and validation
- canonical transaction construction
- Ed25519 signing adapters
- deterministic cross-language test vectors
- polling, retry, and confirmation helpers
- indexer cursor utilities

Until then, keep protocol utilities small, test them against the node implementation, and isolate them from application business logic.
