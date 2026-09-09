# Audit Workflow

The detailed procedure for **audit mode**: assess an existing web repo against
`standards.md`, then remediate in small phased passes. Read this fully before
starting. `standards.md` is the source of truth for what counts as a violation —
this file is about *how to run the audit*, not *what the rules are*.

---

## Step 1 — Detect the stack

Before scanning, establish what you're working with, so findings reference the
right patterns and fixes fit the codebase. Look at:

| Question | Where to look |
|---|---|
| Framework | `package.json` deps (`react`, `vue`, `@angular/core`, `svelte`, `next`, `nuxt`, `astro`); file extensions. |
| Styling approach | CSS Modules, Tailwind (`tailwind.config`), styled-components / Emotion, vanilla-extract, Sass, plain CSS, a UI kit. |
| Component library | `@mui/*`, `@chakra-ui/*`, `radix-ui`, `@headlessui`, `shadcn/ui` (look for `components/ui/`), Angular Material, PrimeNG, Ant. These already handle some rules — note which. |
| Design tokens | A `tokens.*`, `theme.*`, CSS custom properties block, or Tailwind theme. Fixes should use these, not hardcoded values. |
| Animation | `framer-motion`, `@angular/animations`, CSS transitions only, GSAP, Motion One. |
| Routing | File-based (Next/Nuxt/Remix), `react-router`, `@angular/router` — affects scroll-restoration and focus-on-route-change findings. |
| Test setup | `@testing-library`, Playwright, Cypress, `axe-core` / `jest-axe` — affects what "add a test" fixes look like. |
| Monorepo | `pnpm-workspace.yaml`, `turbo.json`, `nx.json`, `packages/*` or `apps/*`. Ask the user which app/package is in scope; don't audit everything. |

Record this in the report's header so the reader knows the context the findings
were written against.

---

## Step 2 — Map the UI surface area

Don't read every file. Build a prioritized list:

1. **Shared UI primitives** — `Button`, `Input`, `Select`, `Checkbox`, `Modal` /
   `Dialog`, `Tooltip`, `Menu`, `Tabs`, `Toast`, form field wrappers. Findings
   here propagate everywhere, so they're the highest leverage and usually belong
   in Phase 1 or 2.
2. **Primary user flows** — auth, onboarding, the main create/edit forms, the
   core list/detail views, checkout or submission flows. Ask the user which
   flows matter most if it isn't obvious.
3. **Async-heavy views** — anything fetching, paginating, streaming, polling,
   uploading.
4. **Global layer** — root layout, error boundaries, the global CSS reset /
   base styles, route configuration, the app shell.

Everything else (one-off marketing pages, rarely-used settings screens) is
lower priority — sample it, don't exhaustively cover it.

---

## Step 3 — Scan against the rules

Work rule by rule across the prioritized surface. For each rule, use its `Check`
line in `standards.md`. Grep is the fast first pass; confirm hits by reading the
surrounding code before recording a finding.

### Grep starting points

These find *candidates*, not confirmed violations — always read the context.

| Rule | Pattern (ripgrep) | What a hit suggests |
|---|---|---|
| 1 | `outline:\s*(none\|0)` | focus suppressed; check for a `:focus-visible` replacement nearby |
| 1 | `:hover` count vs `:active` / `:focus-visible` count | many hover rules, few active/focus rules → missing pressed/focus states |
| 2 | `onClick` / `(click)` on `div` / `span` / `li` with no `role`/`tabindex` | non-semantic clickable; also a keyboard finding (Rule 15/20) |
| 2 | `cursor:\s*pointer` on non-interactive selectors | affordance mismatch |
| 3 | `fetch(` / `axios` / `httpClient` in a handler with no loading state set | unacknowledged action |
| 3 | `setLoading(true)` with no corresponding disable on the trigger | double-submit risk |
| 4 | `.map(` rendering a list with no `length === 0` branch nearby | missing empty state |
| 4 | `catch` blocks that only `console.error` | swallowed error, no error state |
| 5 | `<img` without `width`/`height`/`aspect-ratio` | layout shift |
| 5 | full-view spinner components on content routes | spinner where a skeleton belongs |
| 10 | absence of `prefers-reduced-motion` anywhere in an app with transitions/animation libs | Rule 10 violation, app-wide |
| 11 | validation firing in an `onChange` / `(input)` with no "touched" guard | validates while typing |
| 12 | `reset()` / `setForm(initial)` in a `catch` or after a failed submit | input discarded on error |
| 13 | `placeholder=` on inputs with no associated `<label>` / `for=` | placeholder-as-label |
| 13 | `type="text"` on fields named email/phone/url/number; missing `autocomplete` | input hygiene |
| 15 | `Dialog` / `Modal` implementations with no focus-trap / `inert` / focus-restore | focus management |
| 16 | tap targets: buttons/links with `height` or padding yielding < 44px | touch target |
| 19 | hardcoded color pairs; check computed contrast | contrast |
| 22 | `role=` values, `aria-*` referencing ids; run `axe` if available | broken/missing ARIA |

### Stack-specific checks

- **React** — loading/empty/error handled per query? (`isLoading`/`isError` from
  React Query / SWR used, or bare `useEffect` + `useState` with gaps). `key` on
  lists stable. `useEffect` focus management on route change.
- **Angular** — `async` pipe without `; else loading` / `; else error`
  templates. `(ngSubmit)` buttons without a pending disable.
  `@angular/animations` present but no reduced-motion handling.
- **Vue** — `<Suspense>` / `v-if` loading branches; `<Transition>` usage vs
  reduced-motion.
- **Next / Nuxt / Remix** — `loading.tsx` / `error.tsx` / `not-found.tsx`
  present per route segment. Scroll restoration config. `<Link prefetch>`.

### If a component library handles a rule

Note it and move on — e.g. "Radix `Dialog` provides focus trap + restore +
Escape, Rule 15 satisfied for all modals using it." Only flag the components
that *don't* use the library's primitive, or that override its behavior.

---

## Step 4 — Write `tactile-ux-audit.md`

Write it at the repo root (or the in-scope app's root in a monorepo). Exact
structure:

```markdown
# Tactile UX Audit

**Scanned:** <path / package> · **Date:** <YYYY-MM-DD> · **Standards:** tactile-ux v<n>

**Stack:** <framework>, <styling>, <component lib>, <animation>, <routing>, <test setup>

## Summary

| Bucket | Blocker | High | Medium | Low | Total |
|---|---|---|---|---|---|
| A — Immediate feedback | | | | | |
| B — Async & content states | | | | | |
| C — Motion | | | | | |
| D — Forms & input | | | | | |
| E — Keyboard / touch / perf | | | | | |
| F — Accessibility (WCAG AA) | | | | | |
| **Total** | | | | | |

<2–4 sentence narrative: the overall state, the worst area, the single highest-value fix.>

## Findings

### F-01 · Rule 1 · Interactive states · HIGH
**Where:** `src/components/Button.tsx:24-31`, propagates to ~40 call sites
**Problem:** No `:active` or `:focus-visible` styling; `outline: none` on line 27
with no replacement. Every button in the app lacks a pressed state and a keyboard
focus ring.
**Fix:** Add `:focus-visible` ring using `--color-focus` token; add `:active`
transform (`scale(0.97)`) and fill-darken. One change, whole-app effect.
**Phase:** 1

### F-02 · Rule 4 · Missing empty state · MEDIUM
...

## Remediation plan

### Phase 1 — Global, mechanical (est. 1 PR, ~<n> files)
- F-01 — button focus/active states
- F-07 — app-wide `prefers-reduced-motion` block
- F-11 — `inputmode`/`autocomplete` on 6 form fields
- ...

### Phase 2 — Component-level, additive (est. 1 PR, ~<n> files)
- ...

### Phase 3 — Behavioral, needs review (est. 1–2 PRs)
- ...

## Out of scope / noted for later
<Deeper problems spotted but not in this audit's remit — architectural issues,
missing test infrastructure, design decisions for the user to make.>
```

Finding IDs (`F-01`, `F-02`, …) are stable — reference them in commits and PRs,
and keep them across re-runs so progress is traceable.

Then **stop and present.** Do not write code until the user approves the plan.
Expect them to re-scope: drop findings, merge phases, change priorities.

---

## Step 5 — Severity rubric

Assign severity by *user impact*, not by effort to fix.

| Severity | Definition | Examples |
|---|---|---|
| **blocker** | Breaks the feature for some users, or fails a WCAG 2.2 AA criterion. | Keyboard user can't complete the form; modal has no focus trap; submit button contrast 2.1:1; no visible focus anywhere; form wipes input on error. |
| **high** | Feature works but feels broken or confusing to most users. | No loading state on a 2s save; no empty state (blank screen on zero results); validation errors while typing the first character; no pressed state anywhere. |
| **medium** | Noticeable friction; degrades polish. | Spinner where a skeleton belongs; layout shift on image load; abrupt modal appear/disappear; missing `autocomplete` tokens. |
| **low** | Refinement. | Transition 50ms too long; hover state a little subtle; could prefetch on hover. |

Bucket F findings are **never below high**. An outright AA failure is a
**blocker**; a Bucket F finding that is best-practice-but-not-a-named-AA-failure
(e.g. a redundant ARIA attribute that isn't harmful) can be high.

---

## Step 6 — Severity → phase mapping

Phase is about *risk of the fix*, so severity and phase are correlated but not
identical. A blocker can land in Phase 1 if its fix is mechanical.

| | Phase 1 (mechanical) | Phase 2 (component, additive) | Phase 3 (behavioral) |
|---|---|---|---|
| **Nature** | find-and-replace, attribute/token/CSS additions, single-file changes with app-wide effect | adding states/markup to components; each change self-contained | logic changes: timing, sequencing, optimism, focus choreography, data-flow |
| **Risk** | very low — hard to regress | low — additive, easy to review per-file | moderate — needs testing and per-change review |
| **Typical rules** | 1 (focus/active CSS), 10, 13 (attrs), 16, 19 (tokens), parts of 20 | 1 (stateful variants), 2, 4, 5, 9, 14 | 3, 6, 7, 11, 12, 15, 17, 18 |
| **Review mode** | user reviews finished diff | user reviews finished diff, or per-file | per-change approval by default |

If a single finding spans phases (e.g. Rule 1 needs both a CSS token fix *and* a
new `pending` variant with logic), split it: the mechanical part goes to Phase 1,
the behavioral part to Phase 3, referenced as `F-01a` / `F-01b`.

---

## Step 7 — Execute a phase

1. **Confirm the working mode** for this phase with the user: *auto-apply* (do
   the whole phase, user reviews the finished branch) or *approve-each* (propose
   every change, wait for yes). Default to *approve-each* for Phase 3.
2. **Branch** off the default branch: `tactile-ux/phase-<n>-<slug>` (e.g.
   `tactile-ux/phase-1-mechanical`). Never commit to `main`.
3. **Make the changes**, one finding (or one finding-part) per commit. Commit
   message: `tactile-ux(F-01): add focus-visible + active states to Button`.
   Body: the rule, the before/after in one line, the finding ID.
4. **Keep the diff reviewable.** Soft cap ~400 changed lines or ~15 files. If the
   phase exceeds it, split into `phase-1a` / `phase-1b` branches and tell the
   user there will be two PRs.
5. **Don't scope-creep.** A fix that uncovers a deeper problem → add it to the
   report as a new finding; don't chase it in this commit.
6. **Verify** against the finding's `Check` line. Run the repo's linter, type
   check, tests, and `axe`/`jest-axe` if present. Note in the PR body what you
   verified and what still needs manual/device testing (e.g. "touch targets
   verified in DevTools device mode; needs a real-device pass").
7. **Hand off — do not push, do not open a PR.** Tell the user it's staged and
   give them the command:
   ```
   git push -u origin tactile-ux/phase-1-mechanical && gh pr create \
     --title "Tactile UX — Phase 1: global mechanical fixes" \
     --body-file <path-to-generated-pr-body>
   ```
   Generate a PR body: the phase summary, the finding IDs with one line each,
   the verification notes, and "part N of M phases". Leave running it to the user.
8. **After the user merges**, re-run Steps 1–4 (scan + rewrite the report),
   marking resolved findings `RESOLVED (phase 1)` and keeping their IDs. Then
   start the next phase.

---

## Edge cases

- **No framework / vanilla JS + HTML** — the rules still apply; grep patterns
  shift to raw `addEventListener`, inline handlers, and stylesheet inspection.
  Focus on the global CSS and the shared script files.
- **Very large codebase** — don't try for completeness. Audit the shared
  primitives and the top 3–5 flows the user names, state clearly in the report
  that coverage was sampled, and offer to run further passes on other areas.
- **Repo already has a design system package** — audit the design-system package
  itself first (fixes there cascade), then audit consumers only for places they
  bypass it.
- **User wants just an audit, no fixes** — stop after Step 4. Deliver the report.
- **User wants a specific bucket only** (e.g. "just the accessibility pass") —
  scan and report only that bucket; phasing still applies within it.
- **Findings the user disputes** — record their decision in the report
  (`WONTFIX — <reason>`), don't silently drop it; a future re-run shouldn't
  resurface it as new.
