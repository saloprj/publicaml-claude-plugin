---
name: verify-invoice-payment
description: Verify that a counterparty paid an invoice in crypto — finds the matching on-chain transfer to your wallet, checks asset, amount and date, and AML-screens the payer with PublicAML before you accept it. Use for "did X pay invoice N", "check this payment", "confirm the client paid", "is this tx the payment for the invoice", or when given an invoice plus a tx hash or payer address.
---

# Verify an invoice payment

## 1. Collect the invoice facts

You need:

- **Receiving wallet and chain**: the wallet from the invoice. If it is not given, use the wallet list from the `incoming-payments` skill.
- **Asset**: for example USDT TRC-20, USDT ERC-20, USDC or BTC.
- **Amount due**, and the tolerance (default: exact; allow network-fee shortfall only if the invoice says the payer covers fees).
- **Invoice date or due date.**
- **Something that identifies the payer**: the tx hash they sent, their address, or nothing.

If an invoice file or PDF is attached, read these facts from it. Ask only for what is missing.

## 2. Find the transfer

- **Tx hash given:** call `screen_transaction(tx_hash, chain, focus_address: receiving wallet)`. Confirm three things: the asset, that the recipient is the receiving wallet, and the amount.
- **Otherwise:** call `list_transactions(receiving wallet, chain, since: invoice date - 1 day)` and keep `direction: "in"` in the right asset. Match by amount (exact first, then within tolerance), then by payer address if known. Several partial payments that add up to the total count as one payment; list each.
- Treat token decimals carefully: USDT and USDC have 6 decimals on TRON and Ethereum and 18 on BSC. Use the `value` field, or `amount_raw` / 10^decimals.

## 3. Screen the payer

Call `screen_address(payer, chain)`. If the payer is MEDIUM or higher, also call `explain_risk_path`. If the payer is an exchange hot wallet (label category `cex`), say so: the real payer is that exchange's customer, and the score describes the exchange, not them.

## 4. Verdict

Give one of these:

| Verdict | Meaning |
|---|---|
| **PAID ✓** | amount, asset and wallet match, and the payer is LOW or MEDIUM. Give the tx hash, date (UTC) and amount |
| **PAID, HOLD FOR REVIEW** | matched, but the payer is HIGH or CRITICAL. Give the reason and risk path. Do not release goods or services until compliance clears it |
| **UNDERPAID / OVERPAID** | show expected, received and the difference |
| **WRONG ASSET / WRONG NETWORK** | for example USDT sent on ERC-20 to a TRON invoice. Say what arrived and where |
| **NOT FOUND** | nothing matching since the invoice date. Say what you searched (wallet, asset, window). Suggest asking the payer for the tx hash |

## Rules

- Never confirm a payment from a screenshot or a payer's word alone. It must be visible on-chain.
- A transaction that is unconfirmed or not yet in the index is "not found yet", not "unpaid". Suggest re-checking in a few minutes.
- Do not advise refunding a flagged payer; escalate instead.
