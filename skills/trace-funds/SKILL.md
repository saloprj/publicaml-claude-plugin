---
name: trace-funds
description: Trace where crypto went or came from with PublicAML — follow stolen or suspicious funds forward to exchanges, mixers and bridges, walk a wallet's funding backwards, and continue through DEX swaps. Use for "where did the money go", "trace these funds", "I was hacked / scammed", "where did this wallet get its money", "source of funds".
---

# Trace funds

## Pick the direction

| Question | Tool |
|---|---|
| "Where did the money go?" (theft, scam, cash-out) | `trace_funds(address, chain, max_hops?)`. Start at the default of 3 hops; raise it only if the trail stops at an unlabelled wallet |
| "Where did this wallet's money come from?" | `trace_source_of_funds(address, chain, max_hops?)` |
| A hop goes through a DEX or aggregator swap (ethereum / bsc) | `follow_swap(tx_hash, chain)`. This shows who the swap was really for; continue the trace from that address in the new asset |
| Who does the wallet deal with? | `list_counterparties(address, chain, with_risk: true)`. Filter with `category` (`cex`, `mixer`, `bridge`, `gambling`) |

## For a victim of theft

1. Get the victim address, chain and the theft tx hash or time.
2. Call `trace_funds` from the **thief's** address: the receiver of the theft tx, not the victim.
3. Report endpoints that can still act:
   - **Exchange deposit addresses.** The exchange can freeze the funds. Name the exchange; the victim should report to it with the tx hashes.
   - **USDT or USDC still sitting in a wallet.** Tether or Circle can freeze it, but only on a law-enforcement request.
   - **Mixers and cross-chain bridges.** The trail may end here; say so honestly.
4. Tell the victim to file a police report. Exchanges and Tether act on official requests. Give them the list of addresses and tx hashes to attach.

## Output

Show the path as a short chain:

`victim → 0xthief… (1 hop) → Binance deposit 0x… (2 hops, 12,400 USDT, 2026-11-03)`

Then give a table of endpoints: label, hop, amount and asset, date, tx hash. Add the `publicaml_url` of the key addresses.

## Rules

- Never name an exchange, bridge, relayer or MEV bot as the attacker. They are pass-through services.
- Amounts and hops come from the tool output only. Do not estimate missing hops.
- Recovery is never guaranteed. Do not promise it, and warn about "recovery services" that ask for upfront fees: they are a common second scam.
