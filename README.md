# Design Review Panel

<p align="center">
  <strong>A Review Panel That Sizes Itself to Your Component</strong>
</p>

<p align="center">
  A core trio of design reviewers — plus specialists drafted automatically when your code needs them — examine your UI in parallel and deliver a unified, verified, prioritized report.
</p>

<p align="center">
  <a href="#installation">Install</a> ·
  <a href="#usage">Usage</a> ·
  <a href="#the-panel">The Panel</a> ·
  <a href="#example-output">Example</a>
</p>

---

## What This Does

One command assembles a design review panel for your UI code. Three core reviewers always serve; up to five specialists are **auto-drafted when the target shows relevant signals** (chart imports draft the Data-Viz reviewer, dark-mode classes draft the Token Architect, i18n libraries draft the i18n reviewer…). There is **no fixed cap on panel size** — the panel is as large as the target warrants.

All reviewers run **in parallel**, each with an **exclusive scope**. Findings are cross-referenced, consensus issues are **adversarially verified** by a second wave of agents that try to refute them against your actual source, and everything lands in one actionable report with evidence citations.

**No dependencies.** All review criteria are self-contained — you don't need to install any other skills or plugins.

### Core trio (always on)

| Reviewer | Owns (exclusive) |
|----------|------------------|
| 🎯 **UX Strategist** | Accessibility (WCAG / APCA), touch targets, **3-tier responsive breakpoints** (M3 window size classes), forms, keyboard navigation, empty/error/loading states |
| ✨ **Craft Reviewer** | Composition rhythm, spacing grid, **M3 + Apple HIG typography system**, surface depth, CSS structure, intent vs defaults, **modern CSS** (container queries, `:has()`, CSS nesting, OKLCH, view transitions, `@layer`) |
| 🎬 **Motion & Polish** | Animation decisions, easing curves, duration, hardware acceleration, `prefers-reduced-motion`, hover media queries, scroll-driven animations, asymmetric enter/exit, stagger |

### Specialists (drafted on signal, or via `--add` / `--panel=full`)

| Reviewer | Owns (exclusive) | Drafted when |
|----------|------------------|--------------|
| 🧱 **Token & Theme Architect** | Token architecture (primitive→semantic→component), semantic color tokens, z-index scale, **dark mode & theming** (`light-dark()`, `color-scheme`), token naming | Token files, ≥10 custom properties, or dark-mode markers |
| 🚀 **Performance & Rendering** | Image dimensions/CLS, lazy loading, `fetchpriority`, font loading, `content-visibility`, resting `will-change`, expensive paint | Images, video, `@font-face`, or marketing/landing context |
| 🌍 **i18n & Adaptivity** | Logical properties, RTL safety, text-expansion tolerance, truncation, `Intl.*` locale formats, `lang` attributes | i18n libraries, `dir=`, locale files |
| 🧭 **Content & Microcopy** | Wording of CTAs/errors/empty states, capitalization system, placeholder copy, tone, progressive feedback | Copy-heavy targets or marketing/auth context |
| 📊 **Data-Viz** | Chart color scales, redundant encoding, axes/zero-baseline, legends, tabular numerals, number formatting | Chart libraries, `<canvas>`, data tables, dashboard context |

Scopes stay exclusive even as the panel grows: when a specialist is drafted, overlapping dimensions **transfer ownership** (e.g., semantic color tokens move from UX to the Token Architect for that run), so "2+ reviewers agree" remains a true consensus signal, never a duplicate-checklist artifact.

### The Report

- **Panel composition** — which reviewers served and *why* each specialist was or wasn't drafted
- **Project context detected** — Tailwind / Panda / design-token probes surface the project's real breakpoints and tokens, so reviewers adapt instead of imposing defaults
- **🔴 Urgent Fixes** — issues flagged by 2+ reviewers, each **✓-verified** by an adversarial agent that tried to refute it against your source
- **🟡 Recommended** — important issues from a single reviewer
- **🟢 Polish** — minor improvements
- **⚡ Disagreements & Refuted Findings** — conflicting opinions and any finding the verification wave shot down (never silently deleted)
- **Action Plan** — numbered items ordered by priority
- **Scope caveats** — what each reviewer couldn't evaluate, plus which specialists were *not* drafted so you know what wasn't checked

Every finding cites evidence (line range or region) and a canonical dimension tag. Overall score is the **median of all participating reviewers** (robust to outliers at any panel size). Reports are generated in **English** or **Korean** based on your input language.

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

# Legacy 3-person behavior
/design-review-panel src/components/Button.tsx --panel=core

# Run the entire 8-role roster
/design-review-panel src/app/dashboard/page.tsx --panel=full

# Exact panel of your choosing
/design-review-panel src/components/Hero.tsx --only=craft,motion,perf

# Force-draft specialists on top of auto selection
/design-review-panel src/components/Chart.tsx --add=dataviz,tokens

# Role hint (weights criteria AND feeds draft signals)
/design-review-panel app/marketing/page.tsx --context=marketing

# APCA contrast instead of WCAG 2
/design-review-panel src/components/Card.tsx --contrast=apca

# Verify recommended findings too, or skip verification
/design-review-panel src/components/Nav.tsx --verify=all
/design-review-panel src/components/Nav.tsx --verify=off

# Review a screenshot
/design-review-panel screenshots/dashboard.png --context=dashboard

# Review a small set of files together
/design-review-panel "src/components/Card/*.{tsx,css}"
```

### Grammar

```
<path|glob|image> [--only=<list>] [--add=<list>] [--panel=<mode>] [--context=<hint>] [--contrast=<mode>] [--verify=<mode>]
```

- `--panel` — `auto` (default), `core` (trio only), `full` (entire roster)
- `--only` — exact panel from `ux,craft,motion,tokens,perf,i18n,content,dataviz`
- `--add` — force-draft specialists on top of the panel-mode selection
- `--context` — role hint like `marketing`, `dashboard`, `design-system`, `form`, `landing`, `auth`
- `--contrast` — `wcag` (default) or `apca`
- `--verify` — `urgent` (default: adversarially verify consensus findings), `all`, `off`

### Supported Inputs

- **Source code files** — `.tsx`, `.jsx`, `.vue`, `.svelte`, `.html`, `.css`, etc.
- **Globs / small directories** — up to 5 files reviewed together as one target set
- **Screenshots** — `.png`, `.jpg`, `.webp`, `.gif` (agents Read the image themselves)
- **Large files** (>500 lines) — first 250 + last 50 lines inlined, full file Readable on demand (never inlined once per agent)

### Project Probe

Before dispatching reviewers, the skill probes for `tailwind.config.*`, Panda, `styled-system`, `globals.css`, token files, and `package.json` dependencies (≤ 500ms, time-boxed). Breakpoints and tokens found are handed to reviewers; i18n/chart libraries found feed the specialist draft signals. **Tailwind projects are not flagged for "wrong" breakpoints** — the skill adapts.

---

## The Panel

### 🎯 UX Strategist

Grounded in WCAG 2 / APCA, Apple HIG, and Material Design 3 window size classes:

- **Contrast** — WCAG (4.5:1 text, 3:1 large/UI) or APCA (Lc 75 body, Lc 60 large, Lc 45 non-text) via `--contrast` flag
- Touch targets (≥ 44×44px Apple HIG / ≥ 48dp Material, ≥ 8px spacing)
- Keyboard navigation, focus indicators, semantic alt text & aria-labels
- Loading / error / empty states, visible form labels with required indicators
- **Responsive design** — Mobile ≤ 599 / Tablet 600–839 / Desktop 840–1199 / Large ≥ 1200 (M3 window size classes). Uses **your project's breakpoints** when the probe finds them. Layout must adapt, not just shrink; hamburger requires `aria-expanded` + ESC + body scroll lock

### ✨ Craft Reviewer

Reviews code the way a design lead reviews a junior's work — *"would I put my name on this?"*

- Layout rhythm and intentional proportions (the focal-point test)
- **Spacing grid** — every value a multiple of 4 (Tailwind `p-[13px]` fails)
- **Typography System (M3 + Apple HIG synthesis)** — every text style maps to a *role* (Display / Headline / Title / Body / Label / Caption), not a raw px value. Body ≥ 16px on mobile, real italic cuts only, multi-axis hierarchy beyond size alone
- Surface depth through tonal shifts — the *border-removal test*; one committed depth strategy
- CSS structure quality — no negative-margin hacks, no calc() workarounds, no absolute-position escapes
- **Modern CSS adoption (2025–2026 baseline)** — container queries, `:has()`, `:is()`/`:where()`, CSS nesting, OKLCH / Display-P3, View Transitions API, `@layer` — flagged only where the project would meaningfully benefit
- The *swap test* — would defaults feel any different?

### 🎬 Motion & Polish Reviewer

Based on [Emil Kowalski's](https://emilkowal.ski/) design engineering principles — 17-point animation checklist:

- Easing selection (ease-out for entries, ease-in for exits, custom curves for punch)
- Duration limits (<300ms for UI elements; hero/marketing may exceed deliberately)
- Hardware acceleration (`transform`, `opacity`, `filter`, `clip-path` only)
- Button `:active` states, hover media queries, `prefers-reduced-motion` support
- Asymmetric enter/exit timing, stagger (30–80ms), View Transitions, scroll-driven animations
- Output uses **Before | After | Why** table format with per-row dimension, severity, and confidence

### 🧱 Token & Theme Architect *(specialist)*

Token architecture and theming: primitive→semantic→component tiers, no palette primitives in components, declared z-index scale, one coherent dark-mode mechanism with `color-scheme` and complete theming (shadows and borders adapt too, not just background), no hard-coded inversion-breakers, state variants derived from tokens.

### 🚀 Performance & Rendering *(specialist)*

Load-time rendering outside animation: reserved image space (no CLS), correct lazy/priority loading, `srcset` + modern formats, `font-display` + preload, `content-visibility` on long sections, no resting `will-change`, no large-area blur/backdrop-filter paint bombs, sane DOM depth, video posters.

### 🌍 i18n & Adaptivity *(specialist)*

Survival under translation and RTL: logical properties, flipped directional icons, +30–50% text-expansion tolerance, truncation with full-value affordances, `Intl.*` locale formats, correct `lang` attributes, no text in images, Unicode-safe styling (no letter-spacing on CJK, careful `text-transform`).

### 🧭 Content & Microcopy *(specialist)*

The words themselves: verb-led CTAs, error messages that say what happened and how to fix it, empty states that sell the next action, one capitalization convention, placeholders as examples not instructions, consistent tone, no raw jargon, progressive feedback ("Saving…" → "Saved").

### 📊 Data-Viz *(specialist)*

Charts and numeric displays: intentional categorical palettes, sequential scales for ordered data, redundant (non-color-alone) encoding, zero-baseline bars, labeled axes with units, direct labeling over legends, tabular numerals with right-aligned columns, restrained data-ink, in-frame empty/loading chart states.

---

## Adversarial Verification

After consolidation, every 🔴 Urgent consensus finding is handed to its own verifier agent whose only job is to **refute it** against the actual source: does the cited evidence exist, is the claim technically correct, does project context already mitigate it? Confirmed findings get a ✓; refuted ones move to the ⚡ section with the refutation shown — never silently deleted. `--verify=all` extends this to 🟡 Recommended findings; `--verify=off` skips the wave.

---

## Example Output

### Live Demo

A fictional SaaS landing page (TaskFlow) reviewed and rebuilt using this skill — drag the handle to compare before and after:

**👉 [littlegd.github.io/review-my-design-plz](https://littlegd.github.io/review-my-design-plz/)**

| | Before | After |
|---|---|---|
| 🎯 UX Strategist | 4/10 | 9/10 |
| ✨ Craft Reviewer | 4/10 | 8.8/10 |
| 🎬 Motion & Polish | 4/10 | 8.8/10 |
| **Overall** | **4.0/10** | **8.9/10** |

### Sample Report

```
# 🎨 Design Review Panel Results

**Target:** `src/app/dashboard/page.tsx`
**Panel (5 reviewers):** 🎯 UX · ✨ Craft · 🎬 Motion · 🧱 Tokens · 📊 Data-Viz
**Drafting:** tokens: 14 custom properties + dark: classes · dataviz: recharts import · perf/i18n/content: no signals
**Contrast mode:** wcag · **Verification:** urgent (3 confirmed / 1 refuted)

## Overall

| Reviewer            | Score | Key Findings                       |
|---------------------|-------|------------------------------------|
| 🎯 UX Strategist    | 7/10  | missing focus ring, no aria-label  |
| ✨ Craft Reviewer    | 6/10  | default spacing, flat hierarchy    |
| 🎬 Motion & Polish  | 5/10  | no active state, ease-in used      |
| 🧱 Token & Theme    | 6.5/10| primitives in components           |
| 📊 Data-Viz         | 5.5/10| rainbow palette, no zero baseline  |
| **Overall (median of 5)** | **6.0/10** |                       |

## 🔴 Urgent Fixes (2+ reviewers agree)
1. ✓ No `:active` / press feedback — flagged by Craft + Motion
2. ✓ Chart series colors: palette primitives with no semantic mapping — flagged by Tokens + Data-Viz

## ⚡ Refuted Findings
- ~~"Missing prefers-reduced-motion"~~ — verifier found a global guard in app/globals.css L112

...
```

---

## How It Works

1. You invoke `/design-review-panel` with a file path, glob, or screenshot
2. The skill probes project conventions (breakpoints, tokens, dependencies)
3. The panel is composed: core trio + specialists whose signals fire (no fixed cap)
4. All reviewers launch **in parallel**, each with an exclusive, transfer-adjusted scope
5. Findings are cross-referenced by dimension tag; consensus issues get an **adversarial verification wave**
6. A unified report is generated in your language

```
┌────────────────────────────────────┐
│  /design-review-panel target.tsx   │
└──────────────┬─────────────────────┘
               ▼
      ┌─────────────────┐
      │  Project probe  │──► draft signals
      └────────┬────────┘
    ┌─────┬────┼────┬───────┬─ ── ── ─┐
    ▼     ▼    ▼    ▼       ▼         ▼
┌──────┐┌─────┐┌──────┐┌────────┐┌ ─ ─ ─ ┐
│🎯 UX ││✨Cra ││🎬 Mo ││🧱Tokens││📊 ⋯ 🚀
└──┬───┘└──┬──┘└──┬───┘└───┬────┘└ ─ ┬ ─ ┘
   └───────┴──────┼────────┴─────────┘
                  ▼
        ┌──────────────────┐
        │  Consolidation   │
        └────────┬─────────┘
                 ▼
        ┌──────────────────┐
        │ Verify wave (🔴) │  one refuter per finding
        └────────┬─────────┘
                 ▼
        ┌──────────────────┐
        │  Unified Report  │
        │  🔴✓ 🟡 🟢 ⚡     │
        └──────────────────┘
```

---

## License

MIT

---

<p align="center">
  <sub>Built for designers and engineers who believe every detail matters.</sub>
</p>
