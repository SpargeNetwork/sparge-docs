# Wallet

The Sparge browser wallet creates and signs transactions locally. Private keys stay in browser storage unless you explicitly export them.

## Create a wallet

1. Open **Wallet** in the official Explorer.
2. Choose **Create Wallet**.
3. Read the recovery warning and confirm.
4. Export a wallet backup before receiving funds.

Creating another wallet creates a separate key and address. It does not replace or recover an existing wallet.

## Import a wallet

Use **Import Wallet** and select a trusted Sparge wallet export. Import only on a device and Explorer URL you trust. Never upload wallet files to an issue, support chat, observer, or block explorer form.

## Select and identify wallets

The wallet selector shows locally available wallets by name and shortened address. Shortening is display-only: transactions, links, and copy controls use the complete address or transaction ID.

Give local wallets recognizable names, but always verify the full destination address before sending.

## Receive

The Receive view shows the complete public address. Sharing an address is safe; sharing a private key or wallet export is not.

The displayed balance distinguishes confirmed funds from pending activity. A pending transfer can still fail to confirm and should not be treated as final.

## Send

Enter the destination and amount, then review the confirmation dialog carefully. Check:

- the complete recipient address;
- the amount and fee;
- the selected sending wallet;
- that the Explorer is connected to the expected Sparge network.

After submission, follow the full transaction ID from Wallet Activity. A successful submission means queued, not confirmed.

## Participation status

The wallet distinguishes these states:

- **Not registered**: no confirmed Participant record exists.
- **Pending**: registration is queued but not yet included in a block.
- **Active**: registered and within the activity window.
- **Inactive**: still registered, but temporarily not eligible for Participant rewards.

Registration, heartbeat, and unregister are signed on-chain transactions. They require confirmation like a transfer.

## Reward Maturity

Registered wallets show maturity percentage, stage, multiplier, Registered Height, age, progress to the next stage, remaining blocks, and Active or Inactive eligibility.

Current stages are:

| Participant age | Reward multiplier | Stage |
| ---: | ---: | --- |
| 0-5,100 blocks | 25% | New |
| 5,101-10,200 blocks | 60% | Growing |
| 10,201+ blocks | 100% | Mature |

Inactivity pauses rewards but does not reset maturity. A full unregister removes the record; registering again starts from the New stage.

## Sponsorships

A wallet that sponsors Participants can inspect active and inactive records, available active slots, locked Sponsor Bond, maturity, registration height, and last activity.

The Sponsor pays and signs registration and locks the bond. It does not control the Participant wallet and receives no commission or reward share. The Participant must sign its own future transactions.

The bond returns to the original Sponsor after a successful Participant-initiated unregister. Sponsor reclaim is unavailable in this protocol version. If the Participant loses its key, the bond may remain locked indefinitely.

## Back up and recover

Keep at least one encrypted or physically secured copy of the wallet export outside the browser profile. Test that you can identify the backup without exposing it.

There is no password reset or server-side recovery. Clearing browser storage, replacing a device, or losing the export can permanently remove access.

## Privacy

Addresses, balances, confirmed transactions, participation records, and sponsorship relationships are public chain data. Wallet names and private keys remain local unless you expose them yourself.
