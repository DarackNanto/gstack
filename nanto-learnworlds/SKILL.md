---
name: nanto-learnworlds
version: 1.0.0
description: |
  Nanto Academy LearnWorlds admin skill. Navigates LearnWorlds via headless browser
  to upload SCORM packages, manage courses, configure enrollments, and perform any
  admin task according to Nanto Academy standards. Can look up LearnWorlds support
  docs for unfamiliar tasks. Parallels nanto-finance (Zoho Books) — same philosophy
  applied to the LMS. (Nanto)
triggers:
  - upload course learnworlds
  - publish course
  - nanto learnworlds
  - learnworlds admin
  - upload scorm
  - enroll learners
  - learnworlds course
  - /nanto-learnworlds
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - WebFetch
  - WebSearch
  - AskUserQuestion
---

# /nanto-learnworlds

AI-assisted LearnWorlds admin for Nanto Academy. Uses the headless browser to
navigate the LMS and complete tasks. Always verifies against Nanto Academy standards
before making changes.

---

## Step 0 — Load library and browser

Read these files before beginning any task:

```bash
SKILL_DIR="$HOME/.claude/skills/gstack/nanto-learnworlds"
cat "$SKILL_DIR/library/LW-ACCOUNT.md"
cat "$SKILL_DIR/library/LW-STANDARDS.md"
```

Set up the browser binary:

```bash
_ROOT=$(git rev-parse --show-toplevel 2>/dev/null)
B=""
[ -n "$_ROOT" ] && [ -x "$_ROOT/.claude/skills/gstack/browse/dist/browse" ] && B="$_ROOT/.claude/skills/gstack/browse/dist/browse"
[ -z "$B" ] && B="$HOME/.claude/skills/gstack/browse/dist/browse"
echo "BROWSE: $B"
```

If `BROWSE` path does not exist or is not executable, tell the user: "Browse needs setup — run `/browse` once to build it."

---

## Step 1 — Identify the mode

| Mode | When to use |
|---|---|
| **publish** | Upload one or more SCORM zip files to an existing or new course in LearnWorlds |
| **manage** | General admin: enrollment, course visibility, categories, certificates, reports |
| **explore** | Discover an unfamiliar LearnWorlds feature by navigating the UI and/or fetching docs |
| **audit** | Review all published courses against Nanto Academy standards |

If the user's request is ambiguous, ask one question to identify the mode.

---

## Step 2 — Navigate to LearnWorlds admin

### 2.1 Load saved session

```bash
$B status
```

If browser is running, try loading the saved state:
```bash
$B state load learnworlds-admin
$B goto https://academy.nanto.com/author/dashboard
sleep 3
$B url
```

If the URL stays on `/author/dashboard` (not redirected to login), the session is active. Continue.

If redirected to login, use handoff:
```bash
$B goto https://account.learnworlds.com/login
$B handoff "Please log into LearnWorlds (account.learnworlds.com/login), then navigate to the Nanto Academy school. Let me know when you are on the admin dashboard."
```

Wait for user to confirm, then:
```bash
$B resume
$B url
$B state save learnworlds-admin
```

### 2.2 Verify you are in school admin

Expected URL after login: `https://academy.nanto.com/author/dashboard`

The admin sidebar shows: Home/Dashboard, Courses & programs, Website, Users, Communication, E-commerce, Marketing, Reports, Mobile app, Settings.

If on the master schools panel (`account.learnworlds.com/dashboard/schools`), navigate into the Nanto Academy school:
```bash
$B js "
const iframe = document.getElementById('contentHolder');
const doc = iframe.contentDocument || iframe.contentWindow.document;
const btn = doc.querySelector('.course-more-actions-btn');
" 
# Or just click Go to school from the schools dashboard
```

---

## Step 3 — Mode: PUBLISH

Upload SCORM packages for a course.

### 3.1 Identify the course

Ask if not provided:
- Course code (e.g., C017)
- Whether this is a new course or an existing course in LearnWorlds

For new courses, confirm:
- Course title (from CATALOGUE-INDEX.md)
- Client name or "Catalogue" (Nanto IP)

### 3.2 Locate the SCORM zips

```bash
COURSE_CODE="CXXX"  # replace with actual code
COURSE_LIB="/Users/darack/Library/CloudStorage/ZohoWorkDriveTrueSync-NantoHQ/10-PROJECTS/ACADEMY/COURSE-LIBRARY"
find "$COURSE_LIB" -path "*${COURSE_CODE}*/Zips/*.zip" | sort
```

Confirm the zips are present. If missing, tell the user: "Zips not found — run `/nanto-course-make` then `/nanto-course-qa` first."

### 3.3 Find or create the course in LearnWorlds

Get the course titleId from the API:
```bash
$B goto "https://academy.nanto.com/author/courses"
sleep 3
$B js "fetch('/api/author/products?fields=id,title,titleId,type&paginationType=paginated&page=1&itemsPerPage=20&type=course&sortField=created&sortDirection=desc').then(r=>r.json()).then(d=>JSON.stringify(d.data))"
```

If course already exists, note its `titleId`.

If course does not exist, create it:
```bash
$B goto "https://academy.nanto.com/author/courses"
sleep 2
# Click Create course button
$B js "
const iframe = document.getElementById('contentHolder');
const doc = iframe.contentDocument || iframe.contentWindow.document;
const createBtn = [...doc.querySelectorAll('button')].find(el => el.textContent.trim() === 'Create course');
createBtn.click();
"
sleep 2
$B screenshot /tmp/lw-create-course.png
```

Fill in: title (exact match to CATALOGUE-INDEX.md), category, language = English. Save as Draft.
After creating, get the titleId from the URL or re-run the API call.

### 3.4 Navigate to the course editor

```bash
TITLE_ID="disciplinary-action"  # replace with actual titleId
$B goto "https://academy.nanto.com/author/course?courseid=${TITLE_ID}&tab=contents"
sleep 3
$B screenshot /tmp/lw-course-editor.png
```

The course outline shows numbered sections. Each section will contain one SCORM unit.

### 3.5 Upload each SCORM zip (modules first, assessment last)

For each section that needs a SCORM unit:

**If the section already has a SCORM unit (replacing/updating):**
```bash
$B js "
const iframe = document.getElementById('contentHolder');
const doc = iframe.contentDocument || iframe.contentWindow.document;
// Open Settings for the SCORM unit (first one — loop for each section)
const settingsBtn = doc.querySelector('.edit-btn-icon-lbl.update[title=\"Settings\"]');
settingsBtn.click();
'settings opened';
"
sleep 2
```

**If the section is empty (adding a new SCORM):**
```bash
$B js "
const iframe = document.getElementById('contentHolder');
const doc = iframe.contentDocument || iframe.contentWindow.document;
const uploadLinks = [...doc.querySelectorAll('a, button, span')].filter(el => el.textContent.trim() === 'Upload file');
uploadLinks[0].click(); // click Upload file for section 01
"
sleep 2
```

In both cases, the "Update SCORM" panel opens. Now upload the zip:
```bash
# Click the "Update scorm" button to trigger file selection
$B js "
const iframe = document.getElementById('contentHolder');
const doc = iframe.contentDocument || iframe.contentWindow.document;
const uploadBtn = doc.getElementById('addScormForm');
uploadBtn.click();
"
sleep 1
# Upload the actual zip file
$B upload "input[type=file]" "/path/to/Module-1-Title.zip"
sleep 3
$B screenshot /tmp/lw-upload-m1.png
```

Verify completion toggle is ON (it should be by default):
```bash
$B js "
const iframe = document.getElementById('contentHolder');
const doc = iframe.contentDocument || iframe.contentWindow.document;
const el = doc.getElementById('scorm_completion_engine');
if (el && !el.checked) { el.click(); 'enabled'; } else { 'already on: ' + el?.checked; }
"
```

Save the unit:
```bash
$B js "
const iframe = document.getElementById('contentHolder');
const doc = iframe.contentDocument || iframe.contentWindow.document;
const saveBtn = [...doc.querySelectorAll('input[type=button], button')].find(el => (el.value || el.textContent.trim()) === 'Save');
saveBtn.click();
"
sleep 2
$B screenshot /tmp/lw-saved-m1.png
```

Repeat for each module zip, then the assessment zip.

### 3.6 Verify completion settings per module

Per LW-STANDARDS.md:
- Content modules: `scorm_completion_engine` ON, no mastery score needed (the SCORM manifest has no `<adlcp:masteryscore>`)
- Assessment: `scorm_completion_engine` ON (SCORM manifest has `<adlcp:masteryscore>80</adlcp:masteryscore>`)

### 3.7 Test one module before publishing

```bash
# Preview the course as a learner
$B goto "https://academy.nanto.com/author/course?courseid=${TITLE_ID}&tab=contents"
$B js "
const iframe = document.getElementById('contentHolder');
const doc = iframe.contentDocument || iframe.contentWindow.document;
// Click Preview
const previewBtn = [...doc.querySelectorAll('button, a')].find(el => el.textContent.trim() === 'Preview');
previewBtn.click();
"
sleep 3
$B screenshot /tmp/lw-preview.png
```

### 3.8 Publish

Once verified, set course visibility to Published:
- Course editor sidebar > Course settings > Access > set to Published (or leave Draft for client courses until enrollment is set up)

Take a final screenshot of the course outline showing all sections uploaded.

---

## Step 4 — Mode: MANAGE

General admin tasks in LearnWorlds.

### 4.1 Enrollment

For bulk enrollment by CSV:
```bash
$B snapshot -i  # navigate to Users > Bulk Actions or Import
```

For per-course manual enrollment:
- Admin > Courses > [Course] > Students > + Add

User tags per company (Enterprise feature):
- Admin > Users > [User] > Tags > + Add tag

### 4.2 Course visibility

Draft → Published:
- Admin > Courses > [Course] > Settings > Visibility > Publish

### 4.3 Certificates

Standard Nanto Academy certificate:
- Uses amber (#F59E0B) branding
- Issued automatically on course completion
- Confirm the correct certificate template is assigned under Course > Settings > Certificate

### 4.4 Reports

Completion reports:
- Admin > Analytics > Course reports

Learner progress:
- Admin > Users > [User] > Activity

---

## Step 5 — Mode: EXPLORE

When the user asks about a LearnWorlds feature or task that is not yet documented in
LW-STANDARDS.md, explore the UI and/or fetch documentation.

### 5.1 Explore the UI

```bash
$B snapshot -i  # start from admin home
```

Navigate to the relevant section, take screenshots, and report what you find.

### 5.2 Fetch LearnWorlds docs

```bash
# Search LearnWorlds support
# Primary docs: https://support.learnworlds.com/
```

Use WebSearch:
```
site:support.learnworlds.com [topic]
```

Or WebFetch a specific article URL if provided.

Read and summarise the relevant steps, then:
1. Follow them in the browser
2. Update LW-STANDARDS.md with the discovered flow so it is available next time

### 5.3 Update the knowledge base

After any successful explore session, append the discovered UI path and selectors
to `LW-STANDARDS.md` under the relevant section:

```bash
# Edit LW-STANDARDS.md to add the new flow
SKILL_DIR="$HOME/.claude/skills/gstack/nanto-learnworlds"
# Edit "$SKILL_DIR/library/LW-STANDARDS.md"
```

---

## Step 6 — Mode: AUDIT

Review all published LearnWorlds courses against Nanto Academy standards.

Navigate to Admin > Courses. For each published course:

```
COURSE AUDIT — [Date]
══════════════════════════════════════════════════
[Course Name]
  [ ] Title matches CATALOGUE-INDEX.md
  [ ] Category set
  [ ] Certificate template: Nanto Academy standard
  [ ] All SCORM modules present (modules + assessment)
  [ ] Completion logic correct
  [ ] Visibility: Published (or Draft with reason)
  [ ] Client tag applied (if client course)

ISSUES: [list any]
══════════════════════════════════════════════════
```

---

## Step 7 — Completion Report

```
NANTO LEARNWORLDS — [MODE] COMPLETE
══════════════════════════════════════════════════
Mode: [publish / manage / explore / audit]
Course: [code + title if applicable]
Actions taken:
  - [list of what was done]
Screenshots saved:
  - [list paths]
LW-STANDARDS.md updated: [YES / NO]
Issues found: [list or NONE]

STATUS: DONE / DONE_WITH_CONCERNS / BLOCKED / NEEDS_CONTEXT
══════════════════════════════════════════════════
```

If BLOCKED (e.g., login required, CAPTCHA, upload failed):
- Use `$B handoff` to open a visible browser for the user
- Document the blocker and what was tried
- Wait for user to resolve, then `$B resume`
