---
name: ppt-generator
description: SVG-based PPT generator with 9 themes, 8 layouts, 30+ charts, and 600+ icons
---

# PPT Generator Skill

Professional presentation engine. Generates SVG pages and converts them to native editable PPTX via svg_to_pptx.
Includes 8 layout templates covering dark, light, consulting, tech, and more.

## Prerequisites

- Python ≥ 3.10
- Required packages: `pip install python-pptx lxml`
- Agent capabilities: file write + shell execution
- (Optional) Subagent support for parallel cross review

## When to Use

- User asks to create a PPT / presentation / slides
- User provides content, outline, or data that needs to become a PPTX
- User mentions "make a PPT", "generate slides", "presentation", etc.

## Interaction Flow (must follow)

After receiving a PPT request, **do not generate immediately**. Guide the user step by step:

### Step 1: Confirm Topic

> Got it, I'll make this PPT for you. The topic is "{extracted from user message}" — correct?
>
> How many pages do you need? Or tell me the presentation duration and I'll estimate:
> - 10 min → ~12 pages
> - 20 min → ~18 pages
> - 30 min → ~25 pages
>
> Not sure? I'll default to ~15 pages.

If the user already provided a detailed outline, skip to Step 2.

### Step 2: Pick a Style

> Pick a style (just reply with the number):
>
> **Business / Formal:**
>
> 1. **consultant** — white + blue, strategy & consulting reports
>
> 2. **tech_blue** — blue tech, formal business presentations
>
> 3. **smart_red** — red accent, general business
>
> **Tech / AI:**
>
> 4. **dark_warm** (default) — dark warm tone, AI/tech feel
>
> 5. **ai_ops** — full dark, DevOps/operations
>
> 6. **cloud_orange** — deep navy + orange, cloud/tech architecture
>
> **Creative / Data:**
>
> 7. **exhibit** — light showcase, data-heavy presentations
>
> 8. **pixel_retro** — pixel retro, creative/fun topics

### Step 3: Confirm Outline

Good, {style} it is. Propose a structure based on the topic and style. Use markdown format with clear line breaks. Each page should include a brief content summary. Replace all placeholders with actual content before presenting to user:

> Here's a suggested structure ({N} pages):
>
> **P1 — Cover**
> Title, subtitle, author/date
>
> **P2 — Table of Contents**
> Overview of all sections
>
> **P3 — {Section 1 title}**
> Key points: {brief summary of what this page covers}
>
> **P4 — {Section 2 title}**
> Key points: {brief summary of what this page covers}
>
> ...
>
> **P{N} — Closing**
> Key takeaway, call to action, or contact info
>
> Want to adjust anything, or shall I start generating?

For long presentations (15+ pages), group by chapter and show chapter-level summaries first.

### Step 4: Generate

Only start after user confirms. Send a brief status message:

> Starting generation, this may take a few minutes for longer presentations.

Then execute the Technical Flow below.

### Step 5: Deliver and Iterate

Send the final PPTX with a brief note:

> PPT is ready, {N} pages total.
> Want changes? Just say:
> - "Update the data on page 3"
> - "Add a page about XX"
> - "Make the colors darker"

---

## Technical Flow (executed in Step 4)

⚠️ **Everything below is internal execution detail. NEVER expose SVG rules, code, or technical process to the user.**

```
Read design_spec.md + reference SVGs → Write SVG files → Embed icons → svg_to_pptx → Deliver
```

All paths below are relative to the skill root directory (where this SKILL.md is located).

### Phase 1: Read Design Spec

Load the target style's design spec and reference templates:

```
ppt-master-assets/templates/layouts/{style}/design_spec.md   ← colors, fonts, layout rules
ppt-master-assets/templates/layouts/{style}/01_cover.svg      ← cover reference
ppt-master-assets/templates/layouts/{style}/02_chapter.svg    ← chapter page reference
ppt-master-assets/templates/layouts/{style}/03_content.svg    ← content page reference
ppt-master-assets/templates/layouts/{style}/04_ending.svg     ← ending page reference
```

### Phase 2: Generate SVG Files

Use the `write` tool to create SVG files page by page in a temporary directory (agent decides the path).

**SVG Rules:**
1. `viewBox` must be `0 0 1280 720` (16:9)
2. Strictly follow design_spec.md for colors, fonts, and layout
3. **Do not use**: foreignObject, clipPath, mask, `<style>`, class, `<symbol>`, textPath, `@font-face`, `<animate>`, `<script>`, marker, external CSS, `<iframe>`. Full list in `ppt-master-assets/references/shared-standards.md`
4. File naming: `01_cover.svg`, `02_toc.svg`, `03_chapter1.svg`... in order
5. All text uses `<text>` elements with `font-family`, `font-size`, `fill`
6. Background: full-coverage `<rect>`. Decorations: `<rect>`/`<circle>`/`<line>`/`<path>`
7. Tables: manual `<rect>` + `<text>` layout (no HTML tables)
8. Fill the page — avoid large empty areas
9. Titles should state insights, not category labels

**Suggested order:**
- Cover and ending first (set the tone)
- Chapter dividers next (consistent style)
- Content pages last (data-heavy, one at a time)

### Phase 2.5: Icons (optional)

640+ SVG icons are bundled in `ppt-master-assets/templates/icons/`. Use them to add visual clarity to slides.

**How to use in SVG:**

Use `<use data-icon="...">` placeholder syntax during SVG generation:

```xml
<use data-icon="rocket" x="100" y="200" width="48" height="48" fill="#FF9900"/>
<use data-icon="chart-bar" x="200" y="200" width="48" height="48" fill="#0076A8"/>
```

⚠️ `<use data-icon="...">` is a **temporary placeholder only**. The embed script below replaces them with native `<g>+<path>` elements. The final SVG passed to svg_to_pptx must NOT contain any `<use>` tags.

After all SVGs are generated, run the embed script:

```bash
python3 ppt-master-assets/scripts/svg_finalize/embed_icons.py ppt_svgs/*.svg
```

**Icon index:** See `ppt-master-assets/templates/icons/FULL_INDEX.md` for the complete list, or `icons_index.json` for programmatic lookup.

**Common icons:** `rocket`, `chart-bar`, `chart-line`, `chart-pie`, `lightbulb`, `target`, `shield`, `cog`, `users`, `globe`, `database`, `cloud`, `lock-closed`, `sparkles`, `flag`, `bolt`

### Phase 3: SVG → PPTX Conversion

svg_to_pptx converts SVG elements into **native DrawingML shapes** (not images):
- `<text>` → editable text boxes (double-click to edit)
- `<rect>` → native rectangles (drag, recolor)
- `<circle>` / `<ellipse>` → native circles
- `<line>` / `<path>` → native lines/paths
- Tables (rect + text combos) → editable shape groups

The output PPTX is fully editable in PowerPoint, just like a manually created file.

```python
import sys, os
from pathlib import Path

# Add the skill's scripts directory to path (adjust based on your environment)
SKILL_DIR = Path("path/to/ppt-generator")  # adjust this
sys.path.insert(0, str(SKILL_DIR / "ppt-master-assets" / "scripts"))
from svg_to_pptx import create_pptx_with_native_svg

svgs = sorted(Path("ppt_svgs").glob("*.svg"))
create_pptx_with_native_svg(svgs, Path("output.pptx"),
                            canvas_format="ppt169",
                            use_native_shapes=True,  # must be True, otherwise SVG is embedded as image
                            verbose=True)
```

### Phase 4: Cross Review (optional, requires user confirmation)

After generating the first draft, ask the user:

> First draft is ready. Want to run a cross-review? Multiple reviewers check in parallel — more thorough but takes a few minutes.

If the user agrees and the agent supports subagents, run a review-fix cycle. If subagents are not available, the agent can perform a self-review checking each dimension sequentially.

1. **Round 1 (full review)**: 5 review dimensions (parallel if subagents available)
   - 🎤 Presentation Coach — narrative arc, flow, pacing
   - 👥 Target Audience — simulated audience reaction, comprehension
   - 🔬 Domain Expert — factual accuracy, technical depth
   - 📋 Content Auditor — structure, typos, data consistency
   - 👁️ Visual Inspector — layout, alignment, readability

2. **Fix**: Aggregate all issues, fix by priority (🔴 before ⚠️)

3. **Round 2 (regression)**: Verify the fixes

4. **Exit criteria**: 🔴=0 and ⚠️≤3, or round≥4 force exit

Skip if user says "no review" or "just a quick draft".

### Phase 5: Deliver

Only send the final version. Do not send intermediate versions unless the user asks.

---

## Style Directory Mapping

| # | Style | Directory | Background |
|---|-------|-----------|------------|
| 1 | Dark Warm | dark_warm | Dark cover + light content |
| 2 | Consultant | consultant | White + blue |
| 3 | Cloud Orange | cloud_orange | Deep navy + orange |
| 4 | AI Ops | ai_ops | Dark |
| 5 | Tech Blue | tech_blue | Blue |
| 6 | Smart Red | smart_red | Red |
| 7 | Exhibit | exhibit | Light |
| 8 | Pixel Retro | pixel_retro | Dark |

Default recommendation: dark_warm.

### Design Spec Location

Each template in `ppt-master-assets/templates/layouts/{directory}/` contains:
- `design_spec.md` — full design parameters
- `01_cover.svg` — cover template
- `02_chapter.svg` / `02_toc.svg` — chapter/TOC page
- `03_content.svg` — content page
- `04_ending.svg` — ending page

### Reference Documents

Key docs in `ppt-master-assets/references/`:
- `strategist.md` — strategist role
- `executor-consultant-top.md` — top-tier consulting executor
- `shared-standards.md` — SVG technical constraints (full forbidden element list)

---
