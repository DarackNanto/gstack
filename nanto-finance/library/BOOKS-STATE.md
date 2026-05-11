# Zoho Books — Nanto State Audit
## Last audited: 11 May 2026

**Organisation:** Nanto Ventures Limited
**Zoho Books Org ID:** 887695286
**URL pattern:** `https://books.zoho.com/app/887695286#/[section]`
**Login account:** nantohq25@gmail.com
**Report basis:** Accrual
**Base currency:** KES

---

## Banking Accounts

| Account | Books Balance | Actual Bank Balance | Status |
|---|---|---|---|
| NANTO-KES (xxxx0301) | KES 3,689 | KES 0 (unconfirmed) | Likely stale — needs reconciliation |
| NANTO-USD | $114.13 | $0 (unconfirmed) | Likely stale — needs reconciliation |
| Cash In Hand | KES -402,753 | N/A | CRITICAL — negative cash account |

**Cash In Hand is negative.** This means payments were posted as coming from
Cash In Hand without matching deposits into it. Root cause: expenses or bill
payments recorded with "Cash In Hand" as the payment account when they should
have used NANTO-KES. Must be corrected before any P&L or cash position is
reliable.

**Banking integration status:** Accounts appear to be manually managed — not
connected to live bank feed. DIB Bank Kenya is the banking partner.

---

## Dashboard Snapshot (11 May 2026)

| Metric | Value |
|---|---|
| Receivables (total) | KES 0 |
| Overdue payables | KES 358,554 |
| Starting cash (this period) | KES -223,983 |
| Outgoing | KES 160,317 |
| Projected year-end cash | KES -384,300 |

---

## Invoices

**Status: Zero invoices in Books.**

No sales have been invoiced through Zoho Books. This means:
- Revenue is completely unrecorded
- P&L shows KES 0 income for all periods
- KRA has no documentation of taxable revenue
- The CKS Dialysis Centre gear pump sale (first paying customer) is absent

**CKS gear pump sale must be back-invoiced** with the actual sale date.

---

## Bills (overdue — as at 11 May 2026)

| Vendor | Currency | Amount | Bill Ref | Due Date | Overdue By |
|---|---|---|---|---|---|
| Boxleo 3PL | KES | 1,500 | 0880 | 06/02/2026 | 95 days |
| Boxleo 3PL | KES | 1,500 | 0942 | 09/03/2026 | 64 days |
| Boxleo 3PL | KES | 2,013 | 0804 | 15/12/2025 | 148 days |
| Boxleo 3PL | KES | 1,500 | 0979 | 14/04/2026 | 28 days |
| Shubham Corporation | USD | 1,899 | SO/25-26/110 | 04/10/2025 | 220 days |
| HED | EUR | 300 | 618 | 15/10/2025 | 209 days |
| African Salihiya Cargo | USD | 530 | 20251000387 | 13/10/2025 | 203 days |
| Varni Corporation | USD | 1,860 | (confirm ref) | (confirm) | unknown |
| Volza | USD | 1,180 | 20260425 | 25/04/2026 | PAID |

**Total KES-denominated overdue: KES 6,513 (Boxleo)**
**Total USD-denominated overdue: $4,289+ (Shubham + African Salihiya + Varni)**
**Total EUR-denominated overdue: €300 (HED)**

---

## Expenses

10 expense entries visible as at audit, covering May 2025 to April 2026.

Categories found:
- Consultant Expense
- Advertising
- Telephone
- Government Fees
- Bank Fees

Payment methods used: Undeposited Funds, NANTO-USD, NANTO-KES.
Note: use of "Undeposited Funds" as payment source is incorrect — this account
is for payments received, not payments made. Review these entries.

---

## Items

**Status: Zero items/products set up.**

No SKUs have been created in Books. This means:
- Invoices cannot reference specific products
- Inventory tracking is not possible
- COGS cannot be tracked automatically

Items to create immediately:
- Gear Pump (compatible, generic — by machine model if possible)
- Pressure Transducer
- Flow Sensor
- Valve Component
- Electronic Control Board
- LCD Replacement
- HEDclip (by size variant)
- [Others as sold]

---

## Customers (as at audit)

| Name | Company | Email | Receivables |
|---|---|---|---|
| Nanto HQ | Nanto Venture Ltd | nantohq25@gmail.com | KES 0 |
| PKC | (none entered) | (none) | KES 0 |
| Thomas Kebungo | (none) | nundathomas@yahoo.com | KES 0 |
| zahra shihabuddin | Zihiri clicik ltd | zahrashihabuddin@gmail.com | KES 0 |

**Missing:** CKS Dialysis Centre (Alice) — the first paying customer — is not
in the system at all.

---

## Taxes Configuration

| VAT Rate | % | Status |
|---|---|---|
| General Rate | 16% | Configured |
| Reduced Rate | 8% | Configured |
| Zero Rate | 0% | Configured |

No products have been assigned to tax rates (because no items exist yet).
VAT registration status: NOT YET REGISTERED. Do not apply VAT to invoices
until confirmed.

Withholding Tax module: available but not configured.
e-Invoicing: available but not active.

---

## Settings — Active Modules

Enabled: Quotes, Purchase Orders, Time Tracking, Recurring Invoice, Recurring
Expense, Recurring Bills, Credit Note.

Disabled: Sales Orders, Sales Receipts, Retainer Invoices, Recurring Journals,
Payment Links, Tasks, Fixed Asset, Zoho Inventory Add-on.

**Recommendation:** Enable Zoho Inventory Add-on once items are set up — this
activates proper stock tracking for spare parts.

---

## Priority Fix List

1. **CRITICAL — Fix Cash In Hand negative balance.** Find all expenses posted
   to Cash In Hand and re-post to the correct bank account.
2. **URGENT — Create CKS invoice.** Back-date to actual sale date. Record
   payment received.
3. **URGENT — Create all Items/products** so future invoices use real SKUs.
4. **HIGH — Create new Chart of Accounts** income and COGS accounts split by
   product line. See CHART-OF-ACCOUNTS.md.
5. **MEDIUM — Agree payment plans for overdue bills** with Shubham (220 days),
   HED (209 days), African Salihiya (203 days) and Boxleo (multiple bills).
6. **LOW — Connect bank feeds** to DIB Bank Kenya if API integration is
   available, or set up regular manual import from bank statements.
