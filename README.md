# /checklist

> ## ⚠️ Before you start
>
> Your AI agent can install this skill (copy into `~/.claude/skills/checklist/`, then restart Claude Code to load it). A few things need **you** or your environment:
>
> - For the export/download to work, the generated page is **served over localhost** (the skill does this for you) — just don't open it as a bare `file://`.


> Generate an interactive, shareable HTML checklist that anyone can click through to verify your work.

## What it does

`/checklist` turns "is this actually done?" into a page you can hand to anyone. Invoke it after a batch of work and it discovers what changed (git diff since midnight, your session log, or a scope you name), researches the real UI so the steps reference labels a stranger will actually see, and writes a single self-contained HTML file: ELI5 step-by-step instructions, four verdict buttons per item (Pass / Fail / Needs Attention / Skip), per-item comment fields, a live progress bar, a one-click "Generate Review" summary, and an "Export as HTML" button that bakes the current state into a fresh file you can send to someone else. No build, no dependencies, no server beyond a local static host for the export to work.

It's the manual-QA companion to the automated `/grade` and `/walkthrough` skills — where those drive the verification themselves, `/checklist` produces an artifact a human (you, a teammate, a client) clicks through at their own pace.

It's part of [vibekit](https://github.com/brianharms) — a collection of small tools and Claude Code skills that improve coding with AI.

## Install

This skill is a `SKILL.md` plus a `template.html` reference example. Drop the folder into your Claude Code skills directory.

```bash
git clone https://github.com/brianharms/skill-checklist.git
cd skill-checklist

mkdir -p ~/.claude/skills/checklist
cp SKILL.md template.html ~/.claude/skills/checklist/
```

After copying you should have:

```
~/.claude/skills/checklist/SKILL.md
~/.claude/skills/checklist/template.html
```

Invoke it by typing **`/checklist`** in Claude Code.

## Usage

Run it after some work, optionally with a scope filter:

```
/checklist
/checklist just the voice feature
/checklist since yesterday
```

Claude discovers the relevant changes, researches the UI, builds the checklist data, writes `checklist-YYYY-MM-DD.html` to your project root, serves it on a local port, and tells you the URL:

```
Test plan open at http://localhost:9999/checklist-2026-06-04.html
— 8 items across 3 sections. Export as HTML to share with anyone.
```

Open it, click through each item (Pass / Fail / Needs Attention / Skip), leave comments, then hit **Generate Review** for a copy-pasteable summary or **Export as HTML** to send a filled-in (or blank) copy to someone else. State persists in `localStorage`, so you can close and resume.

## Requirements / Dependencies

- **Claude Code CLI** — this is a Claude Code skill.
- **A browser** — to open and click through the generated checklist.
- **Python 3** (already on macOS/most systems) — the skill serves the file over `localhost` with `python3 -m http.server` so the in-page Export/download works. (`file://` blocks blob downloads, which is why a local server is used.)

No sibling skills required. OS-agnostic — the generated HTML is self-contained and runs anywhere.

## For AI coding agents

If you're an agent working **on** this skill:

**Repo layout:**

```
skill-checklist/
├── SKILL.md        # the skill: discovery → UI research → build data → generate HTML
├── template.html   # a worked, GENERIC example checklist ("Example App") — reference for the output format
├── LICENSE         # MIT
├── .gitignore
└── README.md
```

**`SKILL.md` is the contract** Claude reads at invocation. It defines a 4-phase flow (Discovery → Research the UI → Build the Checklist Data → Generate the HTML) and an exhaustive HTML feature spec (layout, per-item controls, sticky bottom bar, localStorage state, export logic, CSS classes, review output format). `template.html` is a concrete, sanitized example of that output — it is reference material, not something the skill copies verbatim; the skill *generates* a fresh file each run.

**To test a change:** copy `SKILL.md` (+ `template.html`) into `~/.claude/skills/checklist/`, restart Claude Code, make a few changes in a test project, and run `/checklist`. Verify the generated HTML has all four verdict buttons, working export (serve over localhost, click Export, confirm the downloaded file carries `EMBEDDED_STATE`), and a correct Generate-Review summary.

**Invariants — do not break these:**

- **Single file, zero dependencies.** The generated HTML must inline all CSS/JS — no CDN links, no external images, no fonts pulled over the network. This is what makes it shareable.
- **Always serve over localhost for export.** Blob downloads fail under `file://`; keep the local-server step.
- **Keep `template.html` generic.** It must never reference a real project's name, ports, or internal URLs — it was scrubbed to a neutral "Example App" on purpose. Don't reintroduce project-specific content.
- **Project-specific localStorage key.** Use a per-project key (e.g. `myapp-checklist-v1`) so multiple checklists don't collide. The example uses `checklist-v1`.
- **Four verdicts, exact semantics:** Pass / Fail / Needs Attention / Skip, each coloring the item's left border. Don't drop or rename them.
- **Export must always be enabled** — users may want to share a blank checklist before filling anything in.
- **Steps must be followable by someone who's never seen the project** — that's the entire point of the ELI5 instructions. Action steps blue (`.action`), expect steps green (`.expect`).

> Note: this skill is the result of merging the former `/test-plan` skill into `/checklist` (test-plan was a strict superset). If you find old references to `/test-plan`, they point here.

## License

MIT © 2026 Brian Harms / Ritual Industries — [ritual.industries](https://ritual.industries)
