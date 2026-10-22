# BANKSapi for Claude

Build Open Banking integrations with [BANKSapi](https://banksapi.de) faster. This plugin gives
Claude live access to the BANKSapi developer documentation and OpenAPI specifications, and
teaches it the integration flow, so it writes calls against the real API instead of guessing
endpoints and field names.

BANKSapi is a BaFin-regulated Open Banking platform, licensed for account information and
payment initiation under PSD2/ZAG. One REST API reaches banks in Germany, Austria and
Switzerland.

## What's inside

- **BANKSapi docs connector**: a remote MCP server at `https://banksapi.io/docs-mcp`. Claude can
  search the documentation, read a docs page, and fetch a single operation from the OpenAPI
  specifications for BANKS/Connect, AI/Connect and PAY/Connect
- **Integrate skill**: picks the right BANKSapi product, then builds the flow in order, from
  tokens and users through adding a bank access to reading transactions. It covers the parts
  first integrations usually miss: the REG/Protect redirect, consent renewal and issue handling
- **Diagnose skill**: explains a failing call or a bank connection that stopped updating, from
  the HTTP status or BANKSapi message code, and says who has to act: the developer, the end
  customer at their bank, or BANKSapi support

## Use it

Ask Claude in your own words. For example:

- "Add BANKSapi to my Node.js app so users can connect their bank and I can show their
  transactions."
- "My POST to /customer/v2/bankzugaenge returns 451. What do I do with it?"
- "A bank access shows a start_sca relation. What does that mean for my users?"
- "Generate TypeScript types for the BANKSapi Get Transactions response."

In Claude Code you can also call the skills directly with `/banksapi:integrate` and
`/banksapi:diagnose`.

## Data

The plugin reads public BANKSapi documentation from two places, both operated by BANKSapi:

- `https://banksapi.io/docs-mcp`, the documentation connector. It receives the search queries,
  docs page paths and OpenAPI operation names that you or Claude ask for, and nothing else
- `https://docs.banksapi.de`, the documentation website. If the connector isn't connected yet,
  the skills tell Claude to read the same pages there directly, such as
  `https://docs.banksapi.de/llms.txt`. These are plain requests for public pages

Neither needs an account or a sign-in. The documentation is the authority: where it and a skill
differ, the skills tell Claude to follow the documentation.

The plugin doesn't access bank accounts or customer data, doesn't send or ask for API
credentials, runs no local code and stores nothing.

When Claude writes integration code for you, that code calls the BANKSapi API with your own
credentials from your own configuration. The plugin never handles those credentials.

## Before you run the code

To run code that calls the BANKSapi API, you need API credentials from BANKSapi. Request a free
trial at [banksapi.de/en/free-trial](https://banksapi.de/en/free-trial/). The Demo Provider and
the no-code [ATLAS](https://banksapi.io/atlas/) interface let you try the whole flow before you
connect a real bank.

## Support

- Documentation: [docs.banksapi.de](https://docs.banksapi.de)
- Email: support@banksapi.de

## License

[MIT](LICENSE)
