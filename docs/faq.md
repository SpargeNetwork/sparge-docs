# Frequently Asked Questions

## Is Sparge production ready?

No. Sparge is experimental Public Alpha software. Bugs, downtime, migrations, parameter changes, and explicitly announced resets remain possible.

## Is SPRG an investment or guaranteed store of value?

No. Sparge makes no promise of value, liquidity, uptime, permanence, exchange listing, or future direction.

## Do I need to install software to use Sparge?

No. Regular users can use the browser wallet and Explorer on the existing network. Installation is optional for people running an observer.

## Where is my wallet stored?

Browser wallet keys are stored locally in your browser. Sparge does not hold a recoverable server-side account for you.

## Can Sparge recover a lost wallet?

No. Restore from your own wallet export. Without the private key or a valid backup, the wallet cannot be recovered.

## Is a queued transaction confirmed?

No. Queued or Pending means accepted for possible inclusion. It is confirmed only after appearing in a block.

## Why did a pending transaction disappear?

Pending transactions are kept in the producer's process-local mempool. A restart or later validation failure can remove one. Check the full transaction ID and confirmed address history before resubmitting.

## What is a Participant?

A Participant is a registered address that can receive a share of the Participant reward pool while active. Participants do not produce blocks.

## What is a Sponsor?

A Sponsor signs and pays `register_participant` and locks the Sponsor Bond. It gains no control over the Participant and receives no commission or reward share.

## Can I sponsor myself?

Yes. The same wallet then pays registration, locks its own bond, controls the Participant key, and receives its own Participant rewards.

## Does a Sponsor receive part of Participant rewards?

No. Rewards belong to the Participant address. Sponsorship is not referral commission, delegation, or revenue sharing.

## When is the Sponsor Bond returned?

The bond returns to the original Sponsor after the Participant successfully unregisters. Inactivity alone does not release it.

## Can a Sponsor unregister a Participant?

No. Only the Participant can currently unregister. Sponsor reclaim is unavailable in this protocol version.

## What happens if a Participant loses its wallet?

It can no longer sign heartbeats or unregister. Rewards eventually pause through inactivity, while the Sponsor Bond may remain locked indefinitely.

## Why is my Participant status Pending?

Registration has been submitted but not yet included in a block. It becomes Active only after confirmation and successful state application.

## Why is my Participant Inactive?

The Participant has not sent qualifying on-chain activity within the current activity window. It remains registered but does not receive Participant rewards until reactivated.

## Does inactivity reset Reward Maturity?

No. Inactivity pauses rewards but does not reset maturity. Unregistering and registering again does restart maturity.

## Why do new Participants receive reduced rewards?

Reward Maturity gradually increases the Participant share from 25% to 60% and then 100% based on registration age. This encourages stable participation but does not identify people or prevent multiple wallets.

## What does an observer do?

An observer independently synchronizes, verifies, and stores chain state and serves a read-only Explorer. It does not create blocks or accept transactions.

## Is observer listing private?

Public listing is opt-in. Aggregate health can include private observers, while public responses omit raw IPs, internal IDs, hostnames, machine metadata, and latest hashes.

## How are rewards divided?

See [Block reward distribution](protocol.md#block-reward-distribution) and [Reward Maturity](protocol.md#reward-maturity).

## Can I build an application on Sparge?

Yes. Start with the [Builder Guide](developer-guide.md) and [Public API](rpc.md). Public Alpha integrations should verify chain and protocol versions.

## Where are producer and node-development documents?

They are intentionally excluded from the public documentation navigation and maintained separately under `docs/internal/` in the source repository.
