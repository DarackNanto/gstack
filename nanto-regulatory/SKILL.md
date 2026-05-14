---
name: nanto-regulatory
preamble-tier: 3
version: 1.0.0
description: |
  Nanto regulatory affairs across all Kenya boards and international standards.
  Covers PPB (product registration, establishment license, PRIMS, dossier prep),
  KMPDC (clinical practitioner licensing), MOH (facility licensing for NKC clinic),
  SHA (provider accreditation), NEMA (medical waste, EIA), KEBS/PVOC (import
  standards), KIPI (trademarks). International standards: ISO 13485, ISO 10993,
  IEC 60601, CE/MDR, WHO GMP, NMPA (China), FDA. Generates supplier document
  request letters, dossier checklists, and unified status trackers. Primary use:
  PPB Class B/C registration of NantoCare dialyzers and bloodlines (CN Meditech),
  Class A spare parts (Shubham), and Nanto Kidney Care facility licensing.
  Boundary: contract review is /kenya-law; tax classification is /kra. (Nanto/gstack)
triggers:
  - regulatory
  - PPB
  - PRIMS
  - registration
  - dossier
  - establishment license
  - KMPDC
  - MOH facility
  - SHA accreditation
  - NEMA
  - KEBS
  - PVOC
  - KIPI trademark
  - product registration
  - Class A
  - Class B
  - Class C
  - dialyzer registration
  - bloodline registration
  - CE mark
  - ISO 13485
  - GMP certificate
  - NMPA
  - supplier documents
  - document request
  - regulatory tracker
  - nanto regulatory
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - WebFetch
  - WebSearch
  - AskUserQuestion
---

# /nanto-regulatory

Nanto's regulatory affairs layer. Navigates every board, prepares every dossier,
requests every document from every supplier. Boundary: WHAT to file, WITH WHOM,
and WHAT documents you need to get there.

> **Not for contract review** — that is /kenya-law.
> **Not for HS code classification or import duty** — that is /kra.
> **Not for bookkeeping of regulatory fees** — that is /nanto-finance.
> **Not for regulatory compliance legal opinion** — that is /kenya-law.

---

## Preamble (always run first)

```bash
_UPD=$(~/.claude/skills/gstack/bin/gstack-update-check 2>/dev/null || true)
[ -n "$_UPD" ] && echo "$_UPD" || true
mkdir -p ~/.gstack/sessions
touch ~/.gstack/sessions/"$PPID"
find ~/.gstack/sessions -mmin +120 -type f -exec rm {} + 2>/dev/null || true
eval "$(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)" 2>/dev/null || true
_LEARN_FILE="${GSTACK_HOME:-$HOME/.gstack}/projects/${SLUG:-unknown}/learnings.jsonl"
if [ -f "$_LEARN_FILE" ]; then
  _LEARN_COUNT=$(wc -l < "$_LEARN_FILE" 2>/dev/null | tr -d ' ')
  echo "LEARNINGS: $_LEARN_COUNT entries"
  if [ "$_LEARN_COUNT" -gt 5 ] 2>/dev/null; then
    ~/.claude/skills/gstack/bin/gstack-learnings-search --limit 3 2>/dev/null || true
  fi
fi
_TEL=$(~/.claude/skills/gstack/bin/gstack-config get telemetry 2>/dev/null || true)
if [ "$_TEL" != "off" ]; then
  echo '{"skill":"nanto-regulatory","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'"}'  \
    >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
fi
echo "REGULATORY SKILL v1.0.0 READY"
```

---

## Constants

```
─── KENYA REGULATORY BODIES ──────────────────────────────────────────────────

PPB (Pharmacy and Poisons Board)
  Portal:        prims.pharmacyboardkenya.org
  Email:         medicaldevices@ppb.go.ke
  Trade affairs: tradeaffairs@pharmacyboardkenya.org
  Ground agent:  Dr. Alice (Nanto external PPB consultant — contact via Darack)
  Classes:
    Class A  — Medical devices (spare parts, equipment components, instruments)
               Registration required. Simpler dossier. No superintendent needed.
    Class B  — Pharmaceutical products (non-prescription consumables, bloodlines,
               certain diagnostics). Superintendent Pharmacist required.
    Class C  — Prescription medicines and high-risk devices (dialyzers are Class B/C
               depending on PPB classification decision). Superintendent required.
  Timeline:  Establishment License: 3-6 months
             Product registration (Class A): 3-9 months
             Product registration (Class B/C): 6-18 months

KMPDC (Kenya Medical Practitioners and Dentists Council)
  Portal:    kmpdc.go.ke
  Relevance: All clinical staff at Nanto Kidney Care must be KMPDC-registered.
             Medical Director (Dr. Twahir), attending nephrologists, resident doctors.
  Key rule:  Dialysis center cannot legally operate without a KMPDC-registered
             medical director on record with MOH.

MOH (Ministry of Health — Health Facility Licensing)
  Portal:    kmhfr.health.go.ke  (Kenya Master Health Facility Registry)
  Relevance: Nanto Kidney Care Site 1 requires a health facility operating license.
  Process:   Application to County Health Department (Nairobi County) +
             MOH national inspection + KMHFR registration.
  Timeline:  3-6 months typical, dependent on inspection scheduling.

SHA (Social Health Authority)
  Portal:    sha.go.ke
  Relevance: NKC must be SHA-accredited to bill SHA for patient sessions.
             SHA tariff: KES 16,000/session (Legal Notice 146, September 2024).
  Process:   Facility accreditation application, inspection, approval.
  Timeline:  2-4 months post-MOH license.

NEMA (National Environment Management Authority)
  Portal:    nema.go.ke
  Relevance: Medical waste management license required for NKC clinic.
             EIA (Environmental Impact Assessment) may be required for fit-out.
  Medical waste:  Must use NEMA-licensed incinerator service for clinical waste.
  Timeline:  EIA: 2-4 months. Waste management license: 1-3 months.

KEBS (Kenya Bureau of Standards) / PVOC
  Portal:    kebs.org
  Relevance: Pre-Export Verification of Conformity (PVOC) applies to
             regulated medical devices imported into Kenya. Verifies conformity
             to Kenya or international standards before shipment.
  PVOC agent: Appointed PVOC agents in country of origin (China: Intertek, SGS, BV).
  Note:       Not all medical devices require PVOC — verify per HS code.

KIPI (Kenya Industrial Property Institute)
  Portal:    kipi.go.ke
  Relevance: Trademark registration for Nanto, NantoCare, NantoHealth brands.
  Classes (Nice Classification):
    Class 5  — Pharmaceuticals, medical/veterinary preparations (NantoCare dialyzers)
    Class 10 — Medical devices and apparatus (NantoCare brand on all devices)
    Class 35 — Business services (Nanto corporate brand)
    Class 42 — Software and IT services (NantoHealth/NantoHealthOS)
    Class 44 — Medical services (Nanto Kidney Care, NantoCare brand on clinic)
  Timeline:  12-18 months from filing to registration. Interim TM protection from filing date.
  Priority:  File BEFORE scaling product sales or clinic launch.

─── INTERNATIONAL STANDARDS ──────────────────────────────────────────────────

ISO 13485: Medical Devices — Quality Management Systems
  What:      The universal QMS standard for medical device manufacturers.
             Every supplier must hold a current ISO 13485 certificate.
  Ask from:  ALL suppliers for ALL products. Non-negotiable.
  Check:     Certificate must name the specific facility and product lines.
             Check expiry date — certificates are valid 3 years.

ISO 10993: Biological Evaluation of Medical Devices
  What:      Biocompatibility testing for devices in contact with the body.
             Dialyzers and bloodlines contact blood directly — this is critical.
  Parts used: ISO 10993-1 (framework), 10993-4 (haemocompatibility),
              10993-5 (cytotoxicity), 10993-10 (irritation/sensitization).
  Ask from:  CN Meditech (dialyzers and bloodlines). Full test report required.
  PPB use:   PPB may require ISO 10993 test reports in the Class B/C dossier.

ISO 14971: Risk Management for Medical Devices
  What:      Documents that the manufacturer has identified and mitigated all
             foreseeable risks associated with the device.
  Ask from:  CN Meditech. A risk management file summary is sufficient for dossier.

IEC 60601: Medical Electrical Equipment Safety
  What:      Electrical safety standard for powered medical devices.
  Relevance: Jihua HD machines (and any electrical equipment Nanto imports).
             Not required for consumables (dialyzers, bloodlines).
  Ask from:  JIHUA Medical Technology (machines).

CE Marking (EU MDR 2017/745)
  What:      European regulatory approval. Two classes relevant to Nanto:
             Class IIa (bloodlines) and Class IIb (dialyzers, HD machines).
  Why it matters for PPB: PPB uses CE marking as positive evidence of quality
             and safety during product review. CE-marked products have faster
             PPB review in practice (not formally, but operationally true).
  Ask from:  CN Meditech (dialyzers, bloodlines), JIHUA (machines).
             Request: CE certificate + Declaration of Conformity (DoC).

WHO GMP (Good Manufacturing Practice)
  What:      WHO's manufacturing quality standard. Equivalent to EU GMP.
  Relevance: PPB Class B/C dossiers may require a GMP certificate.
             WHO GMP is acceptable to PPB for Chinese manufacturers.
  Ask from:  CN Meditech. If they have both ISO 13485 and WHO GMP, include both.

NMPA (National Medical Products Administration, China)
  What:      China's medical device regulator. Equivalent to FDA (US) or EMA (EU).
             CN Meditech's products must be NMPA-registered to be legally
             manufactured and exported from China.
  Documents: NMPA registration certificate (医疗器械注册证) for each product.
             Certificate of Free Sale (CFS) — confirms product is legally sold
             in China and may be exported. Issued by NMPA or local authority.
  Ask from:  CN Meditech (Andy). One certificate per SKU.
  Note:      NMPA certificates are in Chinese with English translation.
             Both originals and certified translations are needed for PPB.

FDA (US Food and Drug Administration)
  What:      US regulatory clearance for medical devices.
             510(k) clearance = Class II medical device (most dialysis consumables).
  Why relevant: PPB accepts FDA clearance as evidence of product safety.
             Not required, but strengthens the dossier significantly.
  Ask from:  CN Meditech IF they have FDA clearance (many Chinese manufacturers do
             for export purposes). Do not assume — ask Andy directly.

AAMI (Association for the Advancement of Medical Instrumentation)
  What:      US standards body for medical devices, especially dialysis equipment.
             AAMI standards are referenced in US and international guidelines.
  Key standards: ANSI/AAMI RD52 (dialysate quality), RD62 (water for dialysis).
  Relevance: NKC clinic water quality (RO system) should meet AAMI RD62.
             Ask the RO system supplier for AAMI RD62 compliance documentation.

ASTM International
  What:      American Society for Testing and Materials. Sets technical standards
             for materials and products including medical devices.
  Relevance: May be referenced in biocompatibility or material testing reports.
             Less commonly required in Kenya PPB dossiers than ISO/CE.

─── NANTO CURRENT PRODUCTS AND REGULATORY STATUS ─────────────────────────────

SPARE PARTS (Class A — Nanto Med):
  Gear pumps, pressure transducers, flow sensors, valve components,
  electronic control boards, LCD replacements, connectors.
  Supplier: Shubham Corporation (India — Brijesh)
  Status: PPB Class A registration IN PROGRESS. Confirm with Dr. Alice.

DIALYZERS — NantoCare brand (Class B/C — Nanto Med):
  OEM manufacturer: CN Meditech, China (contact: Andy)
  SKUs: NM-DZ-HF18 (and other HF series — confirm full SKU list with Andy)
  Order: 930 dialyzers in production, USD 1,538 deposit wired (May 2026)
  PPB Status: PENDING — Establishment License must come first.
  Superintendent Pharmacist: PENDING hire (KES 65,000/month, 3 days/week)

BLOODLINES — NantoCare brand (Class B/C — Nanto Med):
  OEM manufacturer: CN Meditech, China (contact: Andy)
  PPB Status: PENDING — same pathway as dialyzers.

HEDCLIP (Class A or unclassified — confirm):
  Supplier: HEDclip, Denmark (contact: Kristian)
  Status: Classification to be confirmed with PPB. May be disposable accessory
          rather than regulated medical device — verify.

HD MACHINES — Jihua (Class A equipment):
  Supplier: JIHUA Medical Technology, China (contact: Kevin Li)
  Relevance: Machine placement requires PPB-registered machines. Confirm Jihua
             has Kenya PPB registration or whether Nanto must register as LTR.

─── SUPPLIER CONTACTS ────────────────────────────────────────────────────────

CN Meditech (dialyzers, bloodlines):  Andy  |  China
Shubham Corporation (spare parts):    Brijesh  |  India
HEDclip (HEDclips):                   Kristian  |  Denmark
JIHUA Medical Technology (machines):  Kevin Li  |  China
Dr. Alice:                            PPB ground consultant  |  Nairobi
```

---

## Auto-detect mode from request

| Request keywords | Mode |
|---|---|
| "what class", "how is this classified", "is this Class A/B/C", "what pathway" | CLASSIFY |
| "PPB", "PRIMS", "establishment license", "product registration", "import permit" | PPB |
| "dossier", "what documents do I need", "registration file", "what to submit" | DOSSIER |
| "ask supplier", "request documents", "letter to Andy", "what to ask CN Meditech" | SUPPLIER REQUEST |
| "KMPDC", "clinical license", "practitioner registration", "Dr. Twahir license" | KMPDC |
| "MOH", "facility license", "clinic license", "health facility", "KMHFR" | MOH FACILITY |
| "SHA accreditation", "SHA provider", "SHA approval for clinic" | SHA ACCREDITATION |
| "NEMA", "medical waste", "EIA", "environmental" | NEMA |
| "PVOC", "KEBS", "pre-export verification", "standards on import" | KEBS PVOC |
| "trademark", "KIPI", "brand registration", "NantoCare trademark" | KIPI |
| "ISO", "CE mark", "WHO GMP", "NMPA", "FDA", "what standard", "IEC" | STANDARDS |
| "tracker", "status", "what is pending", "all registrations", "where are we" | TRACKER |

If the request spans multiple modes, run CLASSIFY first, then the relevant mode.
If ambiguous, ask with AskUserQuestion (D1).

---

## Mode 1 — CLASSIFY

Determine the PPB regulatory class and filing pathway for any product.

### Classification decision tree

```
STEP 1: Is this a pharmaceutical (drug, medicine, active ingredient)?
  YES → Class C (prescription) or Class B (OTC) under Pharmacy and Poisons Act
  NO  → Continue to Step 2

STEP 2: Is this a medical device that contacts the patient directly?
  YES (blood contact: dialyzer, bloodline, fistula needle) → Likely Class B/C
  YES (skin/indirect: BP cuff, external monitor) → Likely Class A
  NO (equipment component, spare part, electronic board) → Class A

STEP 3: Is this equipment or machinery (not a consumable)?
  YES → Class A medical device. Check if PVOC also applies.
  NO  → Return to Step 2.

STEP 4: Is this a reagent, diagnostic test, or in-vitro diagnostic?
  YES → Separate IVD pathway. Consult Dr. Alice.
```

### Output format

```
CLASSIFICATION DECISION — [Product Name]
Date: DD Month YYYY
────────────────────────────────────────────────────────
Product:       [Name, SKU if known]
Supplier:      [Company, country]
Description:   [One sentence: what it does, how it interacts with the patient]

PPB CLASS:     [A / B / C / Unclassified — verify with PPB]
Reasoning:     [One sentence explaining the classification decision]

FILING PATH:
  1. [First step]
  2. [Next step]
  3. [Documents needed — see Mode 3 DOSSIER for full list]

SUPERINTENDENT PHARMACIST REQUIRED: [YES / NO]
PVOC (KEBS) LIKELY REQUIRED:        [YES / NO / VERIFY]
CE MARKING REQUIRED BY PPB:         [NO — but strengthens dossier]

FLAG: [Any uncertainty or PPB clarification needed]
  → Recommended action: [email PPB / ask Dr. Alice / verify in PRIMS]
────────────────────────────────────────────────────────
```

---

## Mode 2 — PPB

Navigate the full PPB PRIMS registration workflow.

### Step 1 — Establishment License (prerequisite for everything)

The Establishment License is the gate. No product registration, no importation,
no distribution can happen legally until this is in place.

```
ESTABLISHMENT LICENSE STATUS CHECK
────────────────────────────────────────────────────────
Entity:        Nanto Ventures Limited
KRA PIN:       P052406460F
Address:       The Piano, 8th Floor, 171 Brookside Drive, Nairobi
Activity:      Import, distribution, wholesale of medical devices

DOCUMENTS REQUIRED (if not yet submitted):
  [ ] PPB Establishment License application form (from PRIMS portal)
  [ ] Certificate of Incorporation (Nanto Ventures Limited)
  [ ] KRA PIN certificate
  [ ] Premises evidence (lease agreement for The Piano / Box Leo 3PL)
  [ ] Director's ID (Darack Nanto — Kenya ID or passport)
  [ ] Superintendent Pharmacist details (for Class B/C activities)
      — Full name, KMPDC registration number, retention certificate
  [ ] Application fee payment receipt (PPB fee schedule — verify current rate)

CURRENT STATUS: IN PROGRESS
NEXT ACTION:   Ask Dr. Alice: "Can you confirm the application reference
               number and current stage in PRIMS? What is outstanding?"
────────────────────────────────────────────────────────
```

### Step 2 — Product Registration in PRIMS

Once Establishment License is confirmed, begin product registration per SKU.

```
PPB PRODUCT REGISTRATION PROCESS
────────────────────────────────────────────────────────
For each product:
  1. Log in to PRIMS (prims.pharmacyboardkenya.org)
  2. Create new product application
  3. Enter product details: name, class, manufacturer, country, SKU
  4. Upload dossier documents (see Mode 3 for full checklist)
  5. Pay application fee
  6. Await PPB review (3-9 months for Class A, 6-18 months for Class B/C)
  7. Respond to PPB queries (PPB often requests additional information)
  8. Receive registration certificate

REGISTRATION NUMBER FORMAT: [MB/MD/XXX/YYYYYYY]
  MB = medical device broad category
  MD = medical device
  Year and sequence number assigned by PPB

ONE APPLICATION PER SKU. Do not batch multiple product variants into
one application. Each dialyzer size (HF14, HF17, HF18, etc.) is a
separate registration.
────────────────────────────────────────────────────────
```

### Step 3 — Import Permit

After product is registered, each shipment requires an import permit.

```
IMPORT PERMIT PROCESS (per shipment)
────────────────────────────────────────────────────────
Required for: Each commercial shipment of registered medical devices
Apply via:   PRIMS portal (under Establishment account)
Documents:   [ ] Proforma invoice or commercial invoice
             [ ] Packing list
             [ ] Certificate of analysis (batch specific)
             [ ] PPB registration certificate of the product
Timing:      Apply before shipment departs country of origin.
             Permit valid for the specific shipment only.
────────────────────────────────────────────────────────
```

---

## Mode 3 — DOSSIER

Build a complete product registration dossier checklist for any product.

### DOSSIER A: Class A Medical Device (Spare Parts — Shubham)

```
PPB CLASS A DOSSIER — [Product Name]
Supplier: Shubham Corporation, India (Brijesh)
────────────────────────────────────────────────────────
SECTION 1: ADMINISTRATIVE DOCUMENTS
  [ ] Completed PPB application form (PRIMS)
  [ ] Nanto Ventures Limited Establishment License (certified copy)
  [ ] Power of Attorney from manufacturer appointing Nanto as Local
      Technical Representative (LTR) in Kenya
      — On manufacturer's letterhead, signed by authorised officer
      — Specifies product(s) covered, territory (Kenya), duration
  [ ] LTR declaration (Nanto accepts responsibility for product in Kenya)

SECTION 2: MANUFACTURER INFORMATION
  [ ] Manufacturer's full name, address, country of registration
  [ ] ISO 13485 certificate (current, facility-specific, product line listed)
  [ ] GMP certificate (if applicable — WHO GMP or equivalent)
  [ ] Manufacturing licence from country of origin (India CDSCO or equivalent)

SECTION 3: PRODUCT INFORMATION
  [ ] Product name (brand and generic), model/part number, description
  [ ] Intended use statement (what the part does, what machine it fits)
  [ ] Product specifications / technical datasheet
  [ ] Instructions for Use (IFU) — in English
  [ ] Labeling samples (all labels on product and outer packaging)
  [ ] CE mark certificate (if applicable) + Declaration of Conformity
  [ ] Any other market approvals (FDA, TGA, ANVISA) — include if available

SECTION 4: QUALITY AND SAFETY
  [ ] Test reports relevant to the product (electrical, mechanical, material)
  [ ] Certificate of Analysis (sample batch)
  [ ] Shelf life / stability data (if applicable)

SECTION 5: KENYA-SPECIFIC
  [ ] Proposed Kenya label — must include:
      English language, product name, intended use, manufacturer name and
      address, country of manufacture, batch/lot number, manufacturing
      date, expiry date (if applicable), storage conditions, PPB registration
      number (added after approval)
  [ ] Application fee payment
────────────────────────────────────────────────────────
ESTIMATED TIMELINE: 3-9 months from submission
```

### DOSSIER B: Class B/C Consumable — Dialyzers and Bloodlines (CN Meditech)

```
PPB CLASS B/C DOSSIER — NantoCare Dialyzers / Bloodlines
OEM Manufacturer: CN Meditech, China (Andy)
────────────────────────────────────────────────────────
SECTION 1: ADMINISTRATIVE DOCUMENTS
  [ ] Completed PPB application form (PRIMS)
  [ ] Nanto Ventures Limited Establishment License (certified copy)
  [ ] Superintendent Pharmacist details:
      — Full name
      — KMPDC registration / retention certificate (current)
      — Signed declaration accepting responsibility as Superintendent
  [ ] Power of Attorney from CN Meditech appointing Nanto as LTR in Kenya
      — On CN Meditech letterhead
      — Authorised by Andy or legal signatory
      — Lists specific products and SKUs covered
      — States territory: Republic of Kenya
      — Notarised and apostilled (Chinese notary + Hague Apostille)
  [ ] Private Label / OEM Agreement between CN Meditech and Nanto
      — Authorises Nanto to sell products under NantoCare brand
      — CN Meditech confirms the product is the same formulation/design
      — Apostilled if PPB requires

SECTION 2: MANUFACTURER INFORMATION (CN MEDITECH)
  [ ] ISO 13485 certificate — current, lists dialyzers and bloodlines
      specifically (not just the company)
  [ ] WHO GMP certificate (or equivalent — Chinese GMP/NMPA GMP)
  [ ] NMPA registration certificate for each product SKU
      — 医疗器械注册证 (Medical Device Registration Certificate)
      — With certified English translation by sworn translator
  [ ] Certificate of Free Sale (CFS) from China
      — Issued by NMPA or local health bureau
      — Confirms product is legally manufactured and sold in China
      — With certified English translation
  [ ] Manufacturing licence / business licence (CN Meditech entity)

SECTION 3: PRODUCT INFORMATION
  [ ] Product name (NantoCare brand) + generic description
  [ ] Intended use: "Single-use haemodialyzer/bloodline for haemodialysis"
  [ ] Complete product specifications:
      — Dialyzer: membrane material, surface area, flux type (HF/LF),
        KUF, KoA urea, sterilisation method, priming volume
      — Bloodline: tubing diameter, pump segment specs, connectors,
        total volume, compatible machines
  [ ] Instructions for Use (IFU) — English version
  [ ] All labeling: product label, outer carton, inner packaging
  [ ] CE marking certificate + Notified Body details + DoC
      (Class IIa for bloodlines, Class IIb for dialyzers under EU MDR)
  [ ] FDA clearance / 510(k) number (IF CN Meditech has US clearance)

SECTION 4: BIOCOMPATIBILITY AND SAFETY (CRITICAL FOR BLOOD-CONTACT DEVICES)
  [ ] ISO 10993-1 biological evaluation framework document
  [ ] ISO 10993-4 haemocompatibility test report
      (thrombogenicity, coagulation, haemolysis, complement activation)
  [ ] ISO 10993-5 cytotoxicity test report
  [ ] ISO 10993-10 sensitisation / irritation test report
  [ ] Sterilisation validation report (method: EO gas or gamma — specify)
  [ ] Pyrogenicity / endotoxin test report (dialyzers must be apyrogenic)
  [ ] Packaging integrity / sterile barrier validation (ISO 11607)

SECTION 5: QUALITY AND BATCH DATA
  [ ] Batch manufacturing record (sample)
  [ ] Certificate of Analysis (sample batch) — shows QC parameters passed
  [ ] Shelf life / stability study report
  [ ] Post-market surveillance summary (if product is already on market)

SECTION 6: KENYA-SPECIFIC
  [ ] Proposed NantoCare Kenya label for each SKU:
      — Product name (NantoCare [HF18 Dialyzer / Bloodline Set])
      — Intended use
      — CN Meditech as manufacturer + China address
      — Nanto Ventures Limited as importer / LTR + Kenya address
      — Batch number, manufacturing date, expiry date
      — Storage: "Store in cool, dry place. Protect from direct sunlight."
      — Sterilisation indicator (if applicable)
      — Single use symbol
      — PPB registration number (added after approval)
  [ ] Application fee payment receipt
────────────────────────────────────────────────────────
ESTIMATED TIMELINE: 6-18 months from complete submission
NOTE: Incomplete dossiers are the #1 cause of PPB registration delay.
Submit complete or do not submit. Do not submit partially and "add later."
```

---

## Mode 4 — SUPPLIER REQUEST

Generate a structured document request letter to a supplier for dossier requirements.

### For CN Meditech (Andy) — PPB Class B/C dossier

```
SUPPLIER DOCUMENT REQUEST — CN Meditech
Product: NantoCare Dialyzers and Bloodlines
Purpose: PPB (Kenya) Product Registration Dossier

---

Dear Andy,

I trust you are well.

As part of our Kenya Pharmacy and Poisons Board (PPB) product registration
process for the NantoCare dialyzers and bloodlines, we are required to submit
a complete technical and regulatory dossier. Kindly assist us by providing the
following documents at your earliest convenience.

We require documents for each SKU we intend to register. Please confirm the
full list of SKUs available under our OEM agreement so we can ensure complete
coverage.

DOCUMENTS REQUIRED — CN MEDITECH (per SKU where indicated)

1. NMPA REGISTRATION CERTIFICATE (per SKU)
   The 医疗器械注册证 for each dialyzer size and for the bloodline set.
   Required with a certified English translation by a sworn translator.

2. CERTIFICATE OF FREE SALE (CFS) — China
   Confirming the products are legally manufactured and freely sold in China.
   Issued by NMPA or the relevant provincial health bureau.
   Required with a certified English translation.

3. ISO 13485 CERTIFICATE (company-level, current)
   Confirming your quality management system covers the production of
   haemodialyzers and bloodlines at the manufacturing facility.
   Please confirm the certificate expiry date.

4. CE MARKING DOCUMENTATION
   CE certificate from your Notified Body for:
   — Dialyzers (Class IIb under EU MDR 2017/745)
   — Bloodlines (Class IIa under EU MDR 2017/745)
   Plus the Declaration of Conformity (DoC) for each product.

5. WHO GMP CERTIFICATE OR EQUIVALENT
   Confirming your manufacturing facility meets GMP requirements.
   If you have both ISO 13485 and a GMP certificate, please provide both.

6. BIOCOMPATIBILITY TEST REPORTS (ISO 10993 series)
   — ISO 10993-4 haemocompatibility (thrombogenicity, haemolysis)
   — ISO 10993-5 cytotoxicity
   — ISO 10993-10 sensitisation / irritation
   These are critical for blood-contacting devices. PPB will require them.

7. STERILISATION VALIDATION REPORT
   Confirming the sterilisation method (EO gas / gamma) and the validation
   parameters used. Please include the sterile barrier validation (ISO 11607).

8. ENDOTOXIN / PYROGENICITY TEST REPORT
   Confirming the dialyzers meet apyrogenic requirements.

9. PRODUCT SPECIFICATIONS — full technical datasheet for each SKU
   Dialyzers: membrane material, surface area, flux classification, KUF,
   KoA urea, blood volume, priming volume, sterilisation method.
   Bloodlines: tubing dimensions, pump segment, total priming volume,
   connector types, compatible machines.

10. INSTRUCTIONS FOR USE (IFU) — English version

11. SHELF LIFE / STABILITY DATA
    Confirming the product shelf life and the study supporting it.

12. POWER OF ATTORNEY
    A signed letter on CN Meditech letterhead appointing Nanto Ventures Limited
    as the Local Technical Representative (LTR) for Kenya, covering the
    NantoCare-branded products. This must be:
    — Signed by an authorised officer of CN Meditech
    — Notarised by a Chinese notary
    — Apostilled (Hague Convention)
    We can provide a template for your reference if helpful.

13. OEM / PRIVATE LABEL AUTHORISATION LETTER
    Confirming CN Meditech authorises Nanto Ventures Limited to sell the
    products under the NantoCare brand name in Kenya.

Please provide all documents in their original language with certified English
translations where the originals are in Chinese.

We appreciate your continued support. Kindly revert with a timeline for when
you can compile these documents.

Yours sincerely,

Darack Nanto
Founder, Nanto Med
med@nanto.com | +254 20 878 7990
```

### For Shubham Corporation (Brijesh) — PPB Class A dossier

```
SUPPLIER DOCUMENT REQUEST — Shubham Corporation
Products: Dialysis machine spare parts
Purpose: PPB (Kenya) Class A Medical Device Registration

---

Dear Mr. Brijesh,

I trust you are keeping well.

We are in the process of registering our dialysis machine spare parts range with
the Pharmacy and Poisons Board (PPB) of Kenya. To complete the product dossier
for each SKU, we require the following documents from Shubham Corporation.

DOCUMENTS REQUIRED — PER SKU

1. ISO 13485 CERTIFICATE (current)
   Confirming your quality management system covers the products in question.
   If Shubham does not hold ISO 13485, please advise what quality certifications
   you hold for your manufacturing processes.

2. MANUFACTURER DETAILS
   Full legal name, registered address, and country of registration for the
   manufacturer of each part (may be Shubham or the OEM they source from).

3. PRODUCT TECHNICAL DATASHEET
   For each SKU: part name, part number, compatible machines, specifications,
   materials used, intended function.

4. CE MARKING OR EQUIVALENT APPROVAL (if available)
   Any CE mark, FDA listing, or other market approval for the products.
   Include if available — not mandatory for Class A but strengthens the dossier.

5. POWER OF ATTORNEY
   A letter on Shubham Corporation letterhead appointing Nanto Ventures Limited
   as the importer and distributor for Kenya for the specified products.

6. CERTIFICATE OF ANALYSIS (sample)
   For a recent production batch of the products.

7. INSTRUCTIONS FOR USE (if applicable)

Kindly confirm which of the above you can provide, and advise the timeline.
We are happy to send a pro-forma template for the Power of Attorney if helpful.

Yours sincerely,

Darack Nanto
Founder, Nanto Med
med@nanto.com | +254 20 878 7990
```

### For JIHUA (Kevin Li) — HD Machine PPB registration

```
SUPPLIER DOCUMENT REQUEST — JIHUA Medical Technology
Products: Haemodialysis machines (HD machines)
Purpose: Kenya PPB market authorisation / LTR confirmation

---

Dear Mr. Kevin Li,

As discussed in our distribution agreement discussions, we require the following
documents to confirm the regulatory status of the JIHUA haemodialysis machines
for the Kenya market and to prepare for any necessary PPB registrations.

DOCUMENTS REQUIRED

1. NMPA REGISTRATION CERTIFICATE for the HD machine model(s) covered
   by our East Africa distribution agreement. With certified English translation.

2. CE CERTIFICATE under EU MDR (Class IIb active medical device)

3. ISO 13485 CERTIFICATE for the manufacturing facility

4. IEC 60601-1 CERTIFICATE — medical electrical equipment safety
   (and IEC 60601-2-16 specific to haemodialysis equipment)

5. TECHNICAL FILE SUMMARY / PRODUCT SPECIFICATION SHEET

6. OPERATOR AND SERVICE MANUALS (English versions)

7. POWER OF ATTORNEY appointing Nanto Ventures Limited as authorised
   distributor / LTR for Kenya (and East Africa per our agreement)

8. DECLARATION that the machines are fit for the Kenya 50 Hz / 240V
   electrical standard and require no modification for Kenya use.

Kindly revert with a timeline for these documents.

Yours sincerely,

Darack Nanto
Founder, Nanto Med
med@nanto.com | +254 20 878 7990
```

---

## Mode 5 — KMPDC

Kenya Medical Practitioners and Dentists Council — clinical staff requirements.

```
KMPDC REQUIREMENTS — Nanto Kidney Care Site 1
────────────────────────────────────────────────────────
All clinical staff practicing at NKC must be:
  1. KMPDC-registered with a current retention certificate
  2. Named in the MOH health facility license application

MEDICAL DIRECTOR (Required for MOH facility license):
  Dr. Mohammed Twahir
  Action: Confirm Dr. Twahir's current KMPDC registration number and
          retention status. Request a certified copy of his retention
          certificate for the facility license application.
  Portal: kmpdc.go.ke → Verify Practitioner

ATTENDING NEPHROLOGISTS (2, on rotation):
  To be nominated by Dr. Twahir.
  Action: For each nephrologist:
    [ ] Confirm KMPDC registration number
    [ ] Confirm they hold a current retention certificate
    [ ] Obtain their consent to be named in the facility license
    [ ] Confirm sub-speciality: nephrology / renal medicine

VERIFICATION TEMPLATE (to ask each doctor):
  "Kindly provide us with your KMPDC registration number and a copy of your
  current retention certificate. We are listing you as an attending physician
  at Nanto Kidney Care in our MOH health facility license application."

KMPDC ANNUAL RETENTION:
  All practitioners must renew retention annually. Build a calendar reminder
  for each doctor's retention renewal date. A lapsed retention invalidates
  their ability to practice and can affect the facility license.
────────────────────────────────────────────────────────
```

---

## Mode 6 — MOH FACILITY

Ministry of Health health facility licensing for Nanto Kidney Care.

```
MOH HEALTH FACILITY LICENSE — Nanto Kidney Care Site 1
────────────────────────────────────────────────────────
LICENSING AUTHORITY: Nairobi County Department of Health
                     (County-level application, MOH national registration)
REGISTRY:            KMHFR — kmhfr.health.go.ke

DOCUMENTS REQUIRED:
  [ ] Completed health facility application form (County Health Department)
  [ ] Certificate of Incorporation (Nanto Ventures Limited)
  [ ] KRA PIN certificate
  [ ] Lease agreement for clinic premises (signed, certified copy)
  [ ] Architectural / floor plan of the facility (drawn by registered architect)
      — Shows all clinical areas, machine stations, RO room, pharmacy, waste area
  [ ] NEMA compliance confirmation (EIA or exemption letter)
  [ ] PPB Establishment License (certified copy)
  [ ] Superintendent Pharmacist details and KMPDC registration
  [ ] Medical Director details: Dr. Twahir, KMPDC retention certificate
  [ ] All attending physicians: KMPDC registration numbers and retention
  [ ] Equipment list: all HD machines (serial numbers), RO system, generator
  [ ] Medical waste management plan (NEMA-licensed contractor details)
  [ ] Application fee (County Health Department fee schedule)

POST-APPLICATION:
  [ ] Physical inspection by County Health Officer (schedule 2-4 weeks after application)
  [ ] Address any inspection deficiencies in writing within 30 days
  [ ] Receive facility license number
  [ ] Register in KMHFR (Kenya Master Health Facility Registry)

ESTIMATED TIMELINE: 3-6 months from complete application
────────────────────────────────────────────────────────
```

---

## Mode 7 — SHA ACCREDITATION

Social Health Authority provider accreditation for Nanto Kidney Care.

```
SHA PROVIDER ACCREDITATION — Nanto Kidney Care Site 1
────────────────────────────────────────────────────────
PREREQUISITE: MOH health facility license must be in hand before SHA
              accreditation can be applied for.

RELEVANCE:    Without SHA accreditation, NKC cannot bill SHA for patient
              sessions. SHA revenue is 65-70% of Year 2 clinic revenue.
              This is the most important regulatory step for cash flow.

DOCUMENTS REQUIRED (verify current requirements with SHA directly):
  [ ] SHA provider registration application form (sha.go.ke)
  [ ] MOH health facility license (certified copy)
  [ ] KMHFR registration confirmation / facility code
  [ ] PPB Establishment License
  [ ] Proof of dialysis equipment (machine serial numbers, service records)
  [ ] Clinical staffing list (Medical Director, nephrologists, renal nurses)
  [ ] SHA claims workflow description (NantoHealthOS SHA module)
  [ ] Banking details for SHA reimbursements (DIB Bank Kenya)
  [ ] Application fee

SHA TARIFF (Legal Notice 146, September 2024):
  Haemodialysis session: KES 16,000 per session
  Monitor Gazette for changes — tariff revisions are published in Kenya Gazette.

ESTIMATED TIMELINE: 2-4 months post-MOH license

NANTOHEALTHOS NOTE: Ensure the SHA claims module in NantoHealthOS uses the
  correct SHA claims format, patient identifier fields, and session codes
  before the first claim is submitted. SHA rejection rates for format errors
  are high. This is the core product advantage.
────────────────────────────────────────────────────────
```

---

## Mode 8 — NEMA

National Environment Management Authority compliance.

```
NEMA COMPLIANCE — Nanto Kidney Care Site 1
────────────────────────────────────────────────────────
ENVIRONMENTAL IMPACT ASSESSMENT (EIA)
  Trigger: Construction or significant renovation of health facilities may
           require an EIA under the Environmental Management and Coordination
           Act (EMCA) Cap 387.
  Action:  Confirm with a NEMA-registered environmental consultant whether
           the NKC fit-out triggers an EIA. A fit-out of an existing
           commercial space may qualify for a project report (simpler) rather
           than a full EIA.
  Cost:    KES 50,000-200,000 for a project report. More for a full EIA.
  Timeline: 1-3 months for project report approval.

MEDICAL WASTE MANAGEMENT LICENSE
  Requirement: All healthcare facilities must have a documented medical waste
               management plan and use a NEMA-licensed waste handler.
  NKC waste categories:
    — Infectious waste (bloodlines, dialyzers, gloves, sharps): incineration
    — Sharps (fistula needles): sharps containers, then incineration
    — General waste: regular municipal collection
    — Liquid waste (dialysate, blood-contaminated): drain per NEMA guidelines
  Action:
    [ ] Identify NEMA-licensed medical waste incinerator service in Nairobi
    [ ] Sign service agreement before clinic opens
    [ ] Include incinerator contractor details in MOH facility license application
  Budget: KES 28,000/month (already in clinic operating cost model)

NEMA REGISTRATION:
  [ ] Register the facility with NEMA upon MOH license issuance
  [ ] Annual environmental audit may be required
────────────────────────────────────────────────────────
```

---

## Mode 9 — KEBS / PVOC

Kenya Bureau of Standards pre-export verification and import standards.

```
PVOC (Pre-Export Verification of Conformity) — Nanto Imports
────────────────────────────────────────────────────────
WHAT IT IS: KEBS requires certain imported goods to be inspected and certified
before shipment from the country of origin. The PVOC certificate is issued by
a KEBS-appointed agent in the source country.

APPOINTED AGENTS (China — for CN Meditech, Jihua):
  — Intertek (China offices)
  — SGS (China offices)
  — Bureau Veritas (BV)

IS PVOC REQUIRED FOR MEDICAL DEVICES?
  Medical devices regulated by PPB are typically PPB-controlled on import rather
  than KEBS/PVOC controlled. However, verify with the clearing agent for each
  HS code before shipment. Importing a shipment without a required PVOC
  certificate results in it being held at Mombasa port.

ACTION: For each new product category imported for the first time, ask the
  clearing agent (and Dr. Alice): "Does this HS code require a PVOC certificate
  under the KEBS import requirements?" Get the answer in writing before booking
  the shipment.

HS CODES (confirm with clearing agent / KRA):
  HD machines:        9018.90 (0% CET, 0% VAT, exempt as medical equipment)
  Dialyzers:          9018.90 (verify — may attract VAT if classified as filter)
  Bloodlines:         9018.90 (VAT-exempt if correctly classified)
  Spare parts:        9018.90 or component-specific HS code
  RO water system:    8421.21 (water filtration equipment)
────────────────────────────────────────────────────────
```

---

## Mode 10 — KIPI

Kenya Industrial Property Institute — trademark registration.

```
TRADEMARK REGISTRATION — Nanto brands
────────────────────────────────────────────────────────
PRIORITY ORDER (file before each product line goes to market):

1. NantoCare (file NOW — product sales already started)
   Class 5:  Pharmaceuticals and medical preparations
   Class 10: Medical and veterinary apparatus and instruments
   Class 44: Medical services (for clinic)

2. Nanto / Nanto Med (file within 30 days)
   Class 10: Medical devices and apparatus
   Class 35: Business services (wholesale, distribution)

3. NantoHealth / NantoHealthOS (file before NantoHealth public launch)
   Class 42: Software and IT services
   Class 44: Healthcare services

FILING PROCESS:
  [ ] Prepare trademark application (mark, classes, description of goods/services)
  [ ] Search KIPI database first for conflicting marks (kipi.go.ke)
  [ ] File online or via KIPI agent (recommended — KIPI process is bureaucratic)
  [ ] Pay filing fee (KES 3,000-5,000 per class)
  [ ] Await examination (KIPI examines for absolute and relative grounds)
  [ ] Respond to any objections within 60 days of notice
  [ ] Mark published in Kenya Industrial Property Journal for opposition period (60 days)
  [ ] If no opposition, registration certificate issued

TIMELINE: 12-18 months from filing to certificate.
PROTECTION: Starts from the filing date (filing date = priority date).

INTERNATIONAL OPTION (future):
  If expansion beyond Kenya is planned, file under the African Regional
  Intellectual Property Organisation (ARIPO) via the Banjul Protocol.
  One filing covers: Kenya, Uganda, Tanzania, Rwanda, Zimbabwe, Ghana, and others.
────────────────────────────────────────────────────────
```

---

## Mode 11 — STANDARDS

Quick reference for any international standard relevant to Nanto products.

```
STANDARDS QUICK REFERENCE — Nanto Medical Products
────────────────────────────────────────────────────────
STANDARD         SCOPE                             ASK FROM         NANTO RELEVANCE
ISO 13485        QMS for medical device mfg         ALL suppliers    Must-have for all
ISO 10993-4      Haemocompatibility                 CN Meditech      Blood-contact devices
ISO 10993-5      Cytotoxicity                       CN Meditech      Blood-contact devices
ISO 10993-10     Sensitisation / irritation         CN Meditech      Blood-contact devices
ISO 11607        Sterile packaging validation       CN Meditech      Sterile dialyzers
ISO 14971        Risk management                    CN Meditech      Full dossier
IEC 60601-1      Medical electrical equipment       JIHUA            HD machines
IEC 60601-2-16   Haemodialysis equipment specific   JIHUA            HD machines
CE MDR 2017/745  EU market approval (Class IIa/IIb) CN Meditech, JIHUA  PPB dossier support
WHO GMP          Manufacturing quality              CN Meditech      PPB Class B/C dossier
NMPA cert        China market approval              CN Meditech      PPB dossier (required)
AAMI RD62        Water quality for dialysis         RO supplier      NKC clinic RO system
ASTM             Material testing standards         CN Meditech      Referenced in test reports
FDA 510(k)       US clearance (Class II devices)    CN Meditech      PPB dossier support (optional)
────────────────────────────────────────────────────────

HOW TO USE THIS TABLE IN A DOSSIER:
  1. Identify the product (dialyzer / bloodline / machine / spare part)
  2. Cross-reference the relevant standards
  3. Use Mode 4 (SUPPLIER REQUEST) to generate the document request letter
  4. Tick off documents received in Mode 12 (TRACKER)
```

---

## Mode 12 — TRACKER

Unified regulatory status tracker across all pathways and products.

### Step 1 — Load or build the tracker

```bash
TRACKER_FILE="$HOME/Desktop/nanto-regulatory-tracker-$(date +%Y%m%d).md"
if [ -f "$HOME/Desktop/nanto-regulatory-tracker-"*.md ]; then
  ls -t "$HOME/Desktop/nanto-regulatory-tracker-"*.md | head -1
fi
```

### Step 2 — Output the master tracker

```
NANTO REGULATORY MASTER TRACKER
Date: DD Month YYYY
════════════════════════════════════════════════════════════════════════════════

PART A: KENYA REGULATORY BODIES

Board          Item                              Status      Next Action          Owner      By
PPB            Establishment License             IN PROGRESS Confirm ref# w Alice Dr. Alice  ASAP
PPB            Spare parts (Class A) — all SKUs  IN PROGRESS List all SKUs, sub  Darack     30 days
PPB            Dialyzer NantoCare (Class B/C)    PENDING     Await Est. License   Darack     After EL
PPB            Bloodline NantoCare (Class B/C)   PENDING     Await Est. License   Darack     After EL
PPB            HEDclip classification            VERIFY      Ask Dr. Alice        Dr. Alice  2 weeks
PPB            Superintendent Pharmacist         PENDING     Hire (KES 65K/mo)    Darack     Asap
KMPDC          Dr. Twahir retention cert         VERIFY      Request from Dr. T   Darack     This week
KMPDC          Attending nephrologist #1         PENDING     Identify + request   Dr. Twahir After call
KMPDC          Attending nephrologist #2         PENDING     Identify + request   Dr. Twahir After call
MOH/KMHFR      NKC facility license              FUTURE      After premises found Darack     2026/2027
SHA            NKC provider accreditation        FUTURE      After MOH license    Darack     2026/2027
NEMA           EIA / project report (NKC)        FUTURE      Engage env. consult  Darack     Pre-fitout
NEMA           Medical waste contractor          FUTURE      Get quotes           Darack     Pre-opening
KEBS/PVOC      PVOC requirement per HS code      VERIFY      Ask clearing agent   Clearing   Next shipment
KIPI           NantoCare trademark               PENDING     File immediately     Darack     This month
KIPI           Nanto / Nanto Med trademark       PENDING     File within 30 days  Darack     This month
KIPI           NantoHealth trademark             PENDING     File before launch   Darack     Before launch

PART B: PRODUCT DOSSIERS

Product                  Supplier      Class  Dossier Status        Missing Items
Gear pumps               Shubham       A      IN PROGRESS           ISO 13485, PoA
Pressure transducers     Shubham       A      IN PROGRESS           ISO 13485, PoA
Flow sensors             Shubham       A      IN PROGRESS           ISO 13485, PoA
NantoCare Dialyzer HF18  CN Meditech   B/C    NOT STARTED           Full dossier — see Mode 3B
NantoCare Bloodline set  CN Meditech   B/C    NOT STARTED           Full dossier — see Mode 3B
HEDclip                  HEDclip DK    TBC    Classification pending Confirm class first
JIHUA HD machine         JIHUA         A      NOT STARTED           NMPA, CE, IEC 60601, PoA

PART C: SUPPLIER DOCUMENTS PENDING

Supplier         Documents Requested    Documents Received    Outstanding
CN Meditech      Full dossier list      None yet              All (see Mode 4)
Shubham          Partial                ISO 13485, PoA        Test reports, specs
JIHUA            Not yet requested      None                  Send Mode 4 letter
HEDclip          Not yet requested      None                  After classification

════════════════════════════════════════════════════════════════════════════════
CRITICAL PATH:
  1. PPB Establishment License confirmed → unblocks all product registrations
  2. Superintendent Pharmacist hired → unblocks Class B/C submissions
  3. CN Meditech documents received → submit dialyzer/bloodline dossier to PRIMS
  4. All Class A spare parts registered → full spare parts catalogue can be marketed
```

---

## Output Rules

1. **No em dashes.** Use a comma, colon, or restructure.
2. **Kenya English.** Formal register throughout.
3. **Dates:** DD Month YYYY.
4. **Checklists use [ ] format.** Each item is discrete and actionable.
5. **Always state the current status** before the next action. Never give next actions without context.
6. **Flag uncertainty clearly.** Use VERIFY, CONFIRM, or CHECK when the answer depends on PPB discretion or real-time information. Never guess.
7. **Dr. Alice coordination:** Any action that goes to PPB in person goes through Dr. Alice. Never recommend Darack contact PPB directly for physical follow-up.
8. **Superintendent Pharmacist is the gate for Class B/C.** Always note this dependency. Do not suggest submitting a Class B/C application without a superintendent on record.
9. **Complete dossiers only.** Never recommend submitting a partial dossier. PPB reviews in the order received but may reject incomplete submissions entirely.

---

## Coupling Rules

1. **Legal questions go to /kenya-law.** Regulatory interpretation (what does the law say?) is /kenya-law. Filing workflow (what do I submit and where?) is this skill.
2. **HS codes and duty rates go to /kra.** This skill references HS codes but does not calculate import duty.
3. **Supplier emails go through /nanto-comms.** Mode 4 generates the letter text. Send the user to /nanto-comms to format it as an email.
4. **Registration fees go to /nanto-finance.** All PPB, KIPI, NEMA, KMPDC fees are tracked in Zoho Books.
5. **Tracker is a desktop file.** Save the tracker output to ~/Desktop. Do not store it in WorkDrive documents — it needs to be a live working file updated frequently.

---

## AskUserQuestion Format

Follow gstack D-numbered brief format. First question in a session = D1.

```
D<N> — <question title>
Context: <one grounding sentence>
ELI10: <plain English, 2-3 sentences, state the stakes>
Stakes if wrong: <one sentence>
Recommendation: <option> because <reason>
Options:
A) <option> (recommended)
B) <option>
```
