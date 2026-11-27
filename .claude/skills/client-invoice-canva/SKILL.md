---
name: client-invoice-canva
description: Optional add-on to albanian-invoices - update the user's own Canva invoice design for a client (invoice number, date, period, quantity, totals) and download it as PDF. Use after the portal invoice is prepared, for clients with client_invoice.tool == "canva" in config, or when the user asks to update their Canva invoice.
---

# Client invoice in Canva (optional)

Reads the client's `client_invoice` block from `.albanian-invoices/config.json` (written by `invoice-setup`, in the project folder for project-level skills or in the home folder for user-level skills).
The user is logged in to Canva with their own account; never log in for them.

## Rules
- Ask every question with the **AskUserQuestion** tool (free text via its "Other" option).
- Edit the design named `client_invoice.design_name` in place only if `edit_in_place` is true; otherwise make a copy (ask first).
- First time for a design: look at it, list the fields you think change monthly, and confirm them with the user. Save them to `client_invoice.fields`.
- Also ask on the first run: should the client invoice show the gross amount (usual: the client is billed the full amount, and the fee is deducted only on the state invoice) or the net amount? Save it as `client_invoice.amount`.
- After a successful download, increment `client_invoice.last_invoice_number` in config.
- The user sends the PDF themselves.

## Steps
1. Open the design (from Canva home/search, or `client_invoice.design_url` if set).
2. Update each field in `client_invoice.fields` (typical: invoice no. +1, invoice date by `invoice_date_rule`, "Services for <English month>", quantity, row total, subtotal, total).
3. Verify all values with `browser_find` (regex on the page text).
4. Share > Download (not "Download selection") > PDF > Download.

## Canva quirks (Playwright)
- Double-click a text box to enter edit mode, then Home / Shift+End (or Ctrl+A inside a table cell) to select; type with `page.keyboard.type` (refs vanish in edit mode).
- In tables, Tab moves to the next cell and selects its text.
