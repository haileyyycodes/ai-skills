---
name: tactile-ux
description: >-
  Standards and workflow for tactile, high-quality web UI — interaction feedback,
  pressed/hover/focus states, async and loading states, motion, forms, keyboard
  and touch ergonomics, perceived performance, and WCAG 2.2 AA accessibility. Use
  this whenever building, reviewing, or modifying front-end web UI, even if the
  user never says "UX": any new component, page, form, modal, or interactive
  element should be built against these standards by default. Also use when the
  user wants to audit or fix an existing web repo for UX or accessibility
  problems — phrases like "UX pass", "accessibility review", "make this feel more
  polished", "why does this feel janky", "clean up the front end", or a request
  for a phased plan to fix UI quality. Not for backend, data, infra, or CLI work.
---

# Tactile UX

## Why this skill exists

Front-end code often ships with the happy path wired up and everything else left
undone: no pressed state, no loading state, no empty state, focus that vanishes,
a form that wipes itself on error. Each gap is small; together they make an app
feel cheap and untrustworthy. This skill is the standard for closing those gaps,
plus a workflow for retrofitting it into an existing codebase without drowning
the user in a giant diff.

The full standard is **`references/standards.md`** — 22 rules in 6 buckets (A–E
are the tactile layer, F is WCAG 2.2 AA). Read it before doing work in either
mode. It is written for an agent reader: dense, specific, with a `Check` on every
rule describing how to detect a violation.

## The two modes

**Reference mode** — you are building or changing UI. Apply the standards as you
write code, silently, the way you'd apply any other code-quality bar. No report,
no ceremony.

**Audit mode** — the user wants an existing repo assessed and fixed against the
standards. This runs a read-only analysis first, then remediates in small phased
passes. Never bundles everything into one overwhelming PR.

If it's ambiguous which the user wants, ask.

---

## Reference mode

When writing or modifying a component, page, or interaction:

1. **Bucket F (accessibility) always applies.** Semantic HTML, labels, focus
   management, contrast, `name`/`role`/`value` on custom controls, `aria-live`
   for status. This is not optional and not a "nice to have."

2. **Apply the tactile buckets by relevance to what you're building:**
   - Any interactive element → Rule 1 (states), Rule 2 (affordance), Rule 3
     (acknowledgment).
   - Anything async → Bucket B (loading / empty / error / partial states,
     skeletons, optimistic where low-risk, stale-while-revalidate).
   - Anything that appears, moves, or disappears → Bucket C (motion with a job,
     enter/exit animation, `prefers-reduced-motion`).
   - Any form or input → Bucket D (validation timing, error recovery, input
     hygiene, submit behavior).
   - Modals, menus, routing, long lists → Bucket E (focus management, touch
     targets, perceived performance, scroll continuity).

3. **Match the codebase.** Use the project's existing styling system, animation
   library, state management, and component patterns. Don't introduce a new
   dependency to satisfy a rule if the repo already has a way to do it. If the
   repo has a design system or tokens, use them rather than hardcoding values.

4. **Don't gold-plate.** The standards are a floor, not a maximum. A throwaway
   internal admin screen needs correct focus and states; it does not need
   bespoke motion choreography. Spend effort proportional to how much the surface
   matters.

5. **When a tactile rule and an accessibility rule conflict, accessibility
   wins** and the tactile rule bends.

If the user's request is small (one component), just do it. If it's large (a
whole feature), briefly note which buckets you're applying so the user can
redirect.

---

## Audit mode

Detailed procedure, report template, and severity→phase mapping are in
**`references/audit-workflow.md`**. Read it before starting an audit. Summary:

### Phase 0 — Analysis (read-only, no code changes)

1. Detect the stack (framework, styling approach, component library, test setup)
   so findings reference the right patterns.
2. Scan the codebase against all 22 rules. Prioritize the surfaces that matter:
   primary user flows, shared components, forms, anything with async.
3. Write **`tactile-ux-audit.md`** at the repo root:
   - Summary table: finding counts by bucket and by severity.
   - Findings: each with bucket, rule number, severity
     (blocker / high / medium / low), file:line references, a one-line
     "what's wrong / why it matters", and the proposed fix.
   - Remediation plan: findings grouped into ordered phases.
4. **Stop. Present the report and the plan. Wait for the user to approve** before
   touching any code.

### Phased remediation

Findings are sorted into phases by **risk and blast radius**, not by bucket:

- **Phase 1 — Global, mechanical, low-risk.** Near find-and-replace, hard to get
  wrong: `:focus-visible` styles, `prefers-reduced-motion` block, tap-target
  minimums, contrast token fixes, `inputmode` / `autocomplete` attributes,
  `alt` text. Broad coverage, small diff.
- **Phase 2 — Component-level, additive.** Adding hover/pressed/disabled states,
  button pending states, empty/error states for async views. Touches many files
  but each change is self-contained and low-risk.
- **Phase 3 — Behavioral, needs judgment and testing.** Optimistic UI,
  validation-timing changes, focus trap/restore in modals, scroll restoration,
  stale-while-revalidate. Higher risk; the user reviews each change.

### Rules for remediation

- **One PR per phase**, unless it gets unruly — soft cap ~400 lines changed or
  ~15 files. Split further if exceeded. One concern per commit, clear messages.
- **Ask the user up front how they want to work each phase:**
  - *auto-apply* — make all of the phase's changes on a branch, then the user
    reviews the finished diff; or
  - *approve each* — propose each change and wait for a yes before applying it.
  Respect that choice for the rest of the session. When in doubt, ask again.
- **Never `git push` and never open a PR.** Stage the branch and commits
  locally, then hand the user the exact command
  (`git push -u origin <branch> && gh pr create ...`) or tell them it's ready to
  push. Pushing and PR creation are the user's action.
- **Work on a branch**, never directly on `main` / the default branch. One branch
  per phase (e.g. `tactile-ux/phase-1-mechanical`).
- **Re-run the analysis after each merged phase** and update
  `tactile-ux-audit.md` so remaining work and progress stay visible.
- **Don't refactor beyond the finding.** If a fix reveals a deeper problem, note
  it in the report as a new finding; don't expand the current change to chase it.

---

## Guardrails (both modes)

- The standards file is the source of truth. If you're unsure whether something
  is a violation, check the rule's `Check` line rather than guessing.
- Don't invent numeric thresholds. Use the values in `standards.md`; where it
  gives a range, stay in the range.
- Accessibility findings (Bucket F) are always at least **high** severity, and an
  AA failure is a **blocker**.
- Report honestly. If a surface is fine, say it's fine. Don't manufacture
  findings to pad the audit.
