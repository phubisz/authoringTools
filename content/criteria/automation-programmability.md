---
name: Automation & Programmability
type: criteria
tags: [criteria, automation, cli, api, playwright, ai-agent, bulk-edit, scripting]
---

# Automation & Programmability

## Definition

Automation & Programmability measures how accessible a tool's course content is to **programmatic modification** — whether by a CLI, a REST API, direct file manipulation, or browser automation. The driving scenario: an AI agent (or a script) needs to find and modify a specific element across many slides or many courses — changing a brand name, updating a URL, swapping a color scheme, restructuring quiz logic — without a human clicking through the UI.

**Why it matters**: Manual bulk edits across large course libraries are a bottleneck. Teams managing 100+ courses face this every rebranding, every policy update, every platform migration. The right automation approach can reduce days of work to minutes. For AI-augmented L&D workflows, this is the integration surface that determines whether an agent can be a real collaborator.

---

## Automation Tiers

There are four distinct tiers of automation capability, from most to least reliable:

```
Tier 1 — File Format Access      (most reliable, AI-agent friendly)
Tier 2 — Official REST/CLI API   (reliable, limited scope)
Tier 3 — Browser Automation      (fragile, universal, ToS risk)
Tier 4 — None                    (manual only)
```

---

## Rankings

| Tool | Best Tier Available | Score | Notes |
|------|---------------------|-------|-------|
| [[tools/ispring-suite\|iSpring Suite]] | Tier 1 + CLI | ⭐⭐⭐⭐⭐ (5) | Source is .pptx (python-pptx) + iSpring SDK CLI |
| [[tools/lectora\|Lectora]] | Tier 1 | ⭐⭐⭐⭐ (4) | .awt = zip+XML; XLIFF export; fully scriptable |
| [[tools/articulate-360\|Articulate Storyline 360]] | Tier 1 | ⭐⭐⭐⭐ (4) | .story = zip+XML; no official API; community-proven |
| [[tools/elucidat\|Elucidat]] | Tier 2 | ⭐⭐⭐ (3) | REST API for config/deployment; not content authoring |
| [[tools/dominknow\|dominKnow]] | Tier 3 | ⭐⭐ (2) | No public authoring API; browser automation feasible |
| [[tools/parta-io\|Parta.io]] | Tier 3 | ⭐⭐ (2) | Cloud-only; browser automation only |
| [[tools/easygenerator\|Easygenerator]] | Tier 3 | ⭐⭐ (2) | Cloud-only; browser automation only |
| [[tools/mindsmith\|Mindsmith]] | Tier 3 | ⭐⭐ (2) | Cloud-only; browser automation only |
| [[tools/gomo-learning\|Gomo Learning]] | Tier 3 | ⭐⭐ (2) | Cloud-only; browser automation only |
| Rise 360 (Articulate) | Tier 3 | ⭐⭐ (2) | Cloud; no API; Playwright only |

---

## Tier 1: File Format Access

### Articulate Storyline — `.story` is a zip of XML

The `.story` file is a ZIP archive. Unzip it and you get:

```
story.story (unzipped)/
├── imsmanifest.xml       ← SCORM metadata
├── story.xml             ← master structure: scenes, slides, variables
├── slides/
│   ├── slide1.xml        ← per-slide content, shapes, triggers
│   ├── slide2.xml
│   └── ...
├── media/
│   ├── image1.png
│   └── ...
└── story_content/
    └── data.xml          ← published data (separate from source)
```

**Python automation pattern:**

```python
import zipfile, shutil, os
from pathlib import Path
import xml.etree.ElementTree as ET

def patch_storyline(story_path: str, find: str, replace: str):
    """Find and replace text across all slides in a .story file."""
    story = Path(story_path)
    extract_dir = story.with_suffix('')
    
    # Unzip
    with zipfile.ZipFile(story, 'r') as z:
        z.extractall(extract_dir)
    
    # Walk all XML files
    for xml_file in extract_dir.rglob('*.xml'):
        content = xml_file.read_text(encoding='utf-8')
        if find in content:
            xml_file.write_text(content.replace(find, replace), encoding='utf-8')
            print(f"Patched: {xml_file}")
    
    # Re-zip
    output = story.with_stem(story.stem + '_patched')
    with zipfile.ZipFile(output, 'w', zipfile.ZIP_DEFLATED) as z:
        for f in extract_dir.rglob('*'):
            z.write(f, f.relative_to(extract_dir))
    
    shutil.rmtree(extract_dir)
    print(f"Output: {output}")

# Usage:
patch_storyline('my_course.story', 'Acme Corp', 'New Brand Name')
```

**What Claude Code can do with this:**
- Find/replace text, URLs, variable names across entire course
- Add or remove trigger blocks on specific slides
- Rename variables globally (affects triggers, references, conditions)
- Swap media file references (`image1.png` → `image_v2.png`)
- Modify quiz question text and feedback
- Change slide properties (dimensions, advancement, timings)
- Extract all text for translation, then re-inject XLIFF-style

**Limitations:**
- No official spec — XML schema reverse-engineered by community
- Storyline must be used to re-open and verify after edits (it validates on load)
- Binary/compiled elements (complex animations) may not be safely editable
- Re-importing the patched `.story` for re-publish requires Storyline to be installed

---

### iSpring Suite — Source is `.pptx` (Best Automation Story)

iSpring's source is a standard PowerPoint file. This is the most automation-friendly format in this comparison because:

1. **`python-pptx`** is a mature, well-documented library for reading and writing `.pptx`
2. **iSpring SDK CLI** can batch-publish `.pptx` to HTML5 from the command line

```python
from pptx import Presentation
from pptx.util import Pt
import subprocess

def patch_pptx(pptx_path: str, find: str, replace: str):
    """Replace text across all slides in a PowerPoint."""
    prs = Presentation(pptx_path)
    for slide in prs.slides:
        for shape in slide.shapes:
            if shape.has_text_frame:
                for para in shape.text_frame.paragraphs:
                    for run in para.runs:
                        if find in run.text:
                            run.text = run.text.replace(find, replace)
    output = pptx_path.replace('.pptx', '_patched.pptx')
    prs.save(output)
    return output

def publish_to_html5(pptx_path: str, output_dir: str):
    """Publish using iSpring SDK CLI."""
    subprocess.run([
        'iSpringSDK.exe', 'generate-html',
        pptx_path,
        f'{output_dir}/index.html',
        '--zip'   # produces output.zip ready for LMS upload
    ], check=True)

# Full pipeline:
patched = patch_pptx('course.pptx', 'Old Brand', 'New Brand')
publish_to_html5(patched, './output')
```

**iSpring SDK CLI key commands:**
```bash
iSpringSDK.exe generate-html course.pptx ./output/index.html
iSpringSDK.exe generate-html course.pptx ./output/index.html --zip
iSpringSDK.exe generate-html course.pptx ./output/index.html -r 3-7  # slides 3-7 only
iSpringSDK.exe generate-thumbnails course.pptx ./thumbs/
```

**What this enables end-to-end:**
```
python-pptx patches .pptx → iSpring SDK CLI publishes → .zip SCORM package
         ↑ AI agent writes this code and runs it
```

---

### Lectora — `.awt` is a zip of XML + XLIFF support

Similar to Storyline: `.awt` is a ZIP containing XML. Additionally, Lectora has first-class XLIFF export — the translation workflow produces structured XML files that are far cleaner to automate than raw Storyline XML.

```bash
# Lectora XLIFF round-trip for text changes:
# 1. Export XLIFF from Lectora (UI or scripted)
# 2. Parse and modify with Python xml.etree
# 3. Import XLIFF back (UI or scripted)

# For structural changes: same zip+XML pattern as Storyline
```

---

## Tier 2: Official REST API

### Elucidat REST API

Elucidat has a documented REST API, though it targets **course configuration and deployment**, not content authoring. You cannot create slide content via the API, but you can:

```bash
# Configure a project's SCORM settings
POST /api/v2/projects/{id}/configure
{
  "tracking_mode": "scorm_2004",
  "pass_rate": 80,
  "completion_rate": 100
}

# Get a launch link for a specific learner
GET /api/v2/projects/{id}/launch?user_id=abc123

# Subscribe to events (webhooks)
POST /api/v2/webhooks
{ "event": "project.published", "url": "https://yourserver.com/hook" }
```

**What this enables**: CI/CD-style deployment pipelines — when a course is published, automatically configure it for the right LMS, set pass rates, notify downstream systems.

---

## Tier 3: Browser Automation (Playwright/Puppeteer)

For cloud tools with no API and no accessible file format, browser automation is the only programmatic path.

### When it makes sense
- One-off bulk operations (rebranding an entire library once)
- Internal tooling where ToS risk is accepted
- Prototype/exploration rather than production pipeline

### General approach with Playwright

```javascript
import { chromium } from 'playwright';

const browser = await chromium.launch({ headless: false }); // headful for auth
const page = await browser.newPage();

// Login (may need to handle SSO/2FA manually first time)
await page.goto('https://app.easygenerator.com');
await page.fill('[data-testid="email"]', process.env.EG_EMAIL);
await page.fill('[data-testid="password"]', process.env.EG_PASSWORD);
await page.click('[data-testid="login-btn"]');
await page.waitForNavigation();

// Navigate to a course
await page.goto('https://app.easygenerator.com/courses/course-id-here');

// Find all text matching a pattern and click to edit
const textElements = await page.$$('[contenteditable="true"]');
for (const el of textElements) {
  const text = await el.textContent();
  if (text.includes('Old Brand')) {
    await el.click();
    await page.keyboard.selectAll();
    await page.keyboard.type(text.replace('Old Brand', 'New Brand'));
  }
}
```

### Why this is fragile
- **Selector rot**: any UI update from the vendor breaks your selectors
- **Auth walls**: SSO, 2FA, CAPTCHA
- **Rate limits and bot detection**: cloud tools can flag automated sessions
- **No atomicity**: a crash mid-edit leaves the course in a broken state
- **ToS risk**: most SaaS platforms prohibit automated scraping/modification

### When to prefer it anyway
- The tool has no alternative (Rise 360, Mindsmith, Parta.io)
- The operation is a one-time migration, not an ongoing pipeline
- You control the account and accept the ToS risk

---

## AI Agent (Claude Code) Integration Patterns

### Pattern A — File surgery (Storyline, Lectora, iSpring source)

Claude Code reads the XML/PPTX, understands the structure, writes targeted patches, and hands back a modified file. This is the most reliable approach.

```
User: "Change all instances of 'Module 1' to 'Chapter 1' 
       and add a completion variable check on slide 5"

Claude Code:
  1. Read story.story as zip
  2. Parse XML → locate text nodes + slide 5 trigger blocks
  3. Write patch script
  4. Execute → produce patched .story
  5. Report: 12 text replacements, 1 trigger added
```

### Pattern B — API orchestration (Elucidat)

Claude Code calls the REST API to configure or query courses. Useful for deployment pipelines.

### Pattern C — Playwright agent (cloud tools)

Claude Code writes and runs a Playwright script targeting the specific tool's UI. Best for one-off operations. Can be guided interactively with screenshots.

```javascript
// Claude Code generates this, runs it via Bash tool:
// mcp__playwright__browser_navigate, browser_snapshot, browser_click, etc.
```

### Pattern D — Hybrid: edit source, agent re-publishes

For iSpring: agent patches `.pptx` with python-pptx, then shells out to iSpring SDK CLI to re-publish. Fully automated end-to-end, no browser needed.

---

## Practical Bulk-Edit Scenarios

| Operation | Best Tool | Approach |
|-----------|-----------|----------|
| Rename brand across 50 Storyline courses | Storyline | Batch zip+XML find/replace (Python) |
| Update a URL in 200 iSpring slides | iSpring | python-pptx + SDK CLI republish |
| Change pass rate on 30 Elucidat courses | Elucidat | REST API PATCH loop |
| Add a disclaimer slide to 20 Rise courses | Rise 360 | Playwright (fragile) or manual |
| Translate all text in a Lectora course | Lectora | XLIFF export → edit → reimport |
| Swap a logo image across all Storyline courses | Storyline | Replace media file reference in XML |
| Update variable name in Storyline triggers | Storyline | XML attribute find/replace (careful!) |

---

## Recommended Stack for AI-Agent Course Editing

```
For Storyline-heavy organizations:
  Claude Code + Python (zipfile + xml.etree) + Storyline for final validation

For iSpring-heavy organizations:
  Claude Code + python-pptx + iSpring SDK CLI  ← most complete pipeline

For cloud-tool organizations:
  Claude Code + Playwright MCP  ← fragile but feasible for one-off ops
  + advocate for vendor API access (Elucidat is the only one with a real API)
```
