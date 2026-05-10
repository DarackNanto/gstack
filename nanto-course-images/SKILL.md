---
name: nanto-course-images
version: 1.0.0
description: |
  Generate images for a Nanto Academy course using Gemini Imagen 3.
  Reads the ACAD-CXXX-ImagePrompts file in the course folder, calls the
  Gemini API for each image, and saves them to assets/.
  Replaces the manual Adobe Firefly/Gemini browser step. (Nanto)
triggers:
  - generate course images
  - nanto course images
  - generate images for course
allowed-tools:
  - Bash
  - Read
  - Write
---

# /nanto-course-images

Generate all images for a Nanto Academy course from the ImagePrompts document.

## Step 1 — Identify the course folder

The user may pass a course code (`/nanto-course-images C015`) or run from the
COURSE-LIBRARY directory. Find the folder:

```bash
COURSE_LIB="/Users/darack/Library/CloudStorage/ZohoWorkDriveTrueSync-NantoHQ/10-PROJECTS/ACADEMY/COURSE-LIBRARY"
# Replace C015 with the argument, or list folders and ask
find "$COURSE_LIB" -maxdepth 1 -type d -name "C0*" | sort
```

Confirm the course folder with the user if ambiguous.

## Step 2 — Find the ImagePrompts file

```bash
find "$COURSE_FOLDER" -maxdepth 1 -name "*ImagePrompts*.md" | sort | tail -1
```

Read the file. It is structured in sections:

```
## Image N — Module N: Title

**Filename:** `cxxx-m1-slug.jpg`
...
**Canva AI / DALL-E alternative prompt:**
```
short prompt text here
```
```

Extract every (filename, prompt) pair. Use the **Canva AI / DALL-E alternative prompt**
— it is shorter and optimised for API calls. Never use the Firefly prompt for the API.

## Step 3 — Check API key

```bash
[ -n "$GEMINI_API_KEY" ] && echo "KEY: set" || echo "ERROR: GEMINI_API_KEY not set"
```

Stop immediately if not set. Tell the user: add `export GEMINI_API_KEY="your_key"` to
`~/.zshrc` and run `source ~/.zshrc`.

## Step 4 — Create assets/ folder

```bash
mkdir -p "$COURSE_FOLDER/assets"
```

## Step 5 — Generate each image

For each (filename, prompt) pair:

1. Skip if the file already exists in assets/ (allows safe retry of failed runs).
2. Write `/tmp/nanto_imagen.py`:

```python
import os, sys, json, base64, urllib.request, urllib.error

api_key = os.environ['GEMINI_API_KEY']
prompt  = sys.argv[1]
out     = sys.argv[2]

url  = 'https://generativelanguage.googleapis.com/v1beta/models/imagen-3.0-generate-001:predict'
body = json.dumps({
    'instances':  [{'prompt': prompt}],
    'parameters': {'sampleCount': 1, 'aspectRatio': '16:9'}
}).encode()

req = urllib.request.Request(url, data=body, method='POST')
req.add_header('Content-Type', 'application/json')
req.add_header('x-goog-api-key', api_key)

try:
    with urllib.request.urlopen(req) as r:
        data = json.loads(r.read())
    img = base64.b64decode(data['predictions'][0]['bytesBase64Encoded'])
    with open(out, 'wb') as f:
        f.write(img)
    print(f'OK {os.path.basename(out)} ({len(img):,} bytes)')
except urllib.error.HTTPError as e:
    err = e.read().decode()
    print(f'FAIL {e.code}: {err[:300]}')
    sys.exit(1)
```

3. Run it:
```bash
python3 /tmp/nanto_imagen.py "PROMPT_TEXT" "$COURSE_FOLDER/assets/FILENAME"
```

4. Wait 7 seconds between calls (free-tier rate limit).

If a call returns 404 (model not found), switch to the Gemini Flash fallback:
```python
url  = 'https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash-preview-image-generation:generateContent'
body = json.dumps({
    'contents': [{'parts': [{'text': prompt}]}],
    'generationConfig': {'responseModalities': ['IMAGE']}
}).encode()
# response path: data['candidates'][0]['content']['parts'][0]['inlineData']['data']
```

## Step 6 — Check file sizes

```bash
find "$COURSE_FOLDER/assets" \( -name "*.jpg" -o -name "*.png" \) -exec ls -lh {} \;
```

Flag any file over 300 KB. Compress with macOS built-in `sips`:

```bash
sips --setProperty formatOptions 75 "$FILE" 2>/dev/null
```

Run again until all files are under 300 KB.

## Step 7 — Report

```
NANTO COURSE IMAGES
════════════════════════════════════════════
Course: CXXX — Title
Folder: .../assets/

  ✓ cxxx-m1-slug.jpg       138 KB
  ✓ cxxx-m2-slug.jpg       201 KB
  ✗ cxxx-m3-slug.jpg       FAILED — API error 429

Generated: 4/5    Skipped (exist): 0    Failed: 1
════════════════════════════════════════════
Failed images: re-run /nanto-course-images CXXX — existing files are skipped.
Next step: /nanto-course-qa CXXX
```
