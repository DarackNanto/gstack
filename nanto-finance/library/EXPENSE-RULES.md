# Expense Categorisation Rules — Nanto Ventures Limited
## Last updated: 11 May 2026

Apply these rules when processing M-Pesa statements, receipts or manual
expense entries. Each rule maps a transaction type to the correct Zoho Books
account and flags KRA deductibility.

---

## Rule 1 — Supplier Payments (outbound)

If a payment goes to a known supplier, it is a **bill payment**, not an expense.

| Counterparty contains | Action | Zoho path |
|---|---|---|
| Shubham / Brijesh | Match to Shubham Corporation bill | Purchases > Bills > Record Payment |
| HEDclip / HED / Kristian | Match to HED bill | Purchases > Bills > Record Payment |
| Boxleo / 3PL / Box Leo | Match to Boxleo 3PL bill | Purchases > Bills > Record Payment |
| African Salihiya / cargo / clearing | Match to African Salihiya bill | Purchases > Bills > Record Payment |
| CN Meditech / Andy | Match to CN Meditech bill | Purchases > Bills > Record Payment |
| Varni | Match to Varni Corporation bill | Purchases > Bills > Record Payment |

---

## Rule 2 — Subscription and Technology

| Transaction type | Books account | Notes |
|---|---|---|
| Dialog360 / WhatsApp API | Marketing Technology (NM-MKTTECH) | Monthly $59 — set as Recurring Bill |
| Zoho / Zoho One | Marketing Technology (NM-MKTTECH) | Annual subscription |
| Volza | Consultant Expense or Subscriptions | Trade data service |
| Domain registration | Marketing Technology | |
| Adobe Firefly | Marketing Technology | Image generation subscription |
| Paystack fees | Bank Charges and Fees | Payment processing costs |

---

## Rule 3 — Government and Regulatory Fees

| Transaction type | Books account | KRA deductible? | Notes |
|---|---|---|---|
| KRA payment (tax itself) | NOT an expense — contra-tax account | N/A | Tax paid is not deductible |
| KRA penalty / interest | KRA Penalties (NM-PENALTY) | NO | Track separately — not deductible |
| PPB registration fee | PPB Registration (NM-PPB) | YES | Regulatory cost of doing business |
| PPB application fee | PPB Registration (NM-PPB) | YES | |
| Business registration / BRS | Government Fees (existing Zoho account) | YES | |
| GS1 Kenya barcoding fee | Government Fees | YES | |
| KRA iTax facilitation (consultant) | Professional Fees — Accounting (NM-ACCT) | YES | |

---

## Rule 4 — Professional Services

| Transaction type | Books account | KRA deductible? |
|---|---|---|
| Advocate / lawyer fees | Professional Fees — Legal (NM-LEGAL) | YES |
| Accountant / bookkeeper | Professional Fees — Accounting (NM-ACCT) | YES |
| Dr. Alice (PPB consultant) | Consultant Expense (existing) | YES |
| Any other consultant | Consultant Expense (existing) | YES |
| Medical advisor fees | Consultant Expense | YES |

---

## Rule 5 — Marketing and Sales

| Transaction type | Books account | KRA deductible? | Notes |
|---|---|---|---|
| Meta / Facebook ads | Advertising and Marketing | YES | |
| LinkedIn ads | Advertising and Marketing | YES | |
| Print materials (brochures, cards) | Advertising and Marketing | YES | |
| Trade show fees (KRAcon, WHX) | Trade Shows and Events (NM-EVENTS) | YES | |
| HEDclip samples / giveaways | Medical Samples and Gifts (NM-SAMPLES) | YES | Marketing cost |
| Patient booklet printing | Advertising and Marketing | YES | |
| World Kidney Day sponsorship | Trade Shows and Events | YES | |

---

## Rule 6 — Communication and Office

| Transaction type | Books account | KRA deductible? |
|---|---|---|
| Safaricom M-Pesa / mobile bundle | Telephone and Internet | YES |
| WhatsApp Business (if separate from Dialog360) | Telephone and Internet | YES |
| Internet / fibre | Telephone and Internet | YES |
| Office rent (when applicable) | Rent (create if not present) | YES |
| Courier / DHL local | Freight and Clearing — Inbound (NM-FREIGHT) if goods-related, else Postage | YES |

---

## Rule 7 — Freight and Logistics

| Transaction type | Books account | KRA deductible? | Notes |
|---|---|---|---|
| FedEx Kenya international shipping (inbound goods) | Freight and Clearing — Inbound (NM-FREIGHT) | YES | Part of COGS |
| Customs duty (KRA import declaration) | Import Duty on Goods (NM-DUTY) | YES (as COGS) | Capitalise to inventory if stock-based |
| Clearing agent fee | Freight and Clearing — Inbound (NM-FREIGHT) | YES | COGS component |
| Local delivery to customer | Freight-out (create if needed) | YES | Operating expense |

---

## Rule 8 — Banking

| Transaction type | Books account | KRA deductible? |
|---|---|---|
| DIB Bank Kenya monthly fee | Bank Charges and Fees | YES |
| Bank transfer fees | Bank Charges and Fees | YES |
| Foreign exchange loss | Forex Loss (Zoho creates automatically for multi-currency) | YES |
| M-Pesa transaction fees | Bank Charges and Fees | YES |

---

## Rule 9 — Travel

| Transaction type | Books account | KRA deductible? | Notes |
|---|---|---|---|
| Kenya domestic travel (Nairobi clinic visits) | Travel — Kenya (NM-TRAVEL-KE) | YES | Keep receipts |
| International flights (supplier visits) | Travel — International (NM-TRAVEL-INT) | YES | Business purpose must be documented |
| Accommodation | Travel — International or Kenya | YES | |
| Visa fees | Travel — International | YES | |

---

## Rule 10 — Revenue (Inflows)

If an M-Pesa inflow or bank credit is received:

| Source | Action |
|---|---|
| Known customer name (CKS, Alice, etc.) | Match to outstanding invoice; record payment |
| Investor name | Do NOT post as income — post to Mudharabah Capital (NM-MUDH) or Qard al-Hassan (NM-QARD) — SCHOLAR REVIEW REQUIRED |
| Personal transfer from Darack | Post to Owner's Equity (owner's contribution) |
| Unknown | Flag for Darack's identification before posting |

---

## Shariah Standing Rule

**No riba.** Any transaction involving interest charges (bank overdraft fees
that include interest, late payment interest from suppliers, investment returns
structured as fixed interest) must be flagged immediately. Do not post to a
standard expense account — mark NEEDS-REVIEW and notify Darack.

This applies to:
- Any line item labelled "interest" on a bank statement
- Any payment described as "financing charge" or "penalty interest"
- Any investor return computed as a fixed percentage of capital (not profit)

---

## Common M-Pesa Transaction Patterns

| M-Pesa transaction description pattern | Likely category |
|---|---|
| "Sent to [supplier name]" | Supplier payment — match to bill |
| "Pay Bill [org number]" | Government fee or utility — check PayBill number |
| "Buy Goods [till number]" | Operating expense — categorise by merchant |
| "Withdraw" | Cash withdrawal — needs receipt to categorise |
| "Received from" | Revenue or personal transfer — check source |
| "Airtime" | Telephone and Internet |
| "Charges" | Bank Charges and Fees |
| M-Pesa to M-Pesa (mobile number) | Likely consultant or staff payment — need details |

---

## Payment Account Rules

Which account to use when recording a payment in Books:

| How was it actually paid? | Zoho payment account |
|---|---|
| DIB Bank Kenya KES transfer | NANTO-KES |
| DIB Bank Kenya USD transfer | NANTO-USD |
| M-Pesa (from business M-Pesa line) | NANTO-KES (M-Pesa is KES) |
| Cash physically | Cash In Hand |

**Never use Undeposited Funds as the payment account for outgoing payments.**
Undeposited Funds is for incoming payments not yet deposited to a bank account.
