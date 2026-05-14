---
name: nanto-course-make
version: 1.0.0
description: |
  Full course creation workflow for Nanto Academy. Generates all documents
  (Outline, Script, Assessment, ImagePrompts), builds the SCORM HTML for each
  module and the final assessment, runs package-scorm.js to create zips.
  Follows CDS v2.1 exactly. Run from anywhere — pass the course code as argument.
  After completion, run /nanto-course-images then /nanto-course-qa. (Nanto)
triggers:
  - make course
  - create course
  - nanto course make
  - new academy course
allowed-tools:
  - Bash
  - Read
  - Write
  - AskUserQuestion
---

# /nanto-course-make

Build a complete Nanto Academy course from brief to packaged SCORM.

## Constants

```
COURSE_LIB: /Users/darack/Library/CloudStorage/ZohoWorkDriveTrueSync-NantoHQ/10-PROJECTS/ACADEMY/COURSE-LIBRARY
CDS:        COURSE_LIB/_STANDARDS/ACAD-STD-CourseDevStandard-v2.1-20260503.md
CATALOGUE:  COURSE_LIB/CATALOGUE-INDEX.md
REF_HTML:   COURSE_LIB/C001-DisciplinaryActionKenya/SCORM-Packages/M1-LegalFoundation/index.html
PACKAGER:   /Users/darack/Library/CloudStorage/ZohoWorkDriveTrueSync-NantoHQ/10-PROJECTS/ACADEMY/package-scorm.js
```

Find the latest CDS version dynamically:
```bash
find "$COURSE_LIB/_STANDARDS" -name "ACAD-STD-CourseDevStandard-*.md" | sort | tail -1
```

## Step 1 — Course brief

Ask the user for:

1. **Course code** — next available code from CATALOGUE-INDEX.md (e.g., C017)
2. **Course title** — short, specific (e.g., "Fire Safety Essentials")
3. **Target audience** — who is this for? (e.g., "Warehouse staff, Nairobi")
4. **Number of modules** — typically 3-5 per CDS Section 4
5. **Client or catalogue?** — Catalogue (Nanto IP, generic Kenya workplace) or custom client (specify name)
6. **Duration target** — default 25-45 min total per CDS Section 3.5

Read CATALOGUE-INDEX.md to confirm the code is not already in use.

## Step 2 — Read the CDS

Read the full CDS document. Every document and HTML you generate must follow it without
exception. Key sections to internalise before generating:
- Section 3: Instructional design (ADDIE, Bloom's, andragogy, scenarios, microlearning)
- Section 4: Course anatomy (screen types and counts)
- Section 5: Writing standards (British English, word counts, banned words)
- Section 8: SCORM technical spec (3-package model, manifest templates, mastery scores)
- Section 11: QA checklist (the standard /nanto-course-qa enforces)
- Section 12: Naming conventions

## Step 3 — Create the course folder

```bash
COURSE_FOLDER="$COURSE_LIB/CXXX-ShortName"
mkdir -p "$COURSE_FOLDER/assets"
mkdir -p "$COURSE_FOLDER/SCORM-Packages"
```

Use the course code and a CamelCase short name with no spaces, e.g.:
`C017-FireSafetyEssentials`

## Step 4 — Generate the Outline document

Write `ACAD-CXXX-Outline-v1.0-YYYYMMDD.md` to the course folder.

The Outline must contain:
- Course ID, title, version, date, target audience, duration
- 3-6 learning objectives (Bloom's measurable verbs, Remember through Apply level)
- Module list: number, title, purpose statement, screen count estimate
- Assessment plan: 8-12 questions, what each question tests, Bloom's level
- Image plan: one image per module minimum, brief description

Follow CDS Section 3.2 for objectives and Section 4 for module structure. All character
names in examples must be authentically Kenyan (Wanjiku, Otieno, Fatuma, Kamau, Amina,
Njoroge, Achieng, Mwangi, Halima, Kipchoge).

## Step 5 — Generate the Script document

Write `ACAD-CXXX-Script-v1.0-YYYYMMDD.md` to the course folder.

The Script is the full screen-by-screen content for the content modules only
(not the knowledge checks or final assessment — those go in the Assessment document).

For each module, write every screen per CDS Section 4.1:
- Module Title Screen: module number, title, one-sentence purpose
- Concept Screens (2-4): heading + body text, max 180 words per screen
- Example Screen: realistic Kenyan workplace case
- Scenario Screen: setup + decision point + consequence reveal
- Knowledge Check Screen: 2 MCQs with 4 options each, correct answer marked, feedback text
- Module Summary Screen: 3 plain-language takeaways (not repeat of headings)

Writing rules from CDS Section 5:
- British English throughout
- No sentences over 30 words
- Active voice always
- No banned words (check CDS Section 5.1 for the list)
- Legal references cite specific statute and section
- Scenarios must feel like real decisions, not obvious moral dilemmas

## Step 6 — Generate the Assessment document

Write `ACAD-CXXX-Assessment-v1.0-YYYYMMDD.md` to the course folder.

Contains:
- Final assessment: 8-12 MCQs, each testing application (not exact recall), mapped to
  a learning objective. Pass mark: 80%. Include the correct answer and a distractor
  analysis note for each question.
- Per-module knowledge check questions are already in the Script — list them here for
  reference with their module mapping.

## Step 7 — Generate the ImagePrompts document

Write `ACAD-CXXX-ImagePrompts-v1.0-YYYYMMDD.md` to the course folder.

Format — one section per image:

```markdown
## Image N — Module N: Module Title

**Filename:** `cxxx-mN-slug.jpg`
**Placement:** Module N title screen or [specific screen type]
**Mood:** [one phrase — e.g., "professional, focused, not alarming"]

**Firefly prompt:**
```
[Long, detailed Firefly-style prompt with lighting, composition, East Africa context]
```

**Canva AI / DALL-E alternative prompt:**
```
[Shorter version, same scene, suitable for API call — 1-2 sentences max]
```

**What to avoid:** [specific visual clichés or inappropriate elements]
```

Image rules from CDS Section 7:
- Every module gets at least one image
- No handshakes, no laptop clichés, no generic stock photos
- Kenyan workplace and East African context in every image
- Photorealistic style
- No text overlays
- Format: JPG, 1280×720 minimum, under 300 KB

Use C015's ImagePrompts file as the reference format:
`COURSE-LIBRARY/C015-CybersecurityEssentials/ACAD-C015-ImagePrompts-v1.0-20260408.md`

## Step 8 — Build SCORM packages

Read C001's M1 `index.html` (REF_HTML constant above) as the canonical HTML reference.
It establishes the design system: CSS variables, component classes, SCORM JS API calls,
navigation logic, and screen rendering patterns.

For each module, create a SCORM package folder:

```bash
mkdir -p "$COURSE_FOLDER/SCORM-Packages/M1-SlugName"
```

Write two files per module package:

### `imsmanifest.xml` (content module — no masteryscore)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<manifest identifier="cxxx-m1-content" version="1.0"
  xmlns="http://www.imsproject.org/xsd/imscp_rootv1p1p2"
  xmlns:adlcp="http://www.adlnet.org/xsd/adlcp_rootv1p2"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.imsproject.org/xsd/imscp_rootv1p1p2 imscp_rootv1p1p2.xsd
                      http://www.adlnet.org/xsd/adlcp_rootv1p2 adlcp_rootv1p2.xsd">
  <metadata><schema>ADL SCORM</schema><schemaversion>1.2</schemaversion></metadata>
  <organizations default="cxxx-m1-content-org">
    <organization identifier="cxxx-m1-content-org">
      <title>Module 1: Module Title Here</title>
      <item identifier="item-1" identifierref="resource-1">
        <title>Module 1: Module Title Here</title>
      </item>
    </organization>
  </organizations>
  <resources>
    <resource identifier="resource-1" type="webcontent" adlcp:scormtype="sco" href="index.html">
      <file href="index.html"/>
      <file href="cxxx-m1-imageslug.jpg"/>
    </resource>
  </resources>
</manifest>
```

### `index.html` (SCORM content module)

Generate the full HTML using the C001 M1 file as structural reference. Key requirements:
- Same CSS design system (CSS variables, component classes)
- Same SCORM JS pattern: `LMSInitialize` on load, `lesson_status = passed` on final
  screen, `LMSFinish` on `beforeunload`
- Progress bar tracks current screen / total screens
- Back and Next navigation on every screen
- Knowledge check: Next disabled until both questions answered
- Mobile-first: all content readable at 375px viewport
- No external dependencies except Google Fonts CDN
- Images referenced as relative paths (e.g., `../../assets/cxxx-m1-slug.jpg`)
  — they will be copied into the zip folder by package-scorm.js convention,
  OR copy them directly into the SCORM package folder

Populate the HTML with the module content from the Script document (Step 5).
Each screen type maps to a specific HTML section pattern from the C001 reference.

### Assessment package

Create `SCORM-Packages/Assessment/` with:

**`imsmanifest.xml`** (masteryscore 80):
```xml
<?xml version="1.0" encoding="UTF-8"?>
<manifest identifier="cxxx-assessment" version="1.0"
  xmlns="http://www.imsproject.org/xsd/imscp_rootv1p1p2"
  xmlns:adlcp="http://www.adlnet.org/xsd/adlcp_rootv1p2"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.imsproject.org/xsd/imscp_rootv1p1p2 imscp_rootv1p1p2.xsd
                      http://www.adlnet.org/xsd/adlcp_rootv1p2 adlcp_rootv1p2.xsd">
  <metadata><schema>ADL SCORM</schema><schemaversion>1.2</schemaversion></metadata>
  <organizations default="cxxx-assessment-org">
    <organization identifier="cxxx-assessment-org">
      <title>Final Assessment</title>
      <item identifier="item-1" identifierref="resource-1">
        <title>Final Assessment</title>
        <adlcp:masteryscore>80</adlcp:masteryscore>
      </item>
    </organization>
  </organizations>
  <resources>
    <resource identifier="resource-1" type="webcontent" adlcp:scormtype="sco" href="index.html">
      <file href="index.html"/>
    </resource>
  </resources>
</manifest>
```

**`index.html`** (assessment):
- 8-12 randomised MCQs from the Assessment document
- Pass at 80%: show score, set `lesson_status = passed`, proceed to completion screen
- Fail at <80%: show score, set `lesson_status = failed`, offer one retry, then require course review

## Step 9 — Copy images into SCORM package folders

Each content module's SCORM package needs its image co-located for the zip to be self-contained:

```bash
# For each module: copy the module image(s) into the SCORM package folder
cp "$COURSE_FOLDER/assets/cxxx-m1-slug.jpg" "$COURSE_FOLDER/SCORM-Packages/M1-SlugName/"
```

If images have not been generated yet (/nanto-course-images not yet run), note which
images are missing and continue — the packager will warn about missing files.

## Step 10 — Package with package-scorm.js

```bash
cd "$(dirname "$PACKAGER")"
node package-scorm.js "$COURSE_FOLDER"
```

This reads each `imsmanifest.xml`, derives the zip filename from the `<item title>`,
and writes zips to `SCORM-Packages/Zips/`.

## Step 11 — Completion report

```
NANTO COURSE MAKE — COMPLETE
═══════════════════════════════════════════════════════
Course:  CXXX — Title
Folder:  .../COURSE-LIBRARY/CXXX-ShortName/

Documents generated:
  ✓ ACAD-CXXX-Outline-v1.0-YYYYMMDD.md
  ✓ ACAD-CXXX-Script-v1.0-YYYYMMDD.md
  ✓ ACAD-CXXX-Assessment-v1.0-YYYYMMDD.md
  ✓ ACAD-CXXX-ImagePrompts-v1.0-YYYYMMDD.md

SCORM packages:
  ✓ M1-SlugName/    index.html + manifest
  ✓ M2-SlugName/    index.html + manifest
  ✓ Assessment/     index.html + manifest (masteryscore: 80)

Zips:
  ✓ Module 1 - Title.zip
  ✓ Module 2 - Title.zip
  ✓ Final Assessment.zip

Images: NOT YET GENERATED
═══════════════════════════════════════════════════════
Next steps:
  1. /nanto-course-images CXXX   — generate the images
  2. /nanto-course-qa CXXX       — full QA check before upload
  3. /nanto-learnworlds CXXX     — upload zips and publish in LearnWorlds
═══════════════════════════════════════════════════════
```

Update `CATALOGUE-INDEX.md` to add the new course entry.
