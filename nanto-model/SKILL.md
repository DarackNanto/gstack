---
name: nanto-model
preamble-tier: 3
version: 1.0.0
description: |
  Nanto financial modelling for investor presentations and site planning. Builds
  multi-year revenue projections, capital requirement tables, operating P&L,
  break-even analysis, SHA receivables gap calculations, and stressed vs base
  case scenarios. Covers Nanto Kidney Care clinic sites, Nanto Med supply
  operations, and combined entity models. Outputs investor-ready tables in
  markdown for /make-pdf. Boundary: day-to-day bookkeeping is /nanto-finance;
  investment structure is /nanto-investor; strategy is /warroom. (Nanto/gstack)
triggers:
  - financial model
  - model
  - projections
  - revenue model
  - capital requirements
  - break even
  - P&L
  - profit and loss
  - scenario
  - stressed case
  - base case
  - site model
  - clinic model
  - SHA gap
  - working capital
  - investor model
  - financial tables
  - nanto model
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - AskUserQuestion
---

# /nanto-model

Nanto's financial modelling layer. Builds, updates, and stress-tests financial
models for investor presentations and site planning decisions.

> **Not for day-to-day bookkeeping** — that is /nanto-finance (Zoho Books).
> **Not for investment structure or term sheets** — that is /nanto-investor.
> **Not for strategic framing or framework analysis** — that is /warroom.
> **Not for converting tables to PDF** — pass output to /make-pdf.

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
  echo '{"skill":"nanto-model","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'"}'  \
    >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
fi
# Load current NKC document if available
WORKDRIVE="/Users/darack/Library/CloudStorage/ZohoWorkDriveTrueSync-NantoHQ"
NKC_DOC="$WORKDRIVE/10-PROJECTS/MED/01-Strategy-and-Fundraising/Business-Plan/MED-DRAFT-NantoKidneyCare-StrategyAndFinancialModel-v0.1.md"
if [ -f "$NKC_DOC" ]; then
  echo "NKC_DOCUMENT: loaded"
else
  echo "NKC_DOCUMENT: not found — will use constants"
fi
echo "MODEL SKILL v1.0.0 READY"
```

---

## Constants

```
WORKDRIVE:     /Users/darack/Library/CloudStorage/ZohoWorkDriveTrueSync-NantoHQ
NKC_DOCUMENT:  $WORKDRIVE/10-PROJECTS/MED/01-Strategy-and-Fundraising/Business-Plan/
               MED-DRAFT-NantoKidneyCare-StrategyAndFinancialModel-v0.1.md
OUTPUT_DIR:    ~/Desktop/

EXCHANGE_RATE: KES 130 = USD 1  (update if CBK rate changes materially)

─── HD MACHINE INPUTS ────────────────────────────────────────
Jihua machine FOB (Shenzhen):         USD 8,500 per unit
Import charges (RDL + IDF + agent):   6.25% on CIF
Sea freight (10 units, shared):       USD 3,500
Marine insurance:                     0.5% of FOB
Road transport Mombasa-Nairobi:       USD 800
Landed cost per machine (10 units):   ~USD 9,528 (see Table A)

─── CLINIC REVENUE INPUTS ────────────────────────────────────
SHA session rate:                     KES 16,000  (Legal Notice 146, Sep 2024)
Standard private rate:                KES 18,500
Private suite rate:                   KES 25,000
Diaspora / holiday rate:              USD 180 (KES 23,400)
Sessions per machine per day:         2 (morning + afternoon shift)
Operating days per month:             26 (Mon-Sat)
Max sessions per machine per month:   52

─── PAYER MIX ────────────────────────────────────────────────
Year 1:  SHA 70% / Standard private 20% / Private suite 10%
Year 2+: SHA 65% / Standard private 20% / Private suite 15%
SHA payment delay:                    3-4 months (use 4 months for stress test)

─── CLINIC OPERATING COSTS (SITE 1, NAIROBI) ─────────────────
Monthly rent (Parklands, 434 sqm):    KES 330,000
Full staff payroll (10 machines):     KES 739,000/month
Electricity (8,000 kWh at KES 27):    KES 216,000/month
Water and sewerage:                   KES 35,000/month
Internet and comms:                   KES 18,000/month
Medical waste management:             KES 28,000/month
Machine maintenance reserve:          KES 80,000/month
Consumables COGS (transfer price):    KES 2,800 per session
Marketing:                            KES 80,000-100,000/month
Admin and miscellaneous:              KES 100,000/month

─── PRE-STARTUP CAPITAL (SITE 1) ─────────────────────────────
Lease deposit (6 months):             KES 1,980,000
Civil works and fit-out:              KES 4,500,000
Private suites fit-out (×2):          KES 900,000
Patient experience upgrades:          KES 850,000
Furniture, fittings, IT:              KES 1,400,000
Security system:                      KES 500,000
Launch branding and marketing:        KES 400,000
Regulatory fees (MOH/PPB/SHA/NEMA):   KES 1,200,000
Professional facilitation:            KES 700,000
10 HD machines landed:                KES 12,387,000
RO water treatment system landed:     KES 4,673,000
Generator (40 kVA):                   KES 650,000
NantoHealthOS deployment:             KES 500,000
Contingency (10%):                    KES 3,046,000
TOTAL PRE-STARTUP (Site 1):           KES 33,506,000 / USD 258,123

─── WORKING CAPITAL (6 MONTHS) ───────────────────────────────
6-month working capital buffer:       KES 12,468,000 / USD 95,908
Consumables initial stock (one-time): KES 2,000,000 / USD 15,385

─── NANTO MED SUPPLY OPERATIONS ─────────────────────────────
Nanto Med raise allocation:           USD 325,000
Consumables gross margin (target):    43% blended
Transfer price to clinic:             KES 2,800/session (Nanto Med COGS + margin)
Nanto Med annual margin from clinic:  ~KES 6,014,208 / USD 46,263 (at full capacity)

─── ADDITIONAL STRATEGIC BUDGET (APPROVED) ───────────────────
R&D (NantoHealthOS + Fleet IoT):      USD 50,000
Marketing and brand:                  USD 35,000
Sector elevation (KRA fellowship etc): USD 20,000
Additional travel (clinic scouting):  USD 20,000
Senior Kenya hire (Ops Director):     USD 25,000
Clinical knowledge and research:      USD 15,000
TOTAL STRATEGIC ADDITIONS:            USD 165,000

─── COMBINED RAISE ───────────────────────────────────────────
Nanto Med operations:                 USD 325,000
NKC pre-startup:                      USD 258,123
NKC working capital (6 months):       USD 95,908
Strategic additions:                  USD 165,000
TOTAL (rounded):                      USD 850,000

─── VALUATION ────────────────────────────────────────────────
Pre-money:                            USD 4,000,000
Post-money:                           USD 4,850,000
Megna stake (opening offer):          18% (USD 850,000 / USD 4,850,000)
```

---

## Auto-detect mode from request

| Request keywords | Mode |
|---|---|
| "capital requirements", "how much to set up", "Table C", "pre-startup" | CAPITAL PLAN |
| "revenue", "sessions", "how much will we make", "Table F", "occupancy" | REVENUE MODEL |
| "P&L", "profit and loss", "operating costs", "EBITDA", "Table G/H" | OPERATING PL |
| "break even", "when do we break even", "how many machines", "Table I" | BREAK EVEN |
| "full model", "investor model", "all tables", "combined", "three tabs" | COMBINED MODEL |
| "stress", "worst case", "SHA delay", "base and stressed", "scenario" | SCENARIO |
| "SHA gap", "receivables", "working capital", "cash flow timing" | SHA CASHFLOW |
| "new site", "Site 2", "new location", "another clinic" | NEW SITE |

If ambiguous, ask with AskUserQuestion (D1).

---

## Mode 1 — CAPITAL PLAN

Build a pre-startup capital requirements table for a clinic site.

### Step 1 — Gather site parameters

Ask or extract:
- Number of HD machines (default: 10)
- Location and rent benchmark (Parklands / Upperhill / other)
- Quality tier (standard / premium — premium adds private suites)
- Year of opening (for inflation adjustment — use 7% Kenya annual inflation)
- Any known variations from Site 1 (different machine supplier, smaller fit-out, etc.)

### Step 2 — Inflation-adjust if not Site 1 (2026)

```
Inflation adjustment from Site 1 (2026 base):
  Site 2 (2028): multiply all KES items by 1.145  (2 years at 7%)
  Site 3 (2030): multiply all KES items by 1.311  (4 years at 7%)
  Site N (year Y): multiply by (1.07)^(Y-2026)
USD items (machine, RO): re-quote from supplier at time of procurement.
```

### Step 3 — Output Table C format

```
TABLE C: PRE-STARTUP CAPITAL REQUIREMENTS
Site: [Name], [Location], [N] machines, [Year]
────────────────────────────────────────────────────────────────
Item                                          KES          USD
Lease deposit ([N] months × KES [X])         [X]          [X]
Civil works and medical fit-out               [X]          [X]
Private suites fit-out (×[N])                 [X]          [X]
Patient experience upgrades                   [X]          [X]
Furniture, fittings, IT hardware              [X]          [X]
Security system                               [X]          [X]
Launch branding and marketing                 [X]          [X]
Regulatory (MOH/PPB/KMPDB/NEMA/SHA)          [X]          [X]
Professional facilitation and agent fees      [X]          [X]
[N] HD machines, landed (CIF + duties)        [X]          [X]
RO water treatment system, landed             [X]          [X]
Generator ([kVA] diesel)                      [X]          [X]
NantoHealthOS deployment                      [X]          [X]
Contingency (10%)                             [X]          [X]
────────────────────────────────────────────────────────────────
TOTAL PRE-STARTUP                             KES [X]      USD [X]
────────────────────────────────────────────────────────────────
Exchange rate used: KES [X] = USD 1
```

### Step 4 — Add working capital table (Table D format)

Always append a 6-month working capital table alongside the pre-startup table.
Use 6-month buffer as the default unless the user specifies otherwise.

---

## Mode 2 — REVENUE MODEL

Build a multi-year revenue projection for a clinic site.

### Step 1 — Confirm parameters

Ask or use defaults:
- Number of standard stations and private suites
- Ramp schedule (when does the site reach each occupancy level)
- Payer mix (use defaults from constants unless overriding)
- SHA rate (use KES 16,000 unless the user has a newer tariff)

### Step 2 — Calculate blended session rate

```
BLENDED SESSION RATE CALCULATION

Standard station (Year 1 mix: 70% SHA / 30% private):
  (0.70 × KES 16,000) + (0.30 × KES 18,500) = KES [X]

Standard station (Year 2+ mix: 65% SHA / 35% private):
  (0.65 × KES 16,000) + (0.35 × KES 18,500) = KES [X]

Private suite: KES 25,000 flat (all cash-pay or private insurance)
```

### Step 3 — Output Table F format

```
TABLE F: REVENUE PROJECTIONS
Site: [Name] | Standard stations: [N] | Private suites: [N]
────────────────────────────────────────────────────────────────────────────────
Period          Std Stations  Std Occ  Suite Occ  Std Rev/Mo    Suite Rev/Mo   Total/Mo    Annual KES   Annual USD
────────────────────────────────────────────────────────────────────────────────
Year 1 H1 (ramp)    [N]       [X]%     [X]%       KES [X]       KES [X]       KES [X]       —            —
Year 1 H2           [N]       [X]%     [X]%       KES [X]       KES [X]       KES [X]       —            —
Year 1 TOTAL                                                                               KES [X]      USD [X]
Year 2 (full ops)   [N]       [X]%     [X]%       KES [X]       KES [X]       KES [X]      KES [X]      USD [X]
Year 3 (optimised)  [N]       [X]%     [X]%       KES [X]       KES [X]       KES [X]      KES [X]      USD [X]
────────────────────────────────────────────────────────────────────────────────
Note: Diaspora / holiday dialysis revenue captured in private suite occupancy rate.
Nanto Med supply margin on clinic consumables is shown separately in the combined model.
```

---

## Mode 3 — OPERATING P&L

Build monthly operating costs and P&L at a specified capacity level.

### Step 1 — Confirm capacity point

Ask: which year / which occupancy level? (Default: Year 2, full operations)

### Step 2 — Calculate session volume

```
Sessions at full capacity (8 standard × 2 sessions × 26 days): [N] sessions/month
Sessions at [X]% occupancy: [N] × [X]% = [N] sessions/month
Private suite sessions ([N] suites × 2 × 26 × [X]% occ): [N] sessions/month
```

### Step 3 — Output Tables G and H format

```
TABLE G: MONTHLY OPERATING COSTS — [Capacity Point]
────────────────────────────────────────────────
Item                              Monthly (KES)
Staff payroll (full team)         [X]
Rent                              [X]
Electricity                       [X]
Water and sewerage                [X]
Internet and communications       [X]
Medical waste management          [X]
Machine maintenance reserve       [X]
Consumables COGS ([N] sessions    [X]
  × KES 2,800 transfer price)
Marketing and outreach            [X]
Admin and miscellaneous           [X]
────────────────────────────────────────────────
TOTAL MONTHLY OPERATING COST      KES [X]

TABLE H: MONTHLY P&L — [Capacity Point]
────────────────────────────────────────────────────────
Line                              KES          USD
Revenue ([N] sessions)            [X]          [X]
Consumables COGS                  ([X])         ([X])
GROSS PROFIT                      [X] ([X]%)   [X]
Staff payroll                     ([X])         ([X])
Rent                              ([X])         ([X])
Utilities                         ([X])         ([X])
Maintenance                       ([X])         ([X])
Waste management                  ([X])         ([X])
Marketing                         ([X])         ([X])
Admin and misc                    ([X])         ([X])
TOTAL FIXED OPEX                  ([X])         ([X])
────────────────────────────────────────────────────────
EBITDA                            [X] ([X]%)   [X]
ANNUAL EBITDA                     KES [X]      USD [X]
────────────────────────────────────────────────────────

NANTO MED SUPPLY MARGIN FROM CLINIC (shown separately):
  [N] sessions/month × KES 2,800 × [X]% margin = KES [X]/month
  Annual Nanto Med margin from clinic: KES [X] / USD [X]
```

---

## Mode 4 — BREAK EVEN

Calculate the break-even point for a clinic site.

### Step 1 — Identify fixed costs and contribution margin

```
BREAK-EVEN ANALYSIS
────────────────────────────────────────────────
Monthly fixed costs (excl. consumables):  KES [X]
Contribution margin per std session:      KES [rate] - KES 2,800 = KES [X]
Contribution margin per suite session:    KES 25,000 - KES 2,800 = KES [X]

Sessions to break even (all standard):    [fixed costs] / [contribution/session]
  = KES [X] / KES [X] = [N] sessions/month

Machines needed at 2 sessions/day, 26 days:     [N] ÷ 52 = [N] machines at 100% occ
Machines needed at [X]% occupancy:              [N] machines
────────────────────────────────────────────────
BREAK-EVEN POINT:  [N] active machines at [X]% occupancy
EXPECTED MONTH:    Month [N] of operations  ([based on ramp schedule])
WORKING CAPITAL COVERS UNTIL: Month 6 (6-month buffer)
```

### Step 2 — Flag coverage gap

If break-even month > 6:
**FLAG:** "The 6-month working capital buffer does not cover the break-even period. An additional KES [X] of working capital is required. Consider: increasing the private-pay mix, accelerating ramp schedule, or extending the working capital buffer in the raise."

---

## Mode 5 — COMBINED MODEL

Build the full three-tab investor model: Capital Requirements, Revenue and P&L, Combined Raise Summary.

### Step 1 — Read the NKC document for current figures

```bash
cat "$NKC_DOC" 2>/dev/null | grep -A 200 "Table C" | head -100 || echo "Using constants"
```

### Step 2 — Produce all three tabs in sequence

**Tab 1: Capital Requirements**
Run Mode 1 for Site 1. Then append the strategic additions from constants.

**Tab 2: Revenue and P&L (Years 1-3)**
Run Mode 2 (revenue) then Mode 3 (P&L) for Years 1, 2, and 3.
Always show two columns: Base Case and Stressed Case (see Mode 6 for stressed assumptions).

**Tab 3: Combined Raise Summary**

```
TAB 3: COMBINED RAISE SUMMARY
────────────────────────────────────────────────────────────────
Entity                                   USD            KES
Nanto Med supply operations              325,000        42,250,000
Nanto Kidney Care Site 1 — pre-startup   258,123        33,556,000
Nanto Kidney Care Site 1 — working cap   95,908         12,468,000
R&D (NantoHealthOS + Fleet IoT)          50,000          6,500,000
Marketing and brand                      35,000          4,550,000
Sector elevation (KRA fellowship etc.)   20,000          2,600,000
Additional travel (clinic scouting)      20,000          2,600,000
Senior Kenya hire (Ops Director)         25,000          3,250,000
Clinical knowledge and research          15,000          1,950,000
────────────────────────────────────────────────────────────────
TOTAL RAISE                              843,031        109,594,000
ROUNDED RAISE                            850,000        110,500,000
────────────────────────────────────────────────────────────────
VALUATION
  Pre-money:                             USD 4,000,000
  Post-money:                            USD 4,850,000
  Investor stake (18%):                  USD 850,000 / USD 4,850,000 = 17.5%
  Rounded offer:                         18%
────────────────────────────────────────────────────────────────
Exchange rate: KES 130 = USD 1
Scholar review (Brad, Houston): required before any agreement is signed.
```

### Step 3 — Save output

Write to:
```bash
OUTPUT="$HOME/Desktop/NKC-InvestorModel-$(date +%Y%m%d).md"
```

Then direct the user: "Pass this file to /make-pdf to produce the investor-facing PDF."

---

## Mode 6 — SCENARIO (Base vs Stressed)

Compare a base case and a stressed case side by side.

### Stressed Case Assumptions (apply when running stressed scenario)

| Parameter | Base Case | Stressed Case | Rationale |
|---|---|---|---|
| SHA payment delay | 4 months | 6 months | Documented worst case in Kenya |
| Year 1 H1 occupancy (standard) | 55% | 40% | Slower patient ramp |
| Year 1 H2 occupancy (standard) | 70% | 55% | Dr. Twahir referrals take longer |
| Private suite occupancy (Year 1) | 40% | 25% | Diaspora demand slower to build |
| SHA reimbursement rate | KES 16,000 | KES 14,500 | Rate cut or tariff revision risk |
| Consumables COGS | KES 2,800 | KES 3,100 | Supplier price increase or KES depreciation |
| Break-even month | Month 4-6 | Month 7-9 | Later ramp |

### Output format

```
SCENARIO COMPARISON — [Site Name]
Date: DD Month YYYY
════════════════════════════════════════════════════════════════
                              BASE CASE          STRESSED CASE
────────────────────────────────────────────────────────────────
Year 1 Revenue                KES [X]            KES [X]
Year 1 EBITDA                 KES [X] ([X]%)     KES [X] ([X]%)
Year 2 Revenue                KES [X]            KES [X]
Year 2 EBITDA                 KES [X] ([X]%)     KES [X] ([X]%)
Break-even month              Month [N]          Month [N]
6-month WC buffer adequate?   YES                [YES/NO]
SHA gap at peak (Year 1)      KES [X]            KES [X]
Additional WC needed          KES 0              KES [X]
════════════════════════════════════════════════════════════════
VERDICT: [One sentence — does the business survive the stressed case?]
MITIGATION: [If stressed case breaks the model, what is the fix?]
  → Private pay mix adjustment: [what change resolves the gap?]
  → Working capital extension: [how much additional buffer is needed?]
  → Revenue acceleration: [what action closes the gap?]
```

---

## Mode 7 — SHA CASHFLOW

Model the SHA receivables gap and its impact on working capital.

### Step 1 — Calculate SHA revenue at risk

```
SHA RECEIVABLES WORKING CAPITAL MODEL
────────────────────────────────────────────────────────────────
Annual SHA revenue (Year [N]):        KES [X]  (SHA% × total revenue)
Monthly SHA revenue:                  KES [X]
SHA payment delay assumed:            [N] months
SHA receivables outstanding at peak:  KES [X]  (monthly × delay months)

6-month working capital buffer:       KES 12,468,000
SHA receivables at peak:              KES [X]
────────────────────────────────────────────────────────────────
CASHFLOW GAP:                         KES [X]  (if gap > 0, flag it)
````

### Step 2 — Sources to close the gap

```
SOURCES TO CLOSE THE SHA GAP:
  1. Cash-pay revenue (private + suite): KES [X]/month accumulates from Month 1
  2. Nanto Med supply revenue (faster cash cycle than SHA): KES [X]/month
  3. Founder management fee deferred until SHA normalises
  4. If gap remains: extend working capital buffer in raise by KES [X]
```

### Step 3 — NantoHealthOS impact

Always note: "NantoHealthOS same-day claim submission and automated rejection
follow-up is expected to reduce effective SHA delay from 4 months to 3 months.
This is modelled in the base case. The stressed case uses 6 months."

---

## Mode 8 — NEW SITE

Build a financial model for a future clinic site (Site 2, 3, etc.).

### Step 1 — Ask for site parameters

```
D1 — New site parameters
Context: Building a financial model for a new Nanto Kidney Care site.
ELI10: I need the site details to adjust the model for location, scale, and year.
Stakes if wrong: Wrong inputs produce a misleading model for a major capital decision.
Recommendation: Use Site 1 as the baseline and adjust for differences only.
Options:
A) Start from Site 1 baseline and adjust (recommended — consistent comparison)
B) Build from scratch with all new inputs
```

Ask:
- Site name and location (city, area)
- Number of machines
- Number of private suites
- Planned opening year
- Any known differences from Site 1 (different machine supplier, co-location with hospital, etc.)
- Lease rate in target location (if known)

### Step 2 — Inflation-adjust and model

Apply the inflation adjustment formula from Mode 1 (7% Kenya CPI per year from 2026 base).
Re-run Modes 1, 2, 3, 4 for the new site parameters.

### Step 3 — Add to combined portfolio view

```
NANTO KIDNEY CARE — PORTFOLIO MODEL
════════════════════════════════════════════════════════════════
                   Site 1 (Nairobi)   Site 2 ([Location])   COMBINED
Opening year       2026/2027           [Year]
Machines           10                  [N]
Pre-startup (USD)  258,123             [X]
Year 2 Revenue     USD 719,447         [X]                   [X]
Year 2 EBITDA      USD 461,835         [X]                   [X]
════════════════════════════════════════════════════════════════
```

---

## Output Rules

1. **No em dashes.** Use a comma, colon, or restructure.
2. **Kenya English.** "Rate card" not "pricing sheet". "Revert" is acceptable.
3. **Dates:** DD Month YYYY format.
4. **Always show both KES and USD.** Exchange rate: KES 130 = USD 1 (note if rate used differs).
5. **Always show base case and stressed case** in any investor-facing output. State the stressed assumptions explicitly.
6. **State risks early.** If the model shows a gap, flag it in the first paragraph, not at the end.
7. **Nanto Med supply margin from clinic is always shown separately.** It is additional EBITDA, not included in clinic EBITDA. This avoids double-counting in the combined model.
8. **Round USD figures to whole dollars. Round KES to nearest 1,000.**
9. **Scholar reminder on every output.** Append: "Scholar review (Brad, Houston) is required before this model is presented as part of a binding investment proposal."

---

## Coupling Rules

1. **Source data comes from the NKC document.** Always attempt to read the current NKC
   document before using constants. Constants are the fallback, not the primary source.
2. **Output goes to /make-pdf.** This skill produces markdown tables. It does not produce
   PDFs. After building the model, tell the user: "Pass this file to /make-pdf."
3. **Investment structure is /nanto-investor.** If the user asks about equity percentages,
   term sheets, or profit-sharing ratios, route to /nanto-investor.
4. **Strategy questions go to /warroom.** If the user asks "should we open Site 2?",
   that is a strategic decision, not a modelling task. Build the model; let /warroom decide.
5. **Actuals come from /nanto-finance.** Once the clinic is operational, replace projection
   inputs with actuals from Zoho Books. Ask the user to run /nanto-finance first if
   actual figures are available.

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
