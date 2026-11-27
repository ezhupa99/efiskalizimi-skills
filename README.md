# AI invoices for Albanian freelancers

Prepare your monthly invoice for foreign clients in the Albanian e-invoicing portal (**e-fiskalizimi / self-care**) yourself, for free, with [Claude Code](https://claude.com/claude-code). Accountants often charge around 50 EUR a month for this.

Each month, Claude:
1. gets the official Bank of Albania rate (if today's isn't published yet, it uses the last working day's),
2. calculates the amount (fixed, hourly or daily, with an optional transfer-fee deduction),
3. creates or updates the product in the portal,
4. fills in the invoice and checks the totals,
5. **stops before "Regjistro faturen"**, so you always review and issue the invoice yourself.

Optional: it also updates your own Canva invoice for the client and downloads the PDF.

> ⚠️ This is a helper, not tax advice. You are responsible for what you issue. If you're unsure about VAT or your tax regime, ask an accountant once, then let the tool handle the monthly routine.

## What you need
- [Claude Code](https://claude.com/claude-code)
- [Node.js](https://nodejs.org) (LTS). Claude needs it for the browser connection.
- Chrome or Edge with the **[Playwright Extension](https://chromewebstore.google.com/detail/playwright-extension/mmlmfjhmonkocbjadbfplnigmagldckm)** (open it after installing to copy your token)
- An e-Albania account with access to [e-fiskalizimi (self-care)](https://efiskalizimi-app.tatime.gov.al/self-care/#/login?returnUrl=%2Fdashboard). You always log in yourself.
- Optional: a Canva account with your invoice design

## Install
**Option A: skills CLI** (the skills install into Claude Code globally)
```bash
npx skills add ezhupa99/efiskalizimi-skills
```

**Option B: clone** this repo and open the folder in Claude Code (`claude`). Approve the `playwright` server when asked.

Either way, save the Playwright extension token as an environment variable (keep it private):
- Windows: `setx PLAYWRIGHT_MCP_EXTENSION_TOKEN "your-token"`, then open a new terminal
- macOS/Linux: `export PLAYWRIGHT_MCP_EXTENSION_TOKEN="your-token"` in `~/.zshrc` / `~/.bashrc`

## Setup (one time)
Tell Claude **"set up my invoices"**. It will:
1. **connect to your browser first**, guiding you step by step through Node.js, the Playwright Extension and its token (video guide: coming soon),
2. open Bank of Albania and e-fiskalizimi (you log in yourself),
3. ask for each client's details. Attach an invoice you already issued and Claude reads them from it; otherwise you type them in,
4. ask whether you pay a transfer fee (it's deducted, so the invoice matches what reaches your bank),
5. save everything to `.albanian-invoices/config.json` on your computer: in the folder where Claude is running if you installed the skills in that project, or in your home folder (`~/.albanian-invoices/`) if you installed them at user level. That file is never shared.

Portal choices follow an opinionated best practice, so you don't have to decide anything: invoice type Non-cash, payment method Llogari e transaksionit, one product per client per month (`Client - Month - Year`, service, `Sherbime IT`). Everything is configurable. To change something later, tell Claude (e.g. "change how products are created").

## Every month
1. Log in to e-fiskalizimi in your browser.
2. Tell Claude: **"do my monthly invoices, Example worked 168 hours"**
3. Check the calculation, then let Claude fill in the portal.
4. Claude checks that "Vlera totale ne monedhe" equals what you'll receive in your bank (e.g. 3000 USD − 10 USD fee = 2990 USD). It doesn't hand over the invoice until everything matches.
5. Review the form and click **Regjistro faturen** yourself.

## Skills
| Skill | Purpose |
|---|---|
| `invoice-setup` | One-time check and interview, writes your config |
| `albanian-invoices` | Monthly portal workflow (the main part) |
| `client-invoice-canva` | Optional: Canva invoice for the client |

The portal changes now and then. If a step breaks, fixes to the "Playwright quirks" section in `albanian-invoices/SKILL.md` are welcome. Use another tool for client invoices? Add a skill next to `client-invoice-canva`.
