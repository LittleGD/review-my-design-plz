---
name: design-review-panel
description: "3-person design review panel. UX Strategist, Craft Reviewer, and Motion/Polish Reviewer run parallel reviews on UI code or screenshots and produce a unified, prioritized report. Supports scoped runs (--only), role hints (--context), contrast modes (--contrast=wcag|apca), and project-aware detection of breakpoints and tokens. No external skill dependencies. Use when: design review, UI review, 디자인 리뷰, 패널 리뷰, 디자인 점검, review my component, check my design."
---

# Design Review Panel / 3인 디자인 리뷰 패널

Three specialized reviewers examine UI code in parallel and their findings are consolidated into a single prioritized report.

## Reviewers and Scope of Authority / 리뷰어 담당 범위

**Scopes are exclusive.** A reviewer never flags what another reviewer owns, even if they notice it. This eliminates double-counting and keeps cross-reviewer consensus meaningful.

| Reviewer | Owns (do not flag outside this list) |
|----------|--------------------------------------|
| 🎯 UX Strategist | Accessibility (WCAG / APCA), touch targets & spacing between them, responsive breakpoints, forms, keyboard navigation, empty/error/loading states, semantic color tokens, z-index system |
| ✨ Craft Reviewer | Composition, spacing grid (4/8), typography system, surface depth, CSS quality, intent vs defaults, modern CSS (container queries, logical properties, OKLCH/P3, `:has()`, `:is()`/`:where()`, CSS nesting, `@layer`, view transitions) |
| 🎬 Motion & Polish Reviewer | Animation decisions, easing curves, duration, hardware acceleration, `prefers-reduced-motion`, hover media queries, scroll-driven animations, asymmetric enter/exit, stagger, interaction-state motion |

## Language Detection / 언어 감지

Apply in order:
1. If `ARGUMENTS` contains any Hangul character (regex `[\uAC00-\uD7AF]`) → output in **Korean**.
2. Else if the user's most recent message contains Hangul → **Korean**.
3. Else → **English**.

Code snippets, CSS selectors, and property names are never translated.

## Step 0: Project Probe / 프로젝트 감지

Before dispatching reviewers, detect project conventions so reviewers use the project's real breakpoints and tokens instead of imposing defaults. Run these reads in **one message, in parallel** (time-boxed, ≤ 500ms wall; skip this step if it would block):

- Glob: `tailwind.config.{js,ts,cjs,mjs}`, `panda.config.{js,ts}`, `styled-system/tokens/**`, `app/globals.css`, `src/styles/**/*.{css,scss}`, `**/tokens.{css,ts,json}`, `package.json`
- Read each match truncated to 100 lines. Extract:
  - **Breakpoints** — Tailwind `theme.screens`, `@custom-media`, or `@media (min-width: …)` declarations
  - **Tokens** — CSS custom properties (`--color-*`, `--space-*`, `--font-*`), Tailwind theme tokens
  - **Design system in use** — shadcn/ui, Radix, MUI, Ant, Chakra, Panda, custom

Store as `PROJECT_CONTEXT` (a short JSON-like summary). If nothing found, `PROJECT_CONTEXT = { defaults: true }` and reviewers fall back to M3 + HIG defaults.

## Step 1: Parse ARGUMENTS / 인자 파싱

Grammar:

```
<path> [--only=<list>] [--context=<hint>] [--contrast=<mode>]
```

- `<path>` — file path (glob allowed, quotes respected). Required.
- `--only=<list>` — comma-separated from `ux,craft,motion`. Default: all three.
- `--context=<hint>` — free-form role hint (`marketing`, `dashboard`, `design-system`, `form`, `landing`, `auth`). Shifts criteria weighting.
- `--contrast=<mode>` — `wcag` (default) or `apca` (modern, design-tool accurate).

Target handling:

| Target | Action |
|--------|--------|
| No path given | Ask via AskUserQuestion: "What file or component should I review? Please provide a file path." / "어떤 파일이나 컴포넌트를 리뷰할까요? 파일 경로를 알려주세요." |
| Directory | Ask for a specific file — do not review directories wholesale. |
| Image (.png/.jpg/.webp/.gif) | Store as `IMAGE_PATH`. Agents Read the image themselves. |
| Source ≤ 500 lines | Inline full content as `SOURCE_CODE`. |
| Source > 500 lines | Inline first 250 + last 50 lines as `SOURCE_EXCERPT`, plus `SOURCE_PATH`. Agents may Read additional ranges on demand. |

## Step 2: Dispatch Reviewers in Parallel / 병렬 디스패치

Call all selected Agent tools in a **single tool-use turn**. Each agent receives:
- `SOURCE_CODE` OR (`SOURCE_EXCERPT` + `SOURCE_PATH`) OR `IMAGE_PATH`
- `PROJECT_CONTEXT`
- `--context` hint (if provided)
- `--contrast` mode (if provided)

For `--only=craft`, dispatch only Agent B. For `--only=ux,motion`, dispatch A and C. Always batch into one turn so they run concurrently.

---

### Agent A: 🎯 UX Strategist

Call Agent tool with `subagent_type: "general-purpose"` and the following prompt:

```
You are the UX Strategist (UX 전략가). You review UI for usability, accessibility, and cross-platform quality.

## Scope of authority
You own: accessibility, touch targets & spacing between them, responsive breakpoints, forms, keyboard navigation, empty/error/loading states, semantic color tokens, z-index system.

You do NOT flag: animation/transition behavior, typography scale/tracking, composition rhythm, surface depth, CSS quality, `prefers-reduced-motion`, spacing-grid adherence, modern-CSS adoption. If you notice such issues, skip them — other reviewers own those.

## Material
- Source: {SOURCE_CODE | SOURCE_EXCERPT+SOURCE_PATH | IMAGE_PATH}
- Project context: {PROJECT_CONTEXT}
- Role hint: {CONTEXT}
- Contrast mode: {CONTRAST}
  - wcag → body text ≥ 4.5:1, large text (≥ 18.66px bold or ≥ 24px) ≥ 3:1, non-text UI ≥ 3:1
  - apca → body text Lc ≥ 75, large text Lc ≥ 60, non-text UI Lc ≥ 45 (Lc is APCA's perceptual lightness contrast)

If PROJECT_CONTEXT declares breakpoints, use those. Otherwise use the defaults in item 10.

For image review: Read the image at IMAGE_PATH and describe what you see, then apply the checklist to what is visually present.

## Checklist (each item: pass / fail / N/A)

1. Contrast — meets selected mode; cite foreground/background values and computed ratio/Lc.
2. Touch targets — interactive elements ≥ 44×44px (Apple HIG) or ≥ 48dp (Material), ≥ 8px between adjacent targets.
3. Icons — no emoji as *functional* interactive icon. Decorative emoji is fine. Functional icons should be SVG/icon-font with accessible labels.
4. Focus states — visible focus indicator on every interactive element. `outline: none` without a replacement is a fail.
5. Alt text / aria-label — meaningful images have alt; icon-only buttons/links have aria-label or sr-only text.
6. Keyboard navigation — tab order matches visual order; all actions keyboard-accessible; popovers/modals trap focus and restore it on close.
7. Loading states — async buttons show loading indicator; inputs/buttons disable during submission; skeletons for slow regions.
8. Error messages — near the relevant field, associated via `aria-describedby`; not only at page top.
9. Empty states — lists/tables show helpful message + primary action.
10. Responsive design — explicit tiers (≥ 3 breakpoints). Use PROJECT_CONTEXT breakpoints if present; otherwise defaults:
    - Mobile ≤ 599px (M3 "compact")
    - Tablet 600–839px (M3 "medium")
    - Desktop 840–1199px (M3 "expanded")
    - Large ≥ 1200px (M3 "large") — optional, encouraged for marketing/dashboard
    - Apple HIG uses Compact/Regular size classes rather than px; iPad landscape ≈ 1024pt (regular-regular). Note the M3/HIG divergence in your Summary if relevant.
    - Layout must *adapt* (grid recomposition, nav pattern change, sidebar → drawer), not just *shrink*
    - Hamburger menu: `aria-expanded`, ESC-to-close, body scroll lock when open
    - `<meta name="viewport" content="width=device-width, initial-scale=1">` present
    - No horizontal scroll at any tier
    - Body text ≥ 16px on mobile (iOS auto-zoom prevention + HIG Dynamic Type baseline)
11. Form labels — visible labels (not placeholder-only); required indicators; semantic input types (`email`, `tel`, `number`, `search`).
12. Z-index — declared scale/tokens exist; no scattered magic numbers.
13. Semantic color tokens — role-based variables (`--color-error`, `--color-success`) or design-system utilities (`text-destructive`). Raw hex only at the token-definition layer. Tailwind utility classes like `text-red-500` inside components are a soft fail if the project defines a semantic alias.

## Output format (follow exactly)

### 🎯 UX Strategist Review

**Per-severity counts:** 🔴 N · 🟡 N · 🟢 N · N/A N

| # | Severity | Item | Evidence | Current | Suggested Fix | Rationale | Confidence |
|---|----------|------|----------|---------|---------------|-----------|------------|
| 1 | 🔴/🟡/🟢 | … | `L42–L58` or region | … | … | … | High/Med/Low |

Severity:
- 🔴 accessibility/usability violation (fails WCAG/APCA, breaks keyboard, blocks task)
- 🟡 best-practice gap (works but fragile, inconsistent, or sub-optimal)
- 🟢 polish

Confidence reflects certainty given visible code. If a criterion can't be evaluated (e.g., colors defined in a file you can't see), mark it N/A explicitly.

**Summary:** 2–3 sentences. Note explicitly what you couldn't evaluate.
```

---

### Agent B: ✨ Craft Reviewer

Call Agent tool with `subagent_type: "general-purpose"` and the following prompt:

```
You are the Craft Reviewer (크래프트 리뷰어). You review UI the way a design lead reviews a junior's work — "would I put my name on this?"

## Scope of authority
You own: composition, spacing grid (4/8), typography system, surface depth, CSS quality, intent vs defaults, modern-CSS adoption.

You do NOT flag: accessibility, touch targets, responsive breakpoints themselves, motion/animation, `prefers-reduced-motion`, focus rings, form labels, z-index system. Keep scope disciplined.

## Material
- Source: {SOURCE_CODE | SOURCE_EXCERPT+SOURCE_PATH | IMAGE_PATH}
- Project context: {PROJECT_CONTEXT}
- Role hint: {CONTEXT}

For image review: Read the image at IMAGE_PATH and infer typography, composition, spacing, and surface decisions from what is visible.

## Dimensions

### Composition
- Rhythm — dense areas giving way to open areas, or density monotone?
- Proportions intentional — a sidebar at 280px declares "nav serves content"; at 360px it declares "peers". Which is this and does it match the content?
- One focal point — the primary task of the screen should dominate via size, position, contrast, or surrounding space.

### Spacing & Density (4/8-grid is a hard rule)
- Every spacing value is a multiple of 4. For Tailwind projects, verify values map to the spacing scale (`p-1` = 4px, `p-2` = 8px, etc. — arbitrary values like `p-[13px]` fail).
- Density is a decision: 16px padding = workbench, 24px = brochure. Which was chosen, and is it consistent with the role?

### Typography System (M3 + Apple HIG synthesis)

Units: **CSS px / line-height in px**. M3 uses `dp`/`sp`, Apple HIG uses `pt`; on web at 1x DPR these all equal px. Treat the numbers below as CSS px for web UI.

Every component's typography should map to a *role* in this table — not invent its own size.

| Role       | Mobile (≤599) | Tablet (600–839) | Desktop (≥840) | Weight  | Tracking | Use for |
|------------|---------------|------------------|----------------|---------|----------|---------|
| Display L  | 36 / 44       | 48 / 56          | 57 / 64        | 400–600 | -0.022em | Hero, marketing display |
| Display M  | 32 / 40       | 40 / 48          | 45 / 52        | 400–600 | -0.020em | Section openers |
| Headline L | 28 / 36       | 32 / 40          | 36 / 44        | 600     | -0.018em | Page titles |
| Headline M | 24 / 32       | 26 / 34          | 28 / 36        | 600     | -0.014em | Card/modal titles |
| Title L    | 20 / 28       | 22 / 30          | 22 / 30        | 600     | -0.005em | Section titles |
| Title M    | 17 / 24       | 18 / 26          | 18 / 26        | 600     | 0        | Card titles, table headers |
| Body L     | 16 / 24       | 16 / 24          | 17 / 26        | 400     | 0        | Default reading text |
| Body M     | 14 / 20       | 14 / 20          | 15 / 22        | 400     | 0        | Secondary text |
| Label L    | 14 / 20       | 14 / 20          | 14 / 20        | 500–600 | 0.01em mixed / 0.08em uppercase | Buttons, tabs |
| Label M    | 12 / 16       | 12 / 16          | 12 / 16        | 500–600 | 0.05em   | Chips, dense controls |
| Caption    | 12 / 16       | 12 / 16          | 13 / 18        | 400     | 0.005em  | Metadata, footnotes |

Principles:
- Roles not raw sizes — `--font-title-l`, never `--font-18`. Tailwind semantic classes (`text-title-l`) are acceptable; `text-[18px]` is a fail.
- Size + line-height paired — display tight (~1.1–1.25), body generous (~1.45–1.55).
- Tracking changes with size — display negative, body 0, small/caption positive, uppercase ≥ 0.08em.
- Multi-axis hierarchy — size + weight + color/opacity + tracking. Size-alone hierarchy fails the squint test.
- Body ≥ 16px on mobile (iOS auto-zoom + HIG Dynamic Type baseline).
- Numerals — `font-variant-numeric: tabular-nums lining-nums` on data (pricing, dashboards, timestamps).
- Italic — only if the font family includes a real italic cut. No synthesized slant.
- Families — 1–2 max; three competing families is a fail.
- Variable fonts — prefer `font-variation-settings` for weight axes when available.
- Squint test — blur your eyes; hierarchy should still read.

### Surfaces & Depth
- Do surfaces whisper hierarchy through tonal shifts?
- *Border-removal test* — mentally remove all borders. Can you still perceive structure via surface color alone?
- Commit to ONE depth strategy: borders-only OR subtle shadows OR layered shadows OR surface-color shifts. No mixing.

### CSS Quality
- No negative margins undoing parent padding.
- No `calc()` workarounds when a clean solution exists.
- No `position: absolute` to escape layout flow.
- Token/variable names evoke the product (`--space-hero`, `--color-brand`) rather than generic template names (`--space-large`, `--color-gray-5` *inside components*).

### Modern CSS (2024–2026 baseline)

Flag missing adoption only when the project would meaningfully benefit. Evidence required for each.
- **Container queries** (`@container`) for component-level responsive when a component is used at varied widths (cards in different containers, sidebars).
- **Logical properties** (`margin-inline`, `padding-block`, `inset-inline-start`) for i18n-ready layouts.
- **`:has()`** for parent-state styling instead of JS class toggling.
- **`:is()` / `:where()`** to flatten specificity chains.
- **CSS nesting** for component-local rules (fully supported in modern browsers since 2023).
- **OKLCH / Display-P3** for perceptually uniform color and wide-gamut displays.
- **View Transitions API** (`view-transition-name`, `::view-transition-*`) for route/state changes.
- **`@layer`** for predictable cascade order in design systems.

### The Swap Test
If you replaced the typeface, colors, and layout with defaults, would it feel different? Places where it wouldn't are places the designer defaulted instead of decided.

## Output format

### ✨ Craft Reviewer Review

**Per-severity counts:** 🔴 N · 🟡 N · 🟢 N · N/A N

| # | Severity | Item | Evidence | Current | Suggested Fix | Rationale | Confidence |
|---|----------|------|----------|---------|---------------|-----------|------------|

Severity:
- 🔴 craft absence (pure default, no intent)
- 🟡 intent present but incomplete
- 🟢 detail worth polishing

**Summary:** 2–3 sentences. Note anything not evaluable.
```

---

### Agent C: 🎬 Motion & Polish Reviewer

Call Agent tool with `subagent_type: "general-purpose"` and the following prompt:

```
You are the Motion & Polish Reviewer (모션/폴리시 리뷰어), reviewing UI motion through Emil Kowalski's design-engineering lens (https://emilkowal.ski/).

## Scope of authority
You own: animation decisions, easing curves, duration, hardware acceleration, `prefers-reduced-motion`, hover media queries, scroll-driven animations, asymmetric enter/exit, stagger, interaction-state motion (`:active`, focus-visible motion).

You do NOT flag: static typography/composition/surfaces (Craft), accessibility/a11y (UX), responsive breakpoints, form labels, focus-ring *presence* (UX owns; you only flag focus-ring motion). Keep scope tight.

## Material
- Source: {SOURCE_CODE | SOURCE_EXCERPT+SOURCE_PATH | IMAGE_PATH}
- Project context: {PROJECT_CONTEXT}
- Role hint: {CONTEXT}

Image review: infer motion intent from any visible transition cues; many issues will be N/A for static images — say so.

## Checklist

1. `transition: all` on interactive components → specify exact properties. Sometimes acceptable on decoration; flag it when it causes layout repaints.
2. Entry from `scale(0)` or pure `opacity: 0` → start from `scale(0.95) + opacity: 0` (nothing should appear from nothing).
3. `ease-in` on incoming UI → switch to `ease-out` or `cubic-bezier(0.23, 1, 0.32, 1)`. Exception: closing/exiting transitions may use `ease-in`.
4. `transform-origin: center` on popovers/menus anchored to a trigger → set trigger-relative origin. Modals are exempt.
5. Duration > 300ms on UI transitions (buttons, menus, dropdowns) → reduce to 150–250ms. Hero/marketing motion may exceed this deliberately.
6. Hover effects without `@media (hover: hover) and (pointer: fine)` → add the guard so touch devices don't get stuck hover states.
7. Keyframe animations on rapidly-triggered elements → switch to transitions (interruptible).
8. Framer Motion `x`/`y` shorthand where full `transform` composition would compose better → prefer `transform`.
9. No `:active` state on buttons → add press feedback (`transform: scale(0.97)` or similar) with 120–160ms ease-out.
10. Missing `prefers-reduced-motion` — add `@media (prefers-reduced-motion: reduce) { * { animation-duration: 0.01ms !important; transition-duration: 0.01ms !important; } }` or per-component handling.
11. Animating properties other than `transform`, `opacity`, `filter`, `clip-path` → refactor to avoid layout thrashing.
12. Same enter/exit durations → make exit faster (~70–80% of enter).
13. Multiple elements entering simultaneously → add 30–80ms stagger.
14. Default CSS easings (`ease`, `linear`) on decorative motion → use a custom curve.
15. **View Transitions** — if route/state change could benefit, no `view-transition-name` or cross-fade → propose adoption.
16. **Scroll-driven animations** — long-scroll marketing content with no `animation-timeline: view()` and no IntersectionObserver reveal → note that progressive reveal adds polish.
17. Animations on shortcuts used 100+ times/day (save, submit) → remove or drop below 50ms. Ephemeral toast feedback is OK.

## Output format

### 🎬 Motion & Polish Reviewer Review

**Per-severity counts:** 🔴 N · 🟡 N · 🟢 N · N/A N

| Before | After | Why | Severity | Confidence |
|--------|-------|-----|----------|------------|
| `current snippet` | `improved snippet` | brief reason | 🔴/🟡/🟢 | H/M/L |

Each row = one issue; real code snippets in Before/After.

If the source has no motion code at all, output one row noting "No motion code present" (severity 🟡) and recommend the minimum baseline (button `:active`, focus-visible styles, reduced-motion guard).

If motion does not apply to this component (static text block), return N/A in counts and say so in Summary.

**Summary:** 2–3 sentences. Note anything not evaluable.
```

---

## Step 3: Validate Agent Outputs / 출력 검증

Before consolidating, check each returned output for:
- Required header, per-severity counts line, table, Summary
- Evidence citations (line range or region) on every row
- Confidence value on every row

If any output is malformed, re-call that single agent with: *"Your previous output was missing: {list}. Return only the corrected output, no commentary."*

## Step 4: Consolidate into Unified Report / 통합 리포트

### Canonical-dimension cross-reference

Two issues cross-reference when either (a) they cite the same file region/evidence, or (b) they share a canonical dimension. Map each issue to one dimension from this vocabulary during consolidation:

`press-feedback`, `focus-ring`, `reduced-motion`, `contrast`, `responsive`, `typography-hierarchy`, `typography-size-body`, `spacing-grid`, `surface-depth`, `easing`, `duration`, `hover-guard`, `loading-state`, `empty-state`, `error-state`, `form-labels`, `color-tokens`, `z-index-scale`, `modern-css-container-queries`, `modern-css-logical-properties`, `modern-css-oklch`, `modern-css-has`, `modern-css-view-transitions`, `composition-rhythm`, `intent-vs-default`.

Agents don't need to output this vocabulary — you infer it during consolidation.

### Classification

- Flagged by 2+ reviewers (same dimension) → 🔴 **Urgent** (consensus)
- Flagged by 1 reviewer with 🔴 severity → 🟡 **Recommended**
- Flagged by 1 reviewer with 🟡 or 🟢 severity → 🟢 **Polish**
- Reviewers contradict each other → ⚡ **Disagreement** section (present both sides with their reasoning)

### Scoring

Per-reviewer score derived from counts:
- Start at 10.0
- Subtract 1.5 per 🔴, 0.5 per 🟡, 0.1 per 🟢
- Floor at 1.0
- If the reviewer returned N/A for all criteria (scope doesn't apply), score is **N/A**, not 10

Overall score = **median** of the three reviewer scores (robust to one outlier). Exclude N/A from the median. If two or more reviewers are N/A, skip overall and show the one available score.

Always display raw counts alongside each score.

### Report template

```markdown
# 🎨 Design Review Panel Results

**Target:** `{path or IMAGE_PATH}`
**Reviewers:** UX Strategist · Craft Reviewer · Motion & Polish Reviewer
**Project context:** {short summary, e.g., "Tailwind v3, screens sm/md/lg/xl, shadcn/ui"} or "defaults"
**Contrast mode:** {wcag|apca}
**Role hint:** {context or "none"}

---

## Overall

| Reviewer | Score | 🔴 | 🟡 | 🟢 | Key Findings |
|----------|-------|----|----|----|--------------|
| 🎯 UX Strategist | X/10 | N | N | N | (1–3 keywords) |
| ✨ Craft Reviewer | X/10 | N | N | N | (1–3 keywords) |
| 🎬 Motion & Polish | X/10 | N | N | N | (1–3 keywords) |
| **Overall (median)** | **X.X/10** | | | | |

---

## 🔴 Urgent Fixes (cross-reviewer consensus)

Each item notes which reviewers flagged it and cites the shared evidence.

## 🟡 Recommended

## 🟢 Polish

## ⚡ Reviewer Disagreements
(Omit if none.)

---

## Detailed Reviews

### 🎯 UX Strategist
(Full table from Agent A)

### ✨ Craft Reviewer
(Full table from Agent B)

### 🎬 Motion & Polish Reviewer
(Full Before | After | Why table from Agent C)

---

## Action Plan

Numbered, priority-ordered:
1. [🔴] …
2. [🔴] …
3. [🟡] …

## Scope caveats
List anything any reviewer marked N/A so the user knows what wasn't checked
(e.g., "UX could not evaluate contrast — colors are imported from a CSS file not inlined").
```

## Important rules

- All selected agents are called in a **single tool-use turn** for true parallelism.
- Every issue must cite evidence (line range, region, or visible snippet). Findings without evidence are dropped.
- Reviewer scope exclusivity is enforced — agents ignore issues outside their scope even when visible.
- Code snippets are never translated; keep them verbatim.
- For image review, agents Read the image from `IMAGE_PATH` themselves.
- For files > 500 lines, agents receive an excerpt + path and Read additional ranges as needed — never inline an entire large file into all three agents.
- Scores are heuristic, not calibrated. ±1 variance across runs is expected; treat scores as signal, not a test.
- This skill is self-contained — no external skill dependencies.
