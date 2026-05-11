# Chart of Accounts — Nanto Ventures Limited
## Zoho Books Org ID: 887695286
## Last updated: 11 May 2026

---

## Status

The current Zoho Books CoA is mostly Zoho default accounts with minimal
customisation. The income and COGS sections have no Nanto-specific accounts.
The accounts below must be created before proper management reporting is possible.

Navigate to: Accountant > Chart of Accounts > + New Account

---

## Section 1 — Income Accounts (TYPE: Income)

### Current state
Only default "Sales" account exists. No product-line split.

### Accounts to create

| Account Name | Code | Description |
|---|---|---|
| Spare Parts Sales — Kenya | NM-SPARE-KE | Revenue from dialysis spare parts sold to Kenya customers |
| Spare Parts Sales — Export | NM-SPARE-EX | Revenue from spare parts sold outside Kenya (if any) |
| HEDclip Sales | NM-HEDCLIP | Revenue from HEDclip products (exclusive Africa distribution) |
| Service Revenue — Maintenance | NM-SVC | Revenue from machine maintenance and repair services |
| Machine Placement Revenue | NM-MACH | Revenue from machine placement arrangements (future) |
| NantoCare Dialyzer Sales | NM-DZ | Revenue from NantoCare branded dialyzers — CREATE when PPB-approved |
| NantoCare Bloodline Sales | NM-BL | Revenue from NantoCare branded bloodlines — CREATE when PPB-approved |
| Academy Revenue | NM-ACAD | Revenue from Nanto Academy courses (if handled through same entity) |

---

## Section 2 — Cost of Goods Sold (TYPE: Cost of Goods Sold)

### Current state
No COGS accounts exist.

### Accounts to create

| Account Name | Code | Description |
|---|---|---|
| COGS — Spare Parts | NM-COGS-SPARE | Cost of spare parts sold (purchase price from Shubham, Varni, etc.) |
| COGS — HEDclips | NM-COGS-HED | Cost of HEDclips from HED Denmark |
| COGS — Dialyzers | NM-COGS-DZ | Cost of NantoCare dialyzers from CN Meditech — CREATE when selling |
| COGS — Bloodlines | NM-COGS-BL | Cost of NantoCare bloodlines — CREATE when selling |
| Import Duty on Goods | NM-DUTY | Kenya customs duty on imported medical devices |
| Freight and Clearing — Inbound | NM-FREIGHT | Freight, clearing agent, and logistics costs on inbound shipments |
| 3PL Warehousing | NM-3PL | Boxleo 3PL monthly warehouse fees attributable to goods stored |

---

## Section 3 — Operating Expenses (TYPE: Expense)

### Current accounts (Zoho defaults — keep)
- Advertising and Marketing (use this for WhatsApp ads, Meta spend)
- Bank Charges and Fees
- Consultant / Contractor Fees
- Telephone and Internet

### Accounts to add

| Account Name | Code | Description |
|---|---|---|
| PPB Registration and Licensing | NM-PPB | Pharmacy and Poisons Board fees and compliance costs |
| KRA Penalties and Interest | NM-PENALTY | Tax penalties — NOT deductible for CIT; track separately |
| Professional Fees — Legal | NM-LEGAL | Advocate fees, contract reviews, legal opinions |
| Professional Fees — Accounting | NM-ACCT | Accountant, bookkeeper, iTax filing fees |
| Marketing Technology | NM-MKTTECH | Dialog360 WhatsApp API ($59/month), Zoho One subscription |
| Trade Shows and Events | NM-EVENTS | KRAcon, WHX Nairobi, World Kidney Day sponsorship |
| Medical Samples and Gifts | NM-SAMPLES | HEDclip giveaways, clinical samples — marketing cost |
| Travel — Kenya | NM-TRAVEL-KE | Local travel to visit clinics, PPB, government offices |
| Travel — International | NM-TRAVEL-INT | Flights, accommodation for international supplier visits |

---

## Section 4 — Liability Accounts (TYPE: Liability)

### Accounts to add

| Account Name | Code | Description |
|---|---|---|
| Mudharabah Capital — Investors | NM-MUDH | Investor capital under Mudharabah structure (profit-sharing, not equity) |
| Qard al-Hassan — Received | NM-QARD | Benevolent loans received (two commitments of $5,000 each confirmed) |
| VAT Payable | NM-VATP | Output VAT owed to KRA — activate when VAT-registered |
| WHT Payable | NM-WHTP | Withholding tax deducted from supplier payments, owed to KRA |

**IMPORTANT on Mudharabah accounting:**
Mudharabah capital is NOT equity — it is a liability (a trust held for the
investor, to be returned with profit share at end of term). Do not post
investor capital to equity accounts. Use NM-MUDH as a current liability until
the Mudharabah contract defines the treatment.

**Scholar review required** before posting any investor capital entries.

---

## Section 5 — Asset Accounts (TYPE: Asset)

### Current accounts
- NANTO-KES (bank account)
- NANTO-USD (bank account)
- Cash In Hand (currently negative — must be fixed)

### Accounts to add

| Account Name | Code | Description |
|---|---|---|
| Inventory — Spare Parts | NM-INV-SPARE | Physical stock of spare parts at Boxleo 3PL warehouse |
| Inventory — HEDclips | NM-INV-HED | Physical HEDclip stock |
| Inventory — Dialyzers | NM-INV-DZ | Future — when PPB approved and stock held |
| Prepaid Expenses | NM-PREPAID | Subscriptions or advances paid in advance |
| Security Deposit — Boxleo | NM-DEP-BOX | Any deposit paid to Boxleo 3PL |

---

## Section 6 — Equity Accounts (TYPE: Equity)

### Current
- Owner's Equity (Zoho default)

### Note
Mudharabah investor capital goes to LIABILITIES (NM-MUDH), NOT equity.
Only Darack's own capital contributions go to equity accounts.

---

## Implementation Sequence

Do this in order to avoid mapping issues:

1. Create all Income accounts (Section 1)
2. Create all COGS accounts (Section 2)
3. Create Items/products and link each to the correct Income + COGS account
4. Create Liability accounts (Section 4) — especially Qard al-Hassan if $10,000 received
5. Create additional Expense accounts (Section 3)
6. Create Inventory asset accounts (Section 5) once Inventory Add-on is enabled
7. Correct Cash In Hand negative balance
8. Back-date and enter the CKS invoice using the new accounts
