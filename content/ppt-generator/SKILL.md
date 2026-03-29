---
name: ppt-generator
version: "1.0.0"
description: SVG-based PPT generator with 8 layouts, 9 color themes, 30+ charts, and 640+ icons
---

# PPT Generator Skill

Professional presentation engine. Generates SVG pages and converts them to native editable PPTX via svg_to_pptx.
Includes 8 layout templates covering dark, light, consulting, tech, and more.

## Prerequisites

- Python ≥ 3.10
- Required packages: `pip install python-pptx lxml`
- Agent capabilities: file write + **shell execution** (pure editor agents like Cursor cannot run this skill directly — users must execute commands manually)
- (Optional) Subagent support for parallel cross review

On first run, check if dependencies are installed. If not, run `pip install python-pptx lxml`.

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
> Not sure? I'll default to ~15 pages (about 12-15 min).

If the user already provided a detailed outline, skip to Step 2.

### Step 2: Pick a Style

> Pick a style (just reply with the number):
>
> **Tech / AI:**
>
> 1. **dark_warm** (default) — dark warm tone, AI/tech presentations
>
> 2. **ai_ops** — full dark, DevOps/operations dashboards
>
> 3. **cloud_orange** — deep navy + orange, cloud/tech architecture
>
> **Business / Formal:**
>
> 4. **consultant** — white + blue, strategy & consulting reports
>
> 5. **tech_blue** — blue tech, formal business presentations
>
> 6. **smart_red** — red accent, general business
>
> **Creative / Data:**
>
> 7. **exhibit** — light showcase, data-heavy reports & analytics
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

> Starting generation, {N} pages, this may take a few minutes.

During generation, **report progress to the user every 3-5 pages**:

> Page 5 of 25 done...
> Page 10 of 25 done...
> Page 20 of 25 done, almost there...

Keep it short. Do NOT mention SVG, technical details, or internal steps — the user only cares about page progress.

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
Locate skill dir → Read design_spec → Write SVGs → Embed icons → svg_to_pptx → Deliver
```

### Locating the Skill Directory

Before starting, determine the skill root directory (where this SKILL.md is located). All relative paths below are based on this root. The agent should resolve this based on its platform:
- OpenClaw: typically `~/.openclaw/skills/ppt-generator/` or `~/.openclaw/workspace/skills/ppt-generator/`
- Kiro: typically `~/.kiro/skills/ppt-generator/`
- Other: check where the skill was installed

Store this as `SKILL_DIR` for use in subsequent phases.

### Phase 1: Read Design Spec

Load the target style's design spec and reference templates:

```
{SKILL_DIR}/ppt-master-assets/templates/layouts/{style}/design_spec.md
{SKILL_DIR}/ppt-master-assets/templates/layouts/{style}/01_cover.svg
{SKILL_DIR}/ppt-master-assets/templates/layouts/{style}/02_chapter.svg
{SKILL_DIR}/ppt-master-assets/templates/layouts/{style}/03_content.svg
{SKILL_DIR}/ppt-master-assets/templates/layouts/{style}/04_ending.svg
```

### Phase 2: Generate SVG Files

Create SVG files page by page in a temporary directory (agent decides the path).

**SVG Rules:**
1. `viewBox` must be `0 0 1280 720` (16:9)
2. Strictly follow design_spec.md for colors, fonts, and layout
3. **Forbidden elements**: foreignObject, clipPath, mask, `<style>`, class, `<symbol>`, textPath, `@font-face`, `<animate>`, `<set>`, `<script>`, event attributes, marker, external CSS, `<iframe>`. Full list: `ppt-master-assets/references/shared-standards.md`
4. **Color compatibility**: Do NOT use `rgba()` or `opacity` attribute on `<g>`/`<image>`. Use `fill-opacity`/`stroke-opacity` on individual elements instead. Example: ❌ `fill="rgba(0,80,80,0.5)"` → ✅ `fill="#005050" fill-opacity="0.5"`
5. File naming: `01_cover.svg`, `02_toc.svg`, `03_chapter1.svg`... in order
6. All text uses `<text>` elements with `font-family`, `font-size`, `fill`
7. Background: full-coverage `<rect>`. Decorations: `<rect>`/`<circle>`/`<line>`/`<path>`
8. Tables: manual `<rect>` + `<text>` layout (no HTML tables)
9. Fill the page — avoid large empty areas
10. Titles should state insights, not category labels. Example: ❌ "Market Analysis" → ✅ "Domestic Market CAGR Reached 23% Over 3 Years"

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

After all SVGs are generated, run the full post-processing pipeline (icon embedding, image embedding, text flattening, etc.):

```bash
python3 {SKILL_DIR}/ppt-master-assets/scripts/finalize_svg.py ppt_svgs/
```

This replaces all `<use data-icon="...">` placeholders with native paths and applies other necessary SVG fixes for PPT compatibility. If `finalize_svg.py` is not available, you can run the icon embed step alone:

```bash
python3 {SKILL_DIR}/ppt-master-assets/scripts/svg_finalize/embed_icons.py ppt_svgs/*.svg
```

**Icon index:** See `ppt-master-assets/templates/icons/FULL_INDEX.md` for the complete list, or `icons_index.json` for programmatic lookup.

**Common icons:** `rocket`, `chart-bar`, `chart-line`, `chart-pie`, `lightbulb`, `target`, `shield`, `cog`, `users`, `globe`, `database`, `cloud`, `lock-closed`, `sparkles`, `flag`, `bolt`

### Phase 2.6: Charts (optional)

30+ chart SVG templates are available in `ppt-master-assets/templates/charts/`. Use them as reference when building data visualization slides.

**Available chart types:** bar, stacked-bar, line, area, pie, donut, funnel, waterfall, gantt, radar, scatter, heatmap, treemap, SWOT, comparison, timeline, process-flow, org-chart, and more.

**How to use:** When a slide needs a chart, first `read` the matching chart SVG template (e.g., `bar.svg` for bar charts) to learn its layout structure and style patterns. Then recreate the chart using native SVG elements (`<rect>`, `<line>`, `<text>`, `<circle>`, `<path>`) with your actual data. Do NOT embed chart SVGs directly — they are references only.

### Phase 3: SVG → PPTX Conversion

svg_to_pptx converts SVG elements into **native DrawingML shapes** (not images):
- `<text>` → editable text boxes (double-click to edit)
- `<rect>` → native rectangles (drag, recolor)
- `<circle>` / `<ellipse>` → native circles
- `<line>` / `<path>` → native lines/paths
- Tables (rect + text combos) → editable shape groups

The output PPTX is fully editable in PowerPoint, just like a manually created file.

```python
import sys
from pathlib import Path

SKILL_DIR = Path("...")  # resolve to actual skill root directory
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
| 2 | AI Ops | ai_ops | Dark |
| 3 | Cloud Orange | cloud_orange | Deep navy + orange |
| 4 | Consultant | consultant | White + blue |
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
- `shared-standards.md` — SVG technical constraints (full forbidden element list + compatibility rules)

---
