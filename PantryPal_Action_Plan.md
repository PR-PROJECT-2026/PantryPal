# PantryPal — Action Plan (v1.1 · 2 Oct 2026)

> **Purpose.** Instruction document for the AI and the team: it turns your described workflow into build steps with tech stack, sub-workflows, careful zones, parallel tracks and a **step gate** (no step starts until the previous one runs and passes its tests).
> **Authority order:** (1) the user's latest instruction → (2) this plan → (3) AI judgement.
> **Marker `[CONFIRM-n]`** = something I assumed. Defaults apply until you answer (§14).
> **Companion:** `PantryPal_Learning_Path.md`.
>
> **What changed from v1.0** (after re-reading your full script and the dataset screenshots):
> 1. Every requirement you stated is now an ID (`REQ-1…15`) mapped to a step/task (§1) — nothing is implied.
> 2. Stock is stored as **batches by purchase date** (quantity + expiry per purchase), not one row per item.
> 3. Data cleaning, ingredient identity and the homepage's local/global lists moved into **Step 1**, as you described.
> 4. **Input optimisation (fast input mode, safety thresholds, defaults) is the last feature step (Step 7)**; early steps avoid it.
> 5. Added the **Step Gate** (§8) and a **parking list** of risky topics deferred on purpose (§9).
> 6. Answer to your ingredient-database question (§3): one database, a catalog of linked tables — not separate databases.

---

## 0. AI Operating Rules (strict)

| # | Rule |
|---|------|
| R1 | Before any work: read the handoff (memory) and this plan. State your **Step / Task ID**. |
| R2 | Build **only** what the current Step lists. No extra features; propose, don't build. |
| R3 | One task at a time, in order, using workflow **W1** (test first, interface before code). |
| R4 | No secrets in code. Never use the Supabase **service-role key** in request handlers or the browser. |
| R5 | Every table has **RLS enabled at creation**. Migrations are new files; never edit an applied one. |
| R6 | Don't deviate from Default Decisions (§2) without an ADR (W11) and the user's OK. |
| R7 | After each task give a **3–5 line explanation**: what changed, why, which Learning Path module it uses. |
| R8 | After every action, **update the handoff** (W10). |
| R9 | Unknown → mark `[CONFIRM]` and ask. Never guess silently. |
| R10 | New dependency needs: name, why, one alternative considered. |
| R11 | If a task touches a **careful zone** (§7), say so before continuing. |
| R12 | **Never start Step N+1 until Step N's gate (§8) passes.** Report the gate result. |
| R13 | Anything on the **parking list** (§9) stays out until its step arrives. |

---

## 1. Source of truth

**Idea.** A household website that tracks expiry dates, stores item info, **passively suggests recipes** from the user's ingredients and taste, and reminds the user before items expire or run out.

**Dataset:** `IndianFoodDatasetCSV.csv` (24.15 MB, 15 columns). Your screenshots show 10: `Srno, RecipeName, TranslatedRecipeName, Ingredients, TranslatedIngredients, PrepTimeInMins, CookTimeInMins, TotalTimeInMins, Servings, Cuisine`. The other 5 are not visible — task S0-7 lists them `[CONFIRM-1]`. Observations from the screenshot:
- Ingredients are **one comma-separated string with quantities**: `6 Karela (Bitter Gourd/ Pavakkai) - deseeded, Salt - to taste, 1 Onion - thinly sliced, 3 tablespoon Gr…`
- Pattern per item: `[quantity] [unit] Name (alternate names / separated) - preparation note`, and `to taste` has no number.
- Both `Ingredients` and `TranslatedIngredients` exist; some originals may be non-English → **use the Translated columns**.
- `Servings` and the time columns are numeric → usable for filters and, later, for scaling.

### Requirements ledger (your words → IDs)

| ID | Requirement | Step | Tasks |
|----|-------------|------|-------|
| REQ-1 | Clean the chosen dataset, identify ingredients | 1 | S1-1, S1-2 |
| REQ-2 | Process the data so the website can use it | 1 | S1-2, S1-4 |
| REQ-3 | Database for authentication / user login | 1 | S1-3, S1-5, S1-7 |
| REQ-4 | At first use ask location and taste preferences; store them | 1 | S1-3, S1-5, S1-8 |
| REQ-5 | Minimal professional homepage, menu button, navigation; locality-famous and global-famous recipes (by frequency) | 1 | S1-6, S1-9 |
| REQ-6 | Ingredients page: boxes, click-to-edit quantity, add item, search + edit, expiry-ahead tags | 2 | S2-2…S2-5 |
| REQ-7 | Stock stored by purchase time: purchase date, quantity per date, expiry date | 2 | S2-1, S2-2, S2-4 |
| REQ-8 | Ingredient catalog with entities (liquids, fruits, vegetables, spices…) and specific units — or a better way to handle quantity/identification | 1–2 | §3, S1-2 |
| REQ-9 | Passive recommendation by the user's likeliness + expiry-based ingredient management, with a DB schema built for it | 3 | S3-1…S3-8 |
| REQ-10 | Depletion alert for user-specified ingredients | 4 | S4-4…S4-6 |
| REQ-11 | Fast input mode: first-cook quantity + people served; portion error accepted; warn before safety threshold | 7 | S7-1…S7-4 |
| REQ-12 | Suggestions warn about fast-mode mistakes; red/yellow sign asking to update quantity | 7 | S7-5 |
| REQ-13 | User-set defaults per ingredient to cut repeated input — **last** | 7 | S7-6 |
| REQ-14 | Each step runs and is tested with no bugs/errors at completion | all | §8 gate |
| REQ-15 | Early on, avoid big topics that could crash the site; don't over-focus on input optimisation | all | §9 |

**Build order (yours):** 1 Foundation → 2 Expiry tracking → 3 Recipe engine (core) → 4 Alerts (depletion, cache strategy) → 5 House sharing → 6 Community (public after 3–4 cooks) → last: input optimisation.
**Added by me:** **Step 0** (setup, contracts, gate script) and **Step 8** (hardening, release). No new user features.
**Stack:** React · Node · PostgreSQL · Supabase · Vercel · LFU cache. **Focus:** DB speed, UX friction, frontend appeal. **Team:** 5.

---

## 2. Default decisions (ADR index)

| ID | Decision | Default | Why |
|----|----------|---------|-----|
| ADR-001 | Language | **TypeScript (strict)** `[CONFIRM-9]` | Catches bugs early; shared types UI↔API |
| ADR-002 | Repo | Monorepo, npm workspaces: `apps/web`, `apps/api`, `packages/shared`, `supabase/`, `data/`, `docs/` | One place for contracts |
| ADR-003 | Data access | UI → Node API → Supabase Postgres with the **user's JWT** (RLS always on). Service role only in scheduled jobs | Two locks: API checks + RLS |
| ADR-004 | Hosting | Express on **Vercel** (serverless) `[CONFIRM-6]` | One platform; cache is per-instance (ADR-006) |
| ADR-005 | Scheduler | **pg_cron** runs idempotent SQL functions (Step 4) | Next to the data |
| ADR-006 | Cache | Pure-TS **LFU** in the API (Step 4), best-effort, keys carry `pantry_version` | Correct without cross-instance invalidation |
| ADR-007 | Matching | Set-based scoring in **SQL** + TS oracle for tests | DB speed focus; double-checked |
| ADR-008 | Ingredient identity | `ingredients` + `ingredient_aliases` + `ingredient_categories` + `units`; `is_staple` | See §3 |
| ADR-009 | Time | Expiry/purchase = **DATE**; "today" in household tz (default `Asia/Kolkata`), passed explicitly | Avoids the midnight/UTC bug |
| ADR-010 | Deletes | Batches are **soft-deleted** | Undo, audit |
| ADR-011 | Frontend | Vite + React + Router + TanStack Query + Tailwind; no global state lib | Server state belongs to Query |
| ADR-012 | Testing | Vitest, Testing Library, MSW, Supertest, pgTAP, fast-check, **Playwright smoke from Step 1** | Gate needs runnable checks |
| ADR-013 | Matching input | **Presence-based** until Step 7 (quantities not used for matching) `[CONFIRM-8]` | Units are a bug farm |
| ADR-014 | Community threshold | `PUBLIC_COOK_THRESHOLD = 3`, author excluded `[CONFIRM-4]` | You said 3–4 |
| ADR-015 | Households | Every user gets a **personal household automatically** at signup; one household per user in v1 `[CONFIRM-5]` | No friction; sharing (Step 5) just adds members |
| ADR-016 | Inventory model | **Batches** (one row per purchase) + derived per-ingredient totals; deduction **first-expiring-first-out (FEFO)** | Your purchase-date requirement |
| ADR-017 | Quantity storage | Number + unit, same **dimension** per ingredient; no cross-dimension conversion until Step 7 `[CONFIRM-15]` | Avoids "2 onions = how many grams" crashes |
| ADR-018 | Feature flags | Risky features behind flags in `packages/shared/flags.ts` | Ship without crashing |

---

## 3. Ingredient catalog and quantity design (your question)

**Short answer: one database, several linked tables. Not separate databases.**

Why not separate databases for liquids / fruits / spices: they can't be joined in one SQL query, so the recommendation engine would need many round-trips; three databases triple the connection limits, deploys and failure points — exactly the "crash before running" risk you want to avoid (REQ-15).

**You still get your entities and per-entity units:**

```
ingredient_categories   Liquids · Fruits · Vegetables · Spices · Grains & flours · Dairy · Meat & eggs · Oils & fats · Other
   └─ each has a dimension (volume | mass | count) and a default unit
units                   g, kg (mass) · ml, l, tsp, tbsp, cup (volume) · piece, bunch, clove (count)  — each with a factor to a base unit (g / ml / piece)
ingredients             canonical_name, category_id, primary_unit, is_staple
ingredient_aliases      "aloo", "batata", "potatoes" → potato
```

**Rule:** an ingredient inherits its category's allowed units (it may override). A batch must use a unit from the ingredient's dimension; the API rejects "milk in kg" (Review Focus #3).

**Identifying which item it is (from messy dataset text):** parse each item into `(quantity, unit, name, alternate_names[], prep_note)`, then map `name` + alternates to a canonical ingredient through aliases; unmatched names go to a report that is reviewed **by frequency** (fix the top 100 first). Fuzzy matching is **offline only** (in the data pipeline), never at runtime in early steps.

**Quantity problem, staged:**
- Steps 2–6: store what the user types; match recipes by **presence**; units only validated within a dimension.
- Step 7: recipe quantities (dataset has them) and `Servings` let the app estimate consumption; cross-dimension conversion (e.g., onion pieces ↔ grams) needs a per-ingredient factor — added only then.

---

## 4. Architecture

```
Browser (React SPA, Vercel)
   │ HTTPS + JWT
   ▼
Node API (Express, zod, [LFU cache — Step 4])  ──► Supabase Auth (JWT)
   │ user-JWT client (RLS applies)
   ▼
Supabase Postgres ◄── pg_cron (Step 4) → alerts
   ▲
   └── offline data pipeline (CSV → clean → parse → map → seed)
```

---

## 5. Tech stack and CS domains

| Layer | Tool | First used |
|-------|------|-----------|
| Language | TypeScript, SQL, Python (pandas) for the pipeline | 0 |
| Frontend | React, Vite, Router, TanStack Query, Tailwind | 1 |
| Backend | Node (LTS), Express, zod | 1 |
| Database | PostgreSQL via Supabase; `pg_cron` (Step 4); `pg_trgm` only if measured necessary | 1 |
| Auth | Supabase Auth + RLS | 1 |
| Hosting | Vercel (web + API), Supabase | 1 |
| Cache | Own LFU module | 4 |
| Tests | Vitest, Testing Library, MSW, Supertest, pgTAP, fast-check, Playwright | 0–1 |
| Tooling | Git/GitHub, GitHub Actions, ESLint, Prettier | 0 |

| CS domain | Where | Depth |
|-----------|-------|-------|
| Databases and data modelling | Catalog, batches, views, RLS, indexes, transactions | **Deep** |
| Data structures and algorithms | Sets, scoring, FEFO, LFU, running mean | **Deep** |
| Data engineering / text parsing | Dataset cleaning, ingredient-string parser, alias mapping | Strong |
| Recommendation basics | Content-based scoring, popularity, cold start | Strong |
| Backend / web | REST, JWT, validation | Strong |
| Frontend | React, async state, forms, responsive, a11y | Strong |
| Security | Authorization, RLS, invites, XSS | Strong |
| Caching and performance | Hit ratio, eviction, versioned keys, percentiles | Strong |
| Units and measurement modelling | Dimensions, base units, conversion | Working |
| Distributed/cloud basics | Serverless, cron, idempotency, time zones | Working |
| Software engineering | TDD, Git, CI, contract-first, ADRs | Strong |
| HCI / UX | Friction, empty/error states, colour semantics | Working |
| Statistics (light) | Running mean for consumption estimates | Working |
| ML | Not required for v1 | Skip |

---

## 6. Parallel-work model and sub-workflows

**Tracks:** **DB** (migrations, RLS, SQL functions) · **API** · **UI** · **DATA** (pipeline, catalog, seeds) · **OPS/QA** (CI, deploys, gate, tests).
**Contract-first:** schema, zod schemas in `packages/shared`, and the endpoint list are frozen at the start of each step. UI uses MSW mocks until the API lands; API uses fixtures until real data lands. A contract change needs every track's OK (PR label `contract`).

**W1 Task loop (TDD).** Read task+contract → write types/signature → failing test → minimal code → run all tests → refactor → manual check → commit (`feat:`/`fix:`/`test:`) → explain (R7) → update handoff.
**W2 Database change.** New migration → create table → enable RLS + policies → pgTAP *allowed* and *denied* tests → `supabase db reset` from zero → seed realistic volume → `EXPLAIN ANALYZE` each new query → regenerate types → PR.
**W3 API endpoint.** zod schema in shared → route (validate → user-JWT client → service) → service = logic, repository = SQL → one error shape → Supertest 200/400/401/403/404 → `docs/api.md`.
**W4 UI feature.** Design 4 states (loading/empty/error/success) → build on MSW → keyboard + contrast → wire with TanStack Query → check 360/768/1280 px → **friction count** in the PR.
**W5 Data pipeline.** Raw data immutable → profile → clean → parse → map → validate (`assert`) → deterministic export → load → reconcile counts (in = out + logged rejects).
**W6 Performance.** Budget → baseline → find bottleneck (`EXPLAIN`, profiler) → fix lowest layer first (index → query → cache → UI) → re-measure → log in `docs/perf.md`.
**W7 Git/merge.** Branch `step-N/track/desc`; PR < ~400 lines; `main` always deployable; rebase daily; squash-merge.
**W8 Debugging.** Reproduce → isolate layer (Network → API log → SQL) → one hypothesis → minimal fix → regression test.
**W9 Release.** Preview deploy → DB migration (backward-compatible first) → API → UI → smoke test → tag.
**W10 Handoff update.** After every action: status, change, decisions, blockers, single next action, open `[CONFIRM]`s.
**W11 ADR.** Context → options → decision → consequences (one page, `docs/adr/`).
**W12 Step gate.** Run §8 checklist; write `docs/gates/step-N.md`; tag `step-N-complete`; update handoff.

---

## 7. Careful zones

| # | Zone | Risk | Guardrail |
|---|------|------|-----------|
| Z1 | RLS | Self-referencing policy recurses; missing policy leaks data | `SECURITY DEFINER` helper `is_member()`; pgTAP denied tests per table |
| Z2 | Service-role key | Bypasses RLS | Jobs only; never browser/handlers |
| Z3 | Client-supplied IDs | Someone sends another household's ID | Derive access from JWT + membership |
| Z4 | Dates / time zones | "Expires today" shows expired on a UTC server | DATE type; explicit `today`; boundary tests (−1, 0, 1) |
| Z5 | Ingredient identity | "tomato/tomatoes/tamatar" duplicates | Canonical + aliases; frequency-ordered review |
| Z6 | Staples | Salt/oil/water break "have vs need" | `is_staple` excluded |
| Z7 | Diet labels | Wrong veg/non-veg label destroys trust | Cross-check against ingredients; no allergen promises |
| Z8 | Cache keys | Missing `household_id` leaks suggestions | Key = household + version + today + query; membership check **before** lookup |
| Z9 | Serverless cache | Per-instance, dies on cold start | Accelerator only |
| Z10 | Scheduled jobs | Retries create duplicate alerts | Unique `dedupe_key` + `ON CONFLICT DO NOTHING` |
| Z11 | Invite links | Guessable/reusable | ≥128-bit token, expiry, atomic single-use |
| Z12 | Concurrent edits | Silent last-write-wins | `updated_at` check → 409 |
| Z13 | Community gaming | Author cooks own recipe | Distinct users, exclude author, locked transaction |
| Z14 | Injection / XSS | Sort param in SQL; recipe text as HTML | Whitelist; parameterized; no `dangerouslySetInnerHTML` |
| Z15 | Slow queries | Seq scans as data grows | `EXPLAIN` in every PR with a new query |
| Z16 | Connections | Serverless exhausts DB connections | Supabase pooler string |
| Z17 | Dataset licence/images | Using content without rights | Check licence; attribute; no scraped images |
| Z18 | Colour-only meaning | Unreadable for colour-blind; glass cards fail contrast | Colour + icon + text; contrast over blur |
| **Z19** | **Units / dimensions** | Milk in kg; "2 onions" vs grams; sums across units | Dimension check on every write; sum only in base units within a dimension |
| **Z20** | **Batch deduction (FEFO)** | Two edits drain the same batch; negative stock | One transaction, row locks, `CHECK (quantity >= 0)` |
| **Z21** | **Recipe-string parsing** | Misread `1/2`, ranges, commas inside parentheses, "to taste" | Parenthesis-aware split; parser unit tests on real dataset rows; log rejects |
| **Z22** | **Personal → shared household** | User with a stocked personal pantry joins a friend's household | Defined merge behaviour `[CONFIRM-20]`; transaction; test |
| **Z23** | **Estimates (Step 7)** | Estimated stock drifts; goes negative; nags the user | Clamp ≥ 0; mark `estimated`; confidence decays; nudge limits |
| **Z24** | **Location privacy** | Over-collecting location | Store state/city only; GPS not required `[CONFIRM-19]` |

---

## 8. Step Gate (REQ-14) — a step is "done" only when all pass

| # | Check | Evidence |
|---|-------|----------|
| G1 | Fresh clone → `npm ci` → `npm run gate` passes (lint, typecheck, unit, integration) | CI log |
| G2 | `supabase db reset` builds schema + seeds from zero; pgTAP green | CI log |
| G3 | `npm run dev` starts with documented env; health endpoints OK; **no console/server errors** on this step's flows | Manual run + log |
| G4 | The step's **Playwright smoke test** passes (and every earlier step's smoke) | CI log |
| G5 | All earlier tests still pass (regression) | CI log |
| G6 | Preview deploy works with the step's demo script | Preview URL |
| G7 | Review Focus items relevant to the step have tests | Test names |
| G8 | No open defect above "minor"; known issues listed | `docs/gates/step-N.md` |
| G9 | Tag `step-N-complete`; handoff updated | Git tag, handoff |
| G10 | Each teammate runs the demo script once and explains one task (teach-back) | Checklist |

`npm run gate` (built in S0-8) runs G1–G5 in one command. **If any check fails, the step is not done and the next step is blocked (R12).**

---

## 9. Parking list (REQ-15) — postponed on purpose

| Topic | Why postponed | Arrives |
|-------|---------------|---------|
| Fast input mode, safety thresholds, estimated stock | Input optimisation is last | Step 7 |
| Per-ingredient defaults, default shelf-life suggestions | Input optimisation is last | Step 7 |
| Cross-dimension unit conversion | Crash/bug farm | Step 7 |
| Runtime fuzzy search (`pg_trgm`) | Only if prefix/alias search proves insufficient | Step 2 (measured) |
| LFU cache, `pantry_version` | Cache strategy belongs to Alerts | Step 4 |
| Realtime updates, email, web push | Extra services | After Step 5 / optional |
| Moderation tooling, admin UI | Not described | Optional |
| ML recommender, image hosting | Not described | Not planned |

---

## 10. The steps

### Step 0 — Setup and contracts *(added)*
| ID | Track | Task | ✔ Done when |
|----|-------|------|-------------|
| S0-1 | OPS | Monorepo skeleton, strict TS, ESLint/Prettier | Lint + typecheck pass on a fresh clone |
| S0-2 | OPS | GitHub repo, protected `main`, CI (lint, typecheck, test) | Failing PR cannot merge |
| S0-3 | DB | Supabase dev project + local CLI; `.env.example`; secrets policy | `supabase start` works |
| S0-4 | DB | ERD + schema contract (§11) reviewed by all five | ERD in `docs/`, sign-off in handoff |
| S0-5 | API | Endpoint list + zod schemas in `packages/shared`; `flags.ts` | Both apps import shared and build |
| S0-6 | UI | Design tokens (maroon/plum, saffron, cream), layout, kit page | `/dev/kit` renders all states |
| S0-7 | DATA | Load `IndianFoodDatasetCSV.csv`: list all 15 columns, row count, nulls, distinct `Cuisine` values, `Servings` range, duplicate names; check licence | Findings note resolves `[CONFIRM-1]` |
| S0-8 | OPS | `npm run gate` script + Playwright smoke skeleton ("home page loads") | Gate runs green on an empty app |

**Parallel:** after S0-1, S0-3/4 ∥ S0-5 ∥ S0-6 ∥ S0-7. **Careful:** Z2, Z17. **Gate:** G1–G5.

### Step 1 — Foundation (data cleaning, database/auth, preferences, homepage)
**Goal (REQ-1…5, 8):** a user signs up, answers a short preference form, and lands on a homepage showing local and global popular recipes from the cleaned dataset.
**Contract:** `profiles, households, household_members, user_preferences, ingredient_categories, units, ingredients, ingredient_aliases, recipes, recipe_ingredients, recipe_popularity, region_cuisine_map`; `GET /me`, `PUT /me/preferences`, `GET /home`.

| ID | Track | Task | ✔ Done when |
|----|-------|------|-------------|
| S1-1 | DATA | **Clean:** use Translated columns; fix types; drop/flag empty rows and duplicates by recipe name; validate times and servings | `recipes_clean.csv`; reject log; counts reconcile |
| S1-2 | DATA | **Parse + identify ingredients:** parenthesis-aware split of the ingredient string → `(quantity, unit, name, alternate_names, prep_note)`; build categories, units, canonical ingredients, aliases, staples; map; frequency-ordered unmatched report | Parser unit tests on ≥30 real rows (incl. `1/2`, ranges, "to taste", commas in parentheses); mapped-ingredient % reported |
| S1-3 | DB | Migration 001: `profiles`, `households`, `household_members`, `user_preferences` (location = state/city, diet, liked cuisines, disliked ingredients), signup trigger that creates profile **and personal household**; RLS via `is_member()` | pgTAP: A cannot read B's rows |
| S1-4 | DB | Migration 002: catalog + recipes + `recipe_popularity` + `region_cuisine_map` (read-only for users); seed loader | `supabase db reset` loads all seeds; counts match S1-1/S1-2 |
| S1-5 | API | Auth middleware (verify JWT; user-JWT client); `GET /me`; `PUT /me/preferences` (zod) | 401/401/200 tests; invalid preferences → 400 |
| S1-6 | API | `GET /home`: **local list** (recipes whose cuisine matches the user's region via `region_cuisine_map`) and **global list**, both ranked by `popularity_score` | Works for a user with no location (falls back to global) |
| S1-7 | UI | Sign up / log in / log out, session persistence, protected routes | Refresh keeps session |
| S1-8 | UI | First-use preference screen(s): location, diet, liked cuisines, optional dislikes; skippable; ≤ 3 screens | Friction count + time-to-first-value noted |
| S1-9 | UI | Homepage: top **menu button**, clean navigation, two sections ("Popular near you", "Popular everywhere"), minimal recipe cards, empty states | Verified at 360/768/1280 px |
| S1-10 | OPS | Preview deploy + Playwright smoke: signup → preferences → homepage shows both lists | Smoke green on preview |

**"Famous by frequency" (REQ-5):** the dataset has no popularity numbers. v1 `popularity_score` is a documented **proxy** (how common a recipe's ingredients are across the dataset); it is later blended with real view/cook counts as users arrive `[CONFIRM-16]`.
**Parallel:** S1-1/2 ∥ S1-3 ∥ S1-7/8/9 (MSW). S1-4 needs S1-2's output shape; S1-5/6 need S1-3/4.
**Careful:** Z1, Z2, Z3, Z5, Z17, Z21, Z24; auth redirect URLs (open redirect).
**Gate:** G1–G10 for Step 1.

### Step 2 — Expiry tracking (ingredients page, batches, tags)
**Goal (REQ-6, 7):** manage stock as **batches by purchase date** with expiry tags. **Contract:** `pantry_batches`, view `pantry_ingredient_totals`; `GET /ingredients?q=`, `GET /households/:id/pantry` (grouped by ingredient, batches nested), batch `POST/PATCH/DELETE`.

| ID | Track | Task | ✔ Done when |
|----|-------|------|-------------|
| S2-1 | DB | Migration: `pantry_batches(household_id, ingredient_id, quantity, unit, purchased_on DATE, expiry_date DATE null, added_by, created_at, updated_at, deleted_at, quantity_source default 'entered', last_confirmed_at)` + `CHECK (quantity >= 0)`; view totals; partial index `(household_id, ingredient_id, expiry_date) WHERE deleted_at IS NULL`; RLS | pgTAP + `EXPLAIN` with 10k seeded batches uses the index |
| S2-2 | API | `GET /ingredients?q=` (alias exact → prefix; limit 10) | "aloo"→potato; "tomatoes"→tomato |
| S2-3 | SHARED | `expiryStatus(expiryDate, today)`, `validateQuantity(ingredient, unit, qty)` (dimension rule) | Boundary tests −1, 0, 1, 3, 7, null; "milk in kg" rejected |
| S2-4 | API | Pantry list grouped by ingredient (total, nearest expiry, batches); add batch (purchase date default today, **expiry entered by user**); edit quantity/expiry; soft delete + undo | Supertest all codes; edit stamps `updated_at` |
| S2-5 | UI | **Ingredients page:** one box per ingredient (name, category icon, total, nearest-expiry tag); expand → batches (purchase date · quantity · expiry); click quantity to edit inline | Works with MSW, then API |
| S2-6 | UI | Add-new-item drawer (search → quantity+unit → purchase date → expiry); search bar that finds and edits; tags = colour + icon + text; sort by expiry; filter by status | Add-item friction count recorded |
| S2-7 | OPS | Playwright smoke: add item → appears with correct tag → edit quantity → search finds it | Green |

Status thresholds default: **expired** < 0 days · **use soon** 0–3 · **soon-ish** 4–7 · **fresh** > 7 · **no date** `[CONFIRM-3]` (constants in `packages/shared`).
**Parallel:** S2-1 ∥ S2-3 ∥ S2-5/6 (MSW). S2-2/4 after S2-1.
**Careful:** Z4, Z5, Z12, Z15, Z19, Z20 (editing quantity must keep `>= 0`).
**Gate:** G1–G10 for Step 2.

### Step 3 — Recipe engine (the heart)
**Goal (REQ-9):** passive, ranked suggestions from the user's stock **and taste**, fast and reliable. **Contract:** SQL `suggest_recipes(household_id, user_id, today, filters, limit, offset)`; `GET /households/:id/suggestions`, `GET /recipes/:id`.

**Scoring spec** (`docs/scoring.md`, constants in shared):
- `R` = recipe's required non-staple ingredients; `P` = household's ingredients with **a batch with quantity > 0 that is not expired** (staples excluded).
- `have = |R ∩ P|`, `need = |R|`, `missing = need − have`; coverage `c = have/need` (if `need = 0`, `c = 1`).
- Urgency per matched ingredient: `w = 1/(1+max(d,0))`, `d` = days left on its **earliest-expiring unexpired batch** (no date → `w = 0`); `u = Σw / need`.
- **Taste (likeliness):** `t = 1` if the recipe's cuisine ∈ the user's liked cuisines, else `0`. **Hard filters:** diet mismatch or any disliked ingredient → excluded.
- `score = c + λ·u + τ·t` (defaults `λ = 1`, `τ = 0.25`); show `missing ≤ maxMissing` (default 3); ties → fewer missing → shorter total time.
- Quantities are **not** used (ADR-013). Response also carries each used ingredient's `quantity_source` so Step 7 can warn.

| ID | Track | Task | ✔ Done when |
|----|-------|------|-------------|
| S3-1 | SHARED | TS reference implementation (**oracle**) of the spec | Unit tests on hand-computed cases |
| S3-2 | DB | `suggest_recipes()` + supporting indexes | pgTAP fixture; **property test: SQL ranking == oracle** on random fixtures |
| S3-3 | DB/OPS | Performance on the full dataset | Proposed budget DB ≤ 100 ms, in `docs/perf.md` `[CONFIRM-14]` |
| S3-4 | API | Suggestions endpoint: sort/filter (diet, course, time), pagination; matched/missing/urgent items | Tests: empty pantry, no matches, injection in sort param |
| S3-5 | API | Recipe detail (ingredients with quantities; steps if the dataset has them) | 404 for unknown |
| S3-6 | UI | Suggestions page: cards (match %, missing chips, "uses expiring: onion (2 d)"), sort/tag controls | 4 states verified |
| S3-7 | UI | Expanded recipe card | Keyboard accessible |
| S3-8 | UI | **Passive** home widget "Cook with what's expiring" (top 3) | Appears with no search |
| S3-9 | OPS | Smoke: stock → suggestion appears → expanded card opens | Green |

No cache yet: build `SuggestionService.cacheKey()` as a pure function so Step 4 plugs in.
**Parallel:** S3-1 ∥ S3-6/7/8 (MSW); S3-2 after S3-1; S3-4 after S3-2.
**Careful:** Z5, Z6, Z7, Z14, Z15, Z17.
**Gate:** G1–G10 for Step 3.

### Step 4 — Alerts (expiry reminders, depletion watch, cache)
**Goal (REQ-10):** remind before expiry (with recipe links); alert when a **user-chosen ingredient** runs low; cache repeat reads. **Contract:** `alert_preferences`, `depletion_watches(household_id, ingredient_id, threshold, unit)`, `notifications` (unique `dedupe_key`), `households.pantry_version`.

| ID | Track | Task | ✔ Done when |
|----|-------|------|-------------|
| S4-1 | DB | Migration: tables above + `pantry_version` trigger; RLS | pgTAP denied-cases |
| S4-2 | DB | `scan_expiring_batches()` + `pg_cron` daily (household-local date) | Run twice → zero duplicates (Review Focus #5) |
| S4-3 | API | Notifications list / read / unread count; preferences | 403 on others' rows |
| S4-4 | API+DB | **Depletion watch:** user picks ingredients + threshold; check after every quantity change and in the daily scan; re-arms when restocked | Watch fires once at threshold, again only after restock |
| S4-5 | UI | Bell + dropdown + page; expiry alert deep-links to "recipes using X" | Link opens filtered suggestions |
| S4-6 | UI | "Watch this ingredient" toggle + threshold on the ingredient box | Works for any ingredient |
| S4-7 | API | `LfuCache<K,V>` (capacity + TTL, O(1)) | Unit + fast-check vs naive reference |
| S4-8 | API | Apply cache to suggestions (`sug:{hh}:{version}:{today}:{queryHash}`), recipe detail, ingredient search; log hit ratio | Before/after benchmark; keep only wins |
| S4-9 | OPS | Smoke: expiring item → alert appears; watched item below threshold → alert | Green |

**Parallel:** S4-1/2/4 ∥ S4-7 (pure module) ∥ S4-5/6 (MSW). S4-8 last.
**Careful:** Z8, Z9, Z10, Z4; LFU pollution → TTL.
**Gate:** G1–G10 for Step 4.

### Step 5 — House sharing (shared pantry, member roles)
**Goal:** several people manage one pantry. **Contract:** roles `owner|member`; `household_invites`.

| ID | Track | Task | ✔ Done when |
|----|-------|------|-------------|
| S5-1 | DB | Invites (random token, expiry, max uses, revoked); role-aware RLS | pgTAP per role |
| S5-2 | API | Create/revoke invite, accept (atomic single-use), members list, remove, leave, transfer ownership; last owner can't leave | Race test: two accepts of one single-use code → one wins |
| S5-3 | API | **Joining with an existing personal pantry:** apply the chosen merge behaviour `[CONFIRM-20]` in one transaction | Test: batches move, none lost or duplicated |
| S5-4 | API | Optimistic concurrency on batch edits (409) | Two-client test |
| S5-5 | UI | Members page, invite share, join flow surviving login, role badges | Returns to join flow after login |
| S5-6 | UI/API | Freshness: refetch on focus + 30 s polling | Member A's edit appears for B |

**Parallel:** S5-1 ∥ S5-5 (MSW). **Careful:** Z1, Z3, Z8 (membership check before cache), Z11, Z12, **Z22**.
**Gate:** G1–G10 for Step 5.

### Step 6 — Community recipes (integrity: public after N cooks)
**Contract:** `recipes.source/author_id/visibility`, `cook_logs`, `recipe_reports`. State machine `private → pending → public`.

| ID | Track | Task | ✔ Done when |
|----|-------|------|-------------|
| S6-1 | DB | Columns + `cook_logs` (`unique(recipe_id,user_id)`) + `record_cook()` that inserts and promotes in one locked transaction | Concurrent cooks → exactly one promotion |
| S6-2 | API | Own-recipe CRUD (ingredients from the catalog), submit, cook, report; diet-vs-ingredients validation | Tests per visibility state |
| S6-3 | UI | Recipe creator, "My recipes", badges (Private / Verifying 2/3 / Public), "I cooked this" | Badge updates after a cook |
| S6-4 | API | Visibility rules in suggestions and detail | Private never visible to others |
| S6-5 | API | Material edit of a public recipe → back to pending, count reset `[CONFIRM-4]` | Edit-after-public test |
| S6-6 | API/UI | Minimal report + auto-hide at N `[CONFIRM-13]` | Hidden recipe leaves suggestions |

**Careful:** Z7, Z13, Z14. **Gate:** G1–G10 for Step 6.

### Step 7 — Input optimisation (last feature step; REQ-11, 12, 13)
**Goal:** less typing, with honest warnings when stock is only estimated. **Contract:** `cook_events`, `consumption_profiles(household_id, ingredient_id, per_person_amount, unit, samples)`, `ingredient_defaults`; `quantity_source` / `last_confirmed_at` already exist from Step 2.

**Fast input mode (your idea):**
1. First time the user cooks with an ingredient: they enter **how much they used** and **how many people were served**.
2. The app stores `per_person = used / persons` (a running mean over later cooks: `μ_new = μ + (x − μ)/(n+1)`).
3. Next time: the user enters only **persons**; the app estimates usage = `μ × persons`, deducts it from batches **FEFO** and marks them `estimated`.
4. Portion differences between age groups are an accepted error; the user can correct any quantity at any time (it becomes `entered` again).

**Safety threshold:** `servings_left = remaining / μ`. **Yellow** when `servings_left ≤ T_yellow`, **red** when `≤ T_red` (defaults 3 and 1, user-adjustable `[CONFIRM-18]`).

| ID | Track | Task | ✔ Done when |
|----|-------|------|-------------|
| S7-1 | DB | Migration: `cook_events`, `consumption_profiles`, `ingredient_defaults`; deduction function (FEFO, locked, clamped ≥ 0) | pgTAP: multi-batch deduction, concurrent cooks, never negative |
| S7-2 | API | `POST /cook` (persons, recipe optional; first-time amounts); updates profile; marks `estimated` | Tests: first-time vs repeat; clamping |
| S7-3 | API | Status engine: green/yellow/red per ingredient + confidence (`entered` recent vs `estimated`) | Boundary tests at thresholds |
| S7-4 | UI | Cook flow: persons stepper; first-time amount prompt; one-tap repeat | Friction count ≤ 3 taps on repeat |
| S7-5 | UI/API | **Suggestion warnings:** chip/banner "based on estimated stock", yellow/red markers on ingredients below threshold, "Update quantity" quick action | Estimated + low ingredient shows the right sign |
| S7-6 | UI | Per-ingredient defaults (unit, usual quantity, usual shelf-life days) that prefill the add form | Add-item taps drop vs Step 2 baseline |
| S7-7 | API | Cross-dimension conversion for recipe quantities (per-ingredient factor) — **only if** flagged on `[CONFIRM-15]` | Behind `flags.ts` |
| S7-8 | OPS | Smoke: cook with persons → stock reduces → warning appears when low | Green |

**Careful:** Z19, Z20, Z21, **Z23** (estimates never negative; confidence decays; limit nudges).
**Gate:** G1–G10 for Step 7.

### Step 8 — Hardening and release *(added)*
| ID | Track | Task | ✔ Done when |
|----|-------|------|-------------|
| S8-1 | QA | Full Playwright E2E over every step's smoke path | Green on CI |
| S8-2 | ALL | Performance vs budgets (Lighthouse, API p95, `EXPLAIN`); fix top 3 | Numbers in `docs/perf.md` |
| S8-3 | DB/API | Security pass: RLS audit (every table, denied tests), rate limits, `helmet`, CORS allow-list, dependency audit | Checklist signed |
| S8-4 | UI | UX pass: friction audit, empty/error states, keyboard/contrast, mobile; test with 3–5 real users | Top issues fixed |
| S8-5 | OPS | Demo seed, prod migration via CI, backup note | Fresh environment in one command |
| S8-6 | OPS | Error + request logging, cache hit-ratio log | Forced error visible |
| S8-7 | ALL | README, architecture note, runbook, final handoff | New person runs it from README |

**Project done when** every step's gate is green **on the production URL** and the Review Focus list passes.

---

## 11. Schema contract (v1.1 draft for S0-4)

```
profiles(id = auth.users.id, display_name, created_at)
households(id, name, timezone default 'Asia/Kolkata', pantry_version bigint [Step 4], created_by)
household_members(household_id, user_id unique, role 'owner'|'member', joined_at)
user_preferences(user_id pk, state, city, diet, liked_cuisines text[], disliked_ingredient_ids bigint[], onboarded_at)

ingredient_categories(id, name, dimension 'mass'|'volume'|'count', default_unit)
units(code pk, dimension, to_base numeric)            -- base: g / ml / piece
ingredients(id, canonical_name unique, category_id, primary_unit, is_staple)
ingredient_aliases(alias pk, ingredient_id)

recipes(id, source 'dataset'|'user', title, cuisine, course null, diet null, servings, prep_min, cook_min, total_min,
        instructions null, url null, author_id null, visibility 'private'|'pending'|'public')
recipe_ingredients(recipe_id, ingredient_id, raw_text, quantity null, unit null, prep_note null, is_optional, pk(recipe_id, ingredient_id))
recipe_popularity(recipe_id pk, popularity_score, view_count, cook_count)
region_cuisine_map(state, cuisine)

pantry_batches(id, household_id, ingredient_id, quantity >= 0, unit, purchased_on DATE, expiry_date DATE null,
               added_by, created_at, updated_at, deleted_at null, quantity_source 'entered'|'estimated', last_confirmed_at)
pantry_ingredient_totals  -- VIEW: per household+ingredient: total (base unit), nearest unexpired expiry, batch count

alert_preferences(user_id, household_id, days_before default 3, enabled)                       -- Step 4
depletion_watches(id, household_id, ingredient_id, threshold, unit, enabled, armed)            -- Step 4
notifications(id, user_id, household_id, type, batch_id null, ingredient_id null, dedupe_key unique, created_at, read_at)  -- Step 4
household_invites(token, household_id, created_by, expires_at, max_uses, used_count, revoked)  -- Step 5
cook_logs(id, recipe_id, user_id, household_id, cooked_at, unique(recipe_id, user_id))         -- Step 6
recipe_reports(id, recipe_id, reporter_id, reason, created_at)                                 -- Step 6
cook_events(id, household_id, recipe_id null, persons, cooked_at)                              -- Step 7
consumption_profiles(household_id, ingredient_id, per_person_amount, unit, samples, updated_at) -- Step 7
ingredient_defaults(household_id, ingredient_id, unit, usual_quantity null, shelf_life_days null) -- Step 7
```
Tables arrive with the step that needs them.

---

## 12. Review Focus (must be tested)

1. **Date boundary.** Item expiring today at 23:59 local shows *Use soon*, not *Expired*, whatever the server timezone. (Step 2)
2. **Nothing matches / empty pantry.** Suggestions show a helpful empty state, never blank or error. (Step 3)
3. **Same ingredient, different spelling or wrong unit.** "Tomatoes" when "tomato" exists maps to one ingredient; milk in kg is rejected with a clear message. (Steps 1–2)
4. **Cross-household access.** Guessing another household's ID or invite code returns 403/404 and leaks nothing, including via cache. (Steps 1, 4, 5)
5. **Duplicate alerts.** Re-running the scan or a retry never creates a second notification for the same item/day or the same depletion event. (Step 4)

---

## 13. Definition of Done (every task)

☐ Test first and passing · ☐ Lint/typecheck green · ☐ Authorization/RLS checked (if data touched) · ☐ 4 UI states (if UI) · ☐ `EXPLAIN` (if new query) · ☐ Explanation (R7) · ☐ Handoff updated (R8). **Every step additionally passes the Step Gate (§8).**

---

## 14. Open questions (defaults apply until you answer)

| ID | Question | Default |
|----|----------|---------|
| CONFIRM-1 | The 5 columns not visible in screenshots (likely Course, Diet, Instructions, TranslatedInstructions, URL — **unverified**). Send the Column tab or let S0-7 report it | Recipe card shows steps only if Instructions exists |
| CONFIRM-2 | Roles: only owner/member? | Yes |
| CONFIRM-3 | Expiry thresholds | 0–3 / 4–7 / >7 |
| CONFIRM-4 | Community: 3 or 4 cooks? Author counts? Who may cook a *pending* recipe? Edit-after-public? | 3, author excluded, household + link holders, edits reset to pending |
| CONFIRM-5 | One household per user (personal one auto-created)? | Yes |
| CONFIRM-6 | API on Vercel serverless vs a persistent Node host? | Vercel |
| CONFIRM-7 | Alert channels | In-app only in v1 |
| CONFIRM-8 | Presence-based matching until Step 7? | Yes |
| CONFIRM-9 | TypeScript vs JavaScript | TypeScript |
| CONFIRM-10 | What did the PPT mean by "cache strategy" (what is cached)? | Suggestions, recipe detail, ingredient search |
| CONFIRM-11 | Login methods | Email + password; Google optional |
| CONFIRM-12 | Staples list | Seed (salt, oil, water, turmeric…), editable |
| CONFIRM-13 | Moderation depth | Report + auto-hide only |
| CONFIRM-14 | Performance budgets | DB ≤ 100 ms · API p95 ≤ 300 ms warm · first content ≤ 1 s on 4G |
| CONFIRM-15 | Ingredient catalog: one database with categories + units (my recommendation, §3) vs separate databases | One database, linked tables |
| CONFIRM-16 | "Famous by frequency": what is the frequency of — views, cooks, or dataset commonness? Locality = state/city → regional cuisine | Proxy first, real counts blended later |
| CONFIRM-17 | Taste preferences to ask at first use | Diet, liked cuisines, disliked ingredients |
| CONFIRM-18 | Yellow/red definitions for the safety threshold | Servings left ≤ 3 yellow, ≤ 1 red |
| CONFIRM-19 | Location input | State/city dropdown (no GPS) |
| CONFIRM-20 | User with a stocked personal pantry joins a household | Move their batches into the shared household |
| CONFIRM-21 | Expiry date always typed by the user until Step 7 defaults | Yes |

---

## 15. Handoff protocol

Memory file `pantrypal-handoff` is the live **state** (current step/task, gate results, decisions, blockers, single next action, open `[CONFIRM]`s). This plan is the **spec**. After every action the AI updates the handoff (W10).
