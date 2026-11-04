---
name: josmolmor-design-engineering
description: Audit, refine, debug, and implement high-quality website experiences with strong design judgment. Use when improving an existing UI, portfolio, product website, or frontend repository.
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

1. **Understand** the repository: framework, routes, entry points, styling, tokens, components, scripts, tests, and browser/tooling availability. Read project guidance first.
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

## Additional high-value review checks

Use these checks when relevant; they are prompts for inspection, not rules to apply blindly:

- **Progress and state:** Do revealed, unlocked, read, selected, or filtered states survive navigation when visitors reasonably expect them to? Is a reset deliberate and communicated?
- **No dead ends:** After reading or revealing everything, can the visitor naturally continue, return, or explore related content without scrolling all the way back to the top?
- **Honest affordances:** If an element moves on hover, resembles a control, or features a playful object, does its behavior match the expectation? Is focus state actually legible?
- **Navigation continuity:** Can visitors move between catalogue entries efficiently while retaining real routes, browser history, and open-in-new-tab behavior?
- **Mobile browser edges:** Test Safari's dynamic browser bars, safe areas, and elastic overscroll. Look for background seams, hidden video peeking through, and fixed bars that do not match the page body.
- **Transition quality:** Try rapid repeated route changes, returning from deep scroll, and interrupting animations. Prefer instant navigation over slow or fragile motion.
- **Composition and emphasis:** Inspect actual content shapes, list keylines, code selection, code overflow, image scale, and accent-color frequency. Do not confuse minimalism with undersizing or consistency with sameness.
- **Site-wide coherence:** Inspect the footer and unusual surfaces, not just the homepage. Ensure contrast, badges, content types, and any OS/terminal motif are either coherent or intentionally varied.

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

## Refine behavior

Implement the highest-value changes that are well-supported by the evidence. Do not pause for approval after every minor decision. Ask only if the missing answer materially changes the solution or risk.

## Investigate behavior

Reproduce the issue if possible. Trace the event flow, component lifecycle, route state, CSS/layout behavior, and relevant browser conditions. Form a hypothesis, test it, fix the root cause, and retest the original failure case plus nearby regressions.

## Verification

Use available tools appropriately:

- Run relevant lint, typecheck, tests, and build scripts when practical.
- Inspect links and route behavior.
- Test rapid repeated input and interrupted transitions where relevant.
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
