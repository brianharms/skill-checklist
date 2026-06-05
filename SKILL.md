---
name: checklist
description: Generate an interactive, shareable HTML checklist that anyone can click through to verify your work — ELI5 step-by-step instructions, pass/fail/needs-attention/skip buttons, per-item comments, and one-click export. Use when the user says /checklist, "make a checklist", "verification checklist", "QA checklist", or wants a shareable manual-QA page for completed work.
user-invocable: true
arguments: "Optional scope filter (e.g. 'just voice', 'since yesterday', 'the ripple feature'). If omitted, discovers all changes since midnight."
---

# /checklist — Interactive, Shareable QA Checklist Generator

Generate a standalone, shareable HTML checklist with detailed ELI5 step-by-step instructions, pass/fail/needs-attention/skip verdicts, per-item comments, progress tracking, and one-click export. Anyone — including someone who has never seen the project — can open it in a browser and click through to verify the work.

## Phase 1: Discovery

Identify what needs testing.

**If git repo:**
- `git log --since="midnight" --oneline` + `git diff` for ground truth.
- Cross-reference with SESSION_LOG.md if present.

**If not git repo:**
- Check SESSION_LOG.md in the project root for today's session entries.
- `find` with `-newermt` today, excluding node_modules, .git, build artifacts, logs.

**If scope argument provided:** Filter to matching changes only.

**Multi-project:** If the working directory contains multiple sub-projects (separate directories with their own SESSION_LOG.md or package.json), discover changes in each independently and organize the checklist by project.

## Phase 2: Research the UI

For each project/feature that needs testing, read the relevant source files to understand:
- How the UI is laid out (what's where, section names, button labels)
- How to navigate to each feature being tested
- Specific element names, CSS classes, or landmarks the user can identify
- What ports/URLs to open

Use an Explore agent if needed to gather this context. The goal is to write steps that reference **real UI labels** the user will see — not vague instructions.

## Phase 3: Build the Checklist Data

For each testable change, create a test item with:

```
{
  title: 'Short name of what is being tested',
  desc: 'One-line description of the feature/fix',
  steps: [
    // Alternating ACTION and EXPECT steps
    '<span class="action">Click/type/navigate instruction using real UI labels</span>',
    '<span class="expect">EXPECT: What the user should see or hear</span>',
  ]
}
```

### Step-writing rules:
- **Use real UI labels.** "Click the hexagon icon (OBJECT)" not "go to the object section"
- **Be specific about location.** "In the top-right corner" / "In the settings sidebar" / "Below the OUTPUT SCALE slider"
- **Action steps are blue** — wrap in `<span class="action">`
- **Expect steps are green** — wrap in `<span class="expect">EXPECT: ...</span>`
- **Describe observable outcomes.** "You should see a progress bar animating" not "the feature should work"
- **Include setup steps.** If testing feature X requires enabling feature Y first, say so
- **5-10 steps per item** is the sweet spot. Enough detail that someone unfamiliar can follow along.

Group items by project/section. Each group gets a `title` and `port` (if applicable).

## Phase 4: Generate the HTML

Write the checklist to an HTML file in the project root (or the common parent if multi-project). Use the template below.

**File naming:** `checklist-YYYY-MM-DD.html`

**Serve it locally** so exports work:
```bash
python3 -c "
import http.server, os, socketserver
os.chdir('<PROJECT_DIR>')
handler = http.server.SimpleHTTPRequestHandler
with socketserver.TCPServer(('', 9999), handler) as httpd:
    httpd.serve_forever()
" &>/dev/null &
```

Open `http://localhost:9999/<filename>` in the browser.

Tell the user: "Checklist open at http://localhost:9999/<filename> — [N] items across [M] sections. Export as HTML to share with anyone."

## HTML Template

The generated HTML must include ALL of the following features:

### Layout
- Dark theme (#0d1117 background)
- Sticky progress bar at top with pass/fail/attention/skip counts
- Sections with title + port badge + reviewed count
- Each item: title, description, expandable numbered steps, verdict buttons, comment toggle

### Per-item controls
- **4 verdict buttons:** Pass (green), Fail (red), Needs Attention (amber), Skip (gray)
- **Comment toggle:** "+ comment" button reveals a textarea for notes
- **Steps toggle:** "hide steps" / "show steps" to collapse/expand the numbered instructions
- Verdicts color the left border of the item card

### Bottom bar (sticky)
- **Generate Review** button — opens modal with plain-text summary, copy-to-clipboard
- **Export as HTML** button (always enabled) — downloads a self-contained HTML file with current state baked in via `EMBEDDED_STATE` variable. Recipients can fill in their own verdicts too.

### State management
- All state saved to localStorage (key: `checklist-v1` or project-specific key)
- On load, check for `EMBEDDED_STATE` variable — if present and no localStorage exists, use it as starting data
- **Reset All** button clears localStorage and reloads

### Export logic
The export function must:
1. Build an export state object from current state, with `stepsCollapsed` removed (all steps expanded in export)
2. Serialize as JSON
3. Inject `<script>var EMBEDDED_STATE = ${json};</script>` before the first `<script>` tag in `document.documentElement.outerHTML`
4. Create a Blob, append an `<a>` to the DOM with `.download` attribute, click it, clean up
5. Include data URI fallback for restricted contexts

### CSS classes for steps
```css
.action { color: #58a6ff; }  /* blue — user does something */
.expect { color: #3fb950; font-weight: 500; }  /* green — what should happen */
```

### Review output format
```
# [Project] Test Review — YYYY-MM-DD

## Summary: X pass, X fail, X needs attention, X skipped

## [Section Title] ([port])

[PASS] Item title
[FAIL] Item title
       Comment text indented here
[ATTN] Item title
[SKIP] Item title
[----] Unreviewed item title
```

## Rules

- **Always research the UI before writing steps.** Never write vague instructions. If you can't determine the exact UI label, read the source file.
- **Steps must be followable by someone who has never seen the project.** That's the whole point of ELI5.
- **One HTML file, zero dependencies.** No CDN links, no images, no external CSS. Everything inline.
- **Always serve via localhost** so blob downloads work. `file://` blocks them.
- **Export must always be enabled** — the user may want to share the blank checklist before filling anything in.
- **Use the project name in the localStorage key** to avoid collisions between different checklists.
