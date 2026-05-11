---
name: nanto-course-validate
version: 1.0.0
description: |
  Validate a Nanto Academy course built from customer material against its source document (PPTX or PDF).
  Detects coverage gaps, missing regulation references, unverified quantities, and added content
  not traceable to the source. Produces a structured review checklist saved to the course folder.
  Run after /nanto-course-make for any customer-material course before delivery. (Nanto)
triggers:
  - validate course
  - nanto course validate
  - check course against source
  - course content validation
allowed-tools:
  - Bash
  - Read
  - Write
---

# /nanto-course-validate

Compare a customer's source document (PPTX or PDF) against a generated Nanto Academy course to find coverage gaps, missing regulation references, unverified quantities, and added content not traceable to the source.

**Run after `/nanto-course-make` for every course built from customer material. Not needed for Nanto catalogue courses.**

---

## Step 1 — Identify the course and source document

```bash
COURSE_LIB="/Users/darack/Library/CloudStorage/ZohoWorkDriveTrueSync-NantoHQ/10-PROJECTS/ACADEMY/COURSE-LIBRARY"
```

The user passes a course code (e.g., `CXSKY001`) or a full path. Find the course folder:

```bash
find "$COURSE_LIB" -maxdepth 1 -type d -name "*${CODE}*"
```

Find the source document. Check in order:
1. A `Source/` subfolder inside the course folder
2. Any `.pptx`, `.ppt`, or `.pdf` in the course folder root
3. A client folder at the Academy root level (e.g., `Skyward_Express/`)
4. Ask the user to specify the path

```bash
find "$COURSE_FOLDER" -maxdepth 2 \( -name "*.pptx" -o -name "*.ppt" -o -name "*.pdf" \) | sort
```

## Step 2 — Convert source if needed

**If `.ppt` (old binary format):** Convert with LibreOffice:
```bash
/Applications/LibreOffice.app/Contents/MacOS/soffice --headless --convert-to pptx --outdir /tmp/ "$SOURCE_FILE"
SOURCE_PPTX="/tmp/$(basename "$SOURCE_FILE" .ppt).pptx"
```

**If `.pptx`:** Use directly. **If `.pdf`:** Install pdfplumber and adapt the slides extractor.

Check python-pptx is available:
```bash
python3 -c "from pptx import Presentation; print('ok')" 2>/dev/null || pip3 install python-pptx -q
```

## Step 3 — Write and run the validation script

Write the script to `/tmp/nanto_validate.py`:

```python
#!/usr/bin/env python3
"""
nanto-course-validate: Compare customer source document against generated course HTML.
Usage: python3 /tmp/nanto_validate.py <source.pptx> <scorm_packages_dir>
"""

import sys, re, os, glob
from pptx import Presentation

class HTMLTextExtractor:
    def __init__(self): self._text = []; self._skip = False
    def feed(self, raw):
        from html.parser import HTMLParser
        class P(HTMLParser):
            def __init__(s): super().__init__(); s.text = []
            def handle_starttag(s, tag, attrs):
                if tag in ('script', 'style'): self._skip = True
            def handle_endtag(s, tag):
                if tag in ('script', 'style'): self._skip = False
            def handle_data(s, data):
                if not self._skip:
                    t = data.strip()
                    if t: self._text.append(t)
        P().feed(raw)

def html_raw_content(path):
    with open(path, encoding='utf-8', errors='ignore') as f:
        raw = f.read()
    raw = re.sub(r'<style[^>]*>.*?</style>', ' ', raw, flags=re.DOTALL|re.IGNORECASE)
    raw = re.sub(r'<[^>]+>', ' ', raw)
    raw = raw.replace("\\'", "'").replace('\\"', '"').replace('\\n', ' ').replace('\\t', ' ')
    raw = raw.replace('&amp;', '&').replace('&nbsp;', ' ').replace('&lt;', '<').replace('&gt;', '>')
    raw = raw.replace('&#39;', "'").replace('&mdash;', ' ').replace('&divide;', '/')
    return re.sub(r'\s+', ' ', raw)

def load_html_corpus(pkg_dir):
    files = sorted(glob.glob(os.path.join(pkg_dir, '*/index.html')))
    result = {}
    for f in files:
        pkg = f.replace('\\', '/').split('/')[-2]
        if 'Check' not in pkg and pkg != 'Assessment':
            result[pkg] = html_raw_content(f)
    return result

def extract_pptx(path):
    prs = Presentation(path)
    slides = []
    for i, slide in enumerate(prs.slides):
        texts = []
        for shape in slide.shapes:
            if shape.has_text_frame:
                for para in shape.text_frame.paragraphs:
                    t = para.text.strip()
                    if t: texts.append(t)
        slides.append({'slide': i+1, 'title': texts[0] if texts else '', 'body': ' '.join(texts), 'texts': texts})
    return slides

ACR_SKIP = {
    'A','AN','AM','AS','AT','BE','BY','DO','GO','IF','IN','IS','IT','NO','OF','OK','ON','OR','SO',
    'TO','UP','US','WE','ALL','AND','ARE','CAN','FOR','GET','HAS','HAD','MAY','NOT','NOW','OFF',
    'ONE','OUR','OUT','OWN','PER','PUT','RUN','SAY','SEE','SET','THE','TOO','TWO','USE','WAS',
    'WHO','WHY','YES','YOU','HTML','HTTP','HTTPS','CSS','DIV','NAV','BTN','IMG','SRC','ALT',
    'LMS','API','SCO','ADL','IMS','XSD','UTF','URL','NEW','VAR','LET','NULL','TRUE','FALSE',
    'VOID','THIS','ELSE','RETURN','FUNCTION','CONST','CLASS','STYLE','COLOR','FONT','FLEX',
    'GRID','AUTO','NONE','BLOCK','ITEM','TYPE','BODY','HEAD','META','SPAN','LIST','PASS',
    'FAIL','MARK','SHOW','HIDE','INIT','DONE','LOAD','CONT','REF','DG','ID','II','III','IV',
}

def find_acronyms(text):
    return {a for a in set(re.findall(r'\b[A-Z]{2,7}\b', text)) if a not in ACR_SKIP and not a.isdigit()}

def find_regulation_refs(text):
    patterns = [
        r'DGR\s+[\d]+(?:\.[\d]+)+', r'DGR\s+[\d]+(?:\.[\d]+)?(?!\d)',
        r'ICAO\s+(?:Doc(?:ument)?|Annex)\s+[\d]+', r'Annex\s+\d+',
        r'Table\s+[\d]+(?:\.[\d]+)*[A-Z]?', r'Section\s+[\d]+(?:\.[\d]+)+',
    ]
    refs = set()
    for p in patterns:
        refs.update(re.findall(p, text, re.IGNORECASE))
    return {r.strip() for r in refs if len(r.strip()) >= 5}

def find_quantities(text):
    return set(re.findall(
        r'\b\d+(?:[,\.]\d+)?\s*(?:Wh|mAh|mW|ml|mL|kg|g(?!\w)|V(?!\w)|%|km|cm|mm|litres?|liters?|amps?|watts?|volts?|kPa|psi)\b',
        text, re.IGNORECASE))

def find_class_refs(text):
    found = set()
    for m in re.finditer(r'(?:Class|Division|Div\.?)\s+\d+(?:\.\d+)?', text, re.IGNORECASE):
        found.add(re.sub(r'\s+', ' ', m.group().strip()))
    return found

SKIP_TITLES_RE = re.compile(
    r'^(<number>|agenda|table of contents|contents|objectives|welcome|questions|thank you|end\b|references|module \d|exercise\b|class schedule)',
    re.IGNORECASE)
STOPS = {'that','this','with','from','have','will','been','also','when','where','which','they',
         'their','there','these','those','some','more','than','then','into','onto','must','should',
         'would','could','shall','each','does','were','being','about','above','below','after',
         'before','under','other','every','given','label','labels'}

def check_slide_coverage(slide, html_lower):
    title = slide['title'].strip()
    if not title or len(title) < 3 or SKIP_TITLES_RE.match(title): return 'skip', 0, []
    key_phrases = []
    for block in slide['texts'][1:]:
        if len(block) >= 20:
            words = block.split()
            if len(words) >= 4:
                mid = len(words) // 2
                key_phrases.append(' '.join(words[max(0,mid-2):mid+2]).lower())
    hits = [p for p in key_phrases if p in html_lower]
    if hits and len(hits)/max(len(key_phrases),1) >= 0.3: return 'covered', len(hits)/max(len(key_phrases),1), hits
    words = [w.lower() for w in re.findall(r'\b[a-zA-Z]{6,}\b', slide['body']) if w.lower() not in STOPS]
    seen, unique = set(), []
    for w in words:
        if w not in seen: seen.add(w); unique.append(w)
    sample = unique[:20]
    matched = [w for w in sample if w in html_lower]
    score = len(matched)/len(sample) if sample else 0
    if score >= 0.45: return 'covered', score, matched
    if score >= 0.20: return 'partial', score, matched
    return 'missing', score, matched

def main():
    if len(sys.argv) < 3:
        print("Usage: python3 nanto_validate.py <source.pptx> <scorm_packages_dir>"); sys.exit(1)
    source_path, pkg_dir = sys.argv[1], sys.argv[2]
    slides = extract_pptx(source_path)
    source_text = ' '.join(s['body'] for s in slides)
    html_corpus = load_html_corpus(pkg_dir)
    html_combined = ' '.join(html_corpus.values())
    html_lower = html_combined.lower()

    covered, partial, missing, skipped = [], [], [], []
    for slide in slides:
        status, score, matched = check_slide_coverage(slide, html_lower)
        entry = (slide['slide'], slide['title'], score, matched)
        if status=='covered': covered.append(entry)
        elif status=='partial': partial.append(entry)
        elif status=='missing': missing.append(entry)
        else: skipped.append(slide)

    src_refs = find_regulation_refs(source_text); html_refs = find_regulation_refs(html_combined)
    src_qty = find_quantities(source_text); html_qty = find_quantities(html_combined)
    src_classes = find_class_refs(source_text); html_classes = find_class_refs(html_combined)
    src_acr = find_acronyms(source_text); html_acr = find_acronyms(html_combined)

    def nc(s): return re.sub(r'\s+',' ',s.lower().strip())
    src_norm = {nc(c) for c in src_classes}; html_norm = {nc(c) for c in html_classes}

    customer_terms = set()
    for slide in slides:
        for line in slide['texts']:
            for m in re.finditer(r'\b([A-Z][a-z]+(?:\s+[A-Z][a-z]+){1,3})\b', line):
                t = m.group(1)
                if len(t) >= 8 and t not in {'Dangerous Goods','Class Schedule','Course Objectives',
                    'General Philosophy','Provision Information','Emergency Response','Cargo Aircraft',
                    'Package Markings','Handling Labels','Learning Objectives'}:
                    customer_terms.add(t)
    missing_terms = [t for t in sorted(customer_terms) if t.lower() not in html_lower]

    W = 64
    print(); print("NANTO COURSE CONTENT VALIDATION"); print("═"*W)
    print(f"Source:  {os.path.basename(source_path)} ({len(slides)} slides)")
    print(f"Course:  {len(html_corpus)} content module(s)")
    for pkg in sorted(html_corpus): print(f"         ├── {pkg}")
    total = len(covered)+len(partial)+len(missing)
    print()
    print(f"SLIDE COVERAGE  ({len(covered)} covered / {len(partial)} partial / {len(missing)} missing of {total})"); print("─"*W)
    if covered:
        print(f"\n  COVERED ({len(covered)}):")
        for sn,title,score,_ in covered: print(f"    ✓  Slide {sn:2d} [{int(score*100):3d}%]: {title[:60]}")
    if partial:
        print(f"\n  PARTIAL ({len(partial)}) — confirm manually:")
        for sn,title,score,matched in partial:
            print(f"    ~  Slide {sn:2d} [{int(score*100):3d}%]: {title[:60]}")
            if matched: print(f"               matched: {', '.join(matched[:5])}")
    if missing:
        print(f"\n  MISSING ({len(missing)}) — not traced to HTML:")
        for sn,title,score,matched in missing:
            print(f"    ✗  Slide {sn:2d}: {title[:60]}")
    if skipped: print(f"\n  — {len(skipped)} slide(s) skipped (title/agenda/exercise)")
    print()

    print("REGULATION REFERENCES"); print("─"*W)
    print(f"  Source ({len(src_refs)}): {', '.join(sorted(src_refs)) or 'none'}")
    print(f"  HTML   ({len(html_refs)}): {', '.join(sorted(html_refs)) or 'none'}")
    src_only_refs = src_refs - html_refs; html_only_refs = html_refs - src_refs
    if src_only_refs: print(f"  ⚠  Missing from HTML: {', '.join(sorted(src_only_refs))}")
    if html_only_refs: print(f"  ⚠  In HTML only (verify): {', '.join(sorted(html_only_refs))}")
    print()

    print("SPECIFIC QUANTITIES AND LIMITS"); print("─"*W)
    src_only_qty = src_qty - html_qty; html_only_qty = html_qty - src_qty
    print(f"  Source: {', '.join(sorted(src_qty)) or 'none'}")
    print(f"  HTML:   {', '.join(sorted(html_qty)) or 'none'}")
    if src_only_qty: print(f"  ⚠  In source, absent from HTML: {', '.join(sorted(src_only_qty))}")
    if html_only_qty: print(f"  ⚠  In HTML only — verify vs ops manual: {', '.join(sorted(html_only_qty))}")
    print()

    print("IATA CLASS REFERENCES"); print("─"*W)
    missing_cls = src_norm - html_norm; added_cls = html_norm - src_norm
    print(f"  Source ({len(src_norm)}): {', '.join(sorted(src_norm)) or 'none'}")
    print(f"  HTML   ({len(html_norm)}): {', '.join(sorted(html_norm)) or 'none'}")
    if missing_cls: print(f"  ⚠  In source not in HTML: {', '.join(sorted(missing_cls))}")
    if added_cls:   print(f"  ⚠  In HTML not in source: {', '.join(sorted(added_cls))}")
    print()

    print("ACRONYMS"); print("─"*W)
    src_only_acr = {a for a in (src_acr-html_acr) if len(a)>=3}
    html_only_acr = {a for a in (html_acr-src_acr) if len(a)>=3}
    print(f"  Source: {', '.join(sorted(src_acr))}")
    print(f"  HTML:   {', '.join(sorted(html_acr))}")
    if src_only_acr:  print(f"  ⚠  In source, absent from HTML: {', '.join(sorted(src_only_acr))}")
    if html_only_acr: print(f"  ⚠  In HTML only (verify): {', '.join(sorted(html_only_acr))}")
    print()

    if customer_terms:
        print("CUSTOMER TERMINOLOGY"); print("─"*W)
        if missing_terms:
            print(f"  ⚠  {len(missing_terms)} customer terms not found in HTML:")
            for t in missing_terms[:20]: print(f"       — {t}")
            if len(missing_terms) > 20: print(f"       ... and {len(missing_terms)-20} more")
        else: print("  ✓  All customer terms found in HTML")
        print()

    hard = len(missing)+len(src_only_refs)+len(src_only_qty)+len(missing_cls)
    soft = len(html_only_refs)+len(html_only_qty)+len(html_only_acr)+len(partial)+len(missing_terms)
    print("═"*W)
    print(f"RESULT: {'PASS ✓' if not hard and not soft else 'REVIEW REQUIRED'}")
    if missing:        print(f"  • {len(missing)} slide(s) not traced — check for coverage gaps")
    if src_only_refs:  print(f"  • {len(src_only_refs)} DGR reference(s) in source missing from HTML")
    if html_only_refs: print(f"  • {len(html_only_refs)} reference(s) in HTML not in source — verify accuracy")
    if html_only_qty:  print(f"  • {len(html_only_qty)} quantity/limit(s) in HTML not in source — verify vs ops manual")
    if src_only_qty:   print(f"  • {len(src_only_qty)} quantity/limit(s) in source missing from HTML")
    if missing_cls:    print(f"  • {len(missing_cls)} IATA class(es) in source not in HTML")
    if html_only_acr:  print(f"  • {len(html_only_acr)} acronym(s) added in HTML not in source — review")
    print("═"*W); print()

if __name__ == '__main__':
    main()
```

Run it:
```bash
python3 /tmp/nanto_validate.py "$SOURCE_PPTX" "$COURSE_FOLDER/SCORM-Packages"
```

## Step 4 — Interpret the report

| Section | What it flags |
|---|---|
| **Slide Coverage** | ✓ covered / ~ partial (confirm manually) / ✗ missing |
| **Regulation References** | DGR/ICAO section numbers in source not cited in HTML (not hallucinations — just uncited) |
| **Quantities and Limits** | Numbers in HTML not traceable to source — highest hallucination risk. Every number must be verified against the customer's ops manual and IATA DGR edition |
| **IATA Classes** | Classes mentioned in source but absent from HTML |
| **Acronyms** | Acronyms added by AI that were not in the source |
| **Customer Terminology** | Capitalised phrases from source not in HTML — terminology drift risk |

**Key rule:** Any specific number (Wh limit, ml limit, weight) in the HTML that has no match in the source must be confirmed with the customer before delivery. These come from the AI's general training data, not the customer's document.

## Step 5 — Produce the review checklist

After the automated report, write `ACAD-CXXX-ValidationReport-v1.0-YYYYMMDD.md` to the course folder. Structure:

```markdown
# ACAD-CXXX — Content Validation Report
Source: [filename] | Date: [today]

## REQUIRED BEFORE DELIVERY
[ ] Item — e.g., add DGR 2.3 reference to M2

## VERIFY WITH CUSTOMER
[ ] All quantities (100Wh, 500ml etc.) — confirm match their ops manual edition
[ ] [Specific reference] — confirm accuracy

## INTENTIONAL OMISSIONS
[ ] [Topic] — reason: [why excluded per CDS or scope decision]

## LEGITIMATE ADDITIONS (not hallucinations)
✓ JKIA, KCAA — Kenyan context required by CDS
✓ Wh formula — not in source but necessary for lithium battery module
```

## Notes

- Only runs on **customer-material courses** (PPTX/PDF source → HTML). Skip for Nanto catalogue courses.
- For `.ppt` files: always convert with LibreOffice first (`--convert-to pptx`).
- For PDF sources: coverage matching is less reliable — rely more on the references and quantity checks.
- The script excludes Check and Assessment packages from HTML analysis — those are generated content, not sourced content.
