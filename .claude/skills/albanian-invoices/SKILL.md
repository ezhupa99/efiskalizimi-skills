---
name: albanian-invoices
description: Prepare monthly freelancer invoices for foreign clients in the Albanian e-invoicing portal (efiskalizimi-app.tatime.gov.al) - fetch the Bank of Albania official rate, calculate amounts, create/update the product, fill the invoice form and validate it. Use when the user asks to do / prepare their monthly invoices. NEVER registers/issues anything.
---

# Albanian freelancer invoices (state portal)

All personal data (clients, amounts, fees, date rules) is in `~/.albanian-invoices/config.json` (the user's home folder; on Windows `%USERPROFILE%\.albanian-invoices\config.json`). It lives outside the skill so skill updates never overwrite it and it never ends up in a repo.
- No config there: stop and run the `invoice-setup` skill first.
- This file contains NO personal data: never write any into it.

## Hard rules
- NEVER click "Regjistro faturen" (issue/register). Fill everything, then stop; the user reviews and clicks.
- NEVER log in for the user. They log in to e-Albania with their own credentials.
- Never assume: if anything is missing or mismatched (rate, price, dates, product), stop and ask.
- Do all arithmetic with a script, rounded to 2 decimals. Show the numbers before touching any form.
- `first_run: true` in config: confirm each step with the user before doing it; set it to false after a successful run.
- Helper, not tax advice: for VAT, tax regime or client-treatment questions, point the user to an accountant.

## Links
- Rate: https://www.bankofalbania.org/Tregjet/Kursi_zyrtar_i_kembimit/
- Portal ("e-fiskalizimi", commonly called "self-care"): https://efiskalizimi-app.tatime.gov.al/self-care/#/login?returnUrl=%2Fdashboard (user logs in first)
  - Products: Te dhenat e tatimpaguesit > Lista e produkteve
  - New invoice: Faturat > Lesho fature

## Inputs to ask each month
- Which month (default: current) and which clients (default: all).
- Hours/days for each hourly/daily client.

## Calculation (per client)
- fixed: foreign = `fixed_amount - transfer_fee`
- hourly/daily: foreign = `quantity * rate - transfer_fee`
- `transfer_fee` defaults to 0. Follow the client's config on net vs gross.
- ALL = foreign x rate (Bank of Albania rate, 2 decimals), rounded to 2 decimals.
- Show a table: client, currency, foreign amount, rate, ALL, product name, payment date, supply date. Wait for OK.

## Steps
1. **Rate**: on the rate page, set the period to today-today, choose the currency, table view, "Kerko". Use the "Leke per njesi te monedhes se huaj" row (NOT Bid/Ask).
   - **No rate for today** (too early, the bank hasn't published yet): go back one day at a time to the most recent day that HAS a rate. The bank publishes nothing on weekends or holidays, so on a Monday morning skip Sunday and Saturday and use Friday. If a day has no rate, keep going back.
   - Always tell the user which date's rate you used and why (e.g. "Today's rate isn't published yet, so I used Friday 27.11.2026: 81,79").
2. **Product** (Lista e produkteve): per `portal_defaults` / client `product` overrides. Default name `<product_prefix> - <Albanian month> - <year>`; Albanian months: Janar, Shkurt, Mars, Prill, Maj, Qershor, Korrik, Gusht, Shtator, Tetor, Nentor, Dhjetor. Price = ALL amount (dot decimals).
   - Product already exists: open it and compare the price; if it differs, ASK before Modifiko > Ruaj.
3. **Invoice** (Lesho fature):
   - Tipi i fatures: `portal_defaults.invoice_type` (the portal's default is Cash).
   - NIPT/NID i bleresit dropdown: `portal_defaults.buyer_id_type`; then the buyer ID, name, country, city and address from the client.
   - Data e pageses / Data e furnizimit: from the client's date rules.
   - Valute: `portal_currency_label`; Kursi i kembimit with COMMA decimals (e.g. 81,79).
   - Shto artikujt: search the product name, click it (qty 1, no discount, unless the config says otherwise).
   - Shto menyren e pageses: `portal_defaults.payment_method`, amount auto-filled, Konfirmo.
4. **Validate**: portal LEK total == calculated ALL; "Vlera totale ne monedhe" == foreign amount. Screenshot to `~/.albanian-invoices/screenshots/`.
5. **STOP.** Tell the user it's ready for review; they click "Regjistro faturen".
6. **Client invoice (optional)**: if the client has `client_invoice.tool == "canva"`, use the `client-invoice-canva` skill. Any other tool: remind the user to do it themselves. None: skip.

## Playwright quirks (efiskalizimi-app)
- The app is slow and animated: plain clicks on table rows can time out as "not stable". Use `browser_evaluate` with `(el) => el.click()` on the ref.
- Refs change after each interaction; re-find with `browser_find` or a fresh snapshot.
- Typing into date inputs does NOT commit. Click the input and use the calendar (next-month arrow, then the day). Weekend days look greyed but are clickable.
- Dropdowns: click the combobox, then the option by role=option. The country dropdown filters as you type.
- The exchange rate field accepts comma decimals.
