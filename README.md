# Design Review Panel

<p align="center">
  <strong>A Review Panel That Sizes Itself to Your Component — and Gives You a Verdict</strong>
</p>

<p align="center">
  A core trio of design reviewers — plus specialists drafted automatically when your code needs them — examine your UI in parallel and deliver a unified, verified, prioritized report ending in an explicit <strong>Block / Needs changes / Approve</strong> verdict.
</p>

<p align="center">
  <a href="#installation">Install</a> ·
  <a href="#usage">Usage</a> ·
  <a href="#the-panel">The Panel</a> ·
  <a href="#example-output">Example</a>
</p>

---

## What This Does

One command assembles a design review panel for your UI code. Three core reviewers always serve; up to six specialists are **auto-drafted when the target shows relevant signals** (chart imports draft the Data-Viz reviewer, dialogs and heavy ARIA draft the Deep Accessibility reviewer, dark-mode classes draft the Token Architect…). There is **no fixed cap on panel size** — the panel is as large as the target warrants.

All reviewers run **in parallel**, each with an **exclusive scope**. Findings are cross-referenced and collapsed to root causes, urgent issues are **adversarially verified** by a second wave of agents that try to refute them against your actual source, and everything lands in one actionable report with evidence citations — headed by a verdict, not just a score.

**Review criteria are distilled from the design-engineering canon:** Emil Kowalski's animation skills (frequency gate, duration budgets, exact easing curves), Jakub Krehel's `better-*` suite (escalation triggers, rendered-pair contrast, forms & focus batteries), Paul Bakaus's Impeccable (named anti-slop patterns), Vercel's Web Interface Guidelines, and Apple HIG / Material 3. **No dependencies** — everything is self-contained.

### Core trio (always on)

| Reviewer | Owns (exclusive) |
|----------|------------------|
| 🎯 **UX Strategist** | Contrast (rendered-pair measurement), touch targets (WCAG 24px floor vs HIG 44px bar), **responsive & 320px/200%-zoom reflow**, forms hardening (paste, autocomplete, submit states), destructive-action safety, empty/error/loading states |
| ✨ **Craft Reviewer** | Composition rhythm, spacing grid, **concentric radius & optical alignment**, M3 + Apple HIG typography (with 45–75ch measure & micro-typography), surface depth with numeric thresholds, **named anti-slop patterns** (ghost card, side-tab, nested cards…), modern CSS |
| 🎬 **Motion & Polish** | **Whether-to-animate gate** (frequency tiers + purpose vocabulary), per-surface duration budgets, canonical bezier curves, springs & bounce rules, gesture physics, `prefers-reduced-motion` done right, asymmetric enter/exit, stagger |

### Specialists (drafted on signal, or via `--add` / `--panel=full`)

| Reviewer | Owns (exclusive) | Drafted when |
|----------|------------------|--------------|
| ♿ **Deep Accessibility** | Focus management (trap/restore/`inert`), roving tabindex & APG patterns, ARIA semantics, live regions, accessible-name computation, forced-colors, 200% zoom | Dialogs/popovers, ≥5 aria-* attributes, composite widgets, form/auth context |
| 🧱 **Token & Theme Architect** | Token architecture (primitive→semantic→component), ramp hygiene, borrowed tokens, `primary` collisions, control tokens, **dark mode & theming** (`light-dark()`, `color-scheme`) | Token files, ≥10 custom properties, or dark-mode markers |
| 🚀 **Performance & Rendering** | Image dimensions/CLS, lazy loading, font loading, `content-visibility`, virtualization (>50 rows), GIF→video, layout-read thrash, expensive paint | Images, video, `@font-face`, long lists, or marketing/landing context |
| 🌍 **i18n & Adaptivity** | Logical properties, RTL flip/never-flip rules, text-expansion tolerance, **message construction & pluralization**, `Intl.*` locale formats | i18n libraries, `dir=`, locale files |
| 🧭 **Content & Microcopy** | CTAs, error/empty-state wording, flow vocabulary, toggle/link labels, voice, tone-by-stakes, **AI-copy tells** | Copy-heavy targets or marketing/auth context |
| 📊 **Data-Viz** | Chart palette math (lightness matching, 15° hue rule), redundant encoding, zero-baseline, chart-internal contrast, tabular numerals | Chart libraries, `<canvas>`, data tables, dashboard context |

Scopes stay exclusive even as the panel grows: when a specialist is drafted, overlapping dimensions **transfer ownership** (e.g., keyboard navigation moves from UX to Deep Accessibility for that run), so consensus remains a true signal, never a duplicate-checklist artifact.

### The Report

- **Verdict first** — 🚫 Block / ⚠️ Needs changes / ✅ Approve, derived from finding tiers, never from the score
- **Severity floor** — eight escalation triggers (no accessible name, no focus indicator, zoom-capped viewport, unguarded destructive action…) are 🔴 Urgent even when only one reviewer catches them
- **🔴 Urgent** — escalation triggers and consensus findings, each **✓-verified** by an adversarial agent that tried to refute it against your source
- **🟡 Recommended / 🟢 Polish** — root-cause-collapsed ("raw hex in 7 files → one missing token" is *one* finding), tagged **[auto-fixable]** or **[judgment]**
- **✅ Strengths** — 2–4 things done well with evidence, plus the single **highest-leverage change**
- **🗑️ Considered but Rejected** — candidates examined and cleared, with the criterion that cleared them (anti-padding: zero findings is a valid result)
- **⚡ Disagreements & Refuted Findings** — never silently deleted
- **Coverage table** — "Clear (inspected, nothing found)" vs "Not drafted" vs "Not reviewed", so you know exactly what wasn't checked
- **Trend & Outcomes** — re-runs read the previous snapshot and give every prior finding an explicit status (Fixed / Still open / Won't fix)

Reports are generated in **English** or **Korean** based on your input language.

---

## Installation

### Plugin (Recommended)

```bash
/install github:LittleGD/review-my-design-plz
```

### Manual

```bash
git clone https://github.com/LittleGD/review-my-design-plz.git
cp -r review-my-design-plz/.claude/* ~/.claude/
```

Restart Claude Code after installation.

---

## Usage

```bash
# Auto-sized panel (core trio + signal-drafted specialists)
/design-review-panel src/components/Button.tsx

# Review only what a change touched — findings become Introduced / Regression / Pre-existing
/design-review-panel src/components/Button.tsx --diff
/design-review-panel src/components/Button.tsx --diff=main

# Quick pre-commit sanity check (5 findings max) vs exhaustive audit
/design-review-panel src/components/Nav.tsx --depth=quick
/design-review-panel src/app/dashboard/page.tsx --depth=deep

# Hunt for MISSING craft instead of judging what exists (capped at 5–7, gate-filtered)
/design-review-panel src/components/Card.tsx --mode=opportunities

# Apply the mechanically-safe fixes after the report
/design-review-panel src/components/Form.tsx --apply

# Panel control
/design-review-panel src/components/Button.tsx --panel=core        # legacy trio
/design-review-panel src/app/dashboard/page.tsx --panel=full       # entire 9-role roster
/design-review-panel src/components/Hero.tsx --only=craft,motion,perf
/design-review-panel src/components/Chart.tsx --add=dataviz,tokens

# Context, contrast, verification
/design-review-panel app/marketing/page.tsx --context=marketing
/design-review-panel src/components/Card.tsx --contrast=apca
/design-review-panel src/components/Nav.tsx --verify=all           # verify 🟡 too
/design-review-panel src/components/Nav.tsx --verify=off

# Screenshots and file sets
/design-review-panel screenshots/dashboard.png --context=dashboard
/design-review-panel "src/components/Card/*.{tsx,css}"
```

### Grammar

```
<path|glob|image> [--only=<list>] [--add=<list>] [--panel=<mode>] [--depth=<mode>] [--diff[=<base>]]
                  [--mode=<mode>] [--apply] [--context=<hint>] [--contrast=<mode>] [--verify=<mode>]
```

- `--panel` — `auto` (default), `core` (trio only), `full` (entire roster)
- `--only` — exact panel from `ux,craft,motion,a11y,tokens,perf,i18n,content,dataviz`
- `--add` — force-draft specialists on top of the panel-mode selection
- `--depth` — `quick` (≤5 findings, 🔴🟡 only), `standard` (default, ≤15), `deep` (uncapped, includes 🟢). Caps never drop escalation-trigger findings
- `--diff[=<base>]` — change-scoped review; pre-existing issues are capped, sectioned apart, and excluded from the verdict
- `--mode` — `review` (default) or `opportunities` (generative hunt with a four-question gate and hard cap)
- `--apply` — apply `[auto-fixable]` findings after the report; `[judgment]` findings always ask first
- `--context` — role hint like `marketing`, `dashboard`, `design-system`, `form`, `landing`, `auth`
- `--contrast` — `wcag` (default) or `apca`
- `--verify` — `urgent` (default), `all`, `off`

### Supported Inputs

- **Source code files** — `.tsx`, `.jsx`, `.vue`, `.svelte`, `.html`, `.css`, etc.
- **Globs / small directories** — up to 5 files reviewed together as one target set
- **Screenshots** — `.png`, `.jpg`, `.webp`, `.gif` (agents Read the image themselves)
- **Large files** (>500 lines) — excerpt inlined, full file Readable on demand
- **Diffs** — `--diff` reviews changed hunks, greps the *removed* side for silently-deleted a11y attributes, and statuses every finding

### Project Probe

Before dispatching reviewers, the skill probes for `tailwind.config.*`, Panda, `styled-system`, `globals.css`, token files, and `package.json` dependencies (≤ 500ms, time-boxed). Breakpoints and tokens found are handed to reviewers; i18n/chart libraries found feed the specialist draft signals. **Tailwind projects are not flagged for "wrong" breakpoints** — the skill adapts.

---

## The Panel

### 🎯 UX Strategist

Grounded in WCAG 2 / APCA, Apple HIG, and Material Design 3:

- **Contrast, measured right** — against the *rendered* background pair (nearest painted ancestor, worst region over images, both extremes under glass), per theme; disabled text still has a floor
- **Touch targets with the right citation** — WCAG 2.5.8's 24px legal floor vs the 44px/48dp usability bar; pseudo-element hit-area extension; `touch-action: manipulation`
- **Focus** — `:focus-visible` not bare `:focus`, no positive tabindex, ring perimeter checked against every adjacent background, `scroll-margin-top` under sticky headers
- **Forms hardening** — never block paste (🔴), `autocomplete`/`inputmode`/`spellcheck`, submit disables *after* the request starts, validate on submit with focus-to-first-error, continuous label+control hit targets, unsaved-state warnings
- **Reflow** — content reachable at 320px width and 200% zoom; `user-scalable=no` is an automatic 🔴
- **Destructive actions** — confirmation or undo required; confirmation overuse flagged too (it trains click-through)
- Live regions, heading hierarchy, skip links; loading / error / empty states

### ✨ Craft Reviewer

Reviews code the way a design lead reviews a junior's work — *"would I put my name on this?"*

- **Concentric radius math** — `outerRadius = innerRadius + padding`, the most common thing that makes interfaces feel subtly off; optical alignment (icon-side padding −2px)
- **Typography System (M3 + Apple HIG)** — role table plus **45–75ch measure**, heading rhythm, micro-typography (`…` not `...`, curly quotes, `text-wrap: balance/pretty`), the `min-w-0` truncation bug
- **Surfaces with numbers** — +7/9/12% elevation steps, 0.05–0.12 border alpha, the 3-layer shadow-as-border recipe that collapses to a single ring in dark mode
- **Named anti-slop patterns** — ghost card, side-tab accent, hero eyebrow chip, nested cards, gradient text, oversized h1, icon-tile-stack… flagged by name
- **Browser surfaces** — unthemed `::selection`, `caret-color`, scrollbars: the cheapest tell that a page was assembled, not built
- Spacing grid, modern CSS (container queries, `:has()`, OKLCH, view transitions), the swap test

### 🎬 Motion & Polish Reviewer

Emil Kowalski's design-engineering lens, now with his full decision framework:

- **The gate comes first** — frequency tiers (100+×/day = no animation, *ever*; keyboard-initiated actions never animate) and a closed purpose vocabulary; can't name a purpose → the fix is deletion, not adjustment
- **Remedies in strict order** — delete → reduce → fix easing → fix origin → make interruptible → move to GPU → asymmetric timing → polish
- **Exact reference values** — per-surface duration budgets (button 100–160ms … drawer 300–500ms) and canonical curves (`cubic-bezier(0.23, 1, 0.32, 1)` enter, iOS drawer curve for sheets); a 400ms drawer is *correct*, not a violation
- **Reduced motion done right** — fewer and gentler, not zero; the global `0.01ms` kill-all is itself flagged
- **Springs** — bounce 0 by default, 0.1–0.3 only when the gesture carried momentum; icon cross-fades at scale 0.25→1 + blur 4px→0
- **Gesture physics** — pointer-down feedback, 1:1 tracking, ~10px hysteresis, velocity-based dismissal (>0.11 px/ms), rubber-banding, interruption from the live value
- WCAG 2.2.2 autoplay controls, blur cost caps, SVG `<g>` transforms, `@starting-style`, View Transitions

### ♿ Deep Accessibility *(specialist)*

Screen-reader depth the generalist doesn't reach: dialog focus trap/restore/`inert`, roving tabindex per APG, accessible-name computation (label/aria mismatches break voice control), live regions that actually announce, no focusable children inside `aria-hidden`, forced-colors survival, widgets operable at 200% zoom.

### 🧱 Token & Theme Architect *(specialist)*

Token architecture and theming: primitive→semantic→component tiers, no borrowed tokens ("the fix is add the token, never reuse the nearest one"), `primary` naming collisions, ≥4 text-hierarchy levels, border progressions, control tokens, ramp hygiene with role mapping, dark mode designed rather than inverted.

### 🚀 Performance & Rendering *(specialist)*

Load-time rendering outside animation: reserved image space (no CLS), lazy/priority loading, `srcset` + modern formats, `font-display` + preload + woff2-only, `content-visibility`, virtualization for >50-item lists, GIF→`<video>`, preconnect hints, no resting `will-change`, batched layout reads, no large-area blur bombs.

### 🌍 i18n & Adaptivity *(specialist)*

Survival under translation and RTL: logical properties, the flip/never-flip icon table (chevrons mirror; logos and play buttons never do), `<bdi>` for mixed-direction values, +30–50% text-expansion tolerance, **no concatenated messages** (`count === 1 ? 'item' : 'items'` fails — many languages have 3–6 plural forms), `Intl.*` formats, Unicode-safe styling.

### 🧭 Content & Microcopy *(specialist)*

The words themselves: verb-led CTAs, errors that say what happened + how to fix it, one term per flow ("Continue" XOR "Next"), toggles that label the ON state, links that name their destination, second-person voice, tone-by-stakes (zero playfulness on destructive confirms), and **AI-copy tells** with firing thresholds (buzzwords, aphoristic cadence at 3+ sections, em-dash saturation).

### 📊 Data-Viz *(specialist)*

Charts and numeric displays: palette math (perceived-lightness matching, the 15° hue-collision rule), redundant non-color encoding, zero-baseline bars, chart-internal contrast (marks ≥3:1, labels ≥4.5:1 — a scope no generalist covers), pie ≤5 categories, tabular numerals, in-frame empty/loading states.

---

## Consolidation: How Findings Become a Verdict

1. **Root-cause collapse** — symptoms sharing one cause merge into a single finding listing every location; the root-cause fix outranks its symptoms
2. **Severity floor** — eight escalation triggers are 🔴 regardless of who (or how many) flagged them: no accessible name · no focus indicator · pointer-only path · ignored `prefers-reduced-motion` · clipped at 320px/200% zoom · failed contrast · color-only meaning · unguarded destructive action
3. **Consensus** — 2+ reviewers citing the same evidence region → 🔴 Urgent
4. **Adversarial verification** — every 🔴 gets its own refuter agent checking the evidence exists, the claim is technically correct, project context doesn't already mitigate it, and the **evidence channel supports the claim** (no "contrast fails" from unrendered source, no "uses transition: all" from a screenshot)
5. **Verdict** — any 🔴 remaining → 🚫 Block; only 🟡/🟢 → ⚠️ Needs changes; nothing actionable + full coverage → ✅ Approve. The median score is trend signal; it can never upgrade a Block

---

## Example Output

### Live Demo

A fictional SaaS landing page (TaskFlow) reviewed and rebuilt using this skill — drag the handle to compare before and after:

**👉 [littlegd.github.io/review-my-design-plz](https://littlegd.github.io/review-my-design-plz/)**

### Sample Report

```
# 🎨 Design Review Panel Results

**Target:** `src/app/dashboard/page.tsx`
**Verdict:** 🚫 Block — 2 escalation-trigger findings (no accessible name, color-only meaning)
**Trend:** Block (5🔴) → this run (2🔴)
**Panel (6 reviewers):** 🎯 UX · ✨ Craft · 🎬 Motion · ♿ A11y · 🧱 Tokens · 📊 Data-Viz
**Drafting:** a11y: dialog + 9 aria attrs · tokens: 14 custom properties · dataviz: recharts import · perf/i18n/content: no signals
**Contrast mode:** wcag · **Verification:** urgent (3 confirmed / 1 refuted) · **Depth:** standard

## Overall

| Reviewer            | Score  | Key Findings                        |
|---------------------|--------|-------------------------------------|
| 🎯 UX Strategist    | 7/10   | zoom capped, submit disable bug     |
| ✨ Craft Reviewer    | 6/10   | ghost cards, no measure             |
| 🎬 Motion & Polish  | 5/10   | keyboard action animated            |
| ♿ Deep A11y        | 5.5/10 | icon buttons unnamed, focus lost    |
| 🧱 Token & Theme    | 6.5/10 | primitives in components            |
| 📊 Data-Viz         | 5.5/10 | series 12° apart, no zero baseline  |
| **Overall (median of 6)** | **5.75/10** |                        |

## 🔴 Urgent Fixes
1. ⛔✓ [judgment] Icon-only toolbar buttons have no accessible name — A11y, L88–L104
2. ⛔✓ [auto-fixable] Chart series distinguished by color alone, 12° hue apart — Data-Viz, L142
3. ✓ [auto-fixable] Command-palette open animates 250ms on a keyboard action — Motion + UX (same region), L57

## ✅ Strengths
- Consistent 4/8 spacing grid throughout (all 34 spacing values on-grid)
- Drawer uses the iOS curve at 400ms — correct budget for the surface, L203

**Highest-leverage change:** add semantic series tokens to the chart theme — it resolves
both the color-only encoding and the primitives-in-components findings at once.

## 🗑️ Considered but Rejected
- transform-origin: center on the settings modal — exempt, modals stay centered (Motion)
- 13px metadata caption — maps to project token --text-caption (Craft)

## Coverage
| Reviewer | Findings | Status |
|----------|----------|--------|
| 🌍 i18n  | —        | Not drafted: no i18n signals |
| 🚀 Perf  | —        | Not drafted: no image/font signals |
...
```

---

## How It Works

1. You invoke `/design-review-panel` with a file path, glob, screenshot, or `--diff`
2. The skill probes project conventions (breakpoints, tokens, dependencies) and prior snapshots
3. The panel is composed: core trio + specialists whose signals fire (no fixed cap)
4. All reviewers launch **in parallel**, each with an exclusive, transfer-adjusted scope and a shared contract (injection defense, evidence-channel honesty, considered-but-rejected)
5. Findings are root-cause-collapsed, floored against escalation triggers, and consensus-checked; 🔴 findings get an **adversarial verification wave**
6. A verdict-first report is generated in your language; `--apply` then fixes the safe half

```
┌─────────────────────────────────────────┐
│  /design-review-panel target.tsx --diff │
└──────────────┬──────────────────────────┘
               ▼
      ┌─────────────────┐
      │  Project probe  │──► draft signals + prior snapshot
      └────────┬────────┘
    ┌─────┬────┼────┬───────┬────────┬─ ── ─┐
    ▼     ▼    ▼    ▼       ▼        ▼      ▼
┌──────┐┌─────┐┌──────┐┌──────┐┌────────┐┌ ─ ─ ┐
│🎯 UX ││✨Cra ││🎬 Mo ││♿A11y││🧱Tokens││📊 ⋯
└──┬───┘└──┬──┘└──┬───┘└──┬───┘└───┬────┘└ ─┬─ ┘
   └───────┴──────┼───────┴────────┴────────┘
                  ▼
        ┌───────────────────────┐
        │ Consolidate:          │
        │ root-cause collapse   │
        │ + severity floor      │
        └────────┬──────────────┘
                 ▼
        ┌──────────────────┐
        │ Verify wave (🔴) │  one refuter per finding
        └────────┬─────────┘
                 ▼
        ┌────────────────────┐
        │ 🚫/⚠️/✅ Verdict    │
        │ Report + snapshot  │
        └────────────────────┘
```

---

## License

MIT

---

<p align="center">
  <sub>Built for designers and engineers who believe every detail matters.</sub>
</p>
