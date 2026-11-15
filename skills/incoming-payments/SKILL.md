---
name: incoming-payments
description: Screen every incoming crypto payment to your own wallets for AML risk with PublicAML — lists new incoming transfers, risk-scores each sender, flags HIGH/CRITICAL ones and explains why. Use for "check incoming transactions", "screen our deposits", "AML check on what we received", "who sent us money this week", or a daily/weekly KYT review.
---

# Screen incoming payments (KYT)

## 1. Get the wallets

Look for a wallet list in this order and use the first one found:

1. The wallets the person names in the request.
2. `./publicaml-wallets.txt` in the current project.
3. `~/.claude/publicaml-wallets.txt`.

Format is one wallet per line, as `chain address  # optional note`. Blank lines and lines starting with `#` are ignored. For example:

```
tron TXyz...   # main USDT wallet
ethereum 0xabc...   # treasury
```

If there is no list, ask for the wallets. Offer to save them to `~/.claude/publicaml-wallets.txt` so the next run needs no setup.

## 2. Pull incoming transfers

For each wallet, call `list_transactions(address, chain, since, limit: 100)`:

- `since`: the date the person gives. Otherwise use the date of the last run recorded in `~/.claude/publicaml-last-check.txt`. If there is no record, use the last 7 days.
- Keep only `direction: "in"`.
- **Drop spam.** Drop zero or dust values, unknown airdrop tokens (`symbol: null`, TRC-10 `edge_type: "trc10"` junk), and fake "address-poisoning" transfers. The tell for poisoning is a sender that copies the first and last characters of a real counterparty, with a 0-value transfer. Keep native coins (BTC, ETH, BNB, TRX) and real stablecoins:
  - USDT on TRON: `TR7NHqjeKQxGTCi8q8ZY4pL8otSzgjLj6t`
  - USDT on Ethereum: `0xdAC17F958D2ee523a2206206994597C13D831ec7`
  - USDC on Ethereum: `0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48`
  - USDT on BSC: `0x55d398326f99059fF775485246999027B3197955`
  - USDC on BSC: `0x8AC76a51cc950d9822D68b83fE1Ad97B32Cd580d`
- If `has_more` is true, page with `next_cursor` until you pass `since`.

## 3. Score each distinct sender

Call `screen_address(sender, chain)` once per distinct sender, not once per transfer. For every sender at **HIGH or CRITICAL**, also call `explain_risk_path`.

## 4. Report

Start with one line: "N incoming payments, M senders, K flagged".

Then give a table, flagged rows first:

| Date (UTC) | Wallet | From | Amount | Score / level | What the sender is | Action |

Fill the **Action** column like this:

| Level | Action |
|---|---|
| LOW | accept |
| MEDIUM | accept and note the reason. Indirect exposure only is normal for exchange-funded wallets |
| HIGH | hold. Ask the counterparty for source of funds. Show the risk path |
| CRITICAL, or direct sanctions | do not move or return the funds on your own. Escalate to the compliance officer and keep the evidence: the tx hash and the `publicaml_url` |

Finish by writing today's date to `~/.claude/publicaml-last-check.txt`, so the next run only picks up new transfers.

## Rules

- Never advise sending funds back to a flagged sender. That is a compliance decision, and it can itself be a violation.
- Results are risk signals. Phrase them as "received funds linked to …", never as accusations.
- The person can make this automatic. With `/loop 1d /publicaml:incoming-payments` it runs daily while Claude Code is open.
