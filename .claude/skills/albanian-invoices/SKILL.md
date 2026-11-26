---
name: albanian-invoices
description: Prepare monthly freelancer invoices for foreign clients in the Albanian e-invoicing portal (e-fiskalizimi / self-care, efiskalizimi-app.tatime.gov.al) - fetch the Bank of Albania official rate, calculate amounts, create/update the product, fill the invoice form and validate it. Use when the user asks to do / prepare their monthly invoices. NEVER registers/issues anything.
---

# Albanian freelancer invoices (e-fiskalizimi)

All personal data (clients, amounts, fees, date rules) is in `~/.albanian-invoices/config.json` (Windows: `%USERPROFILE%\.albanian-invoices\config.json`). It lives outside the skill, so skill updates never overwrite it and it never ends up in a repo.
- No config there: stop and run the `invoice-setup` skill first.
- This file contains NO personal data: never write any into it.

## Hard rules
- NEVER click "Regjistro faturen" (issue/register). Fill everything, validate, then stop; the user reviews and clicks.
- NEVER log in for the user. They log in to e-Albania with their own credentials.
- Ask every question with the **AskUserQuestion** tool (free-text answers via its "Other" option). Use plain chat only if the tool isn't available.
- Never assume: if anything is missing or doesn't match (rate, price, dates, product, totals), stop and ask.
- Do all arithmetic with a script, rounded to 2 decimals. Show the numbers before touching any form.
- `first_run: true` in config: confirm each step before doing it; set it to false after a successful run.
- Helper, not tax advice: for VAT, tax regime or client-treatment questions, point the user to an accountant.

## Links
- Rate: https://www.bankofalbania.org/Tregjet/Kursi_zyrtar_i_kembimit/
- E-fiskalizimi ("self-care"): https://efiskalizimi-app.tatime.gov.al/self-care/#/login?returnUrl=%2Fdashboard (user logs in first)
  - Products: Te dhenat e tatimpaguesit > Lista e produkteve
  - New invoice: Faturat > Lesho fature

## Inputs to ask each month
- Which month (default: current) and which clients (default: all).
- Hours/days for each hourly/daily client.

## Calculation (per client)
- gross: fixed = `fixed_amount`; hourly/daily = `hours_or_days * unit_price`
- **expected** (foreign currency, what reaches the bank account) = `gross - transfer_fee` (`transfer_fee` = what the user pays, 0 if none). The fee is ALWAYS deducted.
- ALL = expected x rate (Bank of Albania rate, 2 decimals), rounded to 2 decimals.
- Show a table: client, gross, fee, expected, currency, rate, ALL, product name, payment date, supply date. Confirm with AskUserQuestion before filling anything.

## Steps
1. **Rate**: on the rate page, set the period to today-today, choose the currency, table view, "Kerko". Use the "Leke per njesi te monedhes se huaj" row (NOT Bid/Ask).
   - **No rate for today** (too early, the bank hasn't published yet): go back one day at a time to the most recent day that HAS a rate. The bank publishes nothing on weekends or holidays, so on a Monday morning skip Sunday and Saturday and use Friday. If a day has no rate, keep going back.
   - Always tell the user which date's rate you used and why (e.g. "Today's rate isn't published yet, so I used Friday 27.11.2026: 81,79").
2. **Product** (Lista e produkteve): name `<product_prefix> - <Albanian month> - <year>`; Albanian months: Janar, Shkurt, Mars, Prill, Maj, Qershor, Korrik, Gusht, Shtator, Tetor, Nentor, Dhjetor. Type Sherbim, unit Cope, description `Sherbime IT`, price = ALL amount (dot decimals). Use `portal_defaults.product` / client `product` overrides if present.
   - Product already exists: open it and compare the price; if it differs, ASK before Modifiko > Ruaj.
3. **Invoice** (Lesho fature):
   - Tipi i fatures: ALWAYS **Non-cash** (the portal's default is Cash; change it).
   - NIPT/NID i bleresit dropdown: "Numri Personal i Identifikimit/NIPT subjekt i huaj"; then the buyer ID, name, country, city and address from the client.
   - Data e pageses / Data e furnizimit: from the client's date rules.
   - Valute: `portal_currency_label`; Kursi i kembimit with COMMA decimals (e.g. 81,79).
   - Shto artikujt: search the product name, click it (qty 1, no discount).
   - Shto menyren e pageses: Metoda e pageses is ALWAYS **Llogari e transaksionit**. The amount fills in automatically. Konfirmo.
4. **Validate (mandatory before hand-off)**:
   - "Vlera totale ne monedhe" MUST equal **expected** (what the user receives in the bank account). Example: earns 3000 USD, fee 10 USD → must show 2990,00.
   - Portal LEK total MUST equal the calculated ALL amount.
   - Either one doesn't match: something went wrong along the way. Do NOT hand off. Tell the user what differs, find the cause (product price, rate, quantity, currency), fix it, and validate again.
   - Screenshot the validated form to `~/.albanian-invoices/screenshots/`.
5. **STOP.** Tell the user the invoice is filled and validated; they review it and click "Regjistro faturen".
6. **Client invoice (optional)**: if the client has `client_invoice.tool == "canva"`, use the `client-invoice-canva` skill. Any other tool: remind the user to do it themselves. None: skip.

## Playwright quirks (efiskalizimi-app)
- The app is slow and animated: plain clicks on table rows can time out as "not stable". Use `browser_evaluate` with `(el) => el.click()` on the ref.
- Refs change after each interaction; re-find with `browser_find` or a fresh snapshot.
- Typing into date inputs does NOT commit. Click the input and use the calendar (next-month arrow, then the day). Weekend days look greyed but are clickable.
- Dropdowns: click the combobox, then the option by role=option. The country dropdown filters as you type.
- The exchange rate field accepts comma decimals.
