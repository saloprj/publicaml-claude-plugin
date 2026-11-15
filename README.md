# PublicAML plugin for Claude Code

Free crypto AML screening inside Claude Code, from [PublicAML](https://publicaml.org). No account, no API key.

- **check-address**: risk score 0-100, level, sanctions exposure and the risk path for any Bitcoin, Ethereum, BNB Chain, TRON or Solana address or transaction.
- **incoming-payments**: screen every incoming payment to your wallets (KYT). Flags HIGH and CRITICAL senders and suggests an action for each.
- **verify-invoice-payment**: confirm a counterparty really paid an invoice. It checks the asset, amount and network on-chain, then AML-screens the payer before you accept.
- **trace-funds**: follow stolen funds to exchanges, mixers and bridges, or walk a wallet's source of funds.

## Install

```
/plugin marketplace add saloprj/publicaml-claude-plugin
/plugin install publicaml@publicaml
```

Then ask in plain language, for example:

- "Check TXyz… for AML risk"
- "Screen all incoming payments to our wallets this week"
- "Did ACME pay invoice 1042: 4,800 USDT TRC-20 to TXyz…?"
- "Where did the funds from 0xabc… go?"

Or call a skill directly: `/publicaml:incoming-payments`.

### Your wallets

`incoming-payments` and `verify-invoice-payment` read your wallets from `~/.claude/publicaml-wallets.txt`, or from `./publicaml-wallets.txt` in your project. Put one wallet per line, as `chain address`:

```
tron TXyz...       # main USDT wallet
ethereum 0xabc...  # treasury
```

To run a daily check while Claude Code is open: `/loop 1d /publicaml:incoming-payments`.

## Only the MCP server?

The tools are a public MCP server: `https://mcp.publicaml.org/mcp` (Streamable HTTP, no auth). You can add it to claude.ai, Claude Desktop or any MCP client without this plugin:

```
claude mcp add --transport http publicaml https://mcp.publicaml.org/mcp
```

## Notes

Every tool is read-only: it reads public blockchain data and never moves funds. Wallet addresses you check are sent to PublicAML's API; see the [privacy policy](https://publicaml.org/privacy/). A score is a risk signal, not a legal verdict. Owners can dispute a result on the address page.

MIT licensed. Support: info@publicaml.org
