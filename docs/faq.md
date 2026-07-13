# Frequently Asked Questions

## What is Sparge?

Sparge is an experimental blockchain and JSON API for self-custody wallets, transfers, explorers, games, payment applications, marketplaces, and ecosystem tools.

## Is Sparge production ready?

No. It is public-alpha software. APIs, economics, compatibility, and chain data may change before a stable release.

## Is SPRG an investment or guaranteed store of value?

No. Sparge makes no promise of value, uptime, permanence, exchange listing, or future direction.

## Do I need to run a node?

No. Users and application builders can use an approved public API. Running an Observer Node is optional and is intended for independent verification or operating your own read endpoint.

## Where can I find the public Explorer and Wallet?

The Explorer serves the browser Wallet at `/wallet`. The official repositories do not currently declare a canonical public deployment URL. Use only a URL announced through an official SpargeNetwork channel; do not trust unrelated search results or third-party downloads.

## Where do wallet keys live?

Browser wallet keys remain in browser storage and CLI keys remain in the local wallet file. They are not stored by the node. Wallet exports contain sensitive private key material.

## How do I receive SPRG?

Share your complete `spg_` address. The sender can transfer to it, and you can confirm receipt by transaction ID or address in the Explorer. The balance changes after block inclusion, not merely after submission.

## Why does my transaction say Pending?

The producer accepted it into a temporary mempool but has not yet included it in a block. Pending is expected between broadcast and confirmation. Keep the transaction ID and check it before attempting another payment.

## Can pending transactions disappear?

Yes. The current mempool is not durable and clears on producer restart. A transaction that never confirms may need to be rebuilt using the current nonce and resubmitted. Never resubmit without first checking the original transaction ID and account state.

## Are transaction memos private?

No. Memos are public chain data and can be indexed permanently. Do not include personal or confidential information.

## Can I build a wallet or application without an SDK?

Yes. Sparge uses a documented JSON API and canonical Ed25519 signing format. The [Builder Guide](builder-guide.md) includes JavaScript examples. No official SDK is available yet.

## Does Sparge support smart contracts or tokens?

No smart-contract runtime, custom-token API, or NFT standard is currently exposed. Applications can keep their own off-chain state and use supported SPRG transfers for settlement.

## How do applications receive live updates?

They poll the HTTP API. There is currently no WebSocket, Server-Sent Events, webhook, or subscription service. Shared application backends should poll once and distribute updates to clients.

## How many confirmations are final?

The current protocol confirms inclusion in the chain but has no separate decentralized finality mechanism. Applications must choose a public-alpha confirmation policy appropriate to their risk.

## What does an Observer Node do?

It downloads and validates blocks independently, stores local chain state, and serves a read-only API and Explorer. It cannot produce blocks or accept transaction submissions.

## Is Observer listing private?

Listing is opt-in. Aggregate health includes private observers, while public entries omit IPs, internal IDs, hostnames, usernames, machine metadata, and latest hashes.

## Why is an Observer syncing or mismatched?

Temporary lag is normal. Persistent errors usually indicate an unreachable producer, different genesis, or divergent local observer data. See [Observer troubleshooting](observer.md#troubleshooting).

## How are rewards calculated?

See [Rewards and economics](protocol.md#rewards-and-economics). Reward rules are protocol behavior; application builders should not reproduce them using floating point.

## Where is the endpoint reference?

See the [RPC API](rpc.md). Application workflows and signing are in [Build on Sparge](builder-guide.md).
