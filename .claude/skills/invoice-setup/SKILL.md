---
name: invoice-setup
description: One-time setup for the Albanian freelancer invoice automation - check the prerequisites (browser connection, e-Albania/portal access, optional Canva), then interview the user about their clients, billing and products, and write their personal config.json. Use when the user says "set up my invoices", "configure invoices", or when albanian-invoices finds no config.json. Also use to add/edit a client later.
---

# Invoice setup

Goal: make the `albanian-invoices` skill work for THIS user. Everything personal goes to
`~/.albanian-invoices/config.json` in the user's home folder (Windows: `%USERPROFILE%\.albanian-invoices\config.json`). Create the folder if needed. It stays outside the skill folder so updates never overwrite it and it is never shared. Never write personal data into any SKILL.md.

Go step by step, one topic at a time. Never invent values: if the user doesn't know something, leave it out and say what it's needed for.

## Part 1 - Prerequisites (check them, don't just ask)
1. **Browser connection (Playwright MCP + extension).** Try to open a tab with the Playwright browser tools.
   - Works: continue.
   - No browser tools at all: the Playwright MCP server isn't configured (common when the skills were installed with `npx skills add`).
     1. Check Node.js: `node -v`. Missing: ask the user to install the LTS version from https://nodejs.org.
     2. Ask the user to install the "Playwright MCP Bridge" extension in Chrome or Edge (https://github.com/microsoft/playwright-mcp) and to set its token as the `PLAYWRIGHT_MCP_EXTENSION_TOKEN` environment variable themselves. Never ask them to paste the token into chat.
     3. With their OK, run `claude mcp add playwright --scope user -- npx -y @playwright/mcp@latest --extension` (Edge users: add `--browser=msedge`). The server inherits the token from the environment.
     4. Ask them to restart Claude Code and run setup again.
   - Tools exist but connecting fails: the extension is missing or disabled, or the token env variable isn't set (a new terminal is needed after `setx`).
2. **Bank of Albania**: open https://www.bankofalbania.org/Tregjet/Kursi_zyrtar_i_kembimit/ and confirm a rate can be read (no login needed). If today's rate isn't published yet, the most recent working day's rate is fine for this check.
3. **E-invoicing portal ("e-fiskalizimi" / "self-care")**: open https://efiskalizimi-app.tatime.gov.al/self-care/#/login?returnUrl=%2Fdashboard . Tell the user: "Please log in with your own e-Albania credentials. I never log in for you." Wait, then confirm the logged-in menu (Faturat, Te dhenat e tatimpaguesit) is visible.
4. **Client invoice tool (optional)**: ask: "Do you also send your clients an invoice you make yourself (Canva, Word, Google Docs, an invoicing app...)? This is optional; the main automation is the state portal invoice."
   - Canva: ask them to log in and open their invoice design; confirm you can see it. The `client-invoice-canva` skill takes care of the rest.
   - Anything else: note it in config (`client_invoice.tool`), and tell them that part stays manual for now.

Summarize what passed and what is missing before Part 2.

## Part 2 - Interview
### About the user (once)
- Payment method used in the portal (default "Llogari e transaksionit", i.e. bank transfer).
- Invoice type (default "Non-cash").

### Each client (repeat; ask "Another client?" at the end)
- Legal name, tax/registration ID, country (as written in the portal dropdown, in Albanian, e.g. "Gjermani", "Shtetet e Bashkuara te Amerikes"), city, address.
- Billing: fixed monthly / hourly / daily? Currency? Amount or rate?
- Fees: is a bank/transfer fee deducted from what you receive? Do you invoice the net amount (after fee) or the gross amount in the portal? Default: no fee.
- Payment date rule (e.g. "end of current month", "20th of next month", "15 days after invoice").
- Supply date rule (usually "end of current month", i.e. the billing period).
- Short name to use in product names (e.g. "Acme").
- Anything special about this client (free text, saved as `notes`).
- Client invoice: none / Canva / other. If Canva: the design name, the current invoice number, the invoice date rule, gross or net amount on the client invoice, and which fields change every month.

### Products (show the default, then ask)
Say: "This is how I create the monthly product in the portal by default:
- Name: `<Client short name> - <Albanian month> - <year>`, e.g. `Acme - Nentor - 2026`
- Type: Sherbim (service), unit: Cope, description: `Sherbime IT`
- Price: the total for the month in Lek, quantity 1

Does this fit your work, or do you want something different (another description, a reusable product per client, hours as quantity...)?"
Save their answer in `portal_defaults` (or per client as `product` overrides).

## Part 3 - Write and confirm
- Write `~/.albanian-invoices/config.json` using `config.example.json` (in the `albanian-invoices` skill folder) as the shape. Keep the keys; add `notes` where useful.
- Show a readable summary per client (no raw JSON unless they ask) and an example calculation for the current month with a sample rate.
- Ask them to confirm. Then say: "Next month, open the portal, log in, and tell me 'do my monthly invoices'. The first run is guided: I'll confirm each step with you."
- Set `"first_run": true` in config.
