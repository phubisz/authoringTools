---
name: LLM Ingestion Readiness
type: criteria
tags: [criteria, llm, ai, document-ingestion, content-conversion, rag, vector-database, export]
---

# LLM Ingestion Readiness

## Definition

LLM Ingestion Readiness covers **both directions** of the LLM↔course boundary:

- **Direction A (IN)**: How well a tool accepts documents and prompts and converts them into eLearning courses using LLM capabilities
- **Direction B (OUT)**: How well a tool's course content can be exported to a clean, structured format (Markdown, JSON, XLIFF) suitable for feeding into a RAG system or vector database

**Why it matters**: Organizations increasingly need courses to be part of a bidirectional AI pipeline — not just using AI to *create* content, but also making that content *queryable* by AI. A course library that can't be indexed by a RAG system is a knowledge silo.

---

## Direction A Rankings — Document → Course (LLM IN)

| Tool | Score | Notes |
|------|-------|-------|
| [[tools/mindsmith\|Mindsmith]] | ⭐⭐⭐⭐⭐ (5) | AI-native; built around LLM generation; prompt → full lesson |
| [[tools/parta-io\|Parta.io]] | ⭐⭐⭐⭐ (4) | BYOA; integrate custom LLM workflows; AI on/off toggle |
| [[tools/easygenerator\|Easygenerator]] | ⭐⭐⭐⭐ (4) | EasyAI: Word/PDF → course; strong doc-to-course pipeline |
| [[tools/articulate-360\|Articulate 360]] | ⭐⭐⭐⭐ (4) | AI Assistant; Rise content generation; outline-to-course |
| [[tools/elucidat\|Elucidat]] | ⭐⭐⭐ (3) | Learning Accelerator; AI workflow guidance; limited doc ingestion |
| [[tools/ispring-suite\|iSpring Suite]] | ⭐⭐⭐ (3) | PPT ingestion (native); limited LLM pipeline beyond that |
| [[tools/lectora\|Lectora]] | ⭐⭐⭐ (3) | AI Course Wizard generates drafts; limited deep LLM integration |
| [[tools/gomo-learning\|Gomo Learning]] | ⭐⭐⭐ (3) | AI translation and content suggestions; limited doc-to-course |
| [[tools/dominknow\|dominKnow]] | ⭐⭐⭐ (3) | Limited LLM features; content reuse is manual, not AI-driven |

---

## Direction B Rankings — Course → RAG-Ready Format (LLM OUT)

The key question: **can you get clean, structured, chunking-friendly text out of the tool without manual copy-paste?**

| Tool | Score | Best Export Path | Output Quality |
|------|-------|-----------------|----------------|
| [[tools/ispring-suite\|iSpring Suite]] | ⭐⭐⭐⭐⭐ (5) | `python-pptx` on source `.pptx` → MD, or `MarkItDown` | Excellent: slide titles = natural chunk headers; rich metadata |
| [[tools/lectora\|Lectora]] | ⭐⭐⭐⭐⭐ (5) | XLIFF export → structured XML with full text hierarchy | Excellent: every text unit tagged by type, slide, module |
| [[tools/easygenerator\|Easygenerator]] | ⭐⭐⭐⭐ (4) | XLIFF 1.2 export | Good: structured, but XLIFF is XML not MD — needs one conversion step |
| [[tools/articulate-360\|Articulate 360 (Storyline)]] | ⭐⭐⭐⭐ (4) | `.story` zip+XML extraction → Python → MD | Good: all text accessible; requires custom parser |
| [[tools/articulate-360\|Articulate 360 (Rise)]] | ⭐⭐⭐ (3) | Scrape published HTML → MarkItDown | OK: clean semantic HTML but no machine-readable export |
| [[tools/mindsmith\|Mindsmith]] | ⭐⭐⭐ (3) | Scrape hosted lesson URL → MarkItDown/Jina | OK: dynamic content; lesson URL is stable (remote-hosted SCORM) |
| [[tools/dominknow\|dominKnow]] | ⭐⭐⭐ (3) | Published HTML5 scrape; PDF export → extraction | OK: requires scraping; no structured text export documented |
| [[tools/elucidat\|Elucidat]] | ⭐⭐⭐ (3) | PDF export → MarkItDown | OK: PDF loses structure; no XLIFF-level export confirmed |
| [[tools/parta-io\|Parta.io]] | ⭐⭐⭐⭐⭐ (5) | **Native Markdown export** (Team+); full content + assets (Enterprise) | Excellent: only cloud tool with purpose-built course→MD export |
| [[tools/gomo-learning\|Gomo Learning]] | ⭐⭐ (2) | Published HTML → scrape | Limited: no structured export documented |

---

## Key Insights

> **Parta.io (Team tier+) is the only cloud tool in this comparison with native Markdown export** — purpose-built, no conversion step required. Enterprise tier adds full digital asset management and omnichannel publishing. This is a significant differentiator for organizations building RAG pipelines on top of their course library.

> **All other tools require at least one conversion step.** The question is how clean and structured the intermediate format is.

> **iSpring has the best RAG pipeline by accident**: its PowerPoint source format is natively handled by `python-pptx` and Microsoft's `MarkItDown`, both mature tools. Slide titles become H2 headers, body text becomes paragraphs, speaker notes are preserved — perfect chunking boundaries with zero custom code.

> **XLIFF is underrated as a RAG format**: Lectora and Easygenerator both export XLIFF. It's XML, not Markdown, but it is highly structured — every text unit is tagged with its source, type, and context. One `xliff-to-md` pass produces clean, chunk-ready content with metadata intact.

> **The reverse pipeline is a strategic gap**: tools compete hard on "AI helps you create courses" but almost none have thought about "your course content feeds back into your AI stack." This is the next frontier.

---

## Conversion Pipelines

### Parta.io → Markdown (native, zero conversion)

```
Team/Enterprise plan → Export → "Course export in Markdown"
→ .md file (text) ready for chunking

Enterprise plan → "Integrated digital asset management" + "Omnichannel Publishing"
→ full content bundle (text + media assets) for complete course indexing
```

No code required. The Markdown output can be directly chunked and embedded into a vector DB. The Enterprise asset bundle means images, audio, and other media are co-located with their text references — enabling multimodal RAG if needed.

**Also note**: Enterprise includes "AI model integration via APIs" — the BYOA capability. This means the same tool that generates the RAG-ready Markdown export can also use your organization's own LLM to create the course content in the first place — a complete bidirectional pipeline within a single platform.

---

### iSpring → Markdown (best path for file-based tools)

```python
# Option A: python-pptx (preserves structure)
from pptx import Presentation

def pptx_to_markdown(pptx_path: str) -> str:
    prs = Presentation(pptx_path)
    lines = []
    for i, slide in enumerate(prs.slides, 1):
        title = next((s.text for s in slide.shapes
                      if s.has_text_frame and s.shape_type == 13), f"Slide {i}")
        lines.append(f"\n## {title}\n")
        for shape in slide.shapes:
            if shape.has_text_frame and shape.shape_type != 13:
                for para in shape.text_frame.paragraphs:
                    if para.text.strip():
                        lines.append(para.text.strip())
        # Speaker notes → keep as context for RAG
        if slide.has_notes_slide:
            notes = slide.notes_slide.notes_text_frame.text.strip()
            if notes:
                lines.append(f"\n> **Notes**: {notes}\n")
    return "\n".join(lines)

# Option B: MarkItDown (zero setup)
# pip install markitdown
from markitdown import MarkItDown
md = MarkItDown()
result = md.convert("course.pptx")
print(result.text_content)  # clean markdown, slide titles as headers
```

---

### Storyline → Markdown (custom parser)

```python
import zipfile, xml.etree.ElementTree as ET
from pathlib import Path

def story_to_markdown(story_path: str) -> str:
    lines = []
    with zipfile.ZipFile(story_path) as z:
        # Parse master structure for slide order
        with z.open('story.xml') as f:
            root = ET.parse(f).getroot()
        
        # Parse each slide file
        slide_files = sorted([n for n in z.namelist()
                               if n.startswith('slides/') and n.endswith('.xml')])
        for slide_file in slide_files:
            with z.open(slide_file) as f:
                slide = ET.parse(f).getroot()
            
            # Extract slide title (shape with type title)
            ns = {'a': 'http://schemas.openxmlformats.org/drawingml/2006/main'}
            title_el = slide.find('.//{*}ph[@type="title"]/../..//{*}t')
            title = title_el.text if title_el is not None else slide_file
            lines.append(f"\n## {title}\n")
            
            # Extract all text runs
            for t in slide.findall('.//{*}t'):
                if t.text and t.text.strip():
                    lines.append(t.text.strip())
    
    return "\n".join(lines)
```

---

### XLIFF (Lectora / Easygenerator) → Markdown

```python
import xml.etree.ElementTree as ET

def xliff_to_markdown(xliff_path: str) -> str:
    """Convert XLIFF 1.2 export to chunked Markdown for RAG."""
    tree = ET.parse(xliff_path)
    root = tree.getroot()
    ns = {'x': 'urn:oasis:names:tc:xliff:document:1.2'}
    
    lines = []
    current_file = None
    
    for file_el in root.findall('.//x:file', ns):
        fname = file_el.get('original', 'Unknown')
        if fname != current_file:
            lines.append(f"\n# {fname}\n")
            current_file = fname
        
        for unit in file_el.findall('.//x:trans-unit', ns):
            source = unit.find('x:source', ns)
            note = unit.find('x:note', ns)
            if source is not None and source.text:
                # note often contains the element type (title, body, question, etc.)
                tag = note.text if note is not None else ''
                prefix = '## ' if 'title' in tag.lower() else ''
                lines.append(f"{prefix}{source.text.strip()}")
    
    return "\n".join(lines)
```

---

### Published HTML5 → Markdown (universal fallback)

Works for **any tool** that publishes HTML5 output — Rise 360, Mindsmith, Gomo, Elucidat, dominKnow:

```python
# Option A: MarkItDown on local HTML file or URL
from markitdown import MarkItDown
md = MarkItDown()
result = md.convert("https://your-hosted-mindsmith-lesson-url")
# or: result = md.convert("path/to/published/index.html")
print(result.text_content)

# Option B: Jina Reader API (no install, handles SPAs better)
import requests
url = "https://r.jina.ai/https://your-hosted-lesson-url"
response = requests.get(url, headers={"Accept": "text/markdown"})
markdown = response.text

# Option C: html2text (Python, simple)
import html2text, requests
h = html2text.HTML2Text()
h.ignore_links = False
html = requests.get("https://your-lesson-url").text
markdown = h.handle(html)
```

---

## Chunking Strategy for Course Content in a Vector DB

Course content has natural chunk boundaries that map well to RAG:

```
Chunk level 1 — Course (metadata: title, version, audience)
Chunk level 2 — Module/Scene (metadata: module name, learning objectives)
Chunk level 3 — Slide/Page (metadata: slide title, type)
Chunk level 4 — Text block (metadata: shape type — body/callout/note)
Special chunks — Q&A pairs (question + correct answer + feedback)
```

**Recommended chunk schema:**
```json
{
  "id": "course-slug__module-2__slide-5__body",
  "content": "When handling a customer complaint, first acknowledge the issue...",
  "metadata": {
    "course": "Customer Service Fundamentals",
    "module": "Handling Complaints",
    "slide_title": "The HEAR Framework",
    "content_type": "body_text",
    "tool": "Storyline",
    "version": "2026-04",
    "language": "en"
  }
}
```

**Q&A pairs are especially valuable for RAG** — quiz questions become natural search anchors:
```json
{
  "id": "course-slug__quiz-3__q2",
  "content": "Q: What is the first step in the HEAR framework?\nA: Halt and listen without interrupting.",
  "metadata": { "content_type": "qa_pair", "correct_answer": true }
}
```

---

## Document-to-Course Pipeline Comparison (Direction A)

| Tool | Input Sources | AI Action | Output |
|------|--------------|-----------|--------|
| Mindsmith | Prompts, text input | Full lesson generation + structure | Interactive HTML5 lesson |
| Easygenerator | Word, PDF | Course structure + content extraction | SCORM-ready course |
| Articulate 360 | Text prompts | Outline + lesson content + quiz questions | Rise/Storyline project |
| Parta.io | Via BYOA provider | Configurable per org | Course + project assets |
| Lectora | Text prompts | Draft course structure | Lectora course draft |
| Gomo | Limited | Content suggestions | Partial scaffolding |

---

## Tool Commentary

### iSpring Suite (OUT direction: best)
Source is `.pptx` — a format that Microsoft's own `MarkItDown` handles natively, producing Markdown with slide titles as H2 headers and body text as paragraphs. Speaker notes (often the richest explanatory content) are preserved. No custom parsing code required. Combined with the `python-pptx` extraction path, this gives the cleanest course→RAG pipeline in this comparison.

### Lectora (OUT direction: best)
XLIFF export is purpose-built for structured text round-trips. Every text element is typed (title, body, question, feedback) and associated with its parent container. This is arguably the richest metadata of any export in this comparison — better than MarkItDown on HTML because the structure is explicit, not inferred.

### Easygenerator (OUT direction: good)
XLIFF 1.2 export with optional `Map HTML elements` flag for more granular structure. Combined with its EasyAI doc-to-course pipeline, Easygenerator is strong in both directions — good for organizations wanting a bidirectional AI pipeline.

### Storyline (OUT direction: good with effort)
The `.story` zip+XML extraction path works well and produces clean text. Quiz questions, feedback, and slide notes are all in the XML. Requires a custom parser (see code above) but the result is high-quality structured content. The challenge is that XML namespaces and schema knowledge require either reverse-engineering work or a community-maintained parser library.

### Rise 360 (OUT direction: OK)
Published HTML5 is semantically clean and renders predictably. `MarkItDown` or Jina Reader produces decent Markdown. The limitation is that Rise has no structured text export — you're always working from rendered output, which loses some metadata (block types, learning objective annotations).

### Mindsmith (OUT direction: OK with unique advantage)
Mindsmith's remote-hosted SCORM is a stable, always-current URL. You can point MarkItDown or Jina at the lesson URL and always get the latest version without re-exporting. This is actually a meaningful advantage for keeping a RAG index current — any lesson update is immediately reflected the next time you re-index.

### Parta.io (OUT direction: best cloud tool)
Native Markdown export on Team tier — the only purpose-built course→MD export in this cloud tool comparison. Enterprise adds full asset bundles and Omnichannel Publishing. Combined with BYOA (AI model via APIs on Enterprise), Parta.io is uniquely positioned as a **full bidirectional AI pipeline**: org LLM → course creation → Markdown export → RAG index.

### Gomo / Elucidat (OUT direction: limited)
Website/HTML export is available for parsing but no structured text export is documented. PDF export is the fallback — functional but loses structural metadata.
