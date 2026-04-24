# Design Review Panel

<p align="center">
  <strong>Three Expert Eyes on Every Component</strong>
</p>

<p align="center">
  Three specialized design reviewers examine your UI code in parallel and deliver a unified, prioritized report.
</p>

<p align="center">
  <a href="#installation">Install</a> ·
  <a href="#usage">Usage</a> ·
  <a href="#reviewers">Reviewers</a> ·
  <a href="#example-output">Example</a>
</p>

---

## What This Does

One command triggers up to three parallel design reviews of your UI code, each from a different expert perspective with an **exclusive scope**. The results are cross-referenced and consolidated into a single actionable report with evidence citations.

**No dependencies.** All review criteria are self-contained — you don't need to install any other skills or plugins.

| Reviewer | Owns (exclusive) |
|----------|------------------|
| 🎯 **UX Strategist** | Accessibility (WCAG / APCA), touch targets, **3-tier responsive breakpoints** (Mobile/Tablet/Desktop, M3 window size classes), forms, keyboard navigation, empty/error/loading states, semantic color tokens, z-index system |
| ✨ **Craft Reviewer** | Composition rhythm, spacing grid, **M3 + Apple HIG typography system**, surface depth, CSS structure, intent vs defaults, **modern CSS** (container queries, logical properties, OKLCH, `:has()`, CSS nesting, view transitions, `@layer`) |
| 🎬 **Motion & Polish** | Animation decisions, easing curves, duration, hardware acceleration, `prefers-reduced-motion`, hover media queries, scroll-driven animations, asymmetric enter/exit, stagger |

Scope overlap has been deliberately removed so "2+ reviewers agree" is a true consensus signal, not an artifact of duplicate checklists.

### The Report

- **Project context detected** — Tailwind / Panda / design-token probes surface the project's real breakpoints and tokens, so reviewers adapt instead of imposing defaults
- **🔴 Urgent Fixes** — issues flagged by 2+ reviewers (canonical-dimension consensus)
- **🟡 Recommended** — important issues from a single reviewer
- **🟢 Polish** — minor improvements
- **⚡ Disagreements** — conflicting opinions with both sides' reasoning
- **Action Plan** — numbered items ordered by priority
- **Scope caveats** — what each reviewer couldn't evaluate (N/A) so you know what *wasn't* checked

Every finding cites evidence (line range or region). Overall score is the **median** of three reviewer scores (robust to one outlier), not a naive mean. Reports are generated in **English** or **Korean** based on your input language.

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
# Review a specific file with all three reviewers
/design-review-panel src/components/Button.tsx

# Review without arguments (will ask for target)
/design-review-panel

# Run only selected reviewers
/design-review-panel src/components/Hero.tsx --only=craft,motion

# Pass a role hint so criteria weight appropriately
/design-review-panel app/marketing/page.tsx --context=marketing

# Use APCA contrast instead of WCAG 2
/design-review-panel src/components/Card.tsx --contrast=apca

# Review a screenshot
/design-review-panel screenshots/dashboard.png --context=dashboard
```

### Grammar

```
<path> [--only=<list>] [--context=<hint>] [--contrast=<mode>]
```

- `--only` — subset of `ux,craft,motion` (default: all three)
- `--context` — role hint like `marketing`, `dashboard`, `design-system`, `form`, `landing`, `auth`
- `--contrast` — `wcag` (default) or `apca`

### Supported Inputs

- **Source code files** — `.tsx`, `.jsx`, `.vue`, `.svelte`, `.html`, `.css`, etc.
- **Screenshots** — `.png`, `.jpg`, `.webp`, `.gif` (agents Read the image themselves)
- **Large files** (>500 lines) — first 250 + last 50 lines inlined, full file Readable on demand (not inlined three times)

### Project Probe

Before dispatching reviewers, the skill probes for `tailwind.config.*`, Panda, `styled-system`, `globals.css`, and token files (≤ 500ms, time-boxed). If breakpoints or tokens are found, reviewers use them. Otherwise they fall back to M3 + HIG defaults. This means **Tailwind projects are not flagged for "wrong" breakpoints** — the skill adapts.

---

## Reviewers

### 🎯 UX Strategist

Grounded in WCAG 2 / APCA, Apple HIG, and Material Design 3 window size classes:

- **Contrast** — WCAG (4.5:1 text, 3:1 large/UI) or APCA (Lc 75 body, Lc 60 large, Lc 45 non-text) via `--contrast` flag
- Touch targets (≥ 44×44px Apple HIG / ≥ 48dp Material, ≥ 8px spacing)
- Keyboard navigation, focus indicators, semantic alt text & aria-labels
- Loading / error / empty states, visible form labels with required indicators
- **Responsive design** — Mobile ≤ 599 / Tablet 600–839 / Desktop 840–1199 / Large ≥ 1200 (M3 compact/medium/expanded/large window size classes). Uses **your project's breakpoints** when the probe finds them (Tailwind, Panda, etc.). Layout must adapt, not just shrink; hamburger requires `aria-expanded` + ESC + body scroll lock
- Semantic color tokens over raw hex, consistent z-index scale

### ✨ Craft Reviewer

Reviews code the way a design lead reviews a junior's work — *"would I put my name on this?"*

- Layout rhythm and intentional proportions (the focal-point test)
- **Spacing grid** — every value a multiple of 4 (Tailwind `p-[13px]` fails)
- **Typography System (M3 + Apple HIG synthesis)** — every text style maps to a *role* (Display / Headline / Title / Body / Label / Caption), not a raw px value. Reference scale spans Mobile/Tablet/Desktop with paired sizes + line-heights + weights + tracking. Body ≥ 16px on mobile (iOS auto-zoom + HIG Dynamic Type baseline). Tabular numerals on data, real italic cuts only, multi-axis hierarchy beyond size alone, variable-font axes where available.
- Surface depth through tonal shifts — the *border-removal test*
- One committed depth strategy (no mixing borders + shadows + surface shifts)
- CSS structure quality — no negative-margin hacks, no calc() workarounds, no absolute-position escapes
- **Modern CSS adoption (2024–2026 baseline)** — flags missing use of container queries (`@container`), logical properties (`margin-inline`, `padding-block`), `:has()`, `:is()`/`:where()`, CSS nesting, OKLCH / Display-P3, View Transitions API, and `@layer` cascade control — but only where the project would meaningfully benefit
- The *swap test* — would defaults feel any different?

### 🎬 Motion & Polish Reviewer

Based on [Emil Kowalski's](https://emilkowal.ski/) design engineering principles — 17-point animation checklist:

- Easing selection (ease-out for entries, ease-in for exits, custom curves for punch)
- Duration limits (<300ms for UI elements; hero/marketing may exceed deliberately)
- Hardware acceleration (`transform`, `opacity`, `filter`, `clip-path` only)
- Button `:active` states, hover media queries (`@media (hover: hover) and (pointer: fine)`)
- `prefers-reduced-motion` support, asymmetric enter/exit timing, stagger (30–80ms)
- **View Transitions API** and scroll-driven animations where relevant
- No animations on 100+×/day shortcuts (save, submit) — or drop to <50ms
- Output uses **Before | After | Why** table format with per-row severity + confidence

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

### What the *after* version actually applied

The same content, restructured with everything the panel surfaced:

- **Responsive system** — explicit Mobile/Tablet/Desktop breakpoints (599 / 1023 / 1024) with layouts that *re-compose* (asymmetric features grid 5+7 → 4+4+4 → 12-span), navigation pattern that swaps to a real hamburger with `aria-expanded`, ESC-to-close and body scroll lock, and section padding that steps `80 → 96 → 128px`.
- **Typography system** — Fraunces (display) + Inter (text) on the M3+HIG scale: Display L `36 → 48 → 57px`, Body L `16 → 16 → 17px` with paired line-heights and tracking that turns negative on display sizes. Italic emphasis on key words uses the real Fraunces italic cut. Pricing uses `font-variant-numeric: tabular-nums lining-nums`.
- **Motion** — `cubic-bezier(0.23, 1, 0.32, 1)` on UI transitions, `:active { transform: scale(0.97) }` everywhere clickable, `IntersectionObserver` reveal with geometric stagger sorted by `getBoundingClientRect`, `prefers-reduced-motion` honored, `<noscript>` fallback so animated content is visible without JS.

### Sample Report

```
# 🎨 Design Review Panel Results

**Target:** `src/components/Button.tsx`
**Reviewers:** UX Strategist · Craft Reviewer · Motion & Polish Reviewer

## Overall Score

| Reviewer           | Score | Key Findings              |
|--------------------|-------|---------------------------|
| 🎯 UX Strategist   | 7/10  | missing focus ring, no aria-label |
| ✨ Craft Reviewer   | 6/10  | default spacing, flat hierarchy   |
| 🎬 Motion & Polish | 5/10  | no active state, ease-in used     |
| **Overall**        | **6.0/10** |                          |

## 🔴 Urgent Fixes (2+ reviewers agree)
1. No `:active` / press feedback — flagged by Craft + Motion reviewers
2. Missing `prefers-reduced-motion` — flagged by UX + Motion reviewers

## 🟡 Recommended Improvements
3. Add visible focus ring for keyboard users (UX)
4. Typography relies on size alone — add weight variation (Craft)

...
```

---

## Reference Tables Embedded in the Skill

### Responsive breakpoints (M3 window size classes)

| Breakpoint | Range          | M3 class  | Notes                                                  |
|------------|----------------|-----------|--------------------------------------------------------|
| Mobile     | ≤ 599px        | compact   | Hamburger nav, single column, body ≥ 16px              |
| Tablet     | 600 – 1023px   | medium    | 2-col grids, condensed nav, padding steps up           |
| Desktop    | ≥ 1024px       | expanded  | Full inline nav, multi-col layouts, generous padding   |

### Typography reference scale (M3 + Apple HIG synthesis)

| Role        | Mobile (px / lh) | Tablet      | Desktop     | Weight  | Tracking  |
|-------------|------------------|-------------|-------------|---------|-----------|
| Display L   | 36 / 44          | 48 / 56     | 57 / 64     | 400–600 | -0.022em  |
| Headline L  | 28 / 36          | 32 / 40     | 36 / 44     | 600     | -0.018em  |
| Title L     | 20 / 28          | 22 / 30     | 22 / 30     | 600     | -0.005em  |
| Body L      | 16 / 24          | 16 / 24     | 17 / 26     | 400     | 0         |
| Body M      | 14 / 20          | 14 / 20     | 15 / 22     | 400     | 0         |
| Label L     | 14 / 20          | 14 / 20     | 14 / 20     | 500–600 | 0.01em    |
| Caption     | 12 / 16          | 12 / 16     | 13 / 18     | 400     | 0.005em   |

The Craft Reviewer flags any `--font-18` style raw-size variable, demands `font-variant-numeric: tabular-nums` on numeric data, and rejects synthesized italic when no italic cut exists.

---

## How It Works

1. You invoke `/design-review-panel` with a file path
2. The skill reads your source code
3. Three Agent subprocesses launch **in parallel** — each with embedded review criteria
4. Results are cross-referenced for consensus and conflicts
5. A unified report is generated in your language

```
┌─────────────────────────────────┐
│  /design-review-panel file.tsx  │
└──────────────┬──────────────────┘
               │
    ┌──────────┼──────────┐
    ▼          ▼          ▼
┌────────┐ ┌────────┐ ┌────────┐
│ 🎯 UX  │ │ ✨Craft│ │🎬Motion│
│Strategy│ │Reviewer│ │& Polish│
└───┬────┘ └───┬────┘ └───┬────┘
    │          │          │
    └──────────┼──────────┘
               ▼
    ┌─────────────────────┐
    │  Unified Report     │
    │  🔴 🟡 🟢 ⚡        │
    └─────────────────────┘
```

---

## License

MIT

---

<p align="center">
  <sub>Built for designers and engineers who believe every detail matters.</sub>
</p>
