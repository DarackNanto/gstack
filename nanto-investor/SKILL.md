---
name: nanto-investor
preamble-tier: 3
version: 1.0.0
description: |
  Nanto investor relations and Islamic finance structuring. Term sheet drafting
  (Musharakah, tranche structures, buyback call options), cap table management,
  quarterly investor update reports, due diligence package assembly, scholar
  review coordination (Brad, Houston), and profit-sharing distribution
  calculations. Covers Nanto Ventures Limited equity structure across all
  divisions. Boundary: legal review is /kenya-law; bookkeeping is /nanto-finance;
  investor communication drafting is /nanto-comms; financial projections are
  /nanto-model (future). (Nanto/gstack)
triggers:
  - investor
  - term sheet
  - cap table
  - Megna
  - Musharakah
  - scholar review
  - due diligence
  - investor update
  - profit sharing
  - distribution
  - equity structure
  - Shariah structure
  - raise
  - fundraising
  - investor relations
  - investor package
  - tranche
  - buyback
  - board seat
  - valuation
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - AskUserQuestion
---

# /nanto-investor

Nanto's investor relations layer. Structures, tracks, and reports on all equity
relationships in Nanto Ventures Limited. Boundary: WHAT the deal is, WHO holds
what, and HOW distributions and obligations flow.

> **Not for legal review of contracts** — that is /kenya-law.
> **Not for day-to-day bookkeeping** — that is /nanto-finance (Zoho Books).
> **Not for drafting investor emails or WhatsApp messages** — that is /nanto-comms.
> **Not for multi-year revenue projections** — that is /nanto-model (future skill).

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
  echo '{"skill":"nanto-investor","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'"}'  \
    >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
fi
echo "INVESTOR SKILL v1.0.0 READY"
```

---

## Constants

```
WORKDRIVE:      /Users/darack/Library/CloudStorage/ZohoWorkDriveTrueSync-NantoHQ
NKC_DOCUMENT:  $WORKDRIVE/10-PROJECTS/MED/01-Strategy-and-Fundraising/Business-Plan/
               MED-DRAFT-NantoKidneyCare-StrategyAndFinancialModel-v0.1.md

EXCHANGE_RATE:  KES 130 = USD 1  (update if CBK rate changes materially)
SCHOLAR:        Brad, Houston, Texas, USA  (Shariah review, Islamic finance)

CURRENT RAISE (as of 14 May 2026):
  Total raise:      USD 850,000
  Pre-money:        USD 4.0M
  Post-money:       USD 4.85M
  Opening offer:    18% of Nanto Ventures Limited
  Negotiating floor: 22%  (do not recommend going above this without founder approval)
  Walk-away:        25%
  Structure:        Musharakah (Islamic equity partnership)
  Tranche 1:        USD 500,000 → 12% equity (at signing)
  Tranche 2:        USD 350,000 → 6% equity (at clinic fit-out complete, or Month 6)

CURRENT CAP TABLE (Nanto Ventures Limited):
  Darack Nanto (Founder):   82%  (post-Megna, post-Dr. Twahir NKC stake)
  Megna Home (target):      18%  (opening offer — subject to negotiation)
  Dr. Twahir:               10%  (in Nanto Kidney Care entity ONLY, not holding co.)
  Note: Dr. Twahir's 10% is in the NKC clinical entity. He does not hold equity in
  Nanto Ventures Limited unless he co-invests alongside Megna.

QARD AL-HASSAN (kept separate from Musharakah pool):
  Commitment 1:   USD 5,000  (separate from investor raise)
  Commitment 2:   USD 5,000  (separate from investor raise)

SHARIAH REQUIREMENT:
  Scholar review (Brad, Houston) is REQUIRED before any agreement is signed.
  No exceptions. Flag this in every term sheet and investor update.

PROFIT SHARING (Musharakah):
  Profit ratio is negotiable independently of ownership ratio.
  Proposed: Year 1-2: Megna receives 14% of net profit (below ownership %)
            Year 3+:  Megna receives 18% of net profit (matches ownership %)
  Rationale: Reduced early distribution protects working capital during SHA delay period.
  Losses are shared in proportion to capital contributed (Musharakah rule).
```

---

## Auto-detect mode from request

Read the user's request and route to the correct mode:

| Request keywords | Mode |
|---|---|
| "term sheet", "draft agreement", "tranche", "structure the deal" | TERM SHEET |
| "cap table", "who owns what", "equity split", "dilution" | CAP TABLE |
| "investor update", "quarterly report", "how do I report to Megna" | INVESTOR UPDATE |
| "due diligence", "what documents", "proof package", "data room" | DUE DILIGENCE |
| "scholar review", "Brad", "Shariah", "halal", "Musharakah review" | SCHOLAR REVIEW |
| "distribution", "profit share", "how much do they receive", "payout" | DISTRIBUTION |

If ambiguous, ask with AskUserQuestion (D1).

---

## Mode 1 — TERM SHEET

Draft or review an equity term sheet for a Musharakah investment in Nanto Ventures Limited.

### Step 1 — Load current deal context

Read the NKC document:
```bash
cat "$WORKDRIVE/10-PROJECTS/MED/01-Strategy-and-Fundraising/Business-Plan/MED-DRAFT-NantoKidneyCare-StrategyAndFinancialModel-v0.1.md" 2>/dev/null | head -200 || echo "FILE NOT FOUND — ask user for current deal terms"
```

### Step 2 — Extract deal parameters

Confirm from context or ask (D1):
- Investor name and entity type (individual / company / family office)
- Investment amount (total and tranche split)
- Equity percentage offered
- Investment horizon (years)
- Board representation requested (observer / voting / none)
- Profit sharing ratio Year 1-2 vs Year 3+
- Buyback call option terms (year, valuation formula)

### Step 3 — Draft the term sheet

```
TERM SHEET — MUSHARAKAH EQUITY INVESTMENT
Nanto Ventures Limited
Date: DD Month YYYY
────────────────────────────────────────────────────────
PARTIES
  Rabb-ul-Mal (Investor):  [Investor Name / Entity]
  Mudarib (Manager):       Darack Nanto, Nanto Ventures Limited

STRUCTURE
  Type:                    Musharakah (Declining, or Fixed — specify)
  Shariah Compliance:      Subject to scholar review by [Brad, Houston, TX]
                           No agreement binds until scholar approval received.

INVESTMENT TERMS
  Total Investment:        USD [amount]
  Tranche 1:               USD [amount] at signing → [X]% equity
  Tranche 2:               USD [amount] at [milestone] → [X]% equity
  Total Equity:            [X]% of Nanto Ventures Limited (post-investment)
  Pre-money Valuation:     USD [amount]
  Post-money Valuation:    USD [amount]

WHAT THE EQUITY COVERS
  Nanto Ventures Limited (holding company) owns:
  - Nanto Med (supply: spare parts, consumables, machines)
  - Nanto HealthOS (technology: NantoHealthOS, fleet IoT)
  - Nanto Kidney Care Site 1 (clinical entity, Nairobi)
  - All future divisions activated under Nanto Ventures Limited

PROFIT SHARING (MUSHARAKAH)
  Year 1-2:               [X]% of net distributable profit
  Year 3+:                [X]% of net distributable profit (matches equity %)
  Basis:                  Annual audited net profit, after tax and reserves
  Distribution:           Annual, within 90 days of financial year end
  Loss sharing:           Pro-rata to capital contributed

GOVERNANCE
  Board seat:             [Observer / Voting / None]
  Reporting:              Quarterly financial report (P&L, SHA receivables, cash position)
  Annual review:          Strategic review meeting, DD Month YYYY each year
  Operational veto:       None. Founder retains full operational authority.
  Drag-along:             None. Founder does not grant drag-along rights.

BUYBACK (CALL OPTION)
  Founder call option:    Nanto holds right to repurchase investor's stake after Year [N]
  Investor put option:    Investor may trigger sale from Year [N+1]
  Valuation formula:      [X] × trailing 12-month EBITDA at time of exercise
  Notice period:          90 days written notice to exercise either option

INVESTOR PROTECTIONS
  First right of participation in future fundraising rounds at same or better terms.
  Access to quarterly financial reports.
  Annual strategic review meeting.

WHAT THE INVESTOR DOES NOT RECEIVE
  Operational veto rights over day-to-day decisions.
  Co-management or signing authority.
  Drag-along rights.
  Access to supplier pricing or clinical patient data.

PROHIBITED INSTRUMENTS (SHARIAH)
  No interest-bearing debt (riba).
  No guaranteed returns regardless of profit.
  No short-selling of Nanto equity.

GOVERNING LAW
  Republic of Kenya. Disputes resolved under Kenyan commercial arbitration
  before litigation. Islamic finance scholar opinion is advisory, not binding,
  unless both parties agree otherwise.

CONDITIONS PRECEDENT
  1. Scholar review completed and Musharakah structure approved.
  2. Legal review by investor's Kenyan legal counsel.
  3. Nanto Ventures Limited certificate of incorporation provided.
  4. KYC documentation exchanged.
────────────────────────────────────────────────────────
NEXT STEP: This is a heads-of-terms, not a binding agreement.
Full legal documentation follows after both parties sign this sheet.
Send to /kenya-law for legal review before presenting to investor.
```

### Step 4 — Shariah flag

Always append:

> **Scholar review required.** This term sheet contains an Islamic finance structure (Musharakah). No binding agreement may be signed until Brad (Houston, TX) has reviewed and approved the structure. Allow 2-4 weeks for scholar review. Do not present this as a final term sheet until that approval is received.

---

## Mode 2 — CAP TABLE

Track and update the equity ownership of Nanto Ventures Limited.

### Step 1 — Load current cap table from constants

Use the CURRENT CAP TABLE in the constants section as the baseline.

### Step 2 — Apply any changes the user describes

Model the following scenarios on request:
- New investor at [X]% dilutes all existing holders proportionally
- Tranche 2 not funded: Tranche 1 investor stays at Tranche 1 equity only
- Dr. Twahir co-invests alongside Megna: portion of the 18% goes to him
- Future round at higher valuation: model dilution to all current holders

### Step 3 — Output the cap table

```
CAP TABLE — Nanto Ventures Limited
Date: DD Month YYYY
────────────────────────────────────────────────────────
Shareholder               Shares (%)    Notes
Darack Nanto (Founder)      82%         Operational control. No dilution below majority
                                        without explicit founder consent.
[Investor Name]             18%         Musharakah, [date]. Tranche 1 [X]%, Tranche 2 [X]%.
                                        Scholar review: [pending / approved / date].
────────────────────────────────────────────────────────
TOTAL                      100%

SEPARATE — NANTO KIDNEY CARE (clinical entity only, NOT in holding company):
Dr. Mohammed Twahir         10%         Founding Clinical Partner. NKC entity only.
                                        Buyback call option: Year 4, 6× EBITDA formula.
Nanto Ventures Limited      90%         Holding company stake in NKC clinical entity.
────────────────────────────────────────────────────────
NOTE: Qard al-Hassan commitments (2 × USD 5,000) are separate from the Musharakah
pool and do not appear on the equity cap table.
```

### Step 4 — Dilution check

If the founder would drop below 51% under the proposed scenario:
**FLAG IMMEDIATELY.** "This scenario reduces the founder below majority control. This violates the non-negotiable terms in the investment framework. Do not proceed without explicit founder review."

---

## Mode 3 — INVESTOR UPDATE

Generate a quarterly investor report for Megna Home (or any Musharakah partner).

### Step 1 — Gather financial data

Ask the user to provide or point to:
- P&L for the quarter (revenue, gross profit, EBITDA)
- Cash position at quarter end
- SHA receivables aging (what is owed, what age, how much at risk)
- Key milestones achieved vs. last quarter
- Key risks and what is being done about them

If the Nanto Kidney Care clinic is not yet open, report on Nanto Med supply operations only.

### Step 2 — Generate the report

```
INVESTOR QUARTERLY UPDATE
Nanto Ventures Limited
Quarter: Q[N] [YYYY]  |  Period: DD Month to DD Month YYYY
Prepared: DD Month YYYY
Prepared for: [Investor Name]
════════════════════════════════════════════════════════
FINANCIAL SUMMARY

Revenue (quarter):         KES [X] / USD [X]
Gross Profit:              KES [X] / USD [X]  ([X]% margin)
EBITDA:                    KES [X] / USD [X]  ([X]% margin)
Cash position (end):       KES [X] / USD [X]

SHA RECEIVABLES
  Outstanding:             KES [X] (aging: [X]% <90 days, [X]% 90-180 days)
  Disputed / rejected:     KES [X]  (being resubmitted via NantoHealthOS)
  Collected this quarter:  KES [X]

OPERATIONAL MILESTONES
  Achieved:
    + [milestone 1]
    + [milestone 2]
  Missed and reason:
    - [item]: [reason, revised date]

KEY RISKS
  1. [risk] — [current status and mitigation]
  2. [risk] — [current status and mitigation]

NEXT QUARTER FOCUS
  1. [priority 1]
  2. [priority 2]
  3. [priority 3]

PROFIT DISTRIBUTION
  This quarter distributable profit:  KES [X] / USD [X]
  [Investor]'s share ([X]%):          KES [X] / USD [X]
  Distribution date:                  [DD Month YYYY / "Deferred — see note below"]
  Note (if deferred):                 [SHA receivables gap / re-investment decision]
════════════════════════════════════════════════════════
Prepared by: Darack Nanto, Founder and CEO
Nanto Ventures Limited | KRA PIN: P052406460F
```

### Step 3 — Routing

After producing the report: "Send the written narrative to /nanto-comms for the investor email. Keep the financial tables in this format for the attachment."

---

## Mode 4 — DUE DILIGENCE

Assemble and track the due diligence document checklist for an investor.

### Step 1 — Standard Nanto due diligence package

The following documents constitute the standard proof package for the Nanto Ventures raise:

```
DUE DILIGENCE CHECKLIST — Nanto Ventures Limited
Date: DD Month YYYY
────────────────────────────────────────────────────────
CORPORATE
  [ ] Certificate of Incorporation (Nanto Ventures Limited, PVT-ZQUXALPV)
  [ ] KRA PIN certificate (P052406460F)
  [ ] Registered office confirmation (The Piano, 8th Floor, Nairobi)
  [ ] Founder ID and KYC documentation

CLINICAL PARTNERSHIP
  [ ] Dr. Twahir WhatsApp exchange screenshot (13-14 May 2026)
      "any partnership style you propose. I can provide the patients."
  [ ] Dr. Twahir foreword in Nanto patient education booklet
  [ ] Dr. Twahir KMPDB registration certificate (request from Dr. Twahir)

SUPPLY CHAIN PROOF
  [ ] CN Meditech order confirmation (930 dialyzers, USD 1,538 deposit wired)
  [ ] HEDclip Africa distribution agreement (Kristian, Denmark)
  [ ] Shubham Corporation (Brijesh, India) — correspondence or agreement
  [ ] Jihua HD machine distribution agreement draft (East Africa exclusive, 7 countries)

REGULATORY
  [ ] PPB PRIMS application status screenshot or reference number
  [ ] Dr. Alice (PPB consultant) engagement confirmation

FINANCIAL
  [ ] Nanto Kidney Care financial model (Tables C-I, investor-cleaned PDF)
  [ ] Flynn Medical Ltd 2020 business plan (Kenya market precedent)
  [ ] SHA tariff schedule (Legal Notice 146, September 2024)

MARKET EVIDENCE
  [ ] KRA World Kidney Day 2026 banner with Nanto Med logo
  [ ] CKS Dialysis Centre (Alice) gear pump purchase confirmation
  [ ] WKD HEDclip giveaway — 40+ facility responses (WhatsApp screenshot)
  [ ] NHID architecture document (summary version — internal full doc stays confidential)

SHARIAH
  [ ] Brad (Houston) scholar review status: [Scheduled / In progress / Approved]
  [ ] Musharakah structure summary (one-page, non-technical)
────────────────────────────────────────────────────────
CONFIDENTIAL: Do not share supplier pricing, clinical patient data, or full NHID
architecture with any investor. Share the summary version of NHID only.
```

### Step 2 — Track status

Update each item as: [ ] Missing | [~] In progress | [x] Ready.

Ask the user to confirm which items are ready before the investor package is sent.

### Step 3 — Package for sending

When all critical items are checked:
- Corporate + Clinical Partnership + Supply Chain + Financial = minimum viable package
- Regulatory + Market Evidence = strengthens the package
- Shariah: Brad's confirmation must be in hand or explicitly noted as "in progress, expected [date]"

Send the package assembly to /make-pdf for the financial tables. Send the cover email to /nanto-comms.

---

## Mode 5 — SCHOLAR REVIEW

Coordinate the Shariah compliance review with Brad in Houston, Texas.

### Step 1 — What needs scholar review

The following require Brad's sign-off before any investor signs:

1. The Musharakah equity structure (ownership percentages, profit-sharing ratios)
2. The tranche mechanism (is it a single Musharakah or two sequential Musharakah contributions?)
3. The buyback call option (does the pre-agreed valuation formula create a guaranteed return, which is riba?)
4. The profit-sharing Year 1-2 reduction (is a reduced ratio in early years permissible under Musharakah?)
5. The Qard al-Hassan commitments (confirm they are properly separated from the Musharakah pool)

### Step 2 — Prepare the scholar brief

```
SCHOLAR REVIEW BRIEF — Nanto Ventures Limited
Date: DD Month YYYY
For: [Brad, full name and title if known], Houston, Texas
────────────────────────────────────────────────────────
CONTEXT
  Nanto Ventures Limited is a Kenya-incorporated healthcare company
  (Registration: PVT-ZQUXALPV) raising capital to fund a dialysis
  supply operation (Nanto Med) and a dialysis clinic (Nanto Kidney Care).
  The founder is Muslim. All financial structures must be Shariah-compliant.
  The investor is [Megna Home, a Kenyan entity].

STRUCTURE FOR REVIEW

1. MUSHARAKAH EQUITY
   Investor contributes USD [X] in exchange for [X]% ownership of Nanto
   Ventures Limited. Ownership is proportional to capital contribution.
   There is no guaranteed return. Profit and loss are shared per the ratios below.

2. PROFIT SHARING
   Year 1-2: Investor receives [X]% of net annual distributable profit.
   Year 3+:  Investor receives [X]% of net annual distributable profit.
   (Ownership: [X]% throughout.)
   Question: Is a profit-sharing ratio below the ownership percentage permissible
   in a fixed Musharakah in Years 1-2? The rationale is cash-flow protection during
   the SHA receivables gap period, not profit manipulation.

3. TRANCHE STRUCTURE
   The investment is made in two tranches:
   Tranche 1: USD [X] at signing → [X]% equity
   Tranche 2: USD [X] at milestone → [X]% equity
   Question: Is this a single Musharakah with deferred second contribution, or two
   separate Musharakah contracts? Does the milestone condition create any uncertainty
   (gharar) that would affect validity?

4. BUYBACK CALL OPTION
   Founder holds a call option to repurchase investor's stake after Year [N].
   Valuation formula: [X] × trailing 12-month EBITDA at time of exercise.
   The formula is agreed at signing, not renegotiated at exit.
   Question: Does the pre-agreed formula constitute a guaranteed price (which
   would be riba)? Or is it permissible because it is a formula referencing actual
   future performance, not a fixed amount?

5. QARD AL-HASSAN
   Two separate commitments of USD 5,000 each have been received as interest-free
   loans (Qard al-Hassan). These are separate from the Musharakah pool.
   Question: Confirm the separation is sufficient for them to remain valid Qard
   al-Hassan and not be treated as part of the equity structure.

REQUESTED OUTPUT
  A one-page opinion letter stating:
  1. Whether each structure above is Shariah-compliant as described.
  2. Any modifications required to achieve compliance.
  3. Whether any structure requires further scholarly consultation.
────────────────────────────────────────────────────────
```

### Step 3 — Timeline tracking

| Item | Status | Expected |
|---|---|---|
| Brad contacted to schedule | [ ] | This week |
| Brief sent to Brad | [ ] | Within 3 days of contact |
| Scholar review received | [ ] | Allow 2-4 weeks |
| Scholar approval noted in term sheet | [ ] | Before investor signs |

---

## Mode 6 — DISTRIBUTION

Calculate and document profit-sharing distributions for a Musharakah period.

### Step 1 — Gather inputs

Ask:
- Financial year end date
- Net distributable profit for the period (after tax, after agreed reserves)
- Investor's profit-sharing ratio for this period (Year 1-2 or Year 3+)
- Cash available for distribution (may differ from accounting profit due to SHA receivables)

### Step 2 — Calculate

```
DISTRIBUTION CALCULATION — Musharakah
Period: [YYYY Financial Year / Quarter]
Date calculated: DD Month YYYY
────────────────────────────────────────────────────────
Net distributable profit:      KES [X]  (USD [X] at KES 130/USD)
Investor profit-sharing ratio: [X]%
Investor distribution amount:  KES [X]  (USD [X])
Founder distribution amount:   KES [X]  (USD [X])

Cash available for distribution: KES [X]
SHA receivables outstanding:     KES [X]  (not yet collectible)
Distributable cash this period:  KES [X]

If distributable cash < investor distribution amount:
  → Defer: KES [X] owed to investor, payable when SHA receivables clear.
  → Notify investor in writing per quarterly report.
────────────────────────────────────────────────────────
PAYMENT INSTRUCTIONS (when distributing)
  Investor bank: [on file from term sheet]
  Reference:     NVL-DIST-[YYYY]-[QN]
  Paid by:       DD Month YYYY
```

### Step 3 — Record

Save distribution record to:
```bash
DIST_DIR="$WORKDRIVE/10-PROJECTS/INVESTOR/Distributions"
mkdir -p "$DIST_DIR"
```

Write: `DIST-[YYYY]-Q[N]-[InvestorSlug].md`

---

## Output Rules (apply to all modes)

1. **No em dashes.** Use a comma, colon, or restructure.
2. **Kenya English.** "Kindly", "revert", "rate card". Formal register for all documents.
3. **Dates:** DD Month YYYY format throughout.
4. **Scholar check in every mode.** If any output involves financial structure or agreement,
   append: "Scholar review (Brad, Houston) is required before this document is presented
   as final or signed by any party."
5. **Founder control is non-negotiable.** If any scenario reduces the founder below
   majority, flag it immediately and do not model it without explicit founder approval.
6. **No guaranteed returns.** Any structure that implies a fixed return regardless of
   profit is riba. Flag it and propose a Shariah-compliant alternative.
7. **NKC entity vs holding company.** Dr. Twahir's equity is in the Nanto Kidney Care
   clinical entity only. External investors hold equity in Nanto Ventures Limited.
   Never mix these on the same cap table.

---

## Coupling Rules

1. **Legal review is /kenya-law.** Once a term sheet is drafted here, send it to /kenya-law
   for Kenya Companies Act review before presenting to any investor.
2. **Investor emails are /nanto-comms.** This skill drafts structures and documents,
   not messages. Pass the content to /nanto-comms for the investor email.
3. **Financial projections are not this skill's job.** If the user asks for revenue models
   or capital requirements, read from the NKC document or direct to direct Claude Code work.
   Do not generate financial projections inline.
4. **Due diligence documents are assembled here, not created here.** The actual financial
   model PDF is produced by /make-pdf. The cover page by /nanto-artisan. This skill tracks
   the checklist and coordinates the package.
5. **Qard al-Hassan stays separate.** Never include the two USD 5,000 commitments in any
   equity or profit-sharing calculation.

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
