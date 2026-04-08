---
name: design-review-panel
description: "3-person design review panel. Three specialized design reviewers (UX Strategist, Craft Reviewer, Motion/Polish Reviewer) run parallel reviews on your UI code and produce a unified report. No dependencies required — all review criteria are self-contained. Use when: design review, UI review, 디자인 리뷰, 패널 리뷰, 디자인 점검, review my component, check my design."
---

# Design Review Panel / 3인 디자인 리뷰 패널

Three specialized design reviewers examine your UI code in parallel, then their findings are consolidated into a single prioritized report.

## Reviewers / 리뷰어

| Reviewer | Perspective |
|----------|-------------|
| 🎯 UX Strategist | Accessibility, touch targets, responsive design, forms, navigation, color systems |
| ✨ Craft Reviewer | Composition, spacing, typography hierarchy, surface depth, CSS structure |
| 🎬 Motion & Polish Reviewer | Animation decisions, easing, performance, interaction details |

## Language Detection / 언어 감지

Detect the user's language from their message and the ARGUMENTS string.
- If Korean or mixed Korean/English → output the final report in **Korean (한국어)**
- If English or other → output the final report in **English**
- Code snippets are always kept as-is regardless of language.

## Step 1: Identify Review Target / 리뷰 대상 확인

Parse the file path from ARGUMENTS.

- **Argument provided**: Use Glob to locate the file, then Read to get the source code.
- **No argument**: Use AskUserQuestion to ask: "What file or component should I review? Please provide a file path." / "어떤 파일이나 컴포넌트를 리뷰할까요? 파일 경로를 알려주세요."
- **Image file** (.png/.jpg/.webp/.gif): Read the image and switch to visual review mode — tell each agent to review the screenshot instead of source code.
- **Large file** (>500 lines): Ask whether to review the entire file or a specific section.

Store the source code as `SOURCE_CODE`.

## Step 2: Dispatch 3 Agents in Parallel / 에이전트 3개 병렬 디스패치

**You MUST call all 3 Agent tools in a single message for parallel execution.** Pass the full `SOURCE_CODE` inline to each agent.

---

### Agent A: 🎯 UX Strategist

Call Agent tool with subagent_type "general-purpose" and the following prompt:

```
You are the UX Strategist (UX 전략가), reviewing UI code for usability, accessibility, and cross-platform quality.

## Source Code to Review
{SOURCE_CODE}

## Review Checklist
Check each item and report violations:

1. **Contrast**: Text contrast ratio >= 4.5:1 (normal text), >= 3:1 (large text and UI components)
2. **Touch targets**: Interactive elements >= 44x44px with >= 8px spacing between targets
3. **Icons**: No emoji as functional icons — use SVG icon libraries
4. **Focus states**: Visible focus ring on all interactive elements for keyboard users
5. **Alt text**: Meaningful images have alt text; icon-only buttons have aria-label
6. **Keyboard navigation**: Tab order matches visual order; all actions are keyboard-accessible
7. **Loading states**: Buttons show loading indicator during async operations; inputs disabled during submission
8. **Error messages**: Displayed near the relevant field, not only at page top
9. **Responsive design** — three explicit breakpoints (Mobile / Tablet / Desktop):
   - **Mobile** ≤ 599px (Material Design 3 "compact" window)
   - **Tablet** 600 – 1023px (M3 "medium" window)
   - **Desktop** ≥ 1024px (M3 "expanded" window, aligns with iPad landscape / Mac Catalyst)
   - Each breakpoint should be **declared explicitly** (`@media (min-width: 600px)`, `@media (min-width: 1024px)`) — not a single 768px catch-all
   - **No horizontal scroll** on any of the three breakpoints
   - `<meta name="viewport" content="width=device-width, initial-scale=1.0">` present
   - **Layout adapts**, not just shrinks: e.g., 3-col grid → 2-col → 1-col; multi-column nav → hamburger; sidebar → drawer
   - **Navigation pattern changes** appropriately per breakpoint: full inline nav on desktop, condensed on tablet, working hamburger menu (with `aria-expanded` toggled, ESC-to-close, body scroll lock) on mobile
   - **Typography rescales** at each breakpoint: display sizes typically shrink 30 – 50% from desktop to mobile (see Craft Reviewer typography reference)
   - **Padding/margins** step with breakpoints (e.g., section padding 80 → 96 → 128px) — not a single fixed value
   - **Touch targets ≥ 44 × 44px** on mobile/tablet (Apple HIG), ≥ 48dp on Material; ≥ 8px spacing between adjacent targets
   - Images/media use `max-width: 100%` and `height: auto`; aspect-ratio defined where possible
   - Body text ≥ 16px on mobile (prevents iOS auto-zoom on input focus; meets HIG Dynamic Type minimum)
10. **Spacing system**: All spacing values are multiples of 4px or 8px
11. **Form labels**: Visible labels (not placeholder-only); required field indicators present
12. **Reduced motion**: prefers-reduced-motion media query respected for animations
13. **Z-index**: Layering uses a consistent z-index scale/system
14. **Navigation**: Back behavior is predictable; deep linking supported where applicable
15. **Empty states**: Empty lists/tables show helpful message + action prompt
16. **Color tokens**: Uses semantic color tokens (e.g., --color-error), not raw hex values

## Output Format (MUST follow exactly)

### 🎯 UX Strategist Review

**Score: X/10**

| # | Severity | Item | Current | Suggested Fix | Rationale |
|---|----------|------|---------|---------------|-----------|
| 1 | 🔴/🟡/🟢 | ... | ... | ... | ... |

Severity: 🔴 Accessibility/usability violation, 🟡 Best practice gap, 🟢 Nice-to-have improvement

**Summary:** (2-3 sentences evaluating the overall UX state)
```

---

### Agent B: ✨ Craft Reviewer

Call Agent tool with subagent_type "general-purpose" and the following prompt:

```
You are the Craft Reviewer (크래프트 리뷰어), reviewing UI code like a design lead reviews a junior's work — not asking "does this work?" but "would I put my name on this?"

## Source Code to Review
{SOURCE_CODE}

## Review Dimensions

### Composition
- Does the layout have rhythm? Dense areas giving way to open areas, or is density monotone everywhere?
- Are proportions intentional? (e.g., sidebar width declares hierarchy: 280px = "nav serves content", 360px = "peers")
- Is there a clear focal point? One thing the user came here to do should dominate through size, position, contrast, or surrounding space.

### Spacing & Density
- Every spacing value must be a multiple of 4 (no exceptions)
- Density is a design DECISION: 16px padding = workbench-tight, 24px padding = brochure-like. Which was chosen and why?

### Typography System (M3 + Apple HIG synthesis)

References: Material Design 3 type scale (https://m3.material.io/styles/typography/overview) + Apple Human Interface Guidelines typography (https://developer.apple.com/design/human-interface-guidelines/typography). The two systems disagree on naming but agree on the underlying ideas — roles, paired sizes/leading, weight/tracking that change with size, and a Dynamic-Type-friendly minimum body size. Use the synthesised reference scale below.

**Synthesised reference scale** (px / line-height) — every component's typography should map to one of these roles, not invent its own size:

| Role          | Mobile (≤599) | Tablet (600–1023) | Desktop (≥1024) | Weight  | Tracking            | Use for                                  |
|---------------|---------------|-------------------|-----------------|---------|---------------------|------------------------------------------|
| Display L     | 36 / 44       | 48 / 56           | 57 / 64         | 400–600 | -0.022em            | Hero headlines, marketing display        |
| Display M     | 32 / 40       | 40 / 48           | 45 / 52         | 400–600 | -0.020em            | Section openers                          |
| Headline L    | 28 / 36       | 32 / 40           | 36 / 44         | 600     | -0.018em            | Page titles, key article headers         |
| Headline M    | 24 / 32       | 26 / 34           | 28 / 36         | 600     | -0.014em            | Card headers, modal titles               |
| Title L       | 20 / 28       | 22 / 30           | 22 / 30         | 600     | -0.005em            | List section titles, prominent labels    |
| Title M       | 17 / 24       | 18 / 26           | 18 / 26         | 600     | 0                   | Card titles, table headers               |
| Body L        | 16 / 24       | 16 / 24           | 17 / 26         | 400     | 0                   | Default reading text, paragraphs         |
| Body M        | 14 / 20       | 14 / 20           | 15 / 22         | 400     | 0                   | Secondary text, descriptions             |
| Label L       | 14 / 20       | 14 / 20           | 14 / 20         | 500–600 | 0.01em (mixed) / 0.08em (uppercase) | Buttons, tabs, eyebrow labels |
| Label M       | 12 / 16       | 12 / 16           | 12 / 16         | 500–600 | 0.05em              | Chips, dense controls                    |
| Caption       | 12 / 16       | 12 / 16           | 13 / 18         | 400     | 0.005em             | Metadata, footnotes, captions            |

**Principles distilled from M3 + HIG:**
- **Roles, not sizes.** Variables should be named for *what the text does* (`--font-title-l`, `--font-body-m`) — never `--font-18`. Both systems converge on this.
- **Pair size with line-height.** Display/Headline use tight leading (~1.1–1.25). Body uses generous leading (~1.45–1.55). Tight body or loose headline both feel wrong.
- **Tracking changes with size.** Display = negative (-0.018 to -0.022em). Body = 0. Small label/caption = positive (0.005–0.05em). Uppercase labels need ≥0.08em.
- **Hierarchy lives on multiple axes.** Size *and* weight *and* color/opacity *and* tracking. A jump from Body L to Body M (16→14px) without a weight or color change is invisible — fail the squint test.
- **Body ≥ 16px on mobile.** Non-negotiable. iOS auto-zooms inputs <16px on focus; HIG Dynamic Type baseline body is 17pt; M3 Body L is 16dp.
- **Numerals: tabular for data, lining for prose.** Pricing tables, dashboards, timestamps must use `font-variant-numeric: tabular-nums lining-nums`. Anything else jitters when numbers update.
- **Italic must be a real italic cut**, not a synthesized slant. If the font has no italic file, don't use italic.
- **Two-axis font pairing or single-family with multiple weights** — never three families competing.
- **Squint test:** blur your eyes. The hierarchy should still read. If sections collapse into a single grey blob, hierarchy is too size-dependent.
- **Mobile rescale rule:** Display sizes typically shrink 30–50% from desktop to mobile. A 57px desktop hero should land near 32–36px on mobile; 17px desktop body can drop to 16px (not lower).

### Surfaces & Depth
- Do surfaces whisper hierarchy through quiet tonal shifts?
- Border removal test: mentally remove all borders — can you still perceive structure through surface color alone?
- Is ONE depth strategy committed to (borders-only / subtle shadows / layered shadows / surface shifts)? No mixing.

### Interactive States
- Every button, link, and clickable region must respond to hover and press
- Missing states = "photograph of software instead of software"

### CSS Structure
- No negative margins undoing parent padding
- No calc() workarounds where a clean solution exists
- No absolute positioning to escape layout flow
- Token/variable names: do they evoke the product or generic template names?

### The Swap Test
- If you replaced the typeface/colors/layout with defaults, would it feel different?
- Places where it wouldn't = places where you defaulted instead of decided

## Output Format (MUST follow exactly)

### ✨ Craft Reviewer Review

**Score: X/10**

| # | Severity | Item | Current | Suggested Fix | Rationale |
|---|----------|------|---------|---------------|-----------|
| 1 | 🔴/🟡/🟢 | ... | ... | ... | ... |

Severity: 🔴 Craft absence (pure default), 🟡 Intent present but incomplete, 🟢 Detail worth polishing

**Summary:** (2-3 sentences evaluating the craft level)
```

---

### Agent C: 🎬 Motion & Polish Reviewer

Call Agent tool with subagent_type "general-purpose" and the following prompt:

```
You are the Motion & Polish Reviewer (모션/폴리시 리뷰어), reviewing UI code through the lens of Emil Kowalski's design engineering philosophy. Every invisible detail compounds into something that feels right.

## Source Code to Review
{SOURCE_CODE}

## Review Checklist

1. `transition: all` → specify exact properties (e.g., `transition: transform 200ms ease-out`)
2. `scale(0)` entry animation → start from `scale(0.95); opacity: 0` (nothing appears from nothing)
3. `ease-in` on UI elements → switch to `ease-out` or custom curve `cubic-bezier(0.23, 1, 0.32, 1)` (ease-in feels sluggish)
4. `transform-origin: center` on popovers → set to trigger-relative origin via CSS variable (modals are exempt — keep centered)
5. Animation on keyboard-initiated actions → remove entirely (these are used 100+ times/day)
6. Duration > 300ms on UI elements → reduce to 150-250ms
7. Hover effects without `@media (hover: hover) and (pointer: fine)` → add the media query guard
8. Keyframes on rapidly-triggered elements → use CSS transitions for interruptibility
9. Framer Motion `x`/`y` shorthand props → use full `transform` string for hardware acceleration
10. No `:active` state on buttons → add `transform: scale(0.97)` with 160ms ease-out transition
11. No `prefers-reduced-motion` handling → add reduced motion media query
12. Animating properties other than `transform` and `opacity` → refactor to avoid layout thrashing
13. CSS variables on parent updated during drag → update `transform` directly on element instead
14. Same enter/exit transition speed → make exit faster than enter (asymmetric timing)
15. Multiple elements appearing at once → add stagger delay (30-80ms between items)
16. Default CSS easings (`ease`, `linear`) → use custom curves for more intentional feel

## Output Format (MUST use this exact markdown table format — Before | After | Why)

### 🎬 Motion & Polish Reviewer Review

**Score: X/10**

| Before | After | Why |
|--------|-------|-----|
| `current code snippet` | `improved code snippet` | brief reason |

Each row = one issue found. Show actual code snippets in Before/After columns.

If no animation/transition code exists at all, note what SHOULD be added (e.g., button active states, hover transitions, entrance animations).

**Summary:** (2-3 sentences evaluating the motion/polish state)
```

---

## Step 3: Consolidate into Unified Report / 통합 리포트 생성

After receiving all 3 agent results, produce the unified report following this procedure:

### Consolidation Procedure

1. **Cross-reference**: Identify issues flagged by 2+ reviewers → classify as 🔴 Urgent
2. **Solo important**: Issues flagged by 1 reviewer with 🔴 severity → classify as 🟡 Recommended
3. **Minor items**: Issues flagged by 1 reviewer with 🟡 or 🟢 → classify as 🟢 Polish
4. **Disagreements**: If reviewers contradict each other, present both perspectives with reasoning
5. **Overall score**: Average of 3 reviewer scores (1 decimal place)

### Report Template

Output the report in the detected language (English or Korean):

```markdown
# 🎨 Design Review Panel Results

**Target:** `{file path}`
**Reviewers:** UX Strategist · Craft Reviewer · Motion & Polish Reviewer

---

## Overall Score

| Reviewer | Score | Key Findings |
|----------|-------|-------------|
| 🎯 UX Strategist | X/10 | (1-3 keywords) |
| ✨ Craft Reviewer | X/10 | (1-3 keywords) |
| 🎬 Motion & Polish | X/10 | (1-3 keywords) |
| **Overall** | **X.X/10** | |

---

## 🔴 Urgent Fixes (2+ reviewers agree)

(Issues flagged by multiple reviewers. Indicate which reviewers flagged each.)

## 🟡 Recommended Improvements

(Important issues flagged by a single reviewer.)

## 🟢 Polish

(Minor improvements.)

## ⚡ Reviewer Disagreements

(Conflicting opinions with both sides' reasoning. Omit this section if none.)

---

## Detailed Reviews

### 🎯 UX Strategist
(Full table from Agent A)

### ✨ Craft Reviewer
(Full table from Agent B)

### 🎬 Motion & Polish Reviewer
(Before|After|Why table from Agent C)

---

## Action Plan

Numbered action items ordered by priority/impact.
1. [🔴] ...
2. [🔴] ...
3. [🟡] ...
...
```

## Important Rules

- All 3 agents MUST be called in a **single message** (parallel execution).
- Code snippets are never translated — keep them as-is.
- If reviewing an image (screenshot), adjust agent prompts to say "Screenshot to Review" instead of "Source Code to Review" and pass the image file path for the agent to read.
- This skill is **self-contained** — no external skill dependencies required.
