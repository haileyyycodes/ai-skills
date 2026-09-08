---
name: repo-rundown
description: Gives the user a structured rundown of an unfamiliar codebase so they can start contributing quickly — tech stack, architecture, data flow, repo layout for navigation, and gotchas. Delivered one section at a time with check-ins. Use this whenever the user has just landed in a repo they don't know and wants to get oriented — phrases like "give me a rundown of this repo", "help me understand this codebase", "I just cloned this, walk me through it", "what's the architecture here", "how is this project laid out", "onboard me to this repo", "where do I even start with this". Prefer this over an ad-hoc summary whenever the intent is orientation for future contribution, not a one-off question about a single file.
---

# Repo Rundown

## Why this skill exists

A generic "summarize this repo" answer is usually a shapeless wall of text that doesn't survive the first feature ticket. This skill instead hands the user an orientation they can act on: concrete file paths, real conventions pulled from the code, and the non-obvious things that would otherwise cost them an afternoon. Success is "the user knows where to put their first change," not "I described the repo."

## The five sections

Deliver in this order unless the user asks otherwise. Each leans on the one before it, and the layout recipes are the payoff.

1. **Tech stack** — what it's built with.
2. **Architecture** — the shape, and where the core logic lives.
3. **Data flow** — how a request or event actually moves through the code.
4. **Repo layout** — annotated directory map + "to add X, touch these files" recipes.
5. **Gotchas** — the non-obvious things that will bite a new contributor.

## How to run it

**1. Recon first, mostly silently.** Before presenting anything, explore the repo: dependency manifest(s) and lockfile(s), the directory tree a couple levels deep (skip `node_modules`, `vendor`, `dist`, `build`, `target`), `README` / `CONTRIBUTING` / `Makefile` / CI config / `Dockerfile` / `.env.example`, the entry point(s), and 2–4 core files to confirm conventions rather than guess. `references/discovery-checklist.md` has per-ecosystem lists — read the relevant section if the stack isn't one you know cold. A brief "Taking a look around…" is enough; don't narrate each file.

**2. Pick a delivery mode.** If the user wants something written down or committable → build `REPO-RUNDOWN.md` at the repo root, section by section. Otherwise → deliver inline. Genuinely ambiguous → ask in one line, then go.

**3. One section per turn, then check in.** End with a light, varied prompt offering a real choice — expand what you just covered, or move on: "Want me to go deeper on any of that, or move to the data flow?" Move on at any go-ahead. If the user says "just give me everything" / "don't stop for me," drop the check-ins and deliver all five in order.

## Keep the output tight

The point of the rundown is to get someone oriented fast, so every section has to stay skimmable — a page or so, not an essay.

- **Lead with the takeaway**, then support it. A section should open with the one or two sentences that actually matter, not build up to them.
- **Prose and short bullets beat big tables.** Use a table only for a genuine matrix (e.g. two runtimes × five properties). Don't table-ify a plain list.
- **At most one diagram per rundown**, and only if the relationships genuinely resist a sentence. A five-line ASCII sketch, not a full call graph.
- **Don't restate what `ls` shows.** Annotate the directories that carry meaning; skip the obvious ones.
- **Cut anything that isn't decision-relevant.** If a detail wouldn't change where the user puts their first change or what they watch out for, leave it out — it can come back if they ask to go deeper.
- **Name real paths, not categories.** `src/server/sqlite/schema.ts` earns its place; "the schema layer" doesn't.

## What each section should contain

Pull everything from what's actually in the repo; flag inferences ("looks like…"). If a section has nothing noteworthy, say so in a sentence instead of padding.

- **Tech stack** — a scannable list: language(s) + version, runtime, framework(s), package manager, data stores, build tool, test runner, linter, CI, deploy target. Versions only where they matter. Call out anything unusual or version-pinning.
- **Architecture** — the shape in 2–3 sentences (single app? monorepo? client+server? service+workers?), then the major components and their jobs, the entry point(s), and where domain logic lives vs. framework glue.
- **Data flow** — trace one representative path end to end (e.g. request → middleware → handler → service → data access → DB → back). Then name the other flows (jobs, schedulers, webhooks, sockets), the external services, and where state lives.
- **Repo layout** — an annotated tree of the directories that matter, the organizing convention stated plainly (by feature? by layer? by domain?), and 2–3 concrete recipes drawn from how the codebase actually works ("To add an endpoint: 1) route in `…`, 2) handler in `…`, 3) test in `…`, 4) register in `…`").
- **Gotchas** — only real findings: required env vars vs. what the code reads, services needed for the app or tests, codegen / migration / build steps that aren't obvious, footguns, code that looks wrong but isn't, what the test suite needs, stale docs, deprecated paths still in the tree.

## What this skill is not for

- A specific question about one file or function — just answer it.
- Deep architectural review, refactor proposals, or bug hunting — this is orientation, not audit.
- A repo the user already knows — if they're asking about a specific change in familiar code, help with that directly.
