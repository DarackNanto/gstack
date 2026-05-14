# Nanto Academy — LearnWorlds Course Setup Standards

## SCORM Upload Checklist

When uploading a course to LearnWorlds, always verify each item:

### Course Shell Settings
- [ ] **Title:** Match the CATALOGUE-INDEX.md entry exactly
- [ ] **Category:** Set appropriate category (Kenya Workplace / Health & Safety / Compliance / etc.)
- [ ] **Language:** English
- [ ] **Certificate template:** Nanto Academy standard certificate (amber branding)
- [ ] **Visibility:** Draft until all modules are uploaded and tested
- [ ] **Thumbnail:** Use the module title image or a dedicated course thumbnail

### SCORM Unit Settings (per module)
- [ ] **Unit type:** SCORM 1.2
- [ ] **Completion:** Set to "Completed" when SCORM reports `lesson_status = passed`
- [ ] **Mastery score in LearnWorlds:** Match the `<adlcp:masteryscore>` in manifest (80 for assessment, none for content modules)
- [ ] **Unit title:** Match the `<title>` in imsmanifest.xml exactly

### Course Sequence
Order of units per CDS v2.1:
1. Module 1 — [Title] (content SCORM)
2. Module 2 — [Title] (content SCORM)
3. ... (all content modules in order)
4. Final Assessment (assessment SCORM, masteryscore 80)

### Course Completion Logic
- Course is complete when Final Assessment SCORM reports `lesson_status = passed`
- Content modules: completion is recorded but does not gate the certificate
- Certificate: auto-issued on course completion by LearnWorlds

### Enrollment
- Per-client cohort: use LearnWorlds user tags (one tag per company/client)
- Bulk enrollment: CSV upload via admin > Users
- Per-course pricing: check with Darack before setting (B2B clients get invoiced directly — do not enable self-service checkout for client courses)

## Known LearnWorlds UI Patterns

Last verified: 13 May 2026 via browse session.

### Course Structure in LearnWorlds

Each Nanto course in LearnWorlds has this structure:
```
Course
  Section 01 — Module Title (e.g., "Legal Foundation")
    └─ SCORM unit  (e.g., "ACAD-C001-M1-LegalFoundation-SCORM12")
  Section 02 — Module Title
    └─ SCORM unit
  ...
  Section N — Assessment
    └─ SCORM unit  (the assessment zip)
  Section N+1 — Certification of Completion
    └─ Assessment (native LW)
    └─ Course Cert (certificate activity)
```

### How to Navigate to a Course Editor

The course editor URL pattern:
```
https://academy.nanto.com/author/course?courseid={titleId}&tab=contents
```

The `titleId` comes from the API. Get it with:
```bash
$B goto "https://academy.nanto.com/author/courses"
sleep 3
$B js "fetch('/api/author/products?fields=id,title,titleId,type&paginationType=paginated&page=1&itemsPerPage=20&type=course&sortField=created&sortDirection=desc').then(r=>r.json()).then(d=>JSON.stringify(d.data))"
```

The Course Manager lives in the `#contentHolder` iframe (same-origin, accessible via JS).
To open the course editor from the courses list, use:
```bash
$B js "
const iframe = document.getElementById('contentHolder');
const doc = iframe.contentDocument || iframe.contentWindow.document;
const btn = doc.querySelector('.course-more-actions-btn');
btn.click();
setTimeout(() => {
  const editLi = [...doc.querySelectorAll('.course-more-actions-menu li')].find(li => li.textContent.trim() === 'Edit course');
  if (editLi) editLi.click();
}, 300);
'triggered';
"
sleep 4
$B url  # should now be: .../author/course?courseid={titleId}&tab=contents
```

### SCORM Upload Flow — Updating an Existing SCORM Unit

When a section already has a SCORM unit, click its Settings button:

```bash
# Open Settings panel on the SCORM unit in section 01
$B js "
const iframe = document.getElementById('contentHolder');
const doc = iframe.contentDocument || iframe.contentWindow.document;
const settingsBtn = doc.querySelector('.edit-btn-icon-lbl.update[title=\"Settings\"]');
settingsBtn.click();
'settings opened';
"
sleep 2
```

The "Update SCORM" panel opens. It contains:
- `addScormForm` — input[type=button] — click to select a new zip
- `scorm_completion_engine` — checkbox — must be `checked: true` (ON) — "Use SCORM Completion Rules"
- `scorm_timing_engine` — checkbox — keep ON for time tracking
- Title field — must match `<item title>` in imsmanifest.xml
- Description field — use "SCORM/HTML 5 Package" as default

To upload the new zip:
```bash
$B js "
const iframe = document.getElementById('contentHolder');
const doc = iframe.contentDocument || iframe.contentWindow.document;
document.getElementById('addScormForm')?.click() || doc.getElementById('addScormForm')?.click();
"
sleep 1
# Upload the zip file to the file input
$B upload "[class*=scorm] input[type=file], #scorm-file-input" "/path/to/Module.zip"
```

Verify completion toggle is ON after upload:
```bash
$B js "
const iframe = document.getElementById('contentHolder');
const doc = iframe.contentDocument || iframe.contentWindow.document;
const el = doc.getElementById('scorm_completion_engine');
el ? 'checked: ' + el.checked : 'not found';
"
```

If not checked, click it to enable:
```bash
$B js "
const iframe = document.getElementById('contentHolder');
const doc = iframe.contentDocument || iframe.contentWindow.document;
const el = doc.getElementById('scorm_completion_engine');
if (el && !el.checked) el.click();
"
```

### SCORM Upload Flow — Adding SCORM to an Empty Section

For a new course with empty sections, click "Upload file" in the section footer:
```bash
$B js "
const iframe = document.getElementById('contentHolder');
const doc = iframe.contentDocument || iframe.contentWindow.document;
const uploadLinks = [...doc.querySelectorAll('a, button, span')].filter(el => el.textContent.trim() === 'Upload file');
if (uploadLinks.length > 0) uploadLinks[0].click();
"
sleep 2
# Then follow the Update SCORM panel flow above
```

### Unit-Level Edit Controls (hover over a SCORM unit to reveal)

| Control | Selector | Purpose |
|---|---|---|
| Settings / Update SCORM | `.edit-btn-icon-lbl.update[title="Settings"]` | Opens Update SCORM panel |
| Edit Content | `.edit-btn-icon-lbl.update-direct` | Opens content editor |
| Upload file | `.edit-btn.file-import[title="Upload a learning activity file"]` | Upload a new file |
| Delete | `.edit-btn.delete` | Delete the unit |
| Preview | `.edit-btn-icon-lbl.view` | Preview in player |

### Adding a New Section (for a new course)

At the bottom of the course outline there are "Add section", "Upload files", "Import existing" buttons. Use:
```bash
$B js "
const iframe = document.getElementById('contentHolder');
const doc = iframe.contentDocument || iframe.contentWindow.document;
const addSection = [...doc.querySelectorAll('button, a')].find(el => el.textContent.trim() === 'Add section');
if (addSection) addSection.click();
"
```

### Admin Sidebar Navigation

| Menu item | Selector | URL |
|---|---|---|
| Courses & programs | `@c10` (cursor-pointer "Courses & programs") | /author/courses |
| Users | `@c18` | /author/users |
| Reports | `@c34` | /author/reports |
| Settings | `@c42` | /author/settings |

## LearnWorlds Docs

Primary docs: https://support.learnworlds.com/
SCORM-specific: search "SCORM" in support portal

When the skill encounters an unfamiliar LearnWorlds task:
1. Use WebSearch: `site:support.learnworlds.com [topic]`
2. Fetch the article with WebFetch
3. Follow the steps in the browser
4. Update this file with the discovered flow
