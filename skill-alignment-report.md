# Skill Alignment Report

Comparison of Impeccable skills (`source/skills/`) against preferred guidelines and research.

**Reference sources reviewed:**
- voxls.dev CLAUDE.md + frontend-content.py hook
- Gist: Frontend CLAUDE.md (spectra-the-bot/3b5b12764604223b363c424cfe2cf5e6)
- Gist: Design System Rectification Plan (spectra-the-bot/8c0315cf...)
- agent-platform styling-conventions.md
- agent-platform accessibility.md
- agent-platform research docs (8 files: accessible-premium, color-temperature, dark-mode, micro-interactions, motion-systems, onboarding-ai, typography-chat, executive-summary)

**Skills reviewed:** All 21 skills in `source/skills/` plus `frontend-design/reference/` (7 reference docs).

---

## 1. CSS Architecture

### What the skills say
The frontend-design skill and its reference docs are **framework-agnostic**. They recommend OKLCH color spaces, semantic tokens (`--text-body`, `--text-heading`), two-layer token hierarchies (primitive + semantic), `clamp()` fluid sizing, container queries, and modern CSS features broadly. No specific styling approach is prescribed.

The color reference explicitly recommends:
> "Use two layers: primitive tokens (`--blue-500`) and semantic tokens (`--color-primary: var(--blue-500)`). For dark mode, only redefine the semantic layer."

### What your guidelines say
SCSS Modules exclusively. Flat `$colors` SCSS map generating CSS custom properties and `.bg-*` / `.text-*` utility classes. No Tailwind, no CSS-in-JS, no BEM, no W3C design token JSON, no build pipelines for styles. Hardcode spacing/radius/layout in `px`. Colors via `var(--color-*)`. `lib.scss` grows organically. The rectification plan explicitly kills semantic alias indirection (`semantic.bg.primary -> primitive.gray.50`).

### Gaps

| Skill guidance | Your guideline | Conflict severity |
|---|---|---|
| Two-layer token hierarchy (primitive + semantic) | Flat map, no alias indirection | **High** — directly contradicts rectification plan |
| OKLCH color functions throughout | Hex values in SCSS map, CSS custom properties | Medium — OKLCH is fine for tooling but skills should note hex is acceptable |
| Semantic token naming (`--text-body`) | Named after purpose or color name (`--color-ink`, `--color-violet`) | **High** — skills push a pattern your guidelines reject |
| No mention of SCSS Modules | SCSS Modules are the only approved approach | **High** — skills are stack-agnostic where your projects are opinionated |
| `clamp()` / fluid everything | Fixed `rem` scale with breakpoint-based responsive | Medium — see typography section |

### Recommendation
The `frontend-design/reference/color-and-contrast.md` needs a rewrite of the theming/token section. The two-layer token architecture should be replaced with guidance that acknowledges flat color maps as the primary approach, with two-layer tokens presented as an alternative for larger teams. The SCSS Modules approach should be mentioned as a valid (and often preferred) component styling strategy alongside whatever framework-agnostic advice remains.

---

## 2. Typography

### What the skills say
- Fluid type via `clamp()` as the default approach
- Modular scale (1.25, 1.333, 1.5 ratios)
- Avoid "invisible defaults" (Inter, Roboto, etc.)
- Semantic token naming (`--text-body`, `--font-size-heading`)
- `rem` for font sizes (correct)
- Vertical rhythm based on line-height multiples

### What your guidelines say
- Fixed `rem` scale: d1-d3, h1-h6, b1-b3 as global type classes
- Display classes scale at breakpoints via mixins, not fluid `clamp()`
- Font variables named after the typeface (`$font-untitled-sans`, `$font-space-grotesk`), never after role (`$font-display`, `$font-body`)
- Type classes applied as string classNames in JSX: `className={clsx(styles.card, 'b1')}`
- `font-display: swap` always

### What the research says
- **Fixed `rem` scales preferred for app UIs** — "No major app design system (Material, Polaris, Primer, Carbon) uses fluid type in product UI"
- Chat-specific sizing: 14-16px messages, 16px AI responses, 14px code blocks
- Weight carries more load than size in compact UIs (400→700 at 18px > increasing to 32px)
- Letter-spacing tightens above 20px: `-0.01em` to `-0.02em`
- 7-requirement font selection criteria (x-height, apertures, I/l/1 differentiation, etc.)

### Gaps

| Skill guidance | Your guideline | Research alignment |
|---|---|---|
| Fluid `clamp()` as default | Fixed `rem` + breakpoint mixins | **Research agrees with your guidelines** for app UI |
| Role-based font naming | Typeface-based naming | Skills contradict guidelines |
| Generic type scale advice | Specific d1-d3, h1-h6, b1-b3 system | Skills are generic; guidelines are more useful |
| No chat-specific typography guidance | N/A | Research has detailed chat/AI typography guidance not in skills |

### Recommendation
1. The typography reference should note that fluid `clamp()` is for **marketing/content pages** and fixed `rem` scales with breakpoint adjustments are preferred for **app UI, dashboards, and data-dense interfaces**. (The reference already has this nuance but it's buried — it should lead.)
2. Remove or soften the semantic token naming recommendation. Acknowledge typeface-based naming as a valid approach.
3. Add a chat/AI typography section drawing from the research: message sizing, timestamp hierarchy, code block sizing, AI long-form line-height (1.7-1.85).

---

## 3. Motion & Animation

### What the skills say
- `animate` skill: CSS-first approach, `transform` and `opacity` only, respect `prefers-reduced-motion`
- Motion reference: 100/300/500 duration rule, exponential easing curves, avoid bounce/elastic, CSS custom properties for stagger, `grid-template-rows` for height animation
- Reduced motion via CSS `@media (prefers-reduced-motion: reduce)` block

### What your guidelines say
- **GSAP** is the primary animation tool for sequenced/complex animation
- CSS animations (via `animations.scss` mixins) for simple entrance effects
- Pre-animation state **must** be in CSS, not GSAP — CSS is synchronous, JS is not
- `gsap.killTweensOf(ref.current)` on every `useEffect` teardown, no exceptions
- No shared animation config files — GSAP values inline per component
- Embla for carousel behavior; CSS transitions for carousel motion
- Reduced-motion guard: check `window.matchMedia('(prefers-reduced-motion: reduce)')` and set `gsap.defaults({ duration: 0 })`

### What the research says
- Detailed 6-tier duration scale (50ms instant → 800ms expressive)
- Distance-to-duration and element-size-to-duration mappings
- Exit/enter asymmetry: exits at 60-70% of entrance duration
- Shadow performance trick: animate `::after` opacity, not `box-shadow` directly
- Chat-specific: typing indicator after ~800ms delay, message scale-in 0.95→1.0 in 150ms, scroll-to-bottom 300ms
- Easing personality profiles per product (snappy=Linear/Notion, luxurious=Apple/Stripe)
- Stagger intervals: tight 30-50ms (lists), medium 60-80ms (grids), loose 100-150ms (hero)

### Gaps

| Skill guidance | Your guideline | Conflict severity |
|---|---|---|
| CSS-first animation approach | GSAP-first for complex, CSS for simple | **High** — skills don't mention GSAP at all |
| No pre-animation state rule | CSS must own pre-animation state | **High** — this is a core principle in your guidelines |
| `@media` CSS reduced motion | GSAP runtime guard + `gsap.defaults({ duration: 0 })` | **High** — different implementation strategy |
| No `killTweensOf` guidance | Mandatory on every `useEffect` teardown | **High** — critical cleanup rule missing |
| Generic duration ranges | Research has granular size/distance-to-duration mapping | Medium — skills are less specific |
| No chat-specific motion guidance | N/A | Research has detailed chat interaction timing |
| No carousel guidance | Embla + CSS transitions | Missing |

### Recommendation
This is the largest gap. The `animate` skill and `motion-design.md` reference need significant additions:
1. Add GSAP as a first-class animation approach alongside CSS. Cover: inline per component, local refs, `killTweensOf` teardown rule.
2. Add the pre-animation-state-in-CSS rule explicitly.
3. Add GSAP-specific reduced-motion guidance (runtime guard, `gsap.defaults`).
4. Add the research's duration/size/distance mapping tables — they're more actionable than the current 100/300/500 rule.
5. Add exit/enter asymmetry rule (60-70%).
6. Add chat-specific interaction timing (typing indicators, message appearance, scroll behavior).

---

## 4. Accessibility

### What the skills say
- `audit` skill: checks a11y as one dimension among many (theming, performance, responsive)
- `harden` skill: adds error handling, i18n, edge cases — mentions accessibility tangentially
- `frontend-design` reference docs: focus rings via `:focus-visible`, WCAG contrast ratios, touch targets 44px, reduced motion
- Treatment is **generic principles** spread across multiple skills

### What your guidelines say
Full WCAG 2.2 AA contract with production-ready specifics:
- Contrast ratios per element type, specific color pair requirements
- Focus management: visibility (2px solid, 3:1 contrast), order per flow/screen, trapping per modal type, return targets after dismiss
- ARIA live regions: `polite` vs `assertive` per event, announcement text patterns per flow
- Touch targets: 44px primary CTAs, 24px minimum, 24px spacing when undersized
- Reduced-motion GSAP contract: classification per interaction (essential vs decorative), specific reduced-motion behavior per animation
- Keyboard operability: key mappings per context
- Semantic markup: landmark structure per screen, ARIA per component

### What the research says
- Gradient contrast at worst-case (lightest) stop, not average
- Glass/frosted surfaces: solid text scrim `rgba(10,10,10,0.65)` minimum, never backdrop-filter over uncontrolled content
- Triple-layer focus ring: gap ring + brand ring (3:1+) + decorative glow, include `outline: 2px solid transparent` for Windows High Contrast Mode
- Dark mode contrast recalculation: specific failures called out (`#6B6B6B` on `#1A1A1A` = only 4.0:1)
- Decorative elements: `aria-hidden="true" focusable="false"` for SVG, `alt=""` for images, verify no tab order
- APCA advisory targets: Lc 60+ normal text, Lc 45+ large/bold

### Gaps

| Skill guidance | Your guideline / research | Conflict severity |
|---|---|---|
| Generic "check a11y" in audit skill | Production-ready WCAG contract with per-component specifics | **High** — skills are surface-level |
| Basic focus ring mention | Triple-layer ring, WHCM transparent outline, per-flow focus order/trapping/return | **High** — significantly more nuance needed |
| No ARIA live region guidance | Specific polite/assertive patterns per event | **High** — missing entirely |
| No gradient contrast evaluation | Worst-case stop evaluation, anchor-dark technique | Medium — research technique not in skills |
| No glass/frosted surface a11y | Solid scrim requirements, no backdrop-filter over dynamic content | Medium — research technique not in skills |
| No dark-mode contrast recalculation | Must re-evaluate every pair independently | **High** — skills' dark mode advice doesn't flag this |
| No decorative element ARIA rules | aria-hidden, focusable=false, alt="" patterns | Medium |
| No keyboard nav patterns | Roving tabindex, key mappings per context | Medium — partially in interaction reference |

### Recommendation
1. The `audit` skill needs a dedicated accessibility tier with specific checks, not just "check a11y." Pull from the accessibility contract: contrast pairs, focus management, ARIA patterns.
2. Add gradient contrast evaluation and glass surface accessibility techniques from research to `color-and-contrast.md`.
3. Add dark-mode contrast recalculation requirement to color reference.
4. Add decorative element ARIA patterns to interaction reference.
5. Consider a dedicated accessibility reference doc under `frontend-design/reference/` that consolidates: focus management, live regions, touch targets, reduced-motion contract, keyboard patterns.

---

## 5. Dark Mode

### What the skills say
Color reference covers dark mode generally: lighter surfaces for depth, desaturate accents slightly, dark gray not pure black, two-layer token system for mode switching.

### What your guidelines say
CSS custom properties on `[data-theme="dark"]` and/or `prefers-color-scheme: dark`. Same variable names, different values. No semantic alias indirection. Dual-mode toggle (OS preference + class toggle).

### What the research says
- Surface elevation system: 4 levels, 6-8 lightness points apart, higher = lighter
- Text hierarchy via opacity stack: 0.87 / 0.7 / 0.5 / 0.3 / 0.2
- Border opacity: `rgba(255,255,255,0.08)` for card borders
- Standard shadows invisible on dark — use colored glow or dense multi-layer black
- Never pure `#000000` backgrounds
- Never pure `#FFFFFF` text — use `rgba(255,255,255,0.87)` or off-white like `#EDECF4`
- Brand accent (violet) needs lightening in dark mode for contrast
- Every light-mode contrast pair must be independently verified in dark mode

### Gaps

| Skill guidance | Your guideline / research | Conflict severity |
|---|---|---|
| Two-layer token system for mode switch | Flat map, CSS custom property overrides | **High** — same conflict as color architecture |
| Generic "desaturate accents" | Research gives specific: +20-30% lightness, -10-15% saturation | Medium — skills lack specificity |
| No text opacity stack | Research: 0.87/0.7/0.5/0.3/0.2 hierarchy | Medium — useful pattern not in skills |
| No shadow-in-dark-mode guidance | Research: colored glow or dense black, standard shadows invisible | Medium |
| No contrast re-verification requirement | Research + guidelines: re-evaluate every pair | **High** |

### Recommendation
Add a dark mode section to the color reference (or expand the existing one) covering:
1. Surface elevation pattern (lighter = higher)
2. Text opacity stack
3. Border/shadow behavior in dark mode
4. Accent color lightening requirements
5. Mandatory contrast re-verification
6. Replace the two-layer token guidance with CSS custom property override pattern

---

## 6. Component Structure

### What the skills say
No specific component structure guidance anywhere in the skill set. The frontend-design skill discusses what to build and how it should look, but not file organization or naming conventions.

### What your guidelines say
Strict convention across all projects:
- `ComponentName/index.tsx` + `styles.module.scss`
- `&ChildName` nesting in SCSS (compiles to `chatMessageContent`, never bare `.content`)
- Affectors as `&.sent`, `&.active` (dot, not a new name)
- `clsx` for combining module classes with global type classes
- Type classes as string classNames: `className={clsx(styles.card, 'b1')}`
- No additional files unless clearly needed

### Research says
N/A (research covers design patterns, not file structure).

### Gaps

This is a total gap. The skills produce no guidance on component file structure, SCSS module naming conventions, or how to compose module classes with global classes. This means every skill that produces code (`frontend-design`, `animate`, `adapt`, etc.) will generate components in whatever structure the LLM defaults to — which will not match your conventions.

### Recommendation
This doesn't need to be in every skill, but the `frontend-design` skill or a shared reference should include:
1. Component folder convention (index.tsx + styles.module.scss)
2. SCSS naming: `&ChildName` for children, `&.state` for affectors
3. How to compose: `className={clsx(styles.chatMessage, 'b1')}`
4. Rule: no bare `.content`, `.title`, `.wrapper` class names

This would be a natural addition to the `frontend-design` skill since it's the root skill that others reference via "Use the frontend-design skill."

---

## 7. Onboarding (AI Products)

### What the skills say
The `onboard` skill covers generic first-time UX: empty states, activation flows, progressive disclosure, value demonstration. No AI-specific patterns.

### What the research says
AI-specific onboarding benchmarks and patterns:
- Time-to-first-interaction: target under 30 seconds
- Agent's first message IS the onboarding (conversational, not walkthrough)
- 4-6 suggestion chips sampling capability range
- Zero mandatory walkthroughs (reduce activation rates)
- Loading states as personality reveals
- OAuth-first as primary CTA
- First upsell: session 3+ minimum
- Every signup field costs 10-15% of potential users
- Welcome message: 2-3 sentences max, end with invitation/question
- Mobile-first: design for 375px with keyboard covering bottom third

### Gaps

The `onboard` skill is missing the entire AI product onboarding playbook from the research. Given this is a design skill set likely used with AI products, this is a significant content gap.

### Recommendation
Add an AI/conversational product section to the `onboard` skill covering:
1. Time-to-first-interaction benchmarks
2. Conversational onboarding pattern (first message = onboarding)
3. Suggestion chips as capability sampling
4. Anti-patterns: mandatory walkthroughs, feature dumps, early upsells
5. Loading-as-personality pattern

---

## 8. Micro-Interactions (Chat/AI)

### What the skills say
The `animate` and `delight` skills cover micro-interactions generically. No chat-specific or AI-specific interaction patterns.

### What the research says
Detailed chat interaction specifications:
- Typing indicators: appear after ~800ms, opacity pulse (0.3→1.0→0.3, staggered 150ms), occupy message bubble space
- Message appearance: own messages scale-in (0.95→1.0, translateY 8px→0, 150ms), batched messages 50ms stagger
- Scroll-to-bottom: button tap 300ms smooth, auto-scroll on new message 100ms
- Feedback: visual response under 100ms even if async
- Button state machine: default → loading (immediate on click) → success/error
- History (scroll-up): no animation, instant render

### Gaps

These are production-ready specifications that would significantly improve the quality of AI/chat interface code generated by the skills. Currently absent.

### Recommendation
Add a chat/AI interaction patterns section — either to the `animate` skill, the `delight` skill, or as a new reference doc under `frontend-design/reference/`. This is especially valuable given the primary use case of these skills.

---

## 9. Banned Patterns Enforcement

### What the skills say
The skills have no mechanism for enforcing banned patterns. The `frontend-design` skill has a "DON'T" list covering aesthetic anti-patterns (glassmorphism everywhere, bounce easing, card grids) but nothing about toolchain restrictions.

### What your guidelines say (consistently across all sources)
Hard bans:
- No Tailwind CSS or utility-first frameworks
- No CSS-in-JS (styled-components, emotion, vanilla-extract)
- No BEM naming
- No W3C design token JSON files or token build pipelines
- No shared GSAP config files
- No role-based font variable names

### Gaps

The skills will happily generate Tailwind classes, BEM naming, styled-components, or token JSON because nothing in the skill text prohibits it. The `teach-impeccable` skill collects project context via `.impeccable.md` but it doesn't ask about toolchain constraints — it focuses on audience, brand, and tone.

### Recommendation
Two approaches (not mutually exclusive):
1. Add a "Stack Constraints" section to `teach-impeccable` that asks about CSS approach (modules vs utility vs CSS-in-JS), animation library preferences, and banned patterns. Store in `.impeccable.md`.
2. Add a "Respect Project Conventions" section to `frontend-design` that instructs the LLM to check for existing styling patterns before generating code (look at existing `.module.scss` files, check for tailwind config, etc.).

---

## 10. Skill-Specific Notes

### `typeset`
- Recommends fluid `clamp()` broadly — should note fixed `rem` for app UI
- No chat-specific typography guidance
- Missing: letter-spacing tightening above 20px, weight > size principle for compact UI

### `colorize`
- No dark-mode-specific guidance for adding color
- No gradient contrast evaluation technique
- Missing: tinted neutrals technique (4-8% accent saturation in grays)

### `audit`
- Accessibility section is shallow compared to the contract in your accessibility.md
- No gradient/glass contrast techniques
- No ARIA live region checks
- No dark-mode-specific contrast verification

### `harden`
- Covers i18n text expansion and error handling well
- Missing: ARIA live region patterns for dynamic state
- Missing: decorative element ARIA rules

### `adapt`
- Good responsive coverage
- Missing: breakpoint-mixin approach as an alternative to container queries
- No mention of GSAP considerations for responsive (killing/recreating tweens)

### `animate`
- Entirely CSS-focused — no GSAP guidance
- Missing: pre-animation state in CSS rule
- Missing: killTweensOf teardown rule
- Missing: carousel guidance (Embla)
- Reduced motion treatment is CSS-only

### `onboard`
- Generic patterns, no AI-specific playbook
- Missing: time-to-first-interaction benchmarks, conversational onboarding, suggestion chips

### `optimize`
- Good performance guidance
- No shadow animation trick (::after opacity instead of box-shadow)
- No mention of avoiding `will-change` preemptively

### `delight`
- Generic micro-interaction patterns
- No chat-specific interaction specs
- Missing: typing indicator patterns, message appearance timing

### `teach-impeccable`
- Collects brand/audience/tone context
- Missing: stack constraints (CSS approach, animation library, banned patterns)
- Missing: accessibility requirements level (AA vs AAA, specific needs)

---

## Summary: Priority Recommendations

### P0 — Direct Contradictions (fix these first)

1. **Remove two-layer token architecture from color reference.** Replace with flat color map + CSS custom property override pattern. This is the single biggest philosophical conflict.

2. **Remove semantic token naming from typography reference.** Replace with typeface-based naming as the primary recommendation.

3. **Add GSAP to motion guidance.** Currently completely absent. The animate skill and motion reference are CSS-only, but your projects are GSAP-centric.

4. **Add pre-animation-state-in-CSS rule.** This is a correctness rule (prevents FOUC), not a preference.

### P1 — Significant Gaps (high value additions)

5. **Add dark-mode contrast re-verification requirement** to color reference and audit skill.

6. **Deepen accessibility coverage in audit skill.** Pull specific patterns from the accessibility contract: focus management, ARIA live regions, touch targets.

7. **Add chat/AI interaction patterns** to animate or delight skill. Research has production-ready specs for typing indicators, message appearance, scroll behavior.

8. **Add AI onboarding playbook** to onboard skill. Research has specific benchmarks and patterns.

9. **Add component structure guidance** to frontend-design skill. File convention, SCSS naming, class composition.

10. **Add stack constraint gathering** to teach-impeccable. CSS approach, animation library, banned patterns.

### P2 — Refinements (better specificity)

11. **Clarify fluid vs fixed typography.** The reference already has nuance but it's buried. Lead with: "Fixed rem for app UI, fluid clamp for marketing pages."

12. **Add research-backed duration tables** to motion reference. Size-to-duration, distance-to-duration, exit/enter asymmetry.

13. **Add gradient/glass contrast techniques** from accessible-premium research to color reference.

14. **Add tinted neutrals technique** to color reference (4-8% accent saturation in grays).

15. **Add dark mode section** to color reference: surface elevation, text opacity stack, shadow behavior, accent lightening.

16. **Add shadow animation performance trick** (::after opacity) to motion reference and optimize skill.

---

## Appendix: Source-to-Skill Mapping

Which research insight maps to which skill for implementation:

| Research topic | Target skill(s) | Reference doc |
|---|---|---|
| Accessible premium (gradient contrast, glass surfaces, focus rings) | audit, frontend-design | color-and-contrast.md, interaction-design.md |
| Color temperature (tinted neutrals, shadow tinting) | colorize, frontend-design | color-and-contrast.md |
| Dark mode (elevation, text opacity, contrast) | colorize, audit, frontend-design | color-and-contrast.md (new section) |
| Micro-interactions (chat timing, typing indicators) | animate, delight | motion-design.md (new section) |
| Motion systems (duration tables, exit/enter, stagger) | animate, frontend-design | motion-design.md |
| Onboarding AI (benchmarks, conversational, suggestion chips) | onboard | N/A (skill-level addition) |
| Typography chat (message sizing, code blocks, hierarchy) | typeset, frontend-design | typography.md |
| Executive summary (cross-cutting) | frontend-design | Multiple |
