# Concept Maps

Rough dependency order for each topic. Not a script — use it to judge what's a real prerequisite vs. what can stand alone, and to pick the next single piece to teach. Skip anything the user's calibration answer shows they already have.

## Node.js

1. What Node actually is — JS running outside the browser, on the V8 engine
2. The event loop / single-threaded, non-blocking model (the single most load-bearing concept — most confusion traces back here)
3. Modules: `require`/CommonJS vs `import`/ESM, and why both exist
4. npm & `package.json` (dependencies, scripts, semver basics)
5. Async patterns, in order: callbacks → Promises → async/await (teach the progression, not just async/await in isolation, or "why async/await" won't land)
6. Core built-ins as needed: `fs`, `http`, `process`, `path`
7. Streams (only if relevant — often skippable for web dev)

## React

1. Components as functions that return UI description (JSX is sugar, not magic)
2. Props (data flowing in, read-only) vs state (data a component owns and can change)
3. Rendering & re-renders: what triggers a re-render, why React re-runs the function
4. `useState` as the minimal hook to introduce state
5. `useEffect` — and specifically *why* it exists (syncing with something outside React) before the dependency-array mechanics
6. The virtual DOM / reconciliation — only once rendering basics are solid, as an explanation of *why* React can be efficient, not as a prerequisite to using it
7. Composition & data flow: lifting state up, passing children, prop drilling
8. Context — only after prop drilling has been felt as a pain point, so the motivation is clear

## GraphQL

1. The problem it solves relative to REST (over/under-fetching, multiple round trips) — motivation before mechanics
2. Schema & types: the contract between client and server
3. Queries vs. mutations (read vs. write)
4. Resolvers — the function behind each field, analogous to a controller action per field rather than per endpoint
5. The client side: writing a query, what comes back, basic caching concept
6. The N+1 problem and why it comes up almost immediately in real resolvers
7. Subscriptions (only if relevant to what they're building)

## Postgres

1. The relational model: tables, rows, columns as the basic shape
2. Primary keys & foreign keys — how tables relate
3. Indexes — what they are, why lookups are slow without them, the read/write tradeoff
4. Transactions & ACID, taught with a concrete failure scenario (e.g. "what goes wrong without transactions") rather than the acronym first
5. JSON/JSONB support — relevant since her stack mixes relational and document-ish data
6. Common gotchas: NULL handling, case sensitivity, `SERIAL`/identity columns

## SQL

1. `SELECT` / `FROM` / `WHERE` — the absolute basics
2. `JOIN`s — start with INNER JOIN and the mental model of "combining matching rows," then LEFT JOIN once INNER is solid
3. Aggregation: `GROUP BY`, `HAVING`, aggregate functions
4. Subqueries — as "a query used as a value inside another query"
5. Window functions — only after aggregation is solid, since the distinction (aggregation collapses rows, window functions don't) is the whole point
6. Reading a query plan / `EXPLAIN` basics — useful once she's writing queries against real data volume, not before
