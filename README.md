# BANKSapi

Build Open Banking integrations with [BANKSapi](https://banksapi.de) faster. This plugin gives
your AI assistant live access to the BANKSapi developer documentation and OpenAPI
specifications, and teaches it the integration flow, so it writes calls against the real API
instead of guessing endpoints and field names.

BANKSapi is a BaFin-regulated Open Banking platform, licensed for account information and
payment initiation under PSD2/ZAG. One REST API reaches banks in Germany, Austria and
Switzerland.

## What's inside

- **BANKSapi docs connector**: a remote MCP server at `https://banksapi.io/docs-mcp`. The
  assistant can search the documentation, read a docs page, and fetch a single operation from
  the OpenAPI specifications for BANKS/Connect, AI/Connect and PAY/Connect
- **Integrate skill**: picks the right BANKSapi product, then builds the flow in order, from
  tokens and users through adding a bank access to reading transactions. It covers the parts
  first integrations usually miss: the REG/Protect redirect, consent renewal and issue handling
- **Diagnose skill**: explains a failing call or a bank connection that stopped updating, from
  the HTTP status or BANKSapi message code, and says who has to act: the developer, the end
  customer at their bank, or BANKSapi support

The plugin works in Claude, in ChatGPT and Codex, and in Gemini CLI. It contains only skills
and the remote connector, so it behaves the same everywhere it is installed.

In Gemini CLI, install it with:

```bash
gemini extensions install https://github.com/BANKSapi/claude-plugin
```

## Use it

Ask in your own words. For example:

- "Add BANKSapi to my Node.js app so users can connect their bank and I can show their
  transactions."
- "My POST to /customer/v2/bankzugaenge returns 451. What do I do with it?"
- "A bank access shows a start_sca relation. What does that mean for my users?"
- "Generate TypeScript types for the BANKSapi Get Transactions response."

Where your assistant supports calling skills by name, they are `integrate` and `diagnose`.

## Data

The plugin reads public BANKSapi documentation. It talks to two places, both operated by
BANKSapi, and one service provider:

- `https://banksapi.io/docs-mcp`, the documentation connector. It receives the search queries,
  docs page paths and OpenAPI operation names that you or the assistant ask for, and nothing
  else. Page reads and OpenAPI excerpts are served by BANKSapi itself
- **Inkeep**, the provider behind the BANKSapi documentation search. The connector passes
  search queries to Inkeep to answer them. Only the search tool does this; the other three
  tools answer from BANKSapi's own copy of the documentation
- `https://docs.banksapi.de`, the documentation website. If the connector isn't connected, the
  skills tell the assistant to read the same pages there directly, such as
  `https://docs.banksapi.de/llms.txt`. These are plain requests for public pages

None of them needs an account or a sign-in. Don't put personal or confidential information
into documentation searches. The documentation is the authority: where it and a skill differ,
the skills tell the assistant to follow the documentation.

The plugin doesn't access bank accounts or customer data, doesn't send or ask for API
credentials, runs no local code and stores nothing.

When the assistant writes integration code for you, that code calls the BANKSapi API with your
own credentials from your own configuration. The plugin never handles those credentials.

## Before you run the code

To run code that calls the BANKSapi API, you need API credentials from BANKSapi. You can
request access through the developer hub at [docs.banksapi.de](https://docs.banksapi.de). The
Demo Provider and the no-code [ATLAS](https://banksapi.io/atlas/) interface let you try the
whole flow before you connect a real bank.

## Support

- Documentation: [docs.banksapi.de](https://docs.banksapi.de)
- Support: [banksapi.de/en/support-en](https://banksapi.de/en/support-en/)
- Email: support@banksapi.de
- Privacy policy (German): [banksapi.de/datenschutz](https://banksapi.de/datenschutz/)

## License

[MIT](LICENSE)
