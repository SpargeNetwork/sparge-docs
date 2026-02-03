# Rewards

Block rewards are minted each block and split deterministically:

- 15% → Participants (active, on‑chain participants)
- 70% → Node holders (pooled, paid on payout blocks)
- 10% → Treasury
- 5% → Eligible holders (pooled, paid on payout blocks)

## Participant rewards
Only active participants receive the 15% share.  
Active means lastSeen is within the active window.  
If there are zero active participants, the full participant share goes to the treasury.

## Holder rewards (eligible holders)
Eligibility is based on a **14‑day rolling average balance**.  
Minimum average balance: **1,000 SPRG**.  
Rewards are proportional to each eligible holder’s average balance.

Payouts happen on payout blocks (14‑day cadence in blocks), not every block.  
If there are no eligible holders at payout, the entire holder pool goes to the treasury.

## Node holder rewards
Node holder rewards are also pooled and paid on payout blocks.  
If there are no eligible node holders at payout, the pool goes to the treasury.

## Remainders
All integer remainders from splits or distributions are sent to the treasury.
