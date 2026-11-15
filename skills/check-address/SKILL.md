---
name: check-address
description: Check one crypto address or transaction for AML risk with PublicAML — risk score 0-100, level, what the wallet is, sanctions exposure and where the risk comes from. Use when someone pastes a wallet address or tx hash, or asks "is this address clean / safe / sanctioned / linked to a scam", "check this wallet", "AML score", "risk score".
---

# Check an address or a transaction

Tools come from the `publicaml` MCP server. All of them are read-only.

## Pick the chain

Infer it from the format, ask only when it is truly ambiguous:

| Format | Chain |
|---|---|
| `T…`, 34 chars | `tron` |
| `bc1…`, `1…`, `3…` | `bitcoin` |
| `0x…` 40 hex | `ethereum` or `bsc`: the same address exists on both. Use the chain the person names; if none, screen `ethereum` first and say BNB Chain was not checked |
| base58, 32-44 chars, not `T`/`1`/`3` | `solana` (screening only) |
| 64 hex (with or without `0x`) | a transaction hash: use `screen_transaction` |

## Steps

1. **Address:** `screen_address(address, chain)`.
   **Transaction:** `screen_transaction(tx_hash, chain, focus_address?)`. Pass the person's own wallet as `focus_address` when they give one, so the verdict is from their side.
2. If the level is **MEDIUM or higher**, call `explain_risk_path(address, chain)` and say *where* the risk comes from: the source (sanctioned entity, hack, mixer, scam), the number of hops, and the amount that reached the address.
3. Answer in this order:
   - **Verdict line:** score, level and one plain sentence ("Score 12, LOW: a Binance deposit address, no risky sources found").
   - **What the address is:** the label or category, if known.
   - **Why:** direct vs indirect exposure and the top sources from step 2.
   - **Link:** the `publicaml_url` from the result, for the full report.

## Rules

- A score is a risk signal, not a legal verdict. Never write "this person is a criminal". Write "the address received funds from …".
- **Direct** exposure means the address itself is sanctioned or listed. **Indirect** exposure means funds came through other wallets. Keep the two apart. Indirect exposure 4+ hops away from a big exchange is usually noise; say so.
- If the result is empty or the address is unknown, say "no history in our index". That is not the same as "clean".
- If an owner disputes a result, point them to the dispute form on the `publicaml_url` page.
