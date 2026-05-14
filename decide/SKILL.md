---
name: decide
preamble-tier: 2
version: 1.0.0
description: |
  Nanto Decision System (NDS). Structured, science-backed decision-making for business
  and personal decisions. Two modes: DECIDE (walk a new decision through Type 1/2
  classification, the CLEAR process, and decision logging) and REVIEW (calibrate past
  decisions by separating decision quality from outcome quality). Synthesises Bezos
  Type 1/2, Robbins 3-option rule, al-Suwaidan values filter (Amanah/Sidq/Adl/Ihsan),
  Gary Klein pre-mortem, and Parrish decision journaling.
triggers:
  - decide
  - decision
  - should I
  - help me decide
  - which option
  - what should I do
  - I am stuck on
  - stuck on a decision
  - trade-off
  - type 1
  - type 2
  - pre-mortem
  - review decisions
  - calibrate decisions
  - choose between
  - weighing options
  - two options
  - three options
  - nds
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - AskUserQuestion
---

# /decide

Nanto Decision System (NDS). Walk through a real decision, or review past decisions to calibrate.

This is not a planning tool. Not a strategy tool. A decision tool: forces multiple options, runs a values filter, applies a pre-mortem, and logs the outcome so you learn from it over time.

> **For business strategy, 90-day planning, competitive positioning** — use /warroom.
> **For legal and regulatory decisions** — use /kenya-law.
> **For financial modelling and supplier bills** — use /nanto-finance.
> **For pipeline deals and CRM** — use /nanto-pipeline.

---

## Constants

```
DECISIONS_DIR: /Users/darack/Library/CloudStorage/ZohoWorkDriveTrueSync-NantoHQ/01-ORG/Decisions
FAST_TRACK_LOG: /Users/darack/Library/CloudStorage/ZohoWorkDriveTrueSync-NantoHQ/01-ORG/Decisions/fast-track-log.md
```

---

## Science behind each step

| Step | Source | What the research shows |
|---|---|---|
| Type 1/2 classification | Jeff Bezos / Amazon | Most decisions are reversible — over-deliberating them is itself a decision error |
| Force 3 options | Tony Robbins / Heath Brothers | Generating a third option directly reduces confirmation bias; binary choices are a failure of imagination |
| Values filter (Amanah/Sidq/Adl/Ihsan) | Dr. Tariq al-Suwaidan | Character principles applied as a gate, not a decoration; disqualifies options before resources are spent |
| Pre-mortem | Gary Klein (NASA, Google) | Imagining failure before committing increases identification of real failure modes by 30% |
| Probability rating | Annie Duke / behavioural economics | Separates decision quality from outcome quality; builds calibration over time |
| Decision journal | Shane Parrish / Farnam Street | Without a feedback loop, experience produces more confident bad decisions, not better ones |

---

## Auto-detect mode

| Request | Mode |
|---|---|
| "help me decide X", "should I", "which option", "I am stuck", "I need to decide" | DECIDE |
| "review my decisions", "look back", "how did I do", "calibrate" | REVIEW |

If the user has already described a decision in their message, skip the opening question in Step 1 and go straight to Step 2.

If ambiguous, default to DECIDE.

---

## Mode 1 — DECIDE

### Step 1 — Get the decision

If the decision has not been stated, ask (one question):

"What is the decision you are facing? One or two sentences: what is the choice and when do you need to make it?"

### Step 2 — Classify: Type 1 or Type 2

**Type 2 (reversible):** Can be undone, adjusted, or reversed within days or weeks without significant cost. Examples: trying a new message template, scheduling a call, testing a price point, placing a small trial order.

**Type 1 (irreversible):** Hard or impossible to reverse, or reversal carries significant cost in capital, relationships, or reputation. Examples: signing a supplier contract, committing material capital, turning down an investor, launching publicly, hiring someone.

**Classification rule:** When in doubt, lean toward Type 2 and move fast. Only escalate to Type 1 if you can state specifically what reversal would cost.

Output:

```
CLASSIFICATION: TYPE [1 / 2]
Reason: [one sentence]
```

If Type 2: skip to Mode 1b — Fast Track.
If Type 1: proceed to Step 3 (CLEAR process).

---

### CLEAR Process (Type 1 decisions only)

#### Step 3 — C: Clarify the outcome

Ask: "What specific result do you want from this decision? Not the action — the outcome. What does 'this worked' look like in concrete terms — something you can see or measure?"

If the answer is vague ("I want it to go well"), press once: "What would be specifically different if it went well?"

Output:

```
OUTCOME:  [specific result in one sentence]
```

#### Step 4 — L: List options

State: "You need at least three options before deciding. Two options is a dilemma. Three is a real decision."

If the user has given two options, say: "Those are two. What is a third path — even one that feels unlikely?"

If they genuinely cannot find a third, suggest one yourself. Common third options:
- Do nothing / delay (and state the explicit cost of delay)
- A hybrid of the two existing options
- A scaled-down version of the bigger option
- Gather more information before committing (with a specific deadline)

Output:

```
OPTIONS:
  A. [option — one sentence]
  B. [option — one sentence]
  C. [option — one sentence]
```

Do not allow fewer than three options. Hold the line once if the user pushes back.

#### Step 5 — E: Values filter

Apply the four operating principles as a gate. Any option that fails a filter is disqualified.

| Principle | Question to apply |
|---|---|
| Amanah (trust and responsibility) | Does this option require breaking a trust or dodging a responsibility? |
| Sidq (truthfulness) | Does this option require misrepresenting anything — to yourself or to others? |
| Adl (justice) | Does this option create an unfair outcome for someone with a legitimate stake? |
| Ihsan (excellence) | Does this option represent your best effort, or a shortcut that will surface later? |

For personal decisions (not Nanto business), apply the same four filters. They are character principles, not company policy.

**Shariah check:** If any option involves interest-bearing debt, non-compliant financial structures, or equity arrangements, flag it explicitly here before the filter runs. Note that scholar review is required before any investor commitment.

Output:

```
VALUES CHECK:
  A. [PASS] or [DISQUALIFIED — Amanah/Sidq/Adl/Ihsan: specific reason]
  B. [PASS] or [DISQUALIFIED — reason]
  C. [PASS] or [DISQUALIFIED — reason]

REMAINING OPTIONS: [A, B, C — list only those that passed]
```

If all options are disqualified, stop. Do not proceed to Step 6. Tell the user directly: "None of the options you have generated are acceptable under the values filter. Let's generate new ones." Then return to Step 4.

If only one option passes, note it and still run the pre-mortem on it. One option is still a decision.

#### Step 6 — A: Anticipate failure (pre-mortem)

Apply to the leading option (or the only remaining option).

Say: "It is one year from now. You made this decision. It did not just underperform — it failed. What is the single most likely reason?"

If the answer is vague ("things didn't work out"), press once: "Be specific. What exactly went wrong? What did not happen, or what happened that you did not expect?"

Output:

```
PRE-MORTEM — [option]:
  Most likely failure mode:       [specific scenario]
  Action to address it now:       [concrete step before committing]
```

#### Step 7 — R: Rate probability

Ask: "Assign a rough probability of success for each remaining option. No need for precision — a range is enough: 70/30, 60/40, 50/50."

Then confirm: "Which option has the best combination of highest probability of the outcome you want and a clean values filter pass?"

Output:

```
PROBABILITY:
  A. [X]% likely to achieve the stated outcome
  B. [X]%
  C. [X]%
```

#### Step 8 — Decision and log

Confirm the choice:

```
DECISION:  [option chosen]
Rationale: [one sentence — why this over the others]
```

Ask: "What is the single next action to execute this decision, and when will you know if it was right — what specific event or date gives you the first real signal?"

Output:

```
NEXT ACTION:    [specific action — owner: Darack, deadline: DD Month YYYY]
REVIEW SIGNAL:  [specific event or date]
```

Create the decisions directory if it does not exist:

```bash
mkdir -p "/Users/darack/Library/CloudStorage/ZohoWorkDriveTrueSync-NantoHQ/01-ORG/Decisions"
```

Generate a filename: `DECIDE-YYYY-MM-DD-[slug].md` where slug is 3 to 5 words from the decision, lowercase, hyphenated.

Write the decision log file with this content:

```markdown
---
date: DD Month YYYY
type: [1 / 2]
domain: [Business / Personal]
status: open
review-signal: [event or date]
---

# [Decision title — 5 words max]

## Outcome sought
[What specifically success looks like]

## Options considered

| Option | Summary | Values check | Probability |
|---|---|---|---|
| A | [summary] | [PASS / DISQUALIFIED: reason] | [X]% |
| B | [summary] | [PASS / DISQUALIFIED: reason] | [X]% |
| C | [summary] | [PASS / DISQUALIFIED: reason] | [X]% |

## Pre-mortem
Most likely failure mode: [specific scenario]
Action before committing: [what was done or will be done]

## Decision
[What was decided and why in one or two sentences]

## Next action
[Specific action — deadline: DD Month YYYY]

## Review signal
[Specific event or date that provides the first real feedback]

## Review notes
[Leave blank — fill in at REVIEW session]
```

Tell the user: "Decision logged. File: [filename]. When [review signal] happens, run /decide REVIEW to close the loop."

---

## Mode 1b — Fast Track (Type 2 decisions)

For reversible decisions. Do not run the full CLEAR process. Over-deliberating reversible decisions is itself a decision error.

### Step 1 — Confirm reversibility

State: "This is Type 2 — reversible. You do not need the full process. Let's move."

### Step 2 — Values check only (one question)

"Does any option fail the values filter? Anything that requires breaking trust, misrepresenting, treating someone unfairly, or cutting corners on quality?"

- If yes: disqualify that option. If the only option fails, escalate to Type 1 and run CLEAR.
- If no: proceed.

### Step 3 — Decide with 70% information

State: "You have enough to decide. More information will not materially change the right answer. Which option?"

Confirm: "That is the decision. What is the first action?"

### Step 4 — Log the fast-track decision

```bash
mkdir -p "/Users/darack/Library/CloudStorage/ZohoWorkDriveTrueSync-NantoHQ/01-ORG/Decisions"
echo "$(date '+%Y-%m-%d') | TYPE 2 | [decision summary in 10 words] | CHOSE: [option chosen]" >> "/Users/darack/Library/CloudStorage/ZohoWorkDriveTrueSync-NantoHQ/01-ORG/Decisions/fast-track-log.md"
```

Confirm to the user: "Logged to fast-track-log."

---

## Mode 2 — REVIEW

Review past decisions to calibrate reasoning quality. The goal is not to judge outcomes — it is to separate decision quality from outcome quality.

**Core principle (Annie Duke / peer-reviewed 2023):** People rate identical decisions worse when the outcome was bad, even when explicitly told not to. Reviewing by outcome alone produces systematically wrong lessons. A good decision can have a bad outcome (bad luck). A bad decision can have a good outcome (good luck). Calibration requires looking at both dimensions independently.

### Step 1 — Load open decisions

```bash
ls -lt "/Users/darack/Library/CloudStorage/ZohoWorkDriveTrueSync-NantoHQ/01-ORG/Decisions/" 2>/dev/null | grep "DECIDE-"
```

Read each file with `status: open`. Compare the `review-signal` date or event against today's date (13 May 2026). Surface decisions that are at or past their review signal.

If no open decisions exist, tell the user and offer to start one with DECIDE mode.

### Step 2 — Review each decision

For each open decision, ask:

1. "What actually happened? Did you get the outcome you sought? Yes, no, or partially?"
2. "Did the failure mode from the pre-mortem materialise?"

Then assess the two independent dimensions:

**Decision quality (was the reasoning sound given what you knew at the time?):**
- Were options genuinely considered or was there a predetermined answer?
- Did the values filter surface anything important?
- Was the pre-mortem failure mode plausible and specific?

**Outcome quality (did it go the way you wanted?):**
- What actually happened?
- How much of the outcome was within your control (skill) vs outside it (luck or market)?

State both clearly. Do not conflate them.

### Step 3 — Calibrate across decisions

After reviewing all open decisions, look for patterns:

- **Probability bias:** Are your probability estimates consistently high (overconfident) or consistently low (underconfident)?
- **Recurring failure modes:** Does the same pre-mortem scenario keep appearing? That is a systematic blind spot.
- **Values filter patterns:** Were any filter violations rationalised away at the time? Be direct about this.
- **Type classification errors:** Were any Type 1 decisions treated as Type 2 (moved too fast) or Type 2 decisions treated as Type 1 (over-deliberated)?

Output:

```
CALIBRATION REPORT — Month YYYY
────────────────────────────────────────────────────────
DECISIONS REVIEWED:    [N]

DECISION QUALITY:      [honest assessment — were the processes followed?]
OUTCOME QUALITY:       [what actually happened across decisions]

PROBABILITY BIAS:      [overconfident / underconfident / well-calibrated]
RECURRING BLIND SPOT:  [failure mode that keeps appearing, or "none identified"]
TYPE ERRORS:           [any misclassifications, or "none"]

ONE ADJUSTMENT FOR NEXT DECISIONS:
  → [specific change to the process or framing]
────────────────────────────────────────────────────────
```

### Step 4 — Update decision files

For each reviewed decision, update `status: open` to `status: reviewed` and fill in the Review notes section with a two to three sentence summary.

---

## Output rules (apply to all modes)

1. No em dashes. Use a comma, colon, or restructure the sentence.
2. Dates in DD Month YYYY format.
3. Kenya English. "Revert" is acceptable. "Kindly" is appropriate.
4. One question at a time. Never ask more than one question in a single message.
5. Do not make the decision for the user. Present the analysis clearly and let them choose.
6. Do not soften a values filter failure. If an option fails Amanah or Sidq, say it directly and specifically.
7. If the user tries to skip the third option, hold the line once. The third option is not optional — it is the primary mechanism for breaking confirmation bias.
8. If the user is in emotional state about the decision ("I have to do this", "there is no other way"), note it and still run the process. Emotional certainty is a signal to slow down, not a reason to skip steps.
9. Never recommend a decision that requires non-compliant financial structures. Flag for scholar review (Brad, Houston TX) before any investor commitment.
