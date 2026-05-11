# Nanto Ventures Limited — Tax Profile
## Standing tax baseline for all /kra sessions
### Last reviewed: 11 May 2026

> This file captures Nanto's current tax situation. Verify current status
> before advising — this is not real-time. Update when facts change.

---

## ENTITY DETAILS

| Field | Detail |
|---|---|
| Legal name | Nanto Ventures Limited |
| KRA PIN | P052406460F |
| Registration number | PVT-ZQUXALPV |
| Tax year | 1 January – 31 December (calendar year) |
| Accounting currency | Kenya Shillings (KES); USD for international transactions |
| Bank | DIB Bank Kenya |
| Zoho Books | Primary accounting system — all invoices, expenses, bank feeds |
| Registered office | The Piano, 8th Floor, 171 Brookside Drive, Nairobi, Kenya |
| Director | Darack Nanto (resident: Pearland, Texas, USA) |
| Incorporated | 2025 |

---

## CURRENT TAX STATUS

| Tax type | Status | Notes |
|---|---|---|
| Income tax (CIT) | ACTIVE — PIN registered | No tax due until profitable; must file nil returns if no income |
| VAT | NOT YET REGISTERED | Threshold not yet reached (KES 5M). Decision to register voluntarily is pending — see analysis below |
| PAYE | NOT ACTIVE | No employees. Activate at first hire |
| WHT | PENDING | Will apply on first director's fee drawn, consultant fees paid, or investor distributions |
| Import duty | ACTIVE — first shipment cleared | Via clearing agent at JKIA; Box Leo 3PL handles receiving |
| Instalment tax | PENDING | Due when first profitable year is projected |

---

## VAT REGISTRATION DECISION — PENDING

**Question:** Should Nanto register for VAT voluntarily before reaching KES 5M turnover?

**Arguments FOR early voluntary registration:**
1. Nanto imports goods and pays costs that embed VAT-equivalent charges. If products
   are zero-rated for output VAT, input VAT on Kenya-side costs would be recoverable.
2. DIB Bank and some large customers (hospitals, NGOs) prefer to transact with VAT-registered
   entities — it signals formal business status.
3. Avoids scramble to register when threshold is approaching.

**Arguments AGAINST:**
1. If Nanto's products are EXEMPT (not zero-rated), registration provides no VAT recovery
   benefit — it only creates a monthly compliance burden.
2. If products are standard-rated (16%), registration forces Nanto to charge 16% VAT
   to customers, increasing the price unless competitors are also registered.

**Critical dependency:** The VAT classification of dialysis spare parts (see
VAT-MEDICAL-DEVICES.md) determines whether registration is advantageous. Resolve
the VAT classification question first, then decide on registration.

**Action:** Obtain a formal KRA ruling on VAT classification of Nanto's product
categories before registering.

---

## PRODUCT TAX SUMMARY

| Product | CET Import Duty | VAT on Import | VAT on Sale | Status |
|---|---|---|---|---|
| Dialysis machine spare parts (gear pumps, transducers, sensors, valves, boards) | Likely 0% (HS 9018) | To confirm (zero-rated or exempt?) | Same as import VAT | VERIFY |
| HEDclips (Denmark) | Likely 0% (HS 9018.90.90 or surgical clips) | To confirm | Same | VERIFY |
| Dialyzers (NantoCare brand — CN Meditech OEM) | Likely 0% (HS 9018) | To confirm | Same | VERIFY — PPB Class B/C pending |
| Bloodlines (NantoCare brand — CN Meditech OEM) | Likely 0% (HS 9018) | To confirm | Same | VERIFY — PPB Class B/C pending |

All product VAT classifications must be formally confirmed. See VAT-MEDICAL-DEVICES.md.

---

## IMPORT DUTY — CURRENT SUPPLIERS

| Supplier | Country | Currency | Incoterms (confirm) | Import route |
|---|---|---|---|---|
| CN Meditech | China | USD | TBC with Andy | Air freight via JKIA |
| Shubham Corporation | India | USD | TBC with Brijesh | Air/sea to JKIA |
| HEDclip | Denmark | USD/EUR | TBC with Kristian | Air freight via JKIA |

**Landed cost structure per shipment:**
```
Supplier invoice (CIF or FOB + Kenya freight)
+ CET Import Duty (likely 0% for medical devices — CONFIRM per HS code)
+ IDF: 3.5% of CIF value
+ RDL: 2% of CIF value
+ VAT on import (16% of [CIF + Duty + IDF + RDL] — OR 0%/exempt if classified as medical)
+ Clearing agent fees (variable — Box Leo / freight agent)
+ Last-mile to Box Leo 3PL storage
= Total landed cost
```

---

## MUDHARABAH INVESTOR TAX — OPEN QUESTION

**Raise:** USD 325,000 | Minimum: USD 25,000 | Nisba: 50/50 profit share
**Status:** Pre-close — no investor has yet signed.

**Tax question (unresolved):**
When Nanto distributes the investor's 50% profit share, what is the Kenya tax treatment?

| Scenario | Tax treatment | Action |
|---|---|---|
| Treated as dividend | 5% WHT deducted at source | Not ideal — it is not a dividend in company law terms |
| Treated as interest | 15% WHT | Incorrect — Mudharabah is not a loan |
| Treated as business income (investor's own books) | Investor declares in their own CIT/personal tax | May be the correct treatment — no WHT by Nanto |
| Exempt | No tax at all | Unlikely without specific legislative provision |

**Required action before first distribution:**
1. KRA private ruling (costs approx. KES 5,000–20,000; takes 45–60 days)
2. ICPAK-registered tax practitioner written opinion
3. Coordinate with scholar review (Brad Bradford, Houston) — Shariah compliance
   already required; combine the sessions to address Shariah + Kenya tax together

---

## TAX PLANNING — CURRENT YEAR PRIORITIES

These are the actions available now to legally minimise Nanto's tax exposure:

| Priority | Action | Timing |
|---|---|---|
| 1 | Compile and document ALL pre-commencement expenses (legal, regulatory, PPB, website, marketing before first sale) — deductible in year one | Before first annual return (30 June 2027 for 2026 year) |
| 2 | Resolve VAT classification — this affects pricing, margin, and registration decision | Urgent — before next significant import |
| 3 | Get KRA ruling on Mudharabah WHT treatment — before first investor distribution | Before investor close |
| 4 | Capital allowances — track every equipment purchase (laptops, office setup) to claim in Zoho Books | Ongoing |
| 5 | Ensure Zoho Books records are complete and reconciled monthly — reduces audit risk and makes year-end faster | Ongoing |
| 6 | Register for VAT once classification is resolved and if zero-rated — to recover input VAT | Post-classification ruling |

---

## KEY TAX CONTACTS

| Contact | Role |
|---|---|
| ICPAK (icpak.com) | Find a registered Kenya CPA for formal filings and audit response |
| KRA iTax (itax.kra.go.ke) | All filings and payments |
| KRA helpline (0800 272 272) | Basic queries — use for non-sensitive questions only |
| Brad Bradford (Houston, TX) | Shariah scholar review — required before investor signs |

---

## VERSION HISTORY

| Version | Date | Notes |
|---|---|---|
| 1.0 | 2026-05-11 | Initial profile — built from CLAUDE.md company context |
