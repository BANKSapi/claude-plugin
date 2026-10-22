---
name: integrate
description: Integrate the BANKSapi Open Banking API into an application. Use when the user wants to connect bank accounts in Germany, Austria or Switzerland, read balances or transactions, let end customers add their bank through REG/Protect, categorise transactions with AI/Connect, or asks how to get started with BANKSapi or BANKS/Connect.
---

# Integrate BANKSapi

BANKSapi is a BaFin-regulated Open Banking platform, licensed for account information and payment initiation under PSD2/ZAG. One REST API at `https://banksapi.io` reaches banks in Germany, Austria and Switzerland.

## Where the facts come from

The BANKSapi documentation at https://docs.banksapi.de is the authority. This skill summarises the flow and the usual pitfalls. Where the two differ, follow the documentation, and take every concrete detail from it rather than from memory or from this file.

Read the documentation through the `banksapi-docs` tools:

- Before writing a request, fetch its operation with `get-banksapi-openapi-spec`, passing `spec_name` (`banks-connect`, `ai-connect` or `pay-connect`) and `operation_id`. Use the field names, types and enums it returns
- For a concept or a guide, use `get-banksapi-docs-page` with a page URL such as https://docs.banksapi.de/guide/quickstart.html, or `search-banksapi-docs` with a question
- `get-banksapi-overview` returns the index of every docs page

If those tools aren't available, the BANKSapi docs connector isn't connected yet. Tell the user once that they can connect it from the plugin's **Connectors** tab, then read the same documentation directly:

- Index of every page: https://docs.banksapi.de/llms.txt
- Every guide in one file: https://docs.banksapi.de/llms-full.txt
- OpenAPI specifications: https://docs.banksapi.de/openapi/banksapi-v2-openapi-3_0.yaml for BANKS/Connect, https://docs.banksapi.de/openapi/banksapi_ai_connect.json for AI/Connect and https://docs.banksapi.de/openapi/pay-connect-openapi-3_0.yaml for PAY/Connect

## 1. Pick the product

| The user wants to | Product |
| :- | :- |
| Let their own customers connect bank accounts and read data | BANKS/Connect |
| Connect only their own company's accounts | ONE/Connect |
| Categorise transactions, detect business partners, analyse cashflow | AI/Connect, on top of BANKS/Connect |
| Take Pay by Bank in an online checkout | PAY/Connect |
| Embed ready-made banking UI | WEB/Connect |

If the user holds no PSD2 licence of their own, which is the usual case, they use REG/Protect: end customers enter bank credentials on a BANKSapi-hosted page, never on the user's domain. Confirm this before writing the bank access step, because it changes that step completely. https://docs.banksapi.de/guide/customer.html covers the regulated flow.

## 2. Build the flow in this order

Each step names its `operation_id`.

1. **Client token** (`CreateToken`): `POST /auth/oauth2/token` with `grant_type=client_credentials`. HTTP Basic auth, where the username is `<tenant>/<client>`. This is the only call that sends `application/x-www-form-urlencoded`. Tokens last two hours
2. **User** (`CreateUser`): `POST /auth/mgmt/v1/tenants/{tenant-name}/users` with the client token. Create one technical user per end customer. The answer is `201 Created` with no body
3. **User token** (`CreateToken`): the same endpoint with `grant_type=password` and that user's username and password. Every Customer API call uses this token
4. **Bank access** (`AddBankAccess`): `POST /customer/v2/bankzugaenge` with a new UUID that the application generates as the key and an empty object `{}` as its value. Send the end customer's IP address in a `Customer-IP-Address` header. With REG/Protect the answer is `451` with a `Location` header: append a URL-encoded `callbackUrl` and redirect the end customer there. They come back with `baReentry=ACCOUNT_CREATED`
5. **Read data** (`GetBankAccesses`, then `GetTransactions`): read the bank accesses, then follow each bank product's `relations`, such as `get_kontoumsaetze` for transactions. Add `?tag=categories` for AI/Connect categories
6. **Keep it working**: handle the `start_sca` relation, `get_issues` and `refresh=true` as described below

## 3. Defaults that prevent support tickets

- **Follow relations.** BANKSapi uses HATEOAS. Take URLs from each object's `relations` array instead of building them by hand
- **One new UUID per bank access.** Never reuse the user's ID as the bank access key: the second bank access for that user then has nothing to be keyed by
- **Clear old REG/Protect sessions** with `DeleteAllRegprotectSessions` before starting a new flow, because an abandoned browser tab can leave a session half-finished
- **Plan for consent renewal.** Bank consent usually lasts 180 days, and some banks ask more often. A `start_sca` relation on a bank access means the end customer must confirm again: POST to that relation, and the answer is the same `451` redirect with `useCase=AUTHENTICATE_ACCOUNT`. Old data stays readable meanwhile, so ask at a convenient moment, not on app start
- **Read issues.** A bank access keeps its last good data when a refresh fails. Call `get_issues` to see why, and look up message codes on https://docs.banksapi.de/guide/errors-messages.html
- **Refresh on request only.** Synced bank accesses refresh up to four times a day. To force one, repeat `AddBankAccess` with the existing UUID and `refresh=true`
- **Test with the Demo Provider** before a real bank. https://docs.banksapi.de/guide/demo-provider.html has its credentials, and ATLAS at https://banksapi.io/atlas/ runs the whole flow without code

## 4. Credentials

Never write API credentials into code, examples or the conversation, and don't ask the user to paste them. In generated code, load them from the application's own configuration or secret store, and keep `.env` files out of version control. In examples, use placeholders such as `<tenant>`, `<client>` and `<client-secret>`.

Credentials for the API come from BANKSapi. A free trial is available at `https://banksapi.de/en/free-trial/`.

## 5. What good output looks like

- Code that follows the steps above in order, with the `451` redirect and the `start_sca` renewal handled rather than left as TODOs
- Request and response types taken from the fetched OpenAPI operation
- A short note on which steps the end customer sees: the REG/Protect page and, later, consent renewal
- A link to each docs page you relied on, as a full https://docs.banksapi.de URL, so the user can check it against the current documentation
