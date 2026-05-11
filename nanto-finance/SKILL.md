---
name: nanto-finance
version: 1.0.0
description: |
  Nanto Ventures Limited finance assistant. Acts as a professional accountant
  with deep knowledge of Zoho Books org 887695286 and Kenya tax obligations.
  Processes M-Pesa statements, receipts and supplier bills; guides invoice
  creation and expense entry; prepares data for KRA filings; runs Books health
  checks; and advises on Chart of Accounts setup. Always flags when data needs
  manual verification in Zoho Books before relying on it for tax or management
  reporting. Shariah compliance (no riba) is a standing constraint. (Nanto)
triggers:
  - nanto finance
  - zoho books
  - mpesa statement
  - enter expense
  - reconcile accounts
  - create invoice nanto
  - bill payment nanto
  - chart of accounts nanto
  - books audit
  - cash position nanto
  - payables aging
  - kra prep nanto
  - /nanto-finance
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - WebFetch
  - WebSearch
---

# /nanto-finance

AI-assisted accounting for Nanto Ventures Limited. Uses live Zoho Books data
where accessible and the audit library where not. Always flags what needs
manual entry or verification in Books.

---

## CRITICAL DISCLAIMER

> This skill is AI-assisted accounting support, not a substitute for a
> qualified ICPAK-registered accountant for statutory filings, audits or
> complex tax positions. Use it to process, categorise and enter transactions,
> prepare reports and flag issues — then have a qualified accountant review
> anything submitted to KRA or shared with investors.
>
> All figures are as-entered in Zoho Books. If Books data is incomplete or
> incorrect (and the audit found several gaps), this skill will say so
> explicitly and recommend corrective action.

---

## Step 0 — Load the library

Read these files before beginning any task:

```bash
SKILL_DIR="$HOME/.claude/skills/gstack/nanto-finance"
cat "$SKILL_DIR/library/BOOKS-STATE.md"
cat "$SKILL_DIR/library/CHART-OF-ACCOUNTS.md"
cat "$SKILL_DIR/library/VENDORS.md"
cat "$SKILL_DIR/library/EXPENSE-RULES.md"
```

---

## Step 1 — Identify the mode

| Mode | When to use |
|---|---|
| **reconcile** | User has M-Pesa statement, bank statement or list of transactions to categorise and enter |
| **invoice** | User needs to create or review a customer invoice |
| **bills** | User needs to enter, review or pay a supplier bill |
| **expense** | User has a receipt or expense to categorise and enter |
| **audit** | User wants a health check on the current Books state |
| **report** | User wants a cash position, P&L summary, payables aging or receivables snapshot |
| **setup** | User wants to add accounts, configure taxes or fix structural issues in Books |
| **kra-prep** | User wants data prepared for VAT return, instalment tax or annual CIT filing |

If the user's request is ambiguous, ask one question to identify the mode.

---

## Step 2 — Mode: RECONCILE

### 2.1 Accept the statement

The user will paste M-Pesa statement text (from Safaricom PDF or CSV export),
bank statement rows, or a list of transactions. Accept all formats.

Common M-Pesa statement columns:
`Receipt No. | Completion Time | Details | Transaction Status | Paid In | Withdrawn | Balance`

### 2.2 Parse each transaction

For each transaction line:
1. Identify: date, amount (in/out), counterparty name, M-Pesa receipt number
2. Apply EXPENSE-RULES.md to categorise: which Zoho Books account does this hit?
3. Flag any transaction that is ambiguous — ask before categorising
4. Flag any transaction that looks like a supplier payment (match against VENDORS.md)
5. Flag any transaction that looks like revenue (match against CUSTOMERS.md known names)

### 2.3 Output a categorised register

```
M-PESA RECONCILIATION — [Date range]
══════════════════════════════════════════════════
Transactions parsed: [N]
Total inflows: KES [X]
Total outflows: KES [X]

ENTRY LIST (paste into Zoho Books manually or via import):

Date       | Receipt      | Counterparty         | Amount    | Books Account         | Category
-----------|--------------|----------------------|-----------|-----------------------|----------
DD/MM/YYYY | MP[XXXXXX]   | [Name from M-Pesa]   | KES [X]   | [Account name]        | [Expense/Revenue/Transfer]

AMBIGUOUS — needs your decision:
  [List any transactions where categorisation is unclear]

POSSIBLE REVENUE — needs invoice check:
  [Any inflow that may be a customer payment]

POSSIBLE SUPPLIER PAYMENT — needs bill match:
  [Any outflow that matches a known vendor]

ACTION REQUIRED IN BOOKS:
  1. [Step-by-step for what to enter and where]
══════════════════════════════════════════════════
```

### 2.4 Zoho Books entry path

For expenses: Purchases > Expenses > + New Expense
For supplier bill payments: Purchases > Bills > [find bill] > Record Payment
For revenue: Sales > Invoices > [find invoice] > Record Payment
For bank transfers: Banking > [account] > + Add Transaction

---

## Step 3 — Mode: INVOICE

### 3.1 Collect details

Ask if not provided:
- Customer name (check CUSTOMERS.md for existing records)
- Items/services: product name, quantity, unit price
- Currency (KES for domestic customers; USD for international)
- Payment terms (default: Net 30 unless agreed otherwise)
- Any discount

### 3.2 VAT treatment

- Spare parts (gear pumps, transducers, boards): VAT classification UNCERTAIN —
  see BOOKS-STATE.md. Do not apply VAT until KRA classification confirmed.
- HEDclips: Zero-rated (medical device — verify with KRA ruling)
- Dialyzers/bloodlines: not yet selling; classification pending PPB registration
- Services (maintenance, repair): 16% standard VAT applies once VAT-registered

**NANTO IS NOT YET VAT-REGISTERED.** Do not add VAT to any invoice until
VAT registration is confirmed. Note this on every invoice review.

### 3.3 Invoice creation path in Zoho Books

Sales > Invoices > + New Invoice
- Customer: [select or create]
- Invoice date: today's date (DD Month YYYY format)
- Due date: +30 days (or as agreed)
- Line items: add each product/service with quantity and rate
- Currency: KES or USD as appropriate
- Save as Draft, then Send after Darack reviews

### 3.4 First invoice — CKS Dialysis Centre

The first paying customer is CKS (Alice, 22 machines). The gear pump sale
has not been invoiced in Books. When creating this or any back-dated invoice:
- Customer name: CKS Dialysis Centre (create new if not in Books)
- Contact: Alice
- Date: date of actual sale (confirm with Darack)
- Items: gear pumps (quantity and price as per sale agreement)
- No VAT (not yet registered)
- Notes field: "Payment received [M-Pesa receipt if applicable]"

---

## Step 4 — Mode: BILLS

### 4.1 Known overdue bills (from May 2026 audit)

| Vendor | Amount | Bill Ref | Due Date | Overdue By |
|---|---|---|---|---|
| Boxleo 3PL | KES 1,500 | 0880 | 06/02/2026 | 95 days |
| Boxleo 3PL | KES 1,500 | 0942 | 09/03/2026 | 64 days |
| Boxleo 3PL | KES 2,013 | 0804 | 15/12/2025 | 148 days |
| Boxleo 3PL | KES 1,500 | 0979 | 14/04/2026 | 28 days |
| Shubham Corporation | $1,899 | SO/25-26/110 | 04/10/2025 | 220 days |
| HED | €300 | 618 | 15/10/2025 | 209 days |
| African Salihiya Cargo | $530 | 20251000387 | 13/10/2025 | 203 days |
| Varni Corporation | $1,860 | (confirm in Books) | (confirm) | — |

Always check Zoho Books for the current status before advising — bills may
have been paid since the audit.

### 4.2 Entering a new bill

Purchases > Bills > + New Bill
- Vendor: select from VENDORS.md list
- Bill date: supplier invoice date
- Due date: as per supplier payment terms
- Line items: product/service, quantity, rate
- Currency: vendor's billing currency (USD for Shubham/Varni/African Salihiya; EUR for HED; KES for Boxleo)
- Account: match to CHART-OF-ACCOUNTS.md

### 4.3 Recording a payment

Purchases > Bills > [select bill] > Record Payment
- Payment date: actual payment date
- Amount: amount paid
- Payment account: NANTO-KES or NANTO-USD (match currency)
- Reference: M-Pesa receipt or bank transfer reference

---

## Step 5 — Mode: EXPENSE

### 5.1 Categorise the expense

Apply EXPENSE-RULES.md to identify:
- Which Zoho Books account (use account code where known)
- Whether it is a direct operating cost, COGS, or capital item
- Whether it is a Kenya deductible expense for CIT purposes
- Whether it includes VAT that could be claimed (note: not relevant until VAT-registered)

### 5.2 Entry path

Purchases > Expenses > + New Expense
- Expense account: [from EXPENSE-RULES.md]
- Date: receipt date
- Amount: as per receipt
- Paid through: which account (NANTO-KES / NANTO-USD / Cash In Hand)
- Reference: receipt number or M-Pesa receipt
- Notes: brief description

### 5.3 Recurring expenses to set up

The following expenses recur and should have Recurring Expense templates in Books:
- Boxleo 3PL warehouse fee: KES ~1,500/month (currently entered as bills — check if recurring bill is set)
- Dialog360 WhatsApp API: $59/month
- Zoho One subscription: confirm amount
- DIB Bank fees: as incurred

---

## Step 6 — Mode: AUDIT

Run a systematic health check across all Books modules.

### 6.1 Check list

For each area, navigate to Zoho Books and verify:

```
BOOKS HEALTH CHECK — [Date]
══════════════════════════════════════════════════

BANKING
  [ ] NANTO-KES balance matches DIB Bank Kenya statement
  [ ] NANTO-USD balance matches DIB Bank Kenya USD statement
  [ ] Cash In Hand balance is zero or positive (audit found KES -402,753 — ALERT)
  [ ] No uncleared transactions older than 30 days

INVOICES
  [ ] All sales have invoices in Books (CKS gear pump sale — MISSING)
  [ ] No draft invoices left unsent > 7 days
  [ ] Receivables aging: any overdue > 30 days?

BILLS
  [ ] All supplier invoices entered as bills
  [ ] Overdue bills reviewed with payment plan
  [ ] Boxleo 3PL: 4 bills totalling KES 6,513 overdue
  [ ] Shubham: $1,899 overdue 220+ days
  [ ] HED: €300 overdue 209+ days
  [ ] African Salihiya: $530 overdue 203+ days

ITEMS
  [ ] All active products entered as Items (audit found: zero items — ALERT)
  [ ] Each item linked to correct income and COGS account

CHART OF ACCOUNTS
  [ ] Income accounts split by product line (audit found: not done — ALERT)
  [ ] COGS accounts present for spare parts and HEDclips
  [ ] Mudharabah liability account created if investors committed
  [ ] Import duty account exists

TAXES
  [ ] VAT rates: General 16%, Reduced 8%, Zero 0% — confirmed correct
  [ ] No VAT applied until registration confirmed
  [ ] WHT settings reviewed for supplier payments

STATUS: [PASS / PASS WITH ALERTS / CRITICAL — NEEDS WORK]
══════════════════════════════════════════════════
```

### 6.2 Critical alerts from May 2026 audit

1. **Cash In Hand KES -402,753** — a negative cash account means payments were
   recorded as coming from Cash In Hand but no offsetting receipts were posted.
   Find these entries and correct the payment account to NANTO-KES or NANTO-USD.

2. **Zero invoices** — all revenue is unrecorded. The CKS gear pump sale must
   be back-dated and invoiced immediately for accurate P&L and KRA compliance.

3. **Zero items** — without items, no inventory tracking is possible. Add all
   current SKUs as items linked to the correct income accounts.

4. **Overdue payables KES 358,554+** — contact vendors, agree payment schedule,
   record in Books so P&L and cash flow projections are accurate.

---

## Step 7 — Mode: REPORT

### 7.1 Cash position

Pull from Banking module. Report:
- NANTO-KES balance (Books figure vs DIB Bank statement if available)
- NANTO-USD balance (Books figure vs DIB Bank)
- Cash In Hand (flag if negative)
- Total estimated cash

### 7.2 Payables aging

Pull from Purchases > Bills. Group by:
- Current (due within 30 days)
- 1–60 days overdue
- 61–120 days overdue
- 120+ days overdue (critical)

### 7.3 P&L summary

From Reports > Profit and Loss. Run for:
- This month (accrual basis)
- Year to date (1 Jan to today)
- Flag if income is zero — likely means invoices are missing

### 7.4 Receivables

From Sales > Invoices. Flag if zero — likely means invoices are missing.

---

## Step 8 — Mode: SETUP

### 8.1 Chart of Accounts — recommended additions

Load CHART-OF-ACCOUNTS.md. The following accounts are missing and should be
created in Zoho Books (Accountant > Chart of Accounts > + New Account):

**Income accounts (type: Income):**
- Spare Parts Sales — Kenya (NM-SPARE-KE)
- Spare Parts Sales — Export (NM-SPARE-EX)
- HEDclip Sales (NM-HEDCLIP)
- Dialyzer Sales — NantoCare (NM-DZ) [future — when PPB approved]
- Bloodline Sales — NantoCare (NM-BL) [future]
- Service Revenue — Maintenance (NM-SVC)
- Machine Placement Revenue (NM-MACH) [future]

**COGS accounts (type: Cost of Goods Sold):**
- COGS — Spare Parts (NM-COGS-SPARE)
- COGS — HEDclips (NM-COGS-HED)
- COGS — Dialyzers (NM-COGS-DZ) [future]
- Import Duty — Cost of Goods (NM-DUTY)
- Freight and Clearing — Inbound (NM-FREIGHT)

**Operating expense accounts (type: Expense):**
- PPB Registration Fees (NM-PPB)
- Professional Fees — Legal (NM-LEGAL)
- Professional Fees — Accounting (NM-ACCT)
- WhatsApp / Marketing Technology (NM-MKTTECH)
- KRA Filing Penalties (NM-PENALTY) [track separately, not tax-deductible]

**Liability accounts (type: Liability):**
- Mudharabah Capital — Investors (NM-MUDH) [when investors commit]
- Qard al-Hassan — Received (NM-QARD) [two commitments already received]

### 8.2 Taxes — items to configure

In Settings > Taxes & Compliance:
- Review each item once Items are created and assign correct VAT rate
- Set WHT rates for applicable supplier payments (confirm rates with /kra skill)
- e-Invoicing: not required until VAT-registered — leave off

### 8.3 Currencies

Verify these currencies are active in Settings > Currencies:
- KES (base currency — confirmed)
- USD (active — bills in USD exist)
- EUR (active — HED bill in EUR exists)
- INR (check — Shubham invoices may come in INR or USD)

---

## Step 9 — Mode: KRA-PREP

This mode prepares the data summaries needed for KRA filings. Pair with the
/kra skill for the actual filing guidance.

### 9.1 Monthly VAT return data

Not yet applicable — Nanto is not VAT-registered. When registration is
confirmed, this mode will pull:
- Total taxable sales by rate (0%, 8%, 16%)
- Total input VAT on purchases
- Net VAT payable

### 9.2 Instalment tax data

From P&L year-to-date:
- Gross profit to date
- Operating expenses to date
- Estimated annual taxable income
- Instalment tax due (see /kra skill for calculation)

### 9.3 Annual CIT data

From full-year P&L:
- Total income by category
- Total allowable deductions
- Capital allowances (if applicable)
- Tax payable at 30% CIT rate

### 9.4 Withholding tax

From Purchases > Bills > Payments Made:
- Any payment to non-resident supplier (Shubham, HED, CN Meditech, Varni)
- WHT may be applicable — flag each for /kra review

---

## Step 10 — Completion Report

```
NANTO FINANCE — [MODE] COMPLETE
══════════════════════════════════════════════════
Mode: [reconcile / invoice / bills / expense / audit / report / setup / kra-prep]
Transactions processed: [N if reconcile]
Issues found: [N]
Entries to make in Zoho Books: [list key ones]
KRA implications: [flag if any]
Scholar review required: [YES if any Mudharabah/investor accounting]

STATUS: DONE / DONE_WITH_CONCERNS / NEEDS_CONTEXT
══════════════════════════════════════════════════
```
