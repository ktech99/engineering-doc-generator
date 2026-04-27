---
name: engineering-doc-generator
description: Generate a polished dark-themed HTML explainer document for any engineering artifact — pull requests, repo architecture, system designs, migration plans, post-mortems, roadmaps, bug investigations. Creates a single self-contained HTML file with sidebar navigation, inline SVG diagrams, and color-coded callouts. Use when someone asks to "document this architecture", "generate an HTML explainer for this PR", "make a visual brief of this system", "turn this into a doc the team can read", or any request to render an engineering artifact as a readable page for engineers or non-engineers.
---

# engineering-doc-generator

Generates a single self-contained dark-themed HTML file that explains an engineering artifact — a pull request, repo architecture, system design, migration plan, post-mortem, roadmap, or bug investigation — in a way that's readable by both engineers and non-engineers.

---

## When to use

- "Generate an HTML explainer for [PR / branch / file / module]"
- "Document this architecture as a webpage"
- "Make a visual brief of [thing]"
- "Turn this into a doc the team can read"
- Anything that maps to "render this engineering thing as a page someone who isn't deep in the code can scan and understand"

---

## What to gather before generating

Before writing a single line of HTML, gather:

1. **Subject** — what's being explained. Pull from git (PR diff, branch commits, recent file changes), repo structure, or user-provided source. Run `git log`, `git diff`, `gh pr view`, `find`, `grep` as needed.
2. **Audience** — engineers / mixed / non-technical. Drives jargon density and how much "what" vs "why" lives in the doc.
3. **Outline depth** — quick brief (5 min read) vs deep dive (30+ min). Don't pad.
4. **Diagrams** — architecture (boxes + arrows), tree (file structure), sequence (request walked step by step), comparison (before/after).
5. **Concrete data** — real numbers only (test counts, line counts, commit hashes, file paths). **NEVER invent stats.** Omit a tile rather than fabricate.

Commands to run for a PR:
```bash
gh pr view <number> --json title,body,additions,deletions,changedFiles,commits,reviews
git diff origin/main...<branch> --stat
git log --oneline origin/main...<branch>
```

Commands for repo architecture:
```bash
find . -type f | head -80
wc -l $(find . -name "*.ts" -o -name "*.py" -o -name "*.go") 2>/dev/null | tail -1
git log --oneline -20
```

---

## Document structure

Sidebar-navigated layout. Sections shown one at a time. Scale section count to topic — don't auto-pad.

Recommended sections (drop what doesn't apply):

### Overview
- **01 TL;DR** — stat tiles + one-paragraph summary + 3-card highlights
- **02 Big picture** — full-system architecture SVG
- **03 File tree** — artifacts touched or relevant repo files

### Architecture
- **04 Pipeline / Flow** — sequence-style SVG of a typical run
- **05 Components** — one card per major piece, with what/why/where
- **06 Data model** — schemas, types, persistence
- **07 API surface** — the entry points

### Contracts
- **08 Inputs and outputs**
- **09 Error semantics** — what fails and how

### Observability
*(skip for non-runtime artifacts)*
- **10 Metrics, events, logs** the artifact emits

### Case gallery
- **11 Worked examples**
- **12 Bugs fixed** (post-mortem cards) OR **Decisions made** (ADR cards)

### Roadmap
- **13 What's deferred** — honest TODOs with reasoning
- **14 Open questions**

### Operational
- **15 How to run / promote / roll back**
- **16 Tests**
- **17 Decisions Q&A**

**Scaling rules:**
- PR → collapse to ~5 sections (TL;DR, File tree, Flow, Components, Tests)
- Post-mortem → TL;DR, Timeline, Root cause, Bugs fixed, What's deferred
- Repo architecture → expand to full set
- Migration plan → TL;DR, Big picture, Flow, Inputs/outputs, Operational

---

## Visual design system

### Color tokens (CSS variables)

```css
:root {
  --bg:        #020617;
  --surface:   #0f172a;
  --surface-2: #0b1220;
  --surface-3: #0a1224;
  --border:    #1e293b;
  --text:      #e2e8f0;
  --text-2:    #cbd5e1;
  --muted:     #64748b;
  --emerald:   #34d399;   /* success / done / kept */
  --amber:     #fbbf24;   /* warning / in-progress / partial */
  --violet:    #a78bfa;   /* note / data / structural */
  --cyan:      #22d3ee;   /* info / future / frontend */
  --rose:      #fb7185;   /* error / regression / removed */
  --orange:    #fb923c;   /* external / API boundary */
  --slate:     #94a3b8;   /* neutral / pending */
}
```

### Typography

- **Body:** Inter (400/500/600/700/800) — load from Google Fonts
- **Code/stats:** JetBrains Mono (400/500/600) — same Google Fonts request
- H1: `1.95rem` weight 800
- H2: `1.35rem` weight 700
- Code spans: `rgba(148,163,184,0.12)` bg, JetBrains Mono, cyan text

Google Fonts link (one tag, both families):
```html
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&family=JetBrains+Mono:wght@400;500;600&display=swap" rel="stylesheet">
```

### Atoms

**Stat tile** — number + uppercase label. Grid of 4–6 at TL;DR.
```html
<div class="stat-grid">
  <div class="stat"><span class="stat-val">247</span><span class="stat-lbl">Lines Added</span></div>
</div>
```

**Card** — rounded `0.65rem`, 1px border, surface bg.
```html
<div class="card">...</div>
```

**Callout** — colored variant with title chip. `emerald`=success, `cyan`=info, `violet`=note, `amber`=warning, `rose`=error.
```html
<div class="callout emerald"><span class="chip">Done</span><p>...</p></div>
<div class="callout rose"><span class="chip">Error</span><p>...</p></div>
```

**Chip** — uppercase 0.66rem pill, accent border+bg.
```html
<span class="chip cyan">New</span>
<span class="chip amber">Changed</span>
```

**Code block** — `<pre class="code">` with bg `#0a0f1c`, syntax-color spans:
- `.kw` → violet (keywords)
- `.str` → emerald (strings)
- `.com` → muted (comments)
- `.num` → amber (numbers/literals)
- `.fn` → cyan (function names)
- `.typ` → orange (types/classes)

**Step list** — `<ol class="steps">` with circular numbered counters.

**Tree** — `<pre class="tree">` with span colors:
- `.dir` → cyan
- `.file` → text-2
- `.new` → emerald (added file)
- `.mod` → amber (modified file)
- `.com` → muted (inline comment)

**Q&A** — boxed q+a pair, q in cyan with "Q." monospace prefix.

**Trace/case card** — left-border-accent card. `.fixed` variant = emerald left border.

**Diagram wrap** — `<div class="diagram">` containing inline SVG. Always include a `<p class="caption">` after.

### SVG diagram conventions

- Rounded boxes: `rx="8"` to `rx="10"`, stroke 1.5–2px
- Box fills: accent color at 6–10% opacity; border at 40–50% opacity
- Arrows: 1.5px lines with `<marker>` triangle arrowhead. One color per logical flow.
- Always include `viewBox` and `preserveAspectRatio="xMidYMid meet"`
- Text: Inter 11–13px, weight 700 for box titles, 500 for labels

### Sidebar nav

- Sticky, 240–260px wide, `surface-3` background
- Brand block at top: pulsing emerald dot + document title + subtitle (type of artifact)
- Section groups with uppercase category labels
- Active link: left `2px solid var(--cyan)`, cyan text, subtle bg highlight
- Hash-based show/hide JS — ONE section visible at a time
- Mobile (<900px): sidebar hides, all sections stack and render sequentially

---

## HTML skeleton

Every generated doc must follow this structure:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>[Subject] — Engineering Doc</title>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&family=JetBrains+Mono:wght@400;500;600&display=swap" rel="stylesheet">
  <style>
    /* CSS variables, resets, sidebar, section layout, atoms */
  </style>
</head>
<body>
  <nav class="sidebar">
    <div class="brand">
      <span class="pulse-dot"></span>
      <div>
        <div class="brand-title">[Subject]</div>
        <div class="brand-sub">[Type: PR · Architecture · Post-mortem · etc]</div>
      </div>
    </div>
    <div class="nav-group">
      <div class="nav-label">Overview</div>
      <a href="#tldr" class="nav-link active">TL;DR</a>
      <!-- more links -->
    </div>
  </nav>

  <main class="content">
    <section id="tldr" class="section active">
      <!-- stat grid, summary paragraph, highlight cards -->
    </section>
    <!-- more sections, hidden by default -->

    <footer>
      <span>[Subject] · [version/commit] · Generated [date]</span>
    </footer>
  </main>

  <script>
    // Hash-based navigation: show active section, update nav link
    const links = document.querySelectorAll('.nav-link');
    const sections = document.querySelectorAll('.section');

    function activate(hash) {
      const id = hash.replace('#', '') || 'tldr';
      sections.forEach(s => s.classList.toggle('active', s.id === id));
      links.forEach(l => l.classList.toggle('active', l.getAttribute('href') === '#' + id));
    }

    window.addEventListener('hashchange', () => activate(location.hash));
    activate(location.hash);
  </script>
</body>
</html>
```

Key CSS rules to always include:
```css
.section { display: none; }
.section.active { display: block; }
@media (max-width: 900px) {
  .sidebar { display: none; }
  .section { display: block !important; }
  .content { margin-left: 0; }
}
```

---

## Workflow

1. **Discover subject** — run git/gh/find/grep commands to gather inputs. Ask user for context if insufficient.
2. **Pick section list** — score the topic type, drop non-applicable sections. A PR needs ~5. A full architecture needs up to 17.
3. **Gather concrete numbers** — back every stat tile with a real command. Cite the source in an HTML comment directly above the tile: `<!-- source: git diff --stat output -->`.
4. **Generate one self-contained HTML file** — all CSS inline in `<style>`, all JS inline in `<script>`. One Google Fonts `<link>` is allowed.
5. **Save to** `docs/explainers/<subject-slug>.html` by default. If `docs/explainers/` doesn't exist, create it. If user specifies a path, use that.
6. **Report back briefly** — one short paragraph: what the doc covers, where it's saved, how to open it in a browser.

---

## Quality checklist

Before saving, verify:

- [ ] One HTML file, parses cleanly (no unclosed tags)
- [ ] Sidebar nav has an entry per visible section
- [ ] Every stat tile backed by real source (cited in HTML comment)
- [ ] At least one SVG diagram
- [ ] Mobile-tolerant (viewport meta, sidebar collapses at <900px)
- [ ] No "TBD"/"TODO" unless explicitly in a Roadmap/Open Questions section
- [ ] No invented quotes or non-existent commits
- [ ] Footer: subject + commit/version/date
- [ ] CSS variables used consistently throughout
- [ ] Hash-based nav works: only one section visible at a time on desktop

---

## What NOT to do

- **No invented numbers** — if you can't get a real stat, omit the tile
- **No light theme** — always dark, always `#020617` background
- **No external CSS frameworks** — Tailwind, Bootstrap, etc. are banned. Inline `<style>` only.
- **Not a status dashboard** — this is a document, not a metrics board. Use `visual-dashboard-generator` skill for dashboards.
- **Don't pad section count** — a 50-line PR does not need 17 sections
- **Don't write process commentary** — no "Now I'll generate…" filler in the HTML
- **No placeholder lorem ipsum** — every sentence earns its place

---

## Tone

Senior engineer writing for a colleague — precise, technical, no filler. Not corporate copy. Rigorous enough to review against, friendly enough to read on a Friday afternoon.

One-sentence test: *Would a skeptical senior engineer find this doc worth the 5 minutes it takes to read?* If no, trim or sharpen.
