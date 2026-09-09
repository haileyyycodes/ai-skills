# Tactile UX Standards

The substantive reference for the Tactile UX skill. 22 rules in 6 buckets.

- **Buckets A–E** are the *tactile layer*: interaction quality that makes a web app
  feel responsive and physical. These are distilled from common front-end/UX
  practice and are not externally certified — treat as strong defaults, not law.
- **Bucket F** is the *floor*: WCAG 2.2 Level AA. Externally defined and testable.
  When a tactile rule and an accessibility rule disagree, accessibility wins.

Each rule has: **Standard** (what must be true, with specifics), **Why**, and
**Check** (how to spot a violation). Points that are genuinely debated are marked
**[contested]** with the tradeoff.

Severity labels used by the audit:
- **blocker** — breaks the feature for some users, or fails WCAG AA.
- **high** — feature works but feels broken / confusing to most users.
- **medium** — noticeable friction, degrades polish.
- **low** — refinement.

---

## Bucket A — Immediate feedback

### Rule 1 — Interactive states

**Standard:** Every interactive element (button, link, input, checkbox, toggle,
menu item, tab, card acting as a link) visually distinguishes these states, each
distinct from the others:

| State | Requirement |
|---|---|
| Default | Baseline resting appearance. |
| Hover | Perceptible change on pointer devices (background / border / elevation). Never the *only* affordance — touch has no hover. |
| Focus-visible | Always-visible ring on keyboard focus, via `:focus-visible`. Never `outline: none` without an equivalent replacement. |
| Active / pressed | A *distinct* press response — e.g. `transform: scale(0.97)`, darker fill, shadow collapse. Hover styling alone does not satisfy this. |
| Disabled | Visually muted, `cursor: not-allowed`, native `disabled` or `aria-disabled`. |

Also handle **selected / checked / current** where it applies (`aria-current="page"`
on the active nav item, checked toggles, selected tabs).

**Why:** The pressed state and a reliable focus ring are the difference between an
interface that feels responsive and one that feels like a slideshow. Missing
hover/active states are the most common reason a UI reads as "cheap."

**Check:** Tab through the page — is focus always visible? Click and hold a
button — does it react to the press itself, not just hover? Grep for
`outline:\s*none` / `outline: 0` without a paired `:focus-visible` rule.

**[contested]** Disabled buttons: some accessibility practitioners argue a disabled
submit button is worse than an enabled one that explains why it can't proceed,
because `disabled` removes it from the tab order and gives no feedback on click.
Default here: disabled is fine for controls whose unavailability is obvious from
context; prefer enabled-with-explanation for a form's primary submit.

### Rule 2 — Affordance honesty

**Standard:** Appearance matches behavior.

- If it acts, it looks like it acts: buttons look pressable; links differ from body
  text by more than hue (underline, or a color that also differs in contrast).
- If it doesn't act, it doesn't look interactive: static text isn't link-styled,
  non-clickable cards have no hover elevation, decorative icons have no button chrome.
- `cursor` matches role: `pointer` for navigate/act, default arrow for plain content,
  `not-allowed` for disabled. The common bug is `pointer` on a non-interactive `div`.
- The whole visible target is clickable, not just the text label inside it.
- One primary action per view; secondary actions are visibly lower-weight.

**Why:** Every mismatch between look and behavior costs a hesitation or a misclick,
and those compound across a session.

**Check:** Can you predict what's clickable without hovering? Do hover states appear
on things that don't navigate or act? Is there exactly one visually dominant action
per screen?

**[contested]** "One primary action per view" is a guideline, not a law — dense
tools (spreadsheets, editors) legitimately surface several peer actions. Apply it to
task-oriented and marketing screens; relax for power-user surfaces.

### Rule 3 — Action acknowledgment

**Standard:** Any user action whose result isn't instant and obvious gets immediate
feedback.

- **< 100ms and visible result** — no extra feedback needed (the result *is* the
  feedback).
- **100ms – 1s** — the triggering control enters a pending state (spinner in the
  button, or the control disables) synchronously on the event, before the async work
  resolves.
- **1s – 5s** — pending state continues; for content areas, show skeletons (Rule 5).
  Consider optimistic UI where the mutation is low-risk (Rule 6).
- **> 5s** — show determinate progress: a progress bar, percentage, step text
  ("Uploading 3 of 12"), or an estimated time remaining. An indefinite spinner for
  more than ~5s reads as a hang.
- Controls that trigger a mutation prevent double-submit (disable, or guard in the
  handler) until the result is known.
- Feedback is removed exactly when the operation resolves — no spinners that outlive
  their request, no flicker for sub-100ms responses (delay showing the spinner by
  ~150ms so fast responses never flash one).

**Why:** The user's model is "I did a thing → the thing is happening." A gap there
reads as "it's broken," and the reflex is to click again.

**Check:** In browser dev tools, set the network to "Slow 3G." Click every control
that saves, submits, loads, or navigates. Each one must show a pending state
(spinner, disabled, status text) within one frame of the click — before the result
arrives. A control that stays visually unchanged until the response lands is a
finding. Also confirm fast responses (< 100ms) don't flash a spinner.

---

## Bucket B — Async & content states

### Rule 4 — Explicit states for every async view

**Standard:** Any view or region backed by async data handles all of:

- **loading** — first load, nothing to show yet.
- **empty** — request succeeded, zero results. Designed: says why it's empty and
  what to do next; not a blank panel.
- **error** — request failed. Says what failed, offers a retry, preserves any
  surrounding context.
- **partial** — some data, still loading more (pagination, streaming).
- **success** — the normal populated state.

Empty and error are not afterthoughts — they're specified alongside the happy path.

**Why:** Unhandled async states are where apps look unfinished or broken. A blank
screen with no explanation is indistinguishable from a crash.

**Check:** For each data-backed view: force zero results, force a 500, force a slow
response. Is each one a designed state or an accident?

### Rule 5 — Skeletons over spinners; no layout shift

**Standard:**

- Content areas (lists, cards, tables, article bodies) use skeleton placeholders
  shaped like the final content, not a centered spinner. Spinners are acceptable for
  small in-place actions and full-page transitions.
- Space for async content is reserved before it arrives: images and media have
  width/height or `aspect-ratio`, late-loading regions have a min-height. Cumulative
  Layout Shift stays below 0.1.
- Content does not jump when it loads, when a font swaps, or when an ad/embed
  resolves.

**Why:** Skeletons communicate structure and reduce perceived wait. Layout shift
makes users mis-tap and lose their place — it's one of the most viscerally
"untactile" things a page can do.

**Check:** Run Lighthouse / web-vitals for CLS. Watch the page load on a throttled
connection — does anything shove content down after first paint?

### Rule 6 — Optimistic UI for low-risk mutations

**Standard:** For mutations that are fast, common, and low-consequence — toggles,
likes/stars, reordering, marking read, adding a tag — update the UI immediately on
interaction and reconcile with the server response.

- On failure: roll back to the prior state and surface a non-modal error (toast or
  inline), ideally with a retry.
- Keep an optimistic change visually identical to a confirmed one *unless* failure
  handling needs a distinct "pending" affordance.

Do **not** apply optimism to: payments, destructive actions, anything where a
silent rollback would confuse or where the user needs certainty it persisted.

**Why:** Waiting a round-trip to see a toggle flip makes the whole app feel sluggish
even when latency is modest.

**Check:** Do trivial toggles wait for the network before reflecting the change? Is
there rollback + notification on failure, or does a failed optimistic update leave
the UI lying?

**[contested]** Some teams avoid optimistic UI entirely for consistency /
simplicity, accepting the latency. Defensible; the rule is "optimism where it's
low-risk," not "optimism everywhere."

### Rule 7 — Stale-while-revalidate on refetch

**Standard:** When re-fetching data that's already on screen (tab refocus, poll,
manual refresh, cache expiry), keep showing the existing content and update it in
place when the new data arrives. Show a subtle refreshing indicator if the fetch is
slow. Don't unmount the populated view back to a loading state.

**Why:** Blanking populated content to a spinner on every background refresh is
disorienting and makes the app feel unstable.

**Check:** Switch away from the tab and back. Trigger a manual refresh. Does the
content flash to a skeleton, or update in place?

---

## Bucket C — Motion

### Rule 8 — Transitions with a job

**Standard:** Animation exists to explain a change, not to decorate.

- Transitions show cause and effect and preserve spatial continuity: an element that
  moves or resizes animates between positions rather than popping.
- Duration: ~120–200ms for small UI feedback (hover, press, small toggles),
  ~200–320ms for larger moves (panel open, route transition, list reflow). Longer
  than ~400ms for a routine UI change feels slow.
- Easing: ease-out (decelerate) for elements entering / responding to input;
  ease-in-out for elements moving between two on-screen states. Avoid linear for
  spatial motion.
- Motion never adds latency to an interaction — the result is committed immediately;
  the animation only visualizes it.

**Why:** Well-aimed motion reduces cognitive load (you see *what* changed and
*where it went*). Gratuitous or slow motion does the opposite and wastes the user's
time on every repetition.

**Check:** Does every animation answer "what changed?" Time the longest ones — are
any over ~400ms for a routine change? Does anything animate purely because it can?

**[contested]** Exact durations vary by design language — content-heavy marketing
UIs tend to run longer, dense system UIs shorter. Treat the ranges as a sanity
band, not precise targets.

### Rule 9 — Enter / exit animation for appearing elements

**Standard:** Elements that appear or disappear in response to user action — modals,
popovers, dropdowns, toasts, drawers, list items added/removed, expanding sections —
animate in and out (typically opacity + a small transform), rather than
instantly appearing/vanishing. Exit animation runs to completion before the node is
removed. Keep it short (Rule 8 ranges).

**Why:** Instant appear/disappear reads as a glitch; the eye can't track where the
thing came from or went. A 150ms fade+slide gives the change a physical origin.

**Check:** Open and close every overlay. Add and remove list items. Anything that
just blinks in or out is a finding.

### Rule 10 — Respect `prefers-reduced-motion`

**Standard:** Under `@media (prefers-reduced-motion: reduce)`:

- Remove or substantially shorten non-essential motion — parallax, large slides,
  scale/zoom transitions, auto-playing carousels, attention loops.
- Replace movement with an instant change or a plain opacity fade.
- Keep motion that is essential information (e.g. a progress bar advancing).
- This is a real WCAG concern (2.3.3, AAA) and a vestibular-safety issue; treat it
  as required, not optional.

**Why:** Large motion triggers nausea, dizziness, and migraines for people with
vestibular disorders. Ignoring the preference can make an app unusable for them.

**Check:** Enable "Reduce motion" at the OS level and reload. Do large transitions
and parallax stop? Grep for a `prefers-reduced-motion` block — its absence in any
app with animation is a finding.

---

## Bucket D — Forms & input

### Rule 11 — Validation timing

**Standard:**

- Don't validate a field before the user has finished with it: no errors while
  typing into an untouched field.
- Validate a field on **blur** (first time), then re-validate on **change** only
  *after* it has already shown an error, so the user sees it clear in real time.
- Validate everything on submit; if submit fails, move focus to the first invalid
  field and scroll it into view.
- Positive confirmation (green check) is optional and best reserved for fields with
  meaningful format rules (password strength, unique username).

**Why:** Validating on every keystroke from the first character means the user is
scolded for a half-typed email. Validating only on submit hides problems until the
end. Blur-then-live is the balance.

**Check:** Type a bad email one character at a time — does it error before you leave
the field? Fix an errored field — does the error clear as you type, or only on the
next blur? Submit an invalid form — does focus land on the first problem?

### Rule 12 — Error recovery

**Standard:**

- Each error message sits next to its field, names the problem in plain language,
  and says how to fix it ("Enter a date after today", not "Invalid").
- The field is programmatically tied to its message (`aria-describedby`) and marked
  `aria-invalid="true"`.
- User input is **never** discarded on a failed submit — not the errored fields, not
  the valid ones, not on a server error.
- Server / network errors surface in context (inline or a toast), not as a bare
  console log or a full-page replacement that loses the form.
- A form-level error summary at the top is recommended for long forms, with links to
  each field.

**Why:** Re-entering a whole form because one field was wrong is the fastest way to
make someone abandon a task.

**Check:** Submit a form with several errors, then a server 500. Is any input lost?
Are messages specific and actionable? Are they announced to a screen reader?

### Rule 13 — Input hygiene

**Standard:**

- Correct semantic input: `type="email"` / `tel` / `url` / `number`,
  `inputmode` for the right mobile keyboard, `autocomplete` tokens
  (`email`, `current-password`, `one-time-code`, `street-address`, …).
- Every input has a persistent visible `<label>` (not placeholder-as-label —
  placeholders vanish on focus and fail contrast).
- Show constraints before the error: required marker, expected format, min/max,
  and a live character counter when there's a limit.
- Don't block valid input: allow pasting (including into OTP and password fields),
  don't force-case or strip characters the user can't see happening, be liberal
  about phone/number formatting.
- Set focus to the first field on a dedicated form page/step where that's the
  obvious next action (not on a page where it would scroll past content).

**Why:** These are small and individually minor, but collectively they're the
difference between a form that fills itself and one that fights the user —
especially on mobile.

**Check:** On a phone, does the email field bring up the email keyboard? Does the
browser offer to autofill address/payment? Can you paste into every field? Are
labels visible while typing?

### Rule 14 — Submit behavior

**Standard:**

- On submit, the submit control enters a pending state (spinner / "Saving…") and
  blocks repeat submissions until the request resolves.
- Success is confirmed: a toast, an inline message, a redirect to an obviously
  changed state, or a visible data change. Don't leave the user staring at the form
  wondering if it saved.
- On failure, the control returns to its normal state so the user can retry, and the
  error appears per Rule 12.
- `<form>` submits on Enter from a text field and has a real submit button (not a
  `div` with an `onClick`).

**Why:** Double-charged orders and duplicate records almost always trace back to a
submit button that stayed live during the request with no feedback.

**Check:** Slow the network, submit, and mash the button — does it fire once or
several times? After it succeeds, is there any confirmation?

---

## Bucket E — Keyboard, touch, perceived performance

### Rule 15 — Focus management

**Standard:**

- A visible focus indicator on every focusable element (see Rule 20 for the WCAG
  minimums it must meet).
- Modal/dialog: focus moves into the dialog on open, is trapped while open, returns
  to the triggering element on close. `Escape` closes it. Background content is
  `inert` / `aria-hidden`.
- Newly revealed content (expanded section, loaded step, opened menu) receives focus
  or is the next thing in tab order — focus never lands on nothing or jumps to the
  top of the page.
- Route changes move focus to the main heading or a skip target, and the new page
  title is announced.
- Tab order follows visual/reading order; no positive `tabindex`.

**Why:** Keyboard and screen-reader users navigate entirely by focus. Lost or
trapped focus (outside a modal) makes the app unusable for them and is disorienting
for anyone using the keyboard.

**Check:** Put the mouse away. Open a modal — can you get out with Escape, does focus
return sensibly? Tab through the whole page — does focus ever vanish or backtrack?

### Rule 16 — Touch ergonomics

**Standard:**

- Touch targets are at least **44×44 CSS px** (this skill's tactile floor; WCAG 2.2
  AA minimum is 24×24 — see Rule 20). Small-looking icons get an expanded hit area
  via padding or a pseudo-element.
- At least ~8px between adjacent targets so a thumb doesn't hit two.
- No essential information or action available only on hover — it must also be
  reachable by tap/focus.
- Primary actions sit within comfortable thumb reach on mobile layouts (bottom /
  lower-center), not only top corners.
- Tap feedback is immediate (`:active` state works on touch; control the
  `-webkit-tap-highlight-color` rather than leaving the default flash).
- Custom swipe/drag interactions have a visible affordance and a non-gesture
  fallback.

**Why:** Under-sized, crowded, or hover-gated controls cause mis-taps and dead ends
on the devices most people actually use.

**Check:** On a real phone: can you hit every control first try? Is anything
hover-only? Do buttons respond the instant you touch them?

### Rule 17 — Perceived performance

**Standard:**

- Click/tap feedback is synchronous with the event, before any async result — the UI
  acknowledges the input in the same frame (ties to Rule 3).
- Prefetch likely-next data/routes on intent (link hover, focus, or in-viewport) so
  navigation feels instant.
- Render the shell/layout first, then stream content in; don't wait for everything
  before showing anything.
- Keep interaction handlers cheap: no layout thrash or long tasks on scroll, drag,
  input, or resize; target 60fps. Debounce/throttle expensive work but keep the
  immediate visual response un-debounced.
- Interaction to Next Paint (INP) under 200ms.

**Why:** Perceived speed is mostly about *immediate acknowledgment* and *not
blocking the main thread*, not raw load time. A fast app that janks on scroll feels
slow.

**Check:** Record a Performance trace while scrolling and interacting — any long
tasks, any dropped frames? Does hovering a nav link prefetch it? Does clicking a
route show *anything* before the data arrives?

### Rule 18 — Scroll & focus continuity

**Standard:**

- Back/forward navigation restores the previous scroll position; forward into a new
  page starts at the top.
- Errors, newly opened accordions, and anchor targets scroll into view (respecting
  any sticky header offset, and `scroll-margin-top`).
- No scroll-jacking (hijacking the wheel/trackpad to drive a custom scroll speed or
  animation the user didn't ask for).
- Infinite-scroll lists keep a stable scroll position when new items prepend/append,
  and provide a way to reach the footer (or use "load more").
- Status changes that aren't tied to a focus move are announced via an `aria-live`
  region (ties to Rule 22).

**Why:** Losing scroll position on back-nav, or having the page yank itself around,
breaks the user's spatial memory of where things were.

**Check:** Scroll halfway down a list, open an item, hit back — same position?
Trigger a validation error below the fold — does it scroll into view? Does the wheel
ever behave unexpectedly?

---

## Bucket F — Accessibility (WCAG 2.2, Level AA)

This bucket does not restate WCAG. It targets **conformance level AA of WCAG 2.2**
and calls out the criteria that most often fail in front-end app code. Full text:
https://www.w3.org/WAI/WCAG22/quickref/ (filter to A + AA). Failing any AA criterion
is a **blocker**.

### Rule 19 — Perceivable

**Standard:**

- **Contrast (1.4.3, 1.4.11):** body text ≥ 4.5:1 against its background; large text
  (≥ 24px, or ≥ 19px bold) and UI component boundaries / meaningful graphics ≥ 3:1.
- **Color not the only cue (1.4.1):** state, errors, required fields, chart series,
  links-in-text all carry a non-color signal (icon, underline, text, pattern).
- **Text alternatives (1.1.1):** every informative image has a meaningful `alt`;
  decorative images have `alt=""`; icon-only buttons have an accessible name.
- **Reflow (1.4.10):** usable at 320 CSS px wide with no horizontal scroll and no
  loss of content.
- **Resize / spacing (1.4.4, 1.4.12):** usable at 200% zoom; no clipping when users
  override line-height / letter-spacing.
- **Non-text contrast (1.4.11):** focus rings, input borders, toggle states meet
  3:1.

**Check:** Run axe / Lighthouse for contrast and alt-text. Zoom to 200% and narrow
to 320px. Desaturate the page — is any information lost?

### Rule 20 — Operable

**Standard:**

- **Keyboard (2.1.1, 2.1.2):** all functionality works from the keyboard; no focus
  traps (except properly implemented modals).
- **Focus visible (2.4.7)** and **Focus Appearance (2.4.11, new in 2.2):** the focus
  indicator is not obscured by other content and is large / contrasty enough to
  perceive (roughly: at least a 2px-thick perimeter with 3:1 contrast against
  adjacent colors).
- **Focus not obscured (2.4.11):** sticky headers/footers don't hide the focused
  element.
- **Target size (2.5.8, new in 2.2):** interactive targets ≥ 24×24 px, or have
  sufficient spacing. (This skill's Rule 16 holds the stricter 44px.)
- **Dragging (2.5.7, new in 2.2):** any drag operation has a single-pointer
  (click/tap) alternative.
- **Bypass blocks (2.4.1):** a skip link or landmark structure to jump past repeated
  nav.
- **Page titled (2.4.2), headings & labels (2.4.6), focus order (2.4.3):** unique
  descriptive `<title>`, logical heading outline, focus order matches reading order.
- **No keyboard-only timing / flashing (2.2.1, 2.3.1):** adjustable time limits;
  nothing flashes more than 3×/sec.

**Check:** Full keyboard pass. Tab with a sticky header present — is focus ever
hidden behind it? Check target sizes and drag alternatives. Inspect heading order.

### Rule 21 — Understandable

**Standard:**

- **Labels (3.3.2) & programmatic name (4.1.2):** every control has a visible label
  that is also its accessible name.
- **Error identification & suggestion (3.3.1, 3.3.3):** errors named in text, with a
  fix suggestion (ties to Rule 12).
- **Error prevention (3.3.4):** for legal / financial / data-deleting submissions,
  the action is reversible, checked, or confirmed.
- **Redundant entry (3.3.7, new in 2.2):** don't ask the user to re-enter
  information they already gave in the same process — carry it forward or offer it
  as a choice.
- **Accessible authentication (3.3.8, new in 2.2):** don't require a cognitive
  function test (memorizing/transcribing a code, solving a puzzle) with no
  alternative — allow paste into OTP/password, support password managers, offer
  email link / passkey.
- **Consistent help (3.2.6, new in 2.2):** help affordances (contact link, chat,
  self-serve) appear in the same relative place across pages.
- **On input / on focus (3.2.1, 3.2.2):** focusing or changing a field doesn't
  trigger a surprise context change (auto-submit, navigation).
- **Language (3.1.1):** `<html lang>` is set.

**Check:** Every input has a `<label>`. Try to paste into the OTP field. Change a
`<select>` — does the page navigate on its own? Is help where you left it on the
last page?

### Rule 22 — Robust

**Standard:**

- **Valid name/role/value (4.1.2):** custom controls (tabs, comboboxes, switches,
  menus, dialogs) expose the correct role, current state, and accessible name —
  follow the ARIA Authoring Practices patterns rather than hand-rolling.
- **Status messages (4.1.3):** confirmations, error counts, "3 results",
  async-complete messages are exposed via `role="status"` / `aria-live="polite"`
  (or `assertive` for errors) **without moving focus**.
- **Semantic HTML first:** native `<button>`, `<a>`, `<nav>`, `<main>`, `<ul>`,
  `<table>`, `<label>` before ARIA. ARIA is a patch for gaps in native semantics,
  not a replacement.
- **No broken ARIA:** no references to missing ids, no invalid role combinations, no
  `aria-hidden` on a focusable element.

**Check:** Run axe (catches most ARIA misuse). Operate custom widgets with a screen
reader — is the role and state announced? Do toasts/result-counts get read out
without stealing focus?

---

## How the buckets interact

- **Accessibility (F) is the floor.** If a tactile rule would violate an AA
  criterion, the accessibility rule wins and the tactile rule bends.
- **A and E overlap on feedback and focus** — Rule 3 (acknowledgment), Rule 17
  (perceived performance), and Rule 1 (focus-visible) reinforce each other; a
  finding often satisfies several rules at once. The audit should de-duplicate.
- **C (motion) is always gated by Rule 10** — every animation added under Rules 8–9
  must have its reduced-motion behavior defined.
