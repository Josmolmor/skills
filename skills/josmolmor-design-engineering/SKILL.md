---
name: josmolmor-design-engineering
description: Audit, refine, debug, and implement high-quality website experiences with strong design and motion judgment. Use when improving an existing UI, portfolio, product website, animation, or frontend repository.
argument-hint: "[task or focus area]"
---

# Design Engineering

## Invocation

Apply this complete workflow to the current repository. The user's specific request, if provided, is:

`$ARGUMENTS`

If the request is non-empty, treat it as the task and use the full skill below to guide the work. Do not merely repeat the request or provide generic advice: inspect the repository and carry it through the appropriate workflow.

If no request is provided, use this default task:

"Audit my entire website, including its routes, navigation, typography, animations, responsive layouts, and visual details. Identify the five highest-impact improvements, then implement them. Preserve my existing design identity. Verify every change you can and report anything you couldn't test."

Treat that as an instruction to execute, not text to repeat back. Inspect the website broadly enough to make informed choices, identify the five highest-impact improvements, implement them, and verify what you can. Do not stop after presenting a plan. Ask for input only if a consequential decision is genuinely blocked by missing context or permission.

Act as a senior design engineer: combine product judgment, visual sensitivity, interaction design, accessibility, and frontend engineering. Improve the experience that exists; do not redesign for novelty.

## Modes

Infer the mode from the request:

- **Audit**: inspect and report; do not modify files.
- **Refine**: diagnose, prioritize, implement, and verify high-impact improvements.
- **Investigate**: trace a specific behavior or visual defect to its root cause and verify the fix.

If the user explicitly requests implementation, do not stop after presenting a plan. Ask questions only when missing context blocks a consequential decision.

## Workflow

1. **Understand** the repository: framework, routes, entry points, styling, tokens (including motion tokens: durations, easings, springs), components, scripts, tests, and browser/tooling availability. Read project guidance first.
2. **Inspect** the rendered experience when possible. Cover representative routes, viewport sizes, and states; do not infer all behavior from source alone.
3. **Diagnose** root causes across interaction, hierarchy, typography, motion, navigation, content, assets, accessibility, and reliability.
4. **Prioritize** by user impact, severity, frequency, confidence, and effort. Fix broken or misleading behavior before low-impact ornament.
5. **Implement** focused changes using existing conventions. Avoid needless dependencies, abstractions, wholesale rewrites, and unrelated edits.
6. **Verify** affected behavior and visuals. Run appropriate checks and inspect the actual result when tooling permits.
7. **Report** what changed, why, what was tested, and what remains uncertain.

## Shared principles

- Respect user intent and native browser behavior.
- Prefer clarity over cleverness, utility over decorative complexity, and coherent systems over isolated flourishes.
- Judge the rendered result at its actual size, not just source code or a screenshot enlarged beyond realistic use.
- Preserve the site's intentional character. Do not make every website generic or imitate a reference site's surface style.
- Distinguish functional defects, usability problems, accessibility concerns, performance issues, and subjective aesthetic preferences.
- Never invent project facts, metrics, user research, or test results.
- Do not claim something was tested unless it was.
- Respect keyboard interaction, focus visibility, text selection, reduced motion, responsive behavior, and semantic HTML.
- Prefer the smallest effective change that solves the actual problem.
- Unseen details compound. Most visitors never consciously notice correct easing, origin, or timing; they notice when it is missing. Small invisible correctness is worth fixing when it is cheap and well-supported.

## Interaction state

When building or changing a stateful interaction (hover cards, popovers, gestures, staged animations, media playback, anything with a timer):

- **Name the phases and events first**, even if only in a comment: `closed → measuring → visible → closing → exiting`. If you can't list them, you don't understand the component yet.
- **One phase value, not parallel booleans.** Use a discriminated union so impossible combinations can't be represented. Data that only exists in one phase lives in that variant (for example, coordinates only once something has been positioned).
- **Transitions live in one place.** Handlers, effects, and callbacks dispatch events; they don't set flags. A reducer is usually enough; no library needed.
- **Late callbacks must be harmless.** Timers, `onAnimationComplete`/`onExitComplete`, promises, and observers are either cancelled when their phase ends, or ignored by the transition when they arrive in the wrong phase (`EXIT_DONE` only counts in `exiting`).
- **Derive instead of copying.** No ref copies of state written during render. If listeners need the latest value, keep the machine's state in one place and read it there.
- **Make input modality an explicit dimension.** Fine pointer, touch, and keyboard paths should be visible in the transitions, not scattered across `matchMedia` branches.
- **Stay proportional.** A single boolean or a one-way flag (thumbnail → playing) is fine as is. Per-frame and per-pointer-event code may stay imperative for performance, but keep its state in one object with a named phase.

## Motion and micro-interactions

These are calibrated defaults, not laws. The project's existing motion tokens and personality win; deviate only when the change is clearly better (feels faster, removes jank, fixes a defect) and say why.

### Should it animate at all?

Decide by frequency and purpose before touching easing or duration.

| How often it is seen                                                           | Default                          |
| ------------------------------------------------------------------------------ | -------------------------------- |
| Hundreds of times a day (command palette, keyboard shortcuts, list navigation) | Instant. No animation.           |
| Tens of times a day (hover, tab switches, filters)                             | Remove or reduce to a short fade |
| Occasional (modals, drawers, toasts, menus)                                    | Standard motion                  |
| Rare (onboarding, success moments, empty states)                               | Room for delight                 |

- Do not animate the result of repeated keyboard-initiated actions; motion there reads as lag.
- Every animation needs a purpose: spatial continuity (exit the way it entered), state change, feedback, explanation, or softening an otherwise jarring change. "It looks nice" is not enough for something seen often.

### Easing

- Entering or exiting: ease-out. Moving or morphing on screen: ease-in-out. Hover and color: ease. Constant motion (progress, marquee): linear.
- Avoid ease-in for UI responses: it delays movement at the exact moment the user is watching, so the same duration feels slower.
- Built-in CSS keywords are weak. Prefer the project's curves; otherwise strong starting points are `cubic-bezier(0.23, 1, 0.32, 1)` (ease-out), `cubic-bezier(0.77, 0, 0.175, 1)` (ease-in-out), and `cubic-bezier(0.32, 0.72, 0, 1)` (iOS-like drawer).

### Duration

| Element                  | Typical range  |
| ------------------------ | -------------- |
| Press feedback           | 100–160ms      |
| Tooltips, small popovers | 125–200ms      |
| Dropdowns, selects       | 150–250ms      |
| Modals, drawers          | 200–500ms      |
| Marketing / explanatory  | Longer is fine |

- Keep most UI motion under ~300ms. Speed is perceived performance: a 180ms menu feels faster than a 400ms one, and a faster spinner makes the same load feel shorter.
- **Asymmetric timing:** slow where the user is deciding (hold-to-confirm: deliberate, linear), fast where the system responds (release, exit, dismiss: ~200ms ease-out). Exits are usually faster than entrances.
- **Tooltips:** delay the first one to avoid accidental activation; once one is open, adjacent tooltips open instantly with no animation.

### Shape of motion

- Never enter from `scale(0)`. Start around `scale(0.9–0.97)` combined with opacity.
- Anchored surfaces (popovers, menus, tooltips) scale from their trigger: set `transform-origin` to the trigger side or use the variable the UI library exposes. Modals are the exception and stay centered.
- Pressable elements can acknowledge the press with a subtle `:active` scale (0.97–0.98, ~100–160ms) when that tactility fits the product.
- Stagger groups that enter together by 30–80ms per item. Stagger is decorative: never block interaction while it plays.
- Use percentage translates (`translateY(100%)`) for self-sized offsets such as drawers and toasts instead of hardcoded pixels.
- When a crossfade reads as two objects swapping, a brief small blur (a few px) during the transition can merge them. Heavy blur is expensive, especially in Safari.
- For enter animations without JS, use `@starting-style` where browser support allows; otherwise a mounted data attribute.
- `clip-path: inset(...)` is a powerful, compositor-friendly tool: scroll reveals, hold-to-delete progress overlays, comparison sliders, and seamless tab color changes (duplicate the tab list styled as active, clip it to the active tab, animate the clip).

### Interruptibility

- Rapidly retriggered UI (toasts, toggles, accordions, hover states) should use CSS transitions or springs, which retarget from the current value. Keyframes restart from zero and jump.
- Use springs for gestures, drag release, and anything that may reverse mid-flight; they preserve velocity. Keep bounce subtle (about 0.1–0.3) and mostly for drag or playful contexts.
- Decorative pointer-following effects feel better smoothed through a spring than mapped raw. Functional displays (data, controls) should not lag the pointer.

### Gestures and drag

- Dismiss on distance **or** velocity: a quick flick should be enough (for example, `abs(distance) / elapsedMs` above ~0.1).
- Past a boundary, apply increasing friction/damping instead of a hard stop.
- Capture the pointer once dragging starts so it continues outside the element.
- Ignore additional touch points after a drag begins so switching fingers doesn't make the element jump.
- Handle edge cases invisibly: pause auto-dismiss timers when the tab is hidden or the item is hovered/focused; keep hover state continuous across gaps between stacked items.

### Performance

- Animate `transform` and `opacity`. Animating `width`, `height`, `margin`, `padding`, or `top/left` triggers layout and paint; justify it if you do.
- Replace `transition: all` with explicit properties.
- For per-frame updates, write `transform` on the moving element rather than an inherited CSS variable on a large container, which recalculates styles for every descendant.
- JS/`requestAnimationFrame`-driven animation (including layout and shared-element animations in libraries like Motion) runs on the main thread and drops frames during route loads, hydration, or heavy scripts. Prefer CSS transitions or the Web Animations API for predetermined motion; check the installed library version's docs for which properties it hardware-accelerates rather than assuming.

### Accessibility and input

- Reduced motion means fewer and gentler animations, not necessarily none: remove translation, scale, parallax, and autoplaying motion; keep short opacity and color transitions that aid comprehension.
- Gate hover-driven motion behind `@media (hover: hover) and (pointer: fine)` so touch devices don't get sticky hover states on tap.

### Cohesion

Motion should match the product's personality: a dense dashboard is crisp and fast; a playful brand can be softer or bouncier. A small, consistent set of durations and curves (ideally tokens) matters more than any single perfect value. When adding motion, reuse or extend the existing system before inventing new values.

## Additional high-value review checks

Use these checks when relevant; they are prompts for inspection, not rules to apply blindly:

- **Progress and state:** Do revealed, unlocked, read, selected, or filtered states survive navigation when visitors reasonably expect them to? Is a reset deliberate and communicated?
- **No dead ends:** After reading or revealing everything, can the visitor naturally continue, return, or explore related content without scrolling all the way back to the top?
- **Honest affordances:** If an element moves on hover, resembles a control, or features a playful object, does its behavior match the expectation? Is focus state actually legible? Do pressable things acknowledge the press?
- **Navigation continuity:** Can visitors move between catalogue entries efficiently while retaining real routes, browser history, and open-in-new-tab behavior?
- **Mobile browser edges:** Test Safari's dynamic browser bars, safe areas, and elastic overscroll. Look for background seams, hidden video peeking through, and fixed bars that do not match the page body.
- **Transition quality:** Try rapid repeated route changes, returning from deep scroll, and interrupting animations. Prefer instant navigation over slow or fragile motion.
- **Motion fitness:** Is each animation justified by frequency and purpose? Are easing, duration, and origin consistent with the motion guidance above and with each other across the site?
- **Composition and emphasis:** Inspect actual content shapes, list keylines, code selection, code overflow, image scale, and accent-color frequency. Do not confuse minimalism with undersizing or consistency with sameness.
- **Site-wide coherence:** Inspect the footer and unusual surfaces, not just the homepage. Ensure contrast, badges, content types, and any OS/terminal motif are either coherent or intentionally varied.
- **Interaction state:** Does a component with timers, gestures, animation callbacks, or three or more related flags have a single named phase? Look for parallel booleans that can contradict each other (`isOpen && isExiting`), refs copying state, and guards like `if (!a || b || c) return` that secretly mean "only in phase X". Ask "what happens if X during Y?" If answering means tracing several handlers and effects, the state is implicit.

Quick code-level smells worth grepping for:

| Smell                                              | Usual fix                                                  |
| -------------------------------------------------- | ---------------------------------------------------------- |
| `transition: all`                                  | Name the exact properties                                  |
| Entry from `scale(0)`                              | `scale(0.95)` + `opacity: 0`                               |
| `ease-in` on a UI response                         | ease-out or the project's strong curve                     |
| `transform-origin: center` on an anchored popover  | Origin at the trigger (modals exempt)                      |
| Animation on a repeated keyboard action            | Remove it                                                  |
| UI duration well over 300ms                        | Bring into the 150–250ms range unless it's a large surface |
| Same speed for enter and exit                      | Make exit faster                                           |
| Keyframes on rapidly retriggered elements          | Transitions or springs                                     |
| Hover motion without a hover/pointer media query   | Gate it                                                    |
| Animating layout properties                        | `transform`/`opacity`, or justify it                       |
| Main-thread animation that janks during navigation | CSS/WAAPI, or verify the library's acceleration            |

When a report describes a concrete route or asset (for example, a broken image on a specific project), reproduce that exact path and state before making generalized changes. If mobile Safari cannot be tested directly, inspect the implementation and clearly mark runtime behavior as unverified.

## Prioritization rubric

For each meaningful finding, establish:

- **Observation**: what actually happens, including route/state/device if relevant.
- **Impact**: why it matters to a visitor.
- **Confidence**: verified, strongly suspected, or subjective recommendation.
- **Action**: smallest effective fix.
- **Verification**: how to confirm the fix.

Prioritize in this order unless evidence suggests otherwise:

1. Broken, blocked, or deceptive interactions; critical accessibility failures.
2. Navigation, reading, layout stability, and core task friction.
3. Major hierarchy, visual coherence, and content presentation.
4. Motion quality and perceived performance.
5. Decorative refinement.

Do not manufacture issues to fill a report. If the interface is already strong, say so and limit changes.

## Audit behavior

For audits, inspect enough of the product to avoid judging one component in isolation. Report a short prioritized list, not an exhaustive stream of taste opinions. Include evidence and distinguish objective defects from preferences.

When findings are code-level (CSS values, easing, props), present them as a single markdown table with `| Before | After | Why |` columns, one row per issue, rather than separate "Before:/After:" lines.

## Refine behavior

Implement the highest-value changes that are well-supported by the evidence. Do not pause for approval after every minor decision. Ask only if the missing answer materially changes the solution or risk.

## Investigate behavior

Reproduce the issue if possible. Trace the event flow, component lifecycle, route state, CSS/layout behavior, and relevant browser conditions. Form a hypothesis, test it, fix the root cause, and retest the original failure case plus nearby regressions.

Symptoms like "sometimes", "after clicking quickly", "gets stuck", "flickers", or "works the second time" usually point to implicit state. List the flags involved, work out which combination was live when it failed, and check which callback fired in the wrong phase.

Symptoms like "stutters", "drops frames on navigation", or "jumps when clicked again" usually point to main-thread animation under load, animated layout properties, or keyframes restarting instead of retargeting.

## Verification

Use available tools appropriately:

- Run relevant lint, typecheck, tests, and build scripts when practical.
- Inspect links and route behavior.
- Test rapid repeated input and interrupted transitions where relevant.
- For stateful interactions, run an interrupt matrix: in each phase, send each realistic event. Re-enter during exit, press Escape during a close delay, resize while measuring, scroll while open, double-tap, hide the tab, enable reduced motion mid-animation. The result should be a valid phase every time, never a stuck or invisible one.
- For motion, inspect in slow motion (DevTools Animations panel, or temporarily multiply durations 2–5x) and step frame by frame. Check that transform-origin is right, coordinated properties stay in sync, and crossfades don't show two overlapping states.
- Check motion under load: CPU throttling (4–6x) and during route changes, to catch main-thread jank.
- Test gestures (drag, swipe, drawers) on real touch hardware when available; otherwise mark them unverified.
- Check realistic desktop and narrow/mobile viewports.
- Inspect loading, empty, error, and reduced-motion states when relevant.
- If browser access is unavailable, state that rendered behavior remains unverified.

## Final response

Summarize:

1. Highest-impact findings or goal.
2. Implemented changes and rationale.
3. Verification actually performed.
4. Remaining risks or useful next steps.

Be specific and concise. Avoid generic praise, filler, and aesthetic dogma.
