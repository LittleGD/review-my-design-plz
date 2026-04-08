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

One command triggers three parallel design reviews of your UI code, each from a different expert perspective. The results are cross-referenced and consolidated into a single actionable report.

**No dependencies.** All review criteria are self-contained — you don't need to install any other skills or plugins.

| Reviewer | Focus |
|----------|-------|
| 🎯 **UX Strategist** | Accessibility, touch targets, **3-tier responsive breakpoints** (Mobile/Tablet/Desktop), forms, keyboard navigation, semantic color tokens |
| ✨ **Craft Reviewer** | Composition rhythm, spacing grid, **M3 + Apple HIG typography system**, surface depth, CSS structure, intentional design decisions |
| 🎬 **Motion & Polish** | Animation decisions, easing curves, performance, `:active` states, `prefers-reduced-motion`, hardware acceleration |

### The Report

- **🔴 Urgent Fixes** — issues flagged by 2+ reviewers (consensus)
- **🟡 Recommended** — important issues from a single reviewer
- **🟢 Polish** — minor improvements
- **⚡ Disagreements** — conflicting opinions with both sides' reasoning
- **Action Plan** — numbered items ordered by priority

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
# Review a specific file
/design-review-panel src/components/Button.tsx

# Review without arguments (will ask for target)
/design-review-panel

# Works with any UI file
/design-review-panel app/dashboard/page.tsx
```

### Supported Inputs

- **Source code files** — `.tsx`, `.jsx`, `.vue`, `.svelte`, `.html`, `.css`, etc.
- **Screenshots** — `.png`, `.jpg`, `.webp`, `.gif` (visual review mode)
- **Large files** (>500 lines) — will ask whether to review entirely or by section

---

## Reviewers

### 🎯 UX Strategist

A 16-point checklist grounded in WCAG, Apple HIG, and Material Design 3:

- WCAG contrast ratios (4.5:1 normal, 3:1 large)
- Touch targets (≥ 44×44px on mobile/tablet, ≥ 8px spacing)
- Keyboard navigation, focus rings, semantic alt text & aria-labels
- Loading / error / empty states, visible form labels
- **3-tier responsive design** — explicit `Mobile ≤ 599px`, `Tablet 600–1023px`, `Desktop ≥ 1024px` breakpoints (M3 window size classes), layout *adapts* not just *shrinks*, working hamburger with `aria-expanded` + ESC + body-scroll-lock, padding/margin steps per breakpoint, no `768px` catch-all
- Spacing system on a 4 / 8px grid, semantic color tokens, consistent z-index scale
- `prefers-reduced-motion` respected

### ✨ Craft Reviewer

Reviews code the way a design lead reviews a junior's work — *"would I put my name on this?"*

- Layout rhythm and intentional proportions (the focal-point test)
- **Typography System (M3 + Apple HIG synthesis)** — every text style maps to a *role* (Display / Headline / Title / Body / Label / Caption), not a raw px value. Reference scale spans Mobile/Tablet/Desktop with paired sizes + line-heights + weights + tracking. Body ≥ 16px on mobile (iOS auto-zoom + HIG Dynamic Type baseline). Tabular numerals on data, real italic cuts only, multi-axis hierarchy beyond size alone.
- Surface depth through tonal shifts — the *border-removal test*
- One committed depth strategy (no mixing borders + shadows + surface shifts)
- CSS structure quality — no negative-margin hacks, no calc() workarounds, no absolute-position escapes
- The *swap test* — would defaults feel any different?

### 🎬 Motion & Polish Reviewer

Based on [Emil Kowalski's](https://emilkowal.ski/) design engineering principles — 16-point animation checklist:

- Easing selection (ease-out for entries, custom curves for punch)
- Duration limits (<300ms for UI elements)
- Hardware acceleration (`transform` + `opacity` only)
- Button `:active` states, hover media queries
- `prefers-reduced-motion` support, asymmetric enter/exit timing, stagger
- Output uses **Before | After | Why** table format

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
