---
name: invoice-setup
description: One-time setup for the Albanian freelancer invoice automation - set up the browser connection (Playwright extension) first, then collect each client's details (from an attached previous invoice when possible) and write the user's personal config.json. Use when the user says "set up my invoices", "configure invoices", or when albanian-invoices finds no config.json. Also use to add/edit a client later.
---

# Invoice setup

Goal: make the `albanian-invoices` skill work for THIS user. Everything personal goes to
`.albanian-invoices/config.json`, placed next to where the skills are installed:
- **Skills are project-level** (in `.claude/skills` of the folder where Claude is running): write `<that folder>/.albanian-invoices/config.json`.
- **Skills are user-level** (in `~/.claude/skills`): write `~/.albanian-invoices/config.json` (Windows: `%USERPROFILE%\.albanian-invoices\config.json`).
Create the folder if needed. It stays outside the skill folders so updates never overwrite it. Never write personal data into any SKILL.md. Tell the user once that their details are saved in a config file, and where.

## How to ask
- Ask every question with the **AskUserQuestion** tool: one topic per question, with clear options. Free-text answers (names, addresses, amounts) go through its "Other" option. Use plain chat only if the tool isn't available.
- Never invent values. If the user doesn't know something, leave it out and say what it's needed for.

## Part 1 - Browser connection (ALWAYS FIRST)
Nothing else works without it. Try to open a tab with the Playwright browser tools.
- **Works**: say so and go to Part 2.
- **No browser tools, or connecting fails**: walk the user through this guide one step at a time, confirming each step with AskUserQuestion ("Done" / "I'm stuck") before moving on:
  1. **Node.js**: run `node -v`. If it's missing, ask them to install the LTS version from https://nodejs.org and reopen the terminal.
  2. **Install the Playwright Extension** in Chrome or Edge: https://chromewebstore.google.com/detail/playwright-extension/mmlmfjhmonkocbjadbfplnigmagldckm (Edge can install Chrome Web Store extensions too). Suggest pinning it to the toolbar.
  3. **Copy the token**: click the extension icon; it shows a `PLAYWRIGHT_MCP_EXTENSION_TOKEN` value. Copy it. Never ask the user to paste the token into the chat.
  4. **Save the token** as an environment variable, typed by the user in a terminal:
     - Windows (PowerShell): `setx PLAYWRIGHT_MCP_EXTENSION_TOKEN "the-token"`
     - macOS/Linux: add `export PLAYWRIGHT_MCP_EXTENSION_TOKEN="the-token"` to `~/.zshrc` or `~/.bashrc`
  5. **Connect Claude to the browser**: ask which browser they use (Chrome / Edge), then with their OK run:
     `claude mcp add playwright --scope user -- npx -y @playwright/mcp@latest --extension` (Edge: append `--browser=msedge`).
     Skip this if a `playwright` server already exists (`claude mcp list`).
  6. **Restart**: close Claude Code, open a NEW terminal (so the token loads), start `claude` in the same folder and say "set up my invoices" again. Setup continues here.
  7. **Test**: open a tab. If the extension asks to allow the connection, the user clicks Allow.
- If it still fails: the extension is disabled, the token isn't in the environment (a new terminal is needed after `setx`), or the wrong browser flag was used.

## Part 2 - Portal access
1. **Bank of Albania**: open https://www.bankofalbania.org/Tregjet/Kursi_zyrtar_i_kembimit/ and confirm a rate can be read (no login). The last working day's rate is fine if today's isn't published yet.
2. **E-fiskalizimi ("self-care")**: open https://efiskalizimi-app.tatime.gov.al/self-care/#/login?returnUrl=%2Fdashboard . Tell the user: "Please log in with your own e-Albania credentials. I never log in for you." Wait, then confirm the logged-in menu (Faturat, Te dhenat e tatimpaguesit) is visible.

## Part 3 - Clients (repeat per client)
1. **Ask for a previous invoice first**: "Do you have an invoice you already issued to this client (a PDF or screenshot from e-fiskalizimi, or the invoice you send the client)? Drag it into the chat or give me the file path, and I'll read the details from it."
   - **Yes**: read it and extract the client's legal name, tax/registration ID, country, city, address, currency, and the amount or price per hour/day. Ignore the user's own details. Show what you found and confirm with AskUserQuestion ("Correct" / "Something is wrong"). Ask only for what's missing.
   - **No**: ask for each field.
2. Fields needed:
   - Legal name, tax/registration ID, country (as written in the portal dropdown, in Albanian, e.g. "Gjermani", "Shtetet e Bashkuara te Amerikes"), city, address.
   - Billing: fixed monthly / hourly / daily. Currency. Fixed amount, or price per hour/day (`unit_price`).
   - Currency name in the portal's "Valute" dropdown ("Euro", "Dollar Amerikan"...). Fill it in yourself for EUR/USD; for other currencies, check the dropdown on a new invoice without saving.
   - Short name for product names (suggest one from the client name, e.g. "Acme").
3. **Transfer fee**: ask "Do you pay a transfer fee when this client pays you? Most companies make the freelancer pay it." Options: "Yes, I pay it" (then ask the amount, in the client's currency) / "No, the client covers it" / "We split it" (then ask the amount the user pays).
   - The amount the user pays is ALWAYS deducted from the invoice total: the portal invoice equals what actually arrives in the bank account.
4. **Dates**: ask only for the payment date rule (e.g. "end of current month", "20th of next month"), offered as options. **Never ask about the supply date**: it is always the end of the month. Write `"supply_date_rule": "end of current month"` without asking or mentioning it.
5. **Client invoice** (optional): "Do you also send this client your own invoice (Canva, Word, an invoicing app)?" Options: None / Canva / Other.
   - Canva: ask them to open the design while logged in to Canva, and confirm you can see it. Ask for the design name, current invoice number, and invoice date rule. The `client-invoice-canva` skill confirms the fields on its first run.
   - Other: save the tool name; that part stays manual.
6. Anything special about this client: free text, saved as `notes`.
7. Ask "Add another client?".

## Part 4 - Portal defaults (tell, don't ask)
Don't ask about these. Write them to `portal_defaults` with the values from `config.example.json`, and tell the user once:
"I use a proven setup for the portal, so you don't have to decide anything here:
- Invoice type: Non-cash
- Buyer ID type: foreign subject (NIPT subjekt i huaj)
- Payment method: Llogari e transaksionit
- Product, one per client each month: `<Client> - <Albanian month> - <year>` (e.g. `Acme - Nentor - 2026`), type Sherbim (service), unit Cope, description `Sherbime IT`, price = the month's total in Lek, quantity 1
All of these are configurable. If you ever want something different, just tell me (e.g. 'change how products are created')."
If they ask for a change (now or later), update `portal_defaults` or add a per-client override.

## Part 5 - Write and confirm
- Write `.albanian-invoices/config.json` (project folder or home, as described above) in the shape of `config.example.json` (in the `albanian-invoices` skill folder). Set `"first_run": true`.
- **Don't search for an existing config.** Never look through the user's home or profile folders for a previous config or state. Just write the file at the location above. Only read the config when the user explicitly asks to add or edit a client, and then open it directly at that location.
- Show a readable summary per client (no raw JSON unless asked) and a sample calculation, e.g. "3000 USD - 10 USD fee = 2990 USD on the invoice".
- Confirm with AskUserQuestion. Then say: "When you want to invoice, open e-fiskalizimi, log in, and tell me 'do my monthly invoices'. The first run is guided: I'll confirm each step with you."
