---
name: process-flow-diagram
description: Create polished dark-themed process flow diagrams as self-contained HTML+SVG files. Use when the user asks for workflow diagrams, process maps, approval flows, or automation sequences.
---

# Process Flow Diagram Skill

Create professional process flow diagrams as self-contained HTML files with inline SVG graphics and CSS styling. Optimized for linear, sequential workflows with clear step progression — manual steps, automated steps, integrations, and decision branches.

> **Version 1.1** · MIT License · Authored by [Cocoon AI](mailto:hello@cocoon-ai.com)

## When to Use

Use for:
- Business process documentation
- Automation workflow visualization
- Sprint / development process flows
- Approval workflows and decision trees
- Multi-step procedures with clear sequence
- Onboarding and runbook diagrams

Skip when: the relationships are non-sequential (system component graphs, infrastructure topologies). For those, see [architecture-diagram-generator](https://github.com/Cocoon-AI/architecture-diagram-generator).

## Design System

### Color Palette (Step Types)

| Step Type | Fill (rgba) | Stroke | Icon/Indicator |
|-----------|-------------|--------|----------------|
| Start/End | `rgba(8, 51, 68, 0.4)` | `#22d3ee` (cyan-400) | Pill shape |
| Manual Step | `rgba(6, 78, 59, 0.4)` | `#34d399` (emerald-400) | 👤 actor |
| Automated Step | `rgba(76, 29, 149, 0.4)` | `#a78bfa` (violet-400) | ⚡ or 🤖 |
| Integration/API | `rgba(120, 53, 15, 0.3)` | `#fbbf24` (amber-400) | 🔗 or ☁️ |
| Decision | `rgba(136, 19, 55, 0.4)` | `#fb7185` (rose-400) | Diamond shape |
| Prerequisite | `rgba(30, 41, 59, 0.3)` | `#94a3b8` (slate-400) | Dashed border |

### Typography

Use JetBrains Mono for all text (monospace, technical aesthetic):
```html
<link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;500;600;700&display=swap" rel="stylesheet">
```

Font sizes: 11px for step names, 9px for descriptions, 8px for annotations, 10px for step numbers.

### Visual Elements

**Background:** `#020617` (slate-950) with subtle grid pattern.

**Step boxes:** Rounded rectangles (`rx="8"`) with 1.5px stroke, semi-transparent fills, minimum 140x70px.

**Step number badges:** Small circles with step number, positioned top-left of each step box:
```svg
<circle cx="X" cy="Y" r="12" fill="#1e293b" stroke="STEP_COLOR" stroke-width="1.5"/>
<text x="X" y="Y+4" fill="white" font-size="10" font-weight="600" text-anchor="middle">1</text>
```

**Start/End nodes:** Pill shapes (large rx value):
```svg
<rect x="X" y="Y" width="100" height="40" rx="20" fill="rgba(8, 51, 68, 0.4)" stroke="#22d3ee" stroke-width="2"/>
```

**Decision diamonds:** Rotated squares:
```svg
<rect x="X" y="Y" width="60" height="60" rx="4" transform="rotate(45, CENTER_X, CENTER_Y)" fill="rgba(136, 19, 55, 0.4)" stroke="#fb7185" stroke-width="1.5"/>
```

**Prerequisites box:** Dashed border container at top:
```svg
<rect x="X" y="Y" width="W" height="H" rx="8" fill="rgba(30, 41, 59, 0.2)" stroke="#64748b" stroke-width="1" stroke-dasharray="6,4"/>
```

**Flow arrows:** Use arrowhead marker, with optional labels:
```svg
<marker id="arrowhead" markerWidth="10" markerHeight="7" refX="9" refY="3.5" orient="auto">
  <polygon points="0 0, 10 3.5, 0 7" fill="#64748b" />
</marker>
```

### Layout Patterns

**Horizontal flow (default):** Steps flow left-to-right, wrap to new row if needed.
- Prerequisites at top
- Start node on left
- Steps progress rightward
- End node on right
- Info cards below

**Vertical flow:** Steps flow top-to-bottom. Use for longer processes or when horizontal space is limited.

### ViewBox & Overflow Guidelines

**CRITICAL: Prevent right-edge cutoff**

1. **Calculate viewBox width generously:**
   - Step stride is **220px** (160px box + 60px gap — the 60px gap is needed for arrow labels like `output → input`)
   - Formula: `(number of steps × 220) + 200px padding`
   - **Add +120px** for each inline decision diamond (the diamond's ±57px corners plus its label consume a full step's worth of horizontal budget)
   - **Add +100px** if there is an exit node to the right of the last step
   - For 4 steps + 1 inline decision + exit pill: `(4 × 220) + 120 + 100 + 200 = 1300px`

2. **Match min-width to viewBox AND container max-width:**
   - `.diagram-container svg { min-width: [viewBox width]px; }` — same number as viewBox width, so the SVG never shrinks below its design width
   - `.container { max-width: [viewBox width + 48]px; }` — outer container must accommodate the SVG **plus** the `.diagram-container`'s 24px side padding (48px total). If you set `.container` max-width equal to viewBox width, you'll get a 48px horizontal scroll and the right side will be clipped on export. **Always add 48** to the viewBox width when setting `.container` max-width.
   - Default sizing rule: pick viewBox width V, then set `svg { min-width: Vpx }` and `.container { max-width: (V + 48)px }`. The three numbers (viewBox, svg min-width, container max-width − 48) should always be equal.

3. **Right-side elements need breathing room:**
   - Decision diamonds: keep 120px from right edge of viewBox
   - Exit nodes: keep 80px from right edge
   - Loop-back arrows: account for their curve radius

4. **Default safe viewBox:** `viewBox="0 0 1300 540"` with `min-width: 1300px` and `.container { max-width: 1348px }`
   - **If you change the viewBox width, change all three together:** set `svg { min-width: <viewBox>px }` and `.container { max-width: <viewBox + 48>px }`. The `+ 48` accounts for the `.diagram-container`'s 24px side padding. If these three numbers drift apart you'll get the right side clipped behind a horizontal scroll bar (and that clipped region won't survive PNG/PDF export).
   - Accommodates 4 steps with decision + exit node comfortably
   - Scale up for more complex processes — see "Multi-row wrap" below if you need >5 steps in horizontal flow

### Multi-row wrap pattern

If a horizontal flow needs more than ~5 steps + a decision, wrap to a second row instead of letting the viewBox grow past 1500px. Use a **dashed slate connector** to bridge end-of-row-1 to start-of-row-2:

```svg
<!-- End of row 1 (Step N right edge at, e.g., x=1240, y=180) → top of row 2 (start element at, e.g., x=120, y=423) -->
<path d="M 1240 180 L 1310 180 L 1310 380 L 120 380 L 120 423"
      fill="none" stroke="#94a3b8" stroke-width="1.5" stroke-dasharray="6,3"
      marker-end="url(#arrowhead-slate)"/>
<text x="1320" y="295" fill="#94a3b8" font-size="9">continue</text>
```

Place row 2 at `y = row1_y + 270` to leave room for the connector to clear both rows. Keep the connector's horizontal segment at `y ≈ midway between rows` so it doesn't graze either row's boxes.

### Cyclical loop pattern

For continuously-running processes (monitoring → trigger → action → loop), there is no Start/End pill. Instead, use a **dashed cyan loop-back arrow that travels over the top of the row** to return from the last step to the first:

```svg
<!-- Loop-back: last step (x=1430, y=200) → first step (x=160, y=160) over the top -->
<path d="M 1430 200 L 1460 200 L 1460 110 L 160 110 L 160 160"
      fill="none" stroke="#22d3ee" stroke-width="1.5" stroke-dasharray="6,3"
      marker-end="url(#arrowhead)"/>
<text x="800" y="103" fill="#22d3ee" font-size="9" text-anchor="middle" font-weight="600">↻ resume monitoring</text>
```

Reserve `y = 100–120` (above the actor labels) for the horizontal segment of the loop arrow. Keep "exception path" loops (e.g. QC fail) below the row at `y = row_y + 180+` with a rose dashed stroke.

### Step Box Pattern

```svg
<!-- Step number badge -->
<circle cx="X" cy="Y" r="12" fill="#1e293b" stroke="STROKE_COLOR" stroke-width="1.5"/>
<text x="X" y="Y+4" fill="white" font-size="10" font-weight="600" text-anchor="middle">N</text>

<!-- Step box -->
<rect x="X+5" y="Y+5" width="160" height="80" rx="8" fill="FILL_COLOR" stroke="STROKE_COLOR" stroke-width="1.5"/>

<!-- Actor label (optional, above box) -->
<text x="CENTER_X" y="Y-8" fill="#64748b" font-size="8" text-anchor="middle">ACTOR</text>

<!-- Step title -->
<text x="CENTER_X" y="Y+30" fill="white" font-size="11" font-weight="600" text-anchor="middle">Step Name</text>

<!-- Step description (multi-line if needed) -->
<text x="CENTER_X" y="Y+48" fill="#94a3b8" font-size="9" text-anchor="middle">Description line 1</text>
<text x="CENTER_X" y="Y+62" fill="#94a3b8" font-size="9" text-anchor="middle">Description line 2</text>
```

### Arrow with Label Pattern

```svg
<line x1="X1" y1="Y1" x2="X2" y2="Y2" stroke="#64748b" stroke-width="1.5" marker-end="url(#arrowhead)"/>
<text x="MID_X" y="MID_Y-6" fill="#94a3b8" font-size="8" text-anchor="middle">output → input</text>
```

### Info Cards (Bottom Section)

Three cards for process metadata:
1. **Prerequisites** — What's needed before starting
2. **Inputs/Outputs** — Data flowing through the process
3. **Tools/Integrations** — Systems involved

### Export Toolbar (built-in)

Every diagram ships with a single unobtrusive `⋯` toggle in the header. Click it to reveal three buttons — 📋 Copy (high-DPI PNG to clipboard, scale: 2), 🖼️ PNG (high-DPI PNG download), 📄 PDF (PNG embedded in a one-page PDF via jsPDF). The toolbar collapses back to the icon by default so it doesn't clutter the diagram. All three formats use the same html2canvas capture (with the toolbar excluded and 32px padding around the content), so PDF preserves the dark theme without going through the browser's print dialog.

When generating a new diagram, keep these intact in the template:
- The two CDN scripts in `<head>` (pinned versions, with Subresource Integrity hashes and `crossorigin="anonymous"`):
  - `https://cdn.jsdelivr.net/npm/html2canvas@1.4.1/dist/html2canvas.min.js` — `integrity="sha384-ZZ1pncU3bQe8y31yfZdMFdSpttDoPmOZg2wguVK9almUodir1PghgT0eY7Mrty8H"`
  - `https://cdn.jsdelivr.net/npm/jspdf@2.5.2/dist/jspdf.umd.min.js` — `integrity="sha384-en/ztfPSRkGfME4KIm05joYXynqzUgbsG5nMrj/xEFAHXkeZfO3yMK8QQ+mP7p1/"`
  - SRI ensures generated diagrams are tamper-resistant against CDN compromise. Do not modify the hashes; if the version is bumped, the new hash must be computed fresh.
- `id="report-container"` on the outermost `.container` div (this is what gets captured)
- `.toolbar` markup with `.toolbar-actions` (collapsed by default) and `.toolbar-toggle` (the `⋯` button)
- `.toolbar` CSS + `@media print { .toolbar { display: none !important; } }`
- `copyAsImage()`, `downloadPNG()`, and `downloadPDF()` script before `</body>`, all using `getBoundingClientRect()` + `html2canvas(document.body, { x, y, width, height, ignoreElements })` to capture a precise rect with breathing room and no toolbar

Caveats: clipboard API needs a user gesture and a secure context (https/file/localhost). SVG `<foreignObject>` renders inconsistently in html2canvas — stick to plain `<svg>` shapes and `<text>`. Bump `scale: 2` to `3` or `4` for higher-res output.

## Template

Copy and customize the template at `resources/template.html`. Key customization points:

1. Update the `<title>` and header text
2. Add prerequisites in the prerequisites box
3. Add steps in sequence with appropriate colors
4. Draw arrows between steps with labels
5. Update the three summary cards
6. Update footer metadata

## Output

Always produce a single self-contained `.html` file with:
- Embedded CSS (no external stylesheets except Google Fonts)
- Inline SVG (no external images)
- Two CDN scripts for the export toolbar (html2canvas, jsPDF) — both pinned with SRI hashes

The file should render correctly when opened directly in any modern browser.

## Quality Assurance — Preview & Fix

**IMPORTANT: Always preview the diagram before delivering to the user.**

After creating the HTML file, follow this QA process based on your context:

### Context-Aware QA Approaches

| Context | QA Approach |
|---------|-------------|
| **Claude in Chrome** | Take screenshot with browser tools, visually inspect, fix issues, re-screenshot to verify |
| **Claude.ai with artifacts** | Present file to user — they see rendered preview and can report issues for you to fix |
| **Claude Code CLI** | Inform user to open the HTML file in browser and report any visual issues |
| **API/Agents SDK** | If Puppeteer/Playwright available, screenshot programmatically; otherwise inform user to verify |

### Step 1: Preview the diagram
1. Copy the file to `/mnt/user-data/outputs/` (in Claude.ai sandbox)
2. Use `present_files` to share it with the user
3. **If browser tools available:** Take a screenshot to visually inspect
4. **If no browser tools:** Let user know you've followed the design system but they should verify the rendering

### Step 2: Check for common issues
Inspect the screenshot (or ask user to check) for these problems:

**Layout issues:**
- [ ] Right-side cutoff (decision diamonds, exit nodes, labels clipped)
- [ ] Overlapping elements (boxes, labels, context areas)
- [ ] Elements extending beyond viewBox boundaries

**Connection issues:**
- [ ] Arrows not connecting to box edges (gaps or overshoots)
- [ ] Arrow labels overlapping with the arrow line itself
- [ ] Loop-back arrows not curving smoothly

**Text issues:**
- [ ] Labels overlapping each other
- [ ] Text extending outside boxes
- [ ] Unreadable font sizes

**Spacing issues:**
- [ ] Uneven gaps between steps
- [ ] Step number badges misaligned with boxes
- [ ] Actor labels too close to or overlapping boxes

### Step 3: Fix any issues found
Common fixes:

| Problem | Solution |
|---------|----------|
| Right-side cutoff | Increase viewBox width by 100-200px, increase min-width to match |
| Element overlap | Adjust x/y positions, reduce element width, or expand viewBox |
| Arrow not connecting | Check endpoint coordinates match the target box edge (account for rx radius) |
| Label on arrow line | Move label y-coordinate above (y-10) or below (y+12) the line |
| Diamond edge connection | Diamond corners are at center ± ~45px on each axis |

### Step 4: Re-preview if changes were made
If you made fixes:
- **With browser tools:** Take another screenshot to verify
- **Without browser tools:** Re-present the file and let user confirm the fix

### Coordinate Reference Guide

For a step box at position `(x, y)` with `width=140, height=105`:
- **Left edge:** `x`
- **Right edge:** `x + 140`
- **Top edge:** `y`
- **Bottom edge:** `y + 105`
- **Center:** `(x + 70, y + 52)`
- **Arrow entry (left):** `(x, y + 52)`
- **Arrow exit (right):** `(x + 140, y + 52)`

For a decision diamond centered at `(cx, cy)` with size 64:
- **Left point:** `(cx - 45, cy)`
- **Right point:** `(cx + 45, cy)`
- **Top point:** `(cx, cy - 45)`
- **Bottom point:** `(cx, cy + 45)`

For a pill/exit node at `(x, y)` with `width=100, height=40`:
- **Center:** `(x + 50, y + 20)`
- **Top edge:** `y`
- **Bottom edge:** `y + 40`

## Example Process Types

### Automation Flow (e.g. Sprint Reports)
- Prerequisites: GitHub configured, access tokens
- Steps: Manual input → Automated processing → Review → Integration call
- Mix of manual (emerald) and automated (violet) steps
- See `examples/sprint-report-flow.html`

### Approval Workflow (e.g. AI Governance, IT Change Management)
- Steps with decision diamonds for approval/rejection paths
- Multiple end states possible (Approved → Production, Rejected → exit)
- See `examples/ai-governance-workflow.html` (5 steps + decision + drift loop-back)
- See `examples/it-change-management.html` (two-row wrap pattern, 7 steps + 2 decisions)

### Cyclical Process (e.g. Inventory Reorder)
- No Start/End pills — process is always-on
- Loop-back arrow over the top returns end-of-cycle to start-of-cycle
- Exception paths use a rose dashed stroke below the row
- See `examples/inventory-control.html`

## Worked Example — AI Governance Workflow

This is the linear flow used by `examples/ai-governance-workflow.html`. Use the coordinates below as a starting point when authoring a new 5-step diagram with one inline decision diamond.

**ViewBox sizing:**
- 5 steps × 220 stride = 1100
- + 1 inline diamond = +120
- + start pill + end pill = +220
- + 60 left/right padding = +120
- → `viewBox="0 0 1500 520"`, `svg min-width: 1500px`, `.container max-width: 1548px`

**Element coordinates (y = 180 for arrow midline):**

| Element | x | y | w × h | Center |
|---|---|---|---|---|
| Prerequisites bar | 30 | 20 | 1440 × 70 | — |
| Start pill | 30 | 150 | 100 × 40 | (80, 170) |
| Step 1 (Submit Use Case) — manual | 200 | 140 | 160 × 80 | (280, 180) |
| Step 2 (Risk Classification) — auto | 430 | 140 | 160 × 80 | (510, 180) |
| Step 3 (Ethics Review) — manual | 660 | 140 | 160 × 80 | (740, 180) |
| Decision (Approved?) | — | — | 90 × 90 rotated | (940, 180) |
| Step 4 (Deploy Model) — integration | 1070 | 140 | 160 × 80 | (1150, 180) |
| Step 5 (Monitor & Audit) — auto | 1300 | 140 | 160 × 80 | (1380, 180) |
| In Production pill | 1320 | 280 | 120 × 40 | (1380, 300) |
| Rejected pill | 880 | 330 | 120 × 40 | (940, 350) |

**Flow:** Start → 1 → 2 → 3 → Decision · {Yes → 4 → 5 → ↓ → In Production} · {No → ↓ → Rejected}.

**Drift loop-back:** dashed slate path from Monitor (1380, 220) curving under the row back to Risk Classification (510, 230). Reserve `y = 380–420` for this kind of return arrow so it clears the legend below.
