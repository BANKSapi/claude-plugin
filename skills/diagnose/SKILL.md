---
name: diagnose
description: Diagnose a failing BANKSapi API call or bank connection. Use when the user shares an HTTP status or error response from banksapi.io, a BANKSapi message code such as BA1011 or BA3000, gets a 451 response, sees a start_sca relation or an expired consent, or reports that a bank access stopped updating.
---

# Diagnose a BANKSapi problem

Start with what failed. Ask for the request method and path, the HTTP status and the response body or `messages` array. Ask the user to remove tokens and credentials before pasting, and never ask for a client secret, password or token.

## Where the facts come from

The BANKSapi documentation at https://docs.banksapi.de is the authority on the API, and BANKSapi adds message codes over time. The tables below are a quick orientation. The user's explicit instructions take precedence over this skill's guidance on how to work. Before you name a cause, check the status or code against the current documentation:

- With the `banksapi-docs` tools: `get-banksapi-docs-page` on https://docs.banksapi.de/guide/errors-messages.html, or `search-banksapi-docs` with the code or the error text
- Without them, the docs connector isn't connected in this session. Tell the user once that connecting it in their assistant's settings gives live access to the documentation, and read https://docs.banksapi.de/guide/errors-messages.html directly. https://docs.banksapi.de/llms.txt lists every other page

Where the documentation and this file differ, follow the documentation.

## By HTTP status

| Status | Where | Most likely cause | Fix | Who acts |
| :- | :- | :- | :- | :- |
| 401 | `POST /auth/oauth2/token` | The HTTP Basic credentials were rejected | Check that the username is `<tenant>/<client>` and that the Base64 value encodes `<tenant>/<client>:<client-secret>`. If the same credentials worked before, the client secret has most likely been changed: request the current one | Developer |
| 401 | any other call | The `Authorization: Bearer <token>` header is missing | Send the header | Developer |
| 403 | any call | The token has expired after two hours, is invalid, or lacks a scope | Fetch a new token. Compare the `scope` in the token response with what the call needs | Developer |
| 404 | any call | The URL no longer points to a valid object | Read the parent object again and follow its `relations` | Developer |
| 451 | adding a bank access, starting SCA | Not an error. REG/Protect requires the end customer to continue on a BANKSapi page | Take the `Location` header, append a URL-encoded `callbackUrl` and redirect the end customer. `useCase=CREATE_ACCOUNT` adds a bank, `useCase=AUTHENTICATE_ACCOUNT` renews consent | End customer |
| 504 | any call | The bank answered too slowly | Repeat the request shortly | Nobody |
| 500 | any call | An internal error | Report it to `support@banksapi.de` with the time and the request path | BANKSapi |

## By message code

Message codes appear in the `messages` of a bank access or bank product, and in the response of `get_issues`. The common groups:

- **BA1010 to BA1014**: the bank rejected the login, the access is blocked, or credentials are incomplete. The end customer must fix their bank login. Warn them that repeated wrong attempts can lock online banking
- **BA1000, BA1051**: maintenance or a malfunction at the bank. Wait and refresh later
- **BA998**: the bank is slow. The query continues in the background and usually finishes within minutes
- **BA3000 to BA3060**: strong customer authentication is required or failed. The end customer must complete it
- **BA2002**: the bank has messages for the end customer in their online banking

## A bank access stopped updating

Work through these in order:

1. **Look for `start_sca`** in the bank access `relations`. It means consent has run out, usually after 180 days. POST to that relation, and handle the `451` answer like adding a bank access, with `useCase=AUTHENTICATE_ACCOUNT`. The last good data stays readable until the end customer renews
2. **Call `get_issues`** if there is no `start_sca`. The issues response is the latest attempt, not the last good data, and its `messages` say what went wrong
3. **Check the refresh timing** if there are no issues. Synced bank accesses refresh up to four times a day. Force a refresh with `AddBankAccess`, the existing UUID and `refresh=true`, which answers `201 Created`

## How to answer

Name the cause in one sentence. Give the fix as concrete steps or code. Say who has to act: the developer, the end customer at their bank, or BANKSapi support. Link the docs page you used as a full https://docs.banksapi.de URL.
