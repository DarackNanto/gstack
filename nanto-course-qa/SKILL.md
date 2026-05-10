---
name: nanto-course-qa
version: 1.0.0
description: |
  QA check a Nanto Academy course folder against the Course Development Standard (CDS v2.1).
  Verifies documents, SCORM packages, images, manifests, and HTML correctness.
  Run before packaging any course for LearnWorlds upload. (Nanto)
triggers:
  - qa course
  - nanto course qa
  - check course
  - quality check course
allowed-tools:
  - Bash
  - Read
---

# /nanto-course-qa

Run the CDS v2.1 QA checklist against a course folder. Produces a PASS or GAP REPORT.

## Step 1 — Identify the course folder

```bash
COURSE_LIB="/Users/darack/Library/CloudStorage/ZohoWorkDriveTrueSync-NantoHQ/10-PROJECTS/ACADEMY/COURSE-LIBRARY"
CDS="$COURSE_LIB/_STANDARDS/ACAD-STD-CourseDevStandard-v2.1-20260503.md"
# Find latest CDS if version has changed
find "$COURSE_LIB/_STANDARDS" -name "ACAD-STD-CourseDevStandard-*.md" | sort | tail -1
```

The user may pass a course code (`/nanto-course-qa C015`) or a full path.

## Step 2 — Documents check

Verify these files exist in the course root (use glob — version numbers vary):

| Document | Pattern |
|---|---|
| Outline | `ACAD-CXXX-Outline-*.md` |
| Script | `ACAD-CXXX-Script-*.md` |
| Assessment | `ACAD-CXXX-Assessment-*.md` |
| ImagePrompts | `ACAD-CXXX-ImagePrompts-*.md` |

```bash
for pat in "Outline" "Script" "Assessment" "ImagePrompts"; do
  f=$(find "$COURSE_FOLDER" -maxdepth 1 -name "*-${pat}-*.md" 2>/dev/null | sort | tail -1)
  [ -n "$f" ] && echo "PASS $pat: $f" || echo "FAIL $pat: missing"
done
```

## Step 3 — SCORM packages check

Each module and the assessment must have a folder under `SCORM-Packages/` containing
`index.html` and `imsmanifest.xml`.

```bash
find "$COURSE_FOLDER/SCORM-Packages" -mindepth 1 -maxdepth 1 -type d | sort
```

For each package folder found:

```bash
[ -f "$PKG/index.html" ]      && echo "PASS html: $PKG" || echo "FAIL html missing: $PKG"
[ -f "$PKG/imsmanifest.xml" ] && echo "PASS manifest: $PKG" || echo "FAIL manifest missing: $PKG"
```

## Step 4 — Manifest correctness check

For each `imsmanifest.xml`, determine the package type from the folder name and
verify the `adlcp:masteryscore` rule per CDS v2.1 Section 8:

| Package type | Folder name contains | Expected masteryscore |
|---|---|---|
| Content module | `M1-`, `M2-`, etc. (no "Check") | **absent** (must not appear) |
| Knowledge check | `Check` or `check` | `50` |
| Assessment | `Assessment` | `80` |

```bash
cat "$PKG/imsmanifest.xml" | grep -c "masteryscore"
cat "$PKG/imsmanifest.xml" | grep "masteryscore"
```

Report: PASS or FAIL with the exact rule violated.

## Step 5 — HTML technical checks

For each `index.html`, verify:

```bash
grep -c "LMSInitialize"  "$PKG/index.html"
grep -c "lesson_status"  "$PKG/index.html"
grep -c "LMSFinish"      "$PKG/index.html"
```

All three must appear. Also check there are no external dependencies (CDN links
would break offline):

```bash
grep -E "https?://" "$PKG/index.html" | grep -v "fonts.googleapis.com\|fonts.gstatic.com"
```

Google Fonts CDN links are acceptable. Any other external URL is a FAIL — the
course must load without internet.

## Step 6 — Images check

Count expected images from the ImagePrompts file:

```bash
grep -c "^\*\*Filename:\*\*" "$IMAGEPROMPTS_FILE"
```

Count actual images in assets/:

```bash
find "$COURSE_FOLDER/assets" \( -name "*.jpg" -o -name "*.png" \) | wc -l
```

Check all images are under 300 KB:

```bash
find "$COURSE_FOLDER/assets" \( -name "*.jpg" -o -name "*.png" \) | while read f; do
  size=$(stat -f%z "$f" 2>/dev/null || stat -c%s "$f" 2>/dev/null)
  [ "$size" -gt 307200 ] && echo "FAIL over 300KB: $f ($size bytes)" || echo "PASS size ok: $(basename $f)"
done
```

## Step 7 — Zips check

```bash
find "$COURSE_FOLDER/SCORM-Packages/Zips" -name "*.zip" 2>/dev/null | sort
```

Expected: one zip per module content package, one per knowledge check (if any),
one `Final Assessment.zip`. Report count found vs expected.

## Step 8 — Report

Produce the full report:

```
NANTO COURSE QA REPORT
════════════════════════════════════════════════════
Course:  CXXX — Title
Checked: 2026-05-09
CDS:     v2.1

DOCUMENTS
  ✓ Outline
  ✓ Script
  ✓ Assessment
  ✗ ImagePrompts — missing

SCORM PACKAGES
  ✓ M1-LegalFoundation    html ✓  manifest ✓  masteryscore: absent ✓
  ✓ M2-TypesOfMisconduct  html ✓  manifest ✓  masteryscore: absent ✓
  ✗ Assessment            html ✓  manifest ✓  masteryscore: MISSING (expected 80)

HTML TECHNICAL
  ✓ LMSInitialize present in all packages
  ✓ lesson_status present in all packages
  ✓ LMSFinish present in all packages
  ✓ No unexpected external URLs

IMAGES
  Expected: 5   Found: 4   Over 300KB: 0
  ✗ Missing: cxxx-m3-slug.jpg

ZIPS
  Found: 5 (Module 1, Module 2, Module 3, Module 4, Final Assessment)

════════════════════════════════════════════════════
RESULT: FAIL — 3 gaps found

REQUIRED before packaging:
  1. Add ImagePrompts document (ACAD-CXXX-ImagePrompts-v1.0-YYYYMMDD.md)
  2. Add <adlcp:masteryscore>80</adlcp:masteryscore> to Assessment/imsmanifest.xml
  3. Generate missing image: run /nanto-course-images CXXX
════════════════════════════════════════════════════
```

If all checks pass:
```
RESULT: PASS — course is ready to package and upload to LearnWorlds.
```
