# PantryPal — Learning Path v1.1 (basics → advanced, project-only)

> **Goal.** Learn only what PantryPal needs, deeply enough that you can **explain every line and redesign the project from a blank page**.
> **Companion:** `PantryPal_Action_Plan.md` v1.1 (Steps 0–8). Module IDs (M0…M18) are referenced there.
> **New in v1.1:** parsing the real dataset's ingredient strings (M10), batch inventory + units + taste-based scoring + consumption estimation (new **M18**), views/FEFO in SQL (M6), Playwright smoke tests and the **step gate** (M14), updated roadmap with the input-optimisation step.
> **Style.** Short statements. Every jargon term is explained the first time it appears. Math is done one step at a time.

---

## How to use this path

1. **Learn just-in-time.** Do the modules listed for a Step *before* that Step's tasks (see §Roadmap at the end).
2. **Each module has five parts:** *Why PantryPal needs it* → *Concepts (basic → advanced)* → *Build it* → *Check yourself* → *Done when*.
3. **Teach-back rule.** After the AI writes any task, one teammate explains it back **without looking**. If they can't, the module isn't done.
4. **Rebuild days.** Once per step, rebuild one small piece **without AI** (marked 🔨).
5. Tick the boxes. A concept is ticked only when you can explain it in your own words.

**Size key:** S ≈ a few sessions · M ≈ a week · L ≈ two weeks (for one person, part-time). These are rough.

---

## M0 — Dev environment and Git · *Size S · Needed: Step 0*

**Why:** five people editing one codebase without breaking each other.

**Basics**
- ☐ Terminal: `cd`, `ls`, `mkdir`, `cat`, pipes. (*Pipe* = send one command's output to another.)
- ☐ Node.js and npm: what `package.json`, `node_modules`, scripts are. (*npm* = package manager that downloads libraries.)
- ☐ Environment variables and `.env` files. (*Env var* = a setting kept outside code, e.g. a secret key.)
- ☐ JSON: objects, arrays, strings, numbers, booleans, null.

**Git**
- ☐ Commit, branch, merge, remote, pull request. (*Commit* = a saved snapshot. *Branch* = a parallel line of snapshots.)
- ☐ Rebase vs merge; squash merge; resolving conflicts.
- ☐ Conventional commits (`feat:`, `fix:`); `.gitignore`; why secrets must never be committed.

**Build it:** create a repo, two branches editing the same file, force a conflict, resolve it.
**Check yourself:** What does `main` always have to be? Why is a leaked `.env` an emergency?
**Done when:** every member has opened and merged one PR.

---

## M1 — How the web works · *Size S · Step 0–1*

**Why:** React ↔ Node ↔ Supabase is all HTTP. Every bug lives in one of the three.

**Basics**
- ☐ Client/server model. (*Client* = browser; *server* = program answering requests.)
- ☐ URL anatomy; DNS (name → IP address); HTTPS (encrypted HTTP).
- ☐ HTTP request/response: method, path, headers, body, status code.
- ☐ Methods: GET (read), POST (create), PATCH (partial update), DELETE. Idempotent = repeating gives the same result (GET, DELETE yes; POST no).
- ☐ Status codes: 200, 201, 204, 400, 401 (not logged in), 403 (not allowed), 404, 409 (conflict), 429 (too many requests), 500.

**Intermediate**
- ☐ REST design: nouns in URLs (`/households/:id/items`), verbs from the method.
- ☐ Cookies vs tokens. (*Token* = a string proving who you are, sent in a header.)
- ☐ CORS. (*CORS* = browser rule: a page may call another origin only if that server allows it.)
- ☐ Pagination: why lists are returned in pages.

**Build it:** use the browser Network tab on any site; find one request and name its method, status, and headers. Then call a public API with `curl`.
**Check yourself:** 401 vs 403? Why is POST not idempotent? What does CORS protect?
**Done when:** you can trace "click Add item" to the exact request in Network tab.

---

## M2 — JavaScript → TypeScript · *Size L · Step 0–1*

**Why:** all app code. TypeScript catches mistakes before running.

**JavaScript basics**
- ☐ Variables (`const`, `let`), types, `===`, truthy/falsy.
- ☐ Functions, arrow functions, **closures** (a function remembering the variables around it).
- ☐ Arrays: `map`, `filter`, `reduce`, `find`, `sort` (and that `sort` mutates).
- ☐ Objects, destructuring, spread, optional chaining `?.`, `??`.
- ☐ Modules: `import` / `export`.

**Async (critical)**
- ☐ Event loop: JS does one thing at a time; slow work (network) is handed off and resumes later.
- ☐ Promises and `async/await`; `try/catch`; `Promise.all`.
- ☐ Errors: what happens to an unhandled rejection.

**Dates (project-critical)**
- ☐ `Date` is an *instant* in time, not a calendar day. Local vs UTC. Why `new Date("2026-10-02")` can shift a day.
- ☐ Plan: treat expiry as a **calendar date string** `YYYY-MM-DD`, never an instant.

**TypeScript**
- ☐ Types, interfaces, unions (`'owner' | 'member'`), generics (`LfuCache<K,V>`), narrowing, `unknown` vs `any`.
- ☐ `strict` mode. (*Strict* = the compiler refuses vague types.)
- ☐ **zod**: runtime validation. (*Runtime* = while the program runs; TS types vanish at runtime, so user input must be checked by zod.) Infer types from schemas.

**Build it 🔨:** write `expiryStatus(expiryDate: string, today: string)` with tests for −1, 0, 1, 3, 7, null.
**Check yourself:** Why can't TypeScript validate JSON from the network? What is a closure? Why is "expiry as DATE" safer than a timestamp?
**Done when:** `expiryStatus` passes all boundary tests, written by you.

---

## M3 — HTML, CSS, responsive design, accessibility · *Size M · Step 0–2*

**Why:** *frontend appeal* is a stated priority. Dark glossy glass cards must still be readable.

**Basics**
- ☐ Semantic HTML (`button`, `nav`, `main`, `label`). Why `div` is not a button.
- ☐ Box model: content, padding, border, margin.
- ☐ Flexbox (one-dimensional layout) and Grid (two-dimensional).
- ☐ Units: `rem`, `%`, `vw`; mobile-first with media queries.

**Intermediate**
- ☐ CSS variables = design tokens (palette, spacing). Tailwind: utility classes mapped to tokens.
- ☐ Glass effect: translucent background (`rgba`), `backdrop-filter: blur()`, border, shadow. (*Backdrop-filter* = blurs what is behind the element.)
- ☐ Gradients, shadows, focus rings, transitions (keep subtle).

**Accessibility (a11y)**
- ☐ Contrast ratio (WCAG AA: 4.5:1 for normal text). Test text **over blur**, not over a flat colour.
- ☐ Keyboard: Tab order, visible focus, Enter/Space on buttons.
- ☐ Labels for inputs; `aria-label` only when no visible text.
- ☐ Never use colour alone: expiry = colour **+ icon + text**.

**Build it:** one glass card, three states (default, hover, focus), checked at 360 px with a contrast checker.
**Check yourself:** Why does blur reduce contrast? Flex vs grid for a card list?
**Done when:** the S0-6 kit page passes keyboard-only use.

---

## M4 — React and frontend data flow · *Size L · Step 1–6*

**Why:** the entire UI.

**Basics**
- ☐ Components, JSX, props. (*Props* = inputs to a component.)
- ☐ State with `useState`. (*State* = data that, when changed, re-renders the component.)
- ☐ Rendering model: UI = function of state; re-render on change.
- ☐ Lists and `key`; conditional rendering; controlled inputs and forms.

**Intermediate**
- ☐ `useEffect`: what it is for (sync with outside world) and what it is not for (deriving data).
- ☐ Custom hooks; Context (use sparingly).
- ☐ React Router: routes, params, protected routes, redirect after login.
- ☐ **TanStack Query**: *server state* (data owned by the backend). Query keys, stale time, mutations, invalidation, **optimistic updates** (update UI first, roll back on failure).
- ☐ Loading / empty / error / success states — design all four.

**Testing**
- ☐ Vitest + Testing Library (test what users see, not internals). MSW (mock the network).

**Advanced**
- ☐ Error boundaries; code-splitting (lazy routes); avoiding needless re-renders (measure first).
- ☐ Debounce for autocomplete.

**Build it 🔨:** an ingredient autocomplete input with debounce, keyboard navigation, and a loading state, against an MSW mock.
**Check yourself:** Why is a query key like an address? What goes wrong with optimistic updates on failure? When is `useEffect` the wrong tool?
**Done when:** the S2-6 form works with mocks and you can explain every hook in it.

---

## M5 — Node.js and Express · *Size M · Step 1–5*

**Why:** the API: validation, authorization, orchestration, caching.

**Basics**
- ☐ Node runtime; single-threaded + event loop; why blocking the loop freezes everyone.
- ☐ Express: routes, `req`/`res`, middleware chain. (*Middleware* = a function that runs before your route, e.g. check login.)
- ☐ Reading params, query string, JSON body.

**Intermediate**
- ☐ Layering: **route** (HTTP) → **service** (logic) → **repository** (SQL). Why logic must not live in routes.
- ☐ Validation with zod at the edge; one error shape; error-handling middleware.
- ☐ Auth middleware: read `Authorization: Bearer <JWT>`, verify, attach user.
- ☐ Config via env vars; structured logging.
- ☐ Supertest: test routes without starting a server.

**Advanced**
- ☐ Security middleware: `helmet` (safe headers), CORS allow-list, rate limiting.
- ☐ Serverless: what changes on Vercel — no long-lived process, cold starts, per-instance memory, DB connection pooling.
- ☐ Idempotent endpoints and safe retries.

**Build it 🔨:** a tiny Express app: `GET /health`, `POST /echo` with zod, auth middleware that rejects missing tokens, and Supertest tests.
**Check yourself:** Why validate on the server when the UI already validates? What is lost on a cold start?
**Done when:** S1-2 middleware passes 401/401/200 tests you can explain.

---

## M6 — SQL and relational databases · *Size L · Step 1–6*

**Why:** *database speed* is priority #1, and the recommendation engine is SQL.

**Basics**
- ☐ Table, row, column, primary key, foreign key.
- ☐ `SELECT … WHERE … ORDER BY … LIMIT`; `INSERT`, `UPDATE`, `DELETE`.
- ☐ Data types: `text`, `numeric`, `boolean`, `DATE` vs `timestamptz`, `uuid`, enums.
- ☐ Constraints: `NOT NULL`, `UNIQUE`, `CHECK`, `FOREIGN KEY … ON DELETE`.

**Intermediate**
- ☐ Relationships: one-to-many; **many-to-many via a junction table** (`recipe_ingredients` links recipes ↔ ingredients).
- ☐ `JOIN` (inner, left); `GROUP BY`, `HAVING`, aggregates (`count`, `sum`).
- ☐ Normalization (1NF–3NF): why store each fact once. Aliases table = one canonical ingredient, many names.
- ☐ CTEs (`WITH`), subqueries.
- ☐ Migrations: schema changes as ordered files; never edit an applied one.

**Advanced (project-specific)**
- ☐ **Transactions and ACID.** (*Transaction* = a group of statements that all succeed or all fail. ACID = Atomic, Consistent, Isolated, Durable.)
- ☐ Concurrency: race conditions, `SELECT … FOR UPDATE` (row lock), atomic `UPDATE … WHERE used_count < max_uses`.
- ☐ Functions in PL/pgSQL; **triggers** (`pantry_version` bump); views; `SECURITY DEFINER`.
- ☐ `ON CONFLICT DO NOTHING/UPDATE` (idempotent inserts).
- ☐ Time in SQL: `DATE` arithmetic, `AT TIME ZONE`, "today in Asia/Kolkata".
- ☐ **Views:** a saved query that looks like a table. Your per-ingredient total is a view over batches, so the total is never stored twice (one source of truth).
- ☐ `CHECK (quantity >= 0)`: let the database refuse impossible stock, even if the code has a bug.

**Build it 🔨:** write, by hand, the query "for household H, recipes ranked by coverage" on a 5-recipe fixture. Compute the expected result on paper first.
**Check yourself:** Why a junction table? What can go wrong between "check" and "update" without a lock? `DATE` vs `timestamptz`?
**Done when:** your hand-written ranking query matches the hand-computed table.

---

## M7 — PostgreSQL performance · *Size M · Step 2–4*

**Why:** the first stated focus area.

**Basics**
- ☐ What an **index** is. (*Index* = a sorted lookup structure so the DB doesn't scan every row.)
- ☐ Sequential scan vs index scan.

**Intermediate**
- ☐ B-tree; composite index order (equality columns first, then range/sort).
- ☐ **Partial index** (`WHERE deleted_at IS NULL`): smaller and faster.
- ☐ Reading `EXPLAIN ANALYZE`: node types, estimated vs actual rows, loops, time.
- ☐ **N+1 problem:** 1 query for a list + N for details → fix with a join.
- ☐ Pagination: offset (slow when deep) vs keyset (stable and fast).

**Advanced**
- ☐ `pg_trgm`: trigram similarity and GIN indexes for fuzzy search (math in M9). **Optional:** the plan uses alias + prefix search first and adds trigram only if measurement shows it is needed.
- ☐ Selectivity and statistics; why the planner sometimes ignores an index.
- ☐ Connection pooling: why serverless needs a pooler.
- ☐ Materialized views as a precompute option (only if measured).

**Build it 🔨:** seed 10k rows in `pantry_items`; time the list query with and without the partial index; paste both plans.
**Check yourself:** Why does composite-index column order matter? What does "rows removed by filter" tell you?
**Done when:** S2-2's `EXPLAIN` shows an index scan and you can say why.

---

## M8 — Supabase: Auth, RLS, CLI, cron · *Size L · Step 1–5*

**Why:** identity and **who may see which household's data**.

**Basics**
- ☐ Project anatomy: Postgres, Auth, API keys (**anon** = public, **service role** = bypasses all rules).
- ☐ Sign-up, sign-in, sessions, refresh tokens, email confirmation, redirect URLs.

**Intermediate**
- ☐ **JWT** (JSON Web Token): signed string with claims (`sub` = user id). Anyone can read it; only the issuer can sign it. Why the server must **verify**, not just decode.
- ☐ **Row Level Security**: policies `USING` (which rows you can see) and `WITH CHECK` (which rows you may write). `auth.uid()`.
- ☐ Policy recursion trap; fix with `SECURITY DEFINER` helper `is_member()`.
- ☐ Supabase CLI: local stack, migrations, `db reset`, type generation.

**Advanced**
- ☐ pgTAP: tests for RLS (allowed and denied).
- ☐ `pg_cron`: schedule SQL; remember cron time is UTC-based.
- ☐ Realtime: changes pushed to clients (respects RLS; verify its limits before relying on it).
- ☐ Acting *as the user* from the API (pass the user's JWT to the client).

**Build it 🔨:** two users, two households; write policies; prove with tests that A cannot read B.
**Check yourself:** Why is the service-role key dangerous? What do `USING` and `WITH CHECK` each do?
**Done when:** S1-1's pgTAP suite passes and a teammate can break-test it.

---

## M9 — Data structures and algorithms for PantryPal · *Size L · Step 3–4*

**Why:** the matching score and the LFU cache are the algorithmic heart.

**Basics**
- ☐ Big-O: how work grows with input size (O(1), O(log n), O(n), O(n log n)).
- ☐ Hash map / hash set: average O(1) lookup, insert, delete.
- ☐ Set operations: union, intersection, difference.
- ☐ Sorting; **stable sort** (equal items keep order); top-k selection (heap).
- ☐ Doubly linked list: O(1) remove/insert when you hold the node.

**Matching math (one step at a time)**
1. Recipe needs set `R`; pantry has set `P`.
2. `have = |R ∩ P|`; `need = |R|`; `missing = need − have`.
3. Coverage `c = have / need`. Always between 0 and 1.
4. Urgency weight for one matched item: `w = 1 / (1 + d)` where `d` = days left (≥ 0).
   - `d = 0 → 1`; `d = 1 → 0.5`; `d = 6 → ≈ 0.14`. Decreases as `d` grows, never exceeds 1, never reaches 0.
5. `u = Σw / need`. Since each `w ≤ 1` and there are `have` terms, `u ≤ c`.
6. **Taste term:** `t = 1` if the recipe's cuisine is one the user likes, else `0`.
7. **Hard filters first** (diet mismatch, disliked ingredient → recipe removed). Soft scores only rank what is left.
8. `score = c + λ·u + τ·t` (defaults λ = 1, τ = 0.25). Ties: fewer missing, then shorter time.
   - Why τ is small: taste should break ties between cookable recipes, not push an uncookable recipe to the top.
   - Sanity check: for a fixture pantry, print the top 10 and ask "would I cook this?" before trusting any constant.
- ☐ Why divide by `need` and not by `|P|`? (Dividing by `|P|` rewards recipes just for sharing a big pantry's size, not for being nearly cookable.)

**Fuzzy match math (trigrams)**
- ☐ A *trigram* = every 3-character window of a string padded with spaces.
- ☐ `"tomato"` padded `"  tomato "` → `"  t"," to","tom","oma","mat","ato","to "` = 7.
- ☐ `"tomatoes"` → `"  t"," to","tom","oma","mat","ato","toe","oes","es "` = 9.
- ☐ Intersection = 6 (`"to "` is only in "tomato"). Union = 7 + 9 − 6 = 10.
- ☐ Similarity (Jaccard) = 6 / 10 = **0.6**. The default PostgreSQL threshold is 0.3, so they match.

**LFU cache (full derivation)**
- ☐ *LFU* = Least Frequently Used: evict the entry used the fewest times; tie → least recently used.
- ☐ Structures: `map key → node(value, freq)`; `map freq → doubly linked list` (head = most recent); integer `minFreq`.
- ☐ `get(k)`: take node `n` with freq `f`. Remove `n` from `list[f]` (O(1)). Insert at head of `list[f+1]` (O(1)). If `list[f]` is now empty **and** `f == minFreq`, set `minFreq = f+1`.
- ☐ Why `minFreq = f+1` is exactly right: `n` just landed in `list[f+1]`, so that list is non-empty, and no lower non-empty list exists.
- ☐ `set(k,v)` when full: evict the **tail** of `list[minFreq]`; insert new node with `freq = 1`; set `minFreq = 1`.
- ☐ Every step is a map lookup or a linked-list splice → **O(1)** per operation.
- ☐ Weakness: *cache pollution* — an old, once-popular key keeps a high count forever. Fix: TTL (expire by age).
- ☐ LRU vs LFU: LRU = recency only; LFU = frequency first.

**Optional:** trie (prefix autocomplete) — only if autocomplete from SQL proves too slow.

**Build it 🔨:** implement `LfuCache` from scratch. Test against a naive O(n) version using random operations (property test).
**Check yourself:** Prove `minFreq` stays correct after eviction. Why does the score use a ratio (`have/need`) instead of a count?
**Done when:** your LFU passes the property test and you can draw the three structures from memory.

---

## M10 — Text processing and data cleaning · *Size M · Step 1–3*

**Why:** good suggestions need clean ingredient identity. This is your pandas strength; use it here.

**Basics**
- ☐ pandas: read CSV, inspect (`info`, `describe`, `isna`), vectorized string ops (`.str`), `explode`.
- ☐ Normalization: lowercase, trim, strip punctuation, remove quantities/prep words ("chopped", "to taste").
- ☐ Singular/plural handling (simple rules first).
- ☐ **Parsing your real dataset string.** Example row: `6 Karela (Bitter Gourd/ Pavakkai) - deseeded, Salt - to taste, 1 Onion - thinly sliced`.
  - Split on commas **outside parentheses** (count `(` and `)` as you scan). A plain `split(",")` breaks when a comma sits inside brackets.
  - Per item: preparation note = text after the first ` - `; alternate names = text inside `( … )`, split on `/`.
  - Leading quantity can be an integer, decimal, fraction (`1/2`), mixed (`1 1/2`) or range (`2-3`). `to taste` has **no** quantity.
  - Unit = next word if it is in a known list (`tablespoon`, `teaspoon`, `cup`, `clove`, `inch`…); otherwise it is a count.
  - Regex basics: groups, optional parts, non-greedy `.*?`. Test each rule on ≥30 real rows before trusting it.
  - Use the `Translated…` columns. Send rows you cannot parse to a **reject log**; never guess.

**Intermediate**
- ☐ Alias dictionary: many spellings → one canonical ID (English/Hindi: aloo/potato, dahi/curd/yogurt, atta/wheat flour).
- ☐ Fuzzy matching (trigram or `rapidfuzz`) with a **threshold** and a manual-review queue.
- ☐ Frequency-first review: fix the 100 most common unmatched strings; they cover most of the data.
- ☐ Validation with `assert`; deterministic output (same input → same files).

**Advanced**
- ☐ Evaluate your normalizer: hand-label ~100 strings; report **precision** (how many matches were right) and **recall** (how many true matches were found).
- ☐ Diet-label sanity check (veg recipe containing meat ingredients → flag).
- ☐ Bulk load to Postgres with `COPY`.
- ☐ *(Optional stretch, not v1)* TF-IDF / embeddings for substitutions.

**Build it 🔨:** pipeline notebook → script that outputs `recipes.csv`, `recipe_ingredients.csv`, `unmatched.csv` and a precision/recall note.
**Check yourself:** Why review by frequency? What is the cost of a false match vs a missed match here?
**Done when:** re-running the script regenerates byte-identical outputs.

---

## M11 — Caching and performance engineering · *Size M · Step 3–4, 8*

**Why:** *quick and reliable* suggestions; LFU is in the stack.

**Basics**
- ☐ What a cache is; **hit ratio** = hits / (hits + misses).
- ☐ Where caches live: browser, CDN, API memory, database buffer, TanStack Query.

**Intermediate**
- ☐ Eviction policies (LFU, LRU, TTL) and capacity.
- ☐ **Invalidation** — the hard part. Strategy used here: **versioned keys** (`pantry_version` in the key), so stale entries are simply never read again and age out.
- ☐ Key design: include everything the result depends on (household, version, date, query). Missing one = wrong or leaked data.
- ☐ Cache stampede: many requests miss at once and all hit the DB.

**Advanced**
- ☐ Serverless caveat: memory is per instance, so hit ratio may be low; correctness must not depend on the cache.
- ☐ Measuring: percentiles (p50 = typical, p95 = 1-in-20 slow); tools: `autocannon` (API load), Lighthouse (page).
- ☐ Performance budgets and the rule: **measure → fix lowest layer → re-measure.**

**Build it 🔨:** benchmark `suggest_recipes` through the API with and without the cache; log hit ratio.
**Check yourself:** Why is `household_id` mandatory in the key? What breaks if the key omits `today`?
**Done when:** `docs/perf.md` has a before/after table you produced.

---

## M12 — Security essentials · *Size M · Step 1, 5–6, 8*

**Why:** a shared pantry means other people's data sits next to yours.

- ☐ **Authentication** (who are you) vs **authorization** (what may you do). Most real breaches are authorization bugs ("broken access control").
- ☐ Least privilege: anon key for the browser, user JWT for the API, service role only for jobs.
- ☐ Input validation everywhere; parameterized queries; whitelist sort fields. (*SQL injection* = attacker text becomes SQL.)
- ☐ XSS: attacker script runs in the page. React escapes by default; never use `dangerouslySetInnerHTML` on user text.
- ☐ Secrets management; `.env` never in git; rotate if leaked.
- ☐ Invite tokens: ≥128 bits of randomness, expiry, single-use, revocable.
- ☐ Rate limiting (login, invite accept); CORS allow-list; secure headers.
- ☐ Abuse thinking: how would someone game 3-cook promotion?
- ☐ Privacy: show only display names; collect only what the app needs.

**Build it:** write five "evil tests" for your own API: another household's ID, tampered JWT, SQL in the sort param, expired invite, reused invite.
**Check yourself:** Why is hiding a button not security? What does RLS protect against that API checks alone don't?
**Done when:** all five evil tests fail safely (403/401/400) in CI.

---

## M13 — Time, scheduling, and reliable jobs · *Size S · Step 2, 4*

- ☐ Date vs instant; time zones; India has no DST but users may live elsewhere.
- ☐ Computing "today" per household in SQL and passing it into functions (testable!).
- ☐ Cron syntax (`0 3 * * *`); remember the DB runs cron in UTC; 03:00 UTC ≈ 08:30 IST.
- ☐ **Idempotency**: running twice equals running once. Tool: unique `dedupe_key` + `ON CONFLICT DO NOTHING`.
- ☐ Retries and at-least-once execution; why duplicates must be harmless.
- ☐ Notification patterns: in-app now; email/web push later (web push needs a service worker and has platform limits — only if added).

**Build it 🔨:** `scan_expiring_items()` for a fixture of items; run twice; assert row count is unchanged.
**Check yourself:** What does "at-least-once" imply for your design? Why pass `today` as a parameter?
**Done when:** Review Focus #5 passes.

---

## M14 — Testing and quality · *Size M · every step*

- ☐ Test pyramid: many unit, fewer integration, few end-to-end.
- ☐ TDD loop: red → green → refactor.
- ☐ Fixtures; mocks vs real dependencies; MSW for the network.
- ☐ **Property-based testing** (fast-check): generate random inputs, assert a rule always holds. Used for LFU vs naive, SQL vs oracle.
- ☐ **Oracle testing:** a slow, obviously-correct implementation checks the fast one.
- ☐ pgTAP for SQL and RLS; Supertest for API; Testing Library for UI; **Playwright smoke test per step from Step 1** (a short real-browser run of that step's main path); full end-to-end in Step 8.
- ☐ **Step gate (your "each step must run without bugs" rule):** one command (`npm run gate`) runs lint, typecheck, unit, integration and smoke tests from a fresh clone. A step is done only when it passes, earlier steps' tests still pass (**regression**), and the app starts with no console errors.
- ☐ Feature flags: switch risky features off in code so an unfinished one cannot crash the release.
- ☐ CI with GitHub Actions: tests run on every PR.
- ☐ Coverage is a smoke alarm, not a goal.

**Build it 🔨:** property test "LFU never exceeds capacity and never evicts a higher-frequency key while a lower one exists".
**Check yourself:** What does an oracle test catch that example-based tests miss?
**Done when:** CI blocks a deliberately broken PR.

---

## M15 — UX and HCI for this project · *Size S · Step 1–3, 7–8*

**Why:** *UX friction* is priority #2.

- ☐ Friction = extra steps/thinking between user and goal. **Count taps** for: add an item, see what to cook, invite a member.
- ☐ Time-to-first-value: how long from signup to "I saw something useful".
- ☐ Onboarding: ask the minimum; allow skipping; show value early.
- ☐ Four states: loading (skeletons), empty (tell what to do next), error (say what happened and how to retry), success.
- ☐ Colour semantics for expiry plus icon and text; consistent across the app.
- ☐ Microcopy: "Use soon" beats "Status 2".
- ☐ Warning design (yellow/red stock signs): colour **+ icon + words**, say what to do ("Update quantity"), and limit how often you nag — a warning that appears everywhere gets ignored.
- ☐ Mobile-first: thumbs, tap target size, forms with the right keyboard (`inputmode`).
- ☐ **Usability test:** watch 3–5 people do a task; do not help; note where they pause.

**Build it:** write a task flow for "something expires tomorrow → I cook" and count steps; cut one.
**Check yourself:** What are the empty states for each screen?
**Done when:** a first-time tester adds three items in under a minute without help.

---

## M16 — System design: how to design this project yourself · *Size M · all steps*

**Why:** your explicit goal — you must be able to design it without the AI.

- ☐ Requirements → entities → relationships → ERD.
- ☐ Entities → API resources → screens (the contract-first chain).
- ☐ Sequence diagrams (who calls whom, in order): "accept invite", "daily alert".
- ☐ State machines: recipe `private → pending → public`; invite lifecycle.
- ☐ Trade-offs: SQL-side vs Node-side scoring; polling vs realtime; serverless vs persistent server.
- ☐ Failure thinking: what if the cache is empty? cron fails one night? two people edit one item?
- ☐ Separation of concerns; stateless services; where truth lives (the database).
- ☐ ADRs: write down *why*, not just *what*.

**Design drills (do without AI):**
1. After Step 2 → redraw the ERD and explain each table and index.
2. After Step 4 → whiteboard the suggestion pipeline and the LFU cache, including invalidation.
3. After Step 6 → design a **new feature** (e.g. shopping list) on paper: tables, endpoints, screens, risks.

**Done when:** drill 3 is reviewed by a teammate and survives their questions.

---

## M17 — Deployment and DevOps · *Size S · Step 0–1, 8*

- ☐ Environments: local → preview → production. Env vars per environment.
- ☐ Vercel: projects, preview deploys per PR, serverless functions (check current limits in docs).
- ☐ Supabase: dev vs prod projects; applying migrations safely; backups.
- ☐ Release order for DB changes: backward-compatible first (*expand → migrate → contract*).
- ☐ GitHub Actions: lint, typecheck, test, deploy.
- ☐ Logs and error monitoring; smoke test after deploy.

**Build it:** deploy a "hello" app + API with one secret env var; break it; read the logs.
**Check yourself:** Why migrate the DB before deploying code that needs it?
**Done when:** a PR gets a working preview URL.

---

## M18 — Inventory, units, recommendations and estimation · *Size M · Step 1–7*

**Why:** your batch-by-purchase-date stock, the ingredient catalog with units, taste-based suggestions, and fast input mode.

**Batch inventory**
- ☐ A **batch (lot)** = one purchase: quantity, purchase date, its own expiry. Per-ingredient totals are *derived* (a view), never stored separately.
- ☐ **FEFO** (first-expiring-first-out): when stock is used, take from the batch that expires first.
  - Example: batch A = 200 g (expires 5 Oct), batch B = 300 g (expires 20 Oct). Use 250 g → A becomes 0, B becomes 250 g.
  - Why it must be one **transaction with row locks**: two simultaneous cooks could both read A = 200 g and both take from it.
- ☐ Soft delete (a `deleted_at` stamp) so undo works and history survives.

**Catalog and units**
- ☐ One database, linked tables: `ingredient_categories` → `ingredients` → `ingredient_aliases`, plus `units`. Separate databases cannot be joined in one query and add failure points.
- ☐ **Dimension** = what is measured: mass, volume, or count. **Base unit** per dimension: g, ml, piece.
- ☐ Conversion inside a dimension: `quantity_in_base = quantity × to_base`. Example: 2 kg → 2 × 1000 = 2000 g.
- ☐ Spoon and cup sizes vary by country and recipe source. Pick one constant per unit, write it down, and use it everywhere.
- ☐ **Never convert across dimensions** without a per-ingredient factor (e.g., grams per onion). That is why the plan postpones it to Step 7.
- ☐ Money-style rule for amounts: use `numeric` in the database, not floating point (`0.1 + 0.2` is not exactly `0.3` in floats). Round only for display.

**Recommendation basics**
- ☐ **Content-based filtering:** match an item's features (cuisine, diet, ingredients) to what the user likes. This is your taste term.
- ☐ **Hard constraints vs soft scores:** constraints remove; scores rank.
- ☐ **Popularity ranking and the cold-start problem:** with no usage data, "popular" has no meaning yet.
  - Step 1 uses a proxy (how common a recipe's ingredients are in the dataset).
  - Later blend: `p = (1 − α)·proxy + α·real`, where `real` is the normalized view/cook count and α grows as data arrives.
- ☐ Tune weights by looking at real outputs, not by guessing.

**Consumption estimation (fast input mode) — one step at a time**
1. First cook: user says 300 g used for 3 people → per-person usage `x₁ = 300 / 3 = 100 g`. Mean `μ₁ = 100`, count `n = 1`.
2. Second cook: 500 g for 4 people → `x₂ = 125`.
3. Running mean: `μ_(n+1) = (n·μ_n + x_(n+1)) / (n+1) = μ_n + (x_(n+1) − μ_n) / (n+1)`.
   - Check: `μ₂ = 100 + (125 − 100)/2 = 112.5`, and `(100 + 125)/2 = 112.5` ✓.
   - Why this form: it needs only two stored numbers (`μ`, `n`), not the whole history.
4. Later cooks: user enters only persons `p`. Estimated use = `μ × p`.
5. **Servings left** = `remaining / μ`. Example: 450 g left, `μ = 112.5` → 4 servings. With yellow at ≤ 3 and red at ≤ 1, this is still green.
6. **Accepted error:** age groups eat different amounts. Each estimate can be off, and errors add up over many estimated deductions — so the app marks stock `estimated`, lets the user correct it anytime, and nudges when stock is low.
7. **Clamp** estimates at 0 — never show negative stock.

**Build it 🔨:** (a) a pure function `fefoDeduct(batches, amount)` with tests: one batch, many batches, exact, more than available, zero. (b) a running-mean estimator with a property test: its result equals the plain average of all inputs.
**Check yourself:** Why store `(μ, n)` and not every past cook? Why is a total derived, not stored? What breaks if two cooks deduct at the same time without a lock? Why not convert onions to grams in Step 2?
**Done when:** both pure functions pass their tests and you can redo the 450 g example on paper.

---

## Roadmap — learn before each step

| Step | Must-have modules (before starting) | Deepen during the step |
|------|-------------------------------------|------------------------|
| **0 Setup** | M0, M1 (basics), M2 (core), M14 (basics, gate idea) | M17 (basics), M16 (ERD basics) |
| **1 Foundation** (clean data, auth, preferences, homepage) | M2 (async, zod), M3 (basics), M4 (basics + Router), M5 (basics), M6 (basics), M8 (Auth, RLS), M10 (cleaning + parsing), M18 (catalog, units) | M12 (authN/Z), M15, M18 (popularity, cold start) |
| **2 Expiry tracking** (batches, ingredients page) | M6 (joins, constraints, migrations, views), M7 (indexes, EXPLAIN), M4 (forms, Query), M13 (dates), M18 (batches, FEFO, units) | M3 (a11y, colour), M9 |
| **3 Recipe engine** | M9 (sets, scoring incl. taste), M6 (CTEs, functions), M7 (advanced), M18 (content-based scoring) | M14 (oracle/property tests), M15 |
| **4 Alerts + cache** | M9 (LFU), M11, M13, M8 (pg_cron) | M14 (property tests), M5 (serverless) |
| **5 House sharing** | M8 (advanced RLS), M12, M6 (transactions, locks) | M16 (sequence diagrams) |
| **6 Community** | M6 (locking, triggers), M12 (abuse, XSS), M16 (state machines) | M15 (usability test) |
| **7 Input optimisation** | M18 (estimation, thresholds, FEFO), M6 (locking), M15 (warning design) | M14 |
| **8 Hardening** | M14 (Playwright E2E, CI), M12 (audit), M11 (budgets), M15, M17 | M16 (drill 3) |

**Step-gate habit:** at the end of every step, run `npm run gate`, fix what fails, and only then start the next step.

---

## Milestones (are you ready to design it yourself?)

- **Level 1 — after Step 2:** you can redraw the ERD, explain RLS, and trace a click to SQL and back.
- **Level 2 — after Step 4:** you can whiteboard the suggestion engine with the score formula, the LFU structures, and cache invalidation.
- **Level 3 — after Step 6:** you can design a new feature end-to-end on paper and defend the trade-offs.

**Final exam (team):** pick any task from the Action Plan, close the laptop, and explain: the data, the contract, the algorithm, the risk, the test. Then rebuild one piece live without AI.

---

## Master checklist

Copy this table into your tracker and update it as modules finish.

| Module | Owner | Status | Teach-back done |
|--------|-------|--------|-----------------|
| M0 Dev env + Git | | ☐ | ☐ |
| M1 Web basics | | ☐ | ☐ |
| M2 JS → TS | | ☐ | ☐ |
| M3 HTML/CSS/a11y | | ☐ | ☐ |
| M4 React | | ☐ | ☐ |
| M5 Node/Express | | ☐ | ☐ |
| M6 SQL | | ☐ | ☐ |
| M7 Postgres perf | | ☐ | ☐ |
| M8 Supabase | | ☐ | ☐ |
| M9 DSA | | ☐ | ☐ |
| M10 Text + data cleaning | | ☐ | ☐ |
| M11 Caching/perf | | ☐ | ☐ |
| M12 Security | | ☐ | ☐ |
| M13 Time + jobs | | ☐ | ☐ |
| M14 Testing | | ☐ | ☐ |
| M15 UX/HCI | | ☐ | ☐ |
| M16 System design | | ☐ | ☐ |
| M17 Deploy/DevOps | | ☐ | ☐ |
| M18 Inventory, units, recommendations, estimation | | ☐ | ☐ |
