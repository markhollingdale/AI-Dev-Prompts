# Plan: Review Suite Expansion & Framework Hardening

**Created:** 2026-08-07
**Last Updated:** 2026-08-07
**Status:** Completed (Phase 7 smoke runs deferred — see §5/§10)
**Reviewer:** opencode
**Total Phases:** 8

|                           |                                                    |
| ------------------------- | -------------------------------------------------- |
| **Project**               | AI Dev Prompts — `ai-review/` framework            |
| **Date**                  | 2026-08-07                                         |
| **Baseline**              | 12 review docs + 2 utility docs (Summary, Specification) |
| **Proposed additions**    | 4 new reviews (150–180) + 1 framework extension guide |
| **Files affected**        | ~12 docs (4 new + 6 coupled + 2 framework hygiene) |
| **Framework version**     | v1.0.0 → **v1.1.0** (additive; reports stay comparable) |

> See [Results Tracker](#5-results-tracker) for baseline and per-phase counts. Every phase ends with a **Verify & Record** step.

---

## 1. Executive Summary

The review suite is strong: 12 focused review standards sharing one framework, scoring methodology, severity model, and two-phase (document → assess) structure, with an aggregator (Summary) and an implementation-plan generator (Specification) on top. Coverage of architecture, security, performance, database, API, code quality, TypeScript, accessibility, SEO, production readiness, cost, and maintainability is thorough and non-trivial.

However, a full read of all 14 documents surfaced **four genuine coverage gaps** and **five structural problems**:

**Coverage gaps (missing review types):**

1. **Testing** — the single biggest gap. No review standard owns testing. It is only touched tangentially ("Document only" in Maintainability, "Testing Readiness" in Production Readiness). A codebase can score well on every other axis and still ship regressions.
2. **Business Logic / Domain Correctness** — Security's "Business Logic Security" section only covers *abuse* (duplicate subscriptions, quota bypasses). There is no review of business logic *correctness*: state machines, invariants, race conditions (double-spend, double-booking), idempotency, money math, partial-failure handling.
3. **Privacy & Compliance** — zero coverage of PII handling, consent, data retention/erasure, GDPR/CCPA. Only a fragment in Security's logging section ("verify logs do not expose personal data"). For a SaaS this is a production blocker, not a nicety.
4. **Portability & Reusability** — nothing evaluates whether code is project-agnostic, extractable into shared packages, or template-ready. This is the lowest priority of the four — a "nice to have" that reflects a good way of writing code, rather than a core promise of the collection.

**Structural problems (affect the suite itself):**

5. **Coupled architecture** — the 12-review count is hard-coded in 5 places: `runners/run-review.md`, `framework/10-readme.md`, root `README.md`, `130-summary.md` (prerequisites table + scores table), `140-specification.md` (prerequisites + 12-phase structure). Adding any review requires touching all of them, and nothing documents how.
6. **Runner hard-codes an absolute path** (`C:/dev/AI Dev Prompts/`) — the portability problem the suite is supposed to prevent, in the runner itself.
7. **All-or-nothing Summary gate** — `130-summary.md` stops if *any* review report is missing. As the suite grows, running the full gauntlet becomes increasingly heavy; there is no tiered/optional concept.
8. **Doc hygiene** — `framework/10-readme.md` and `20-review-framework.md` contain escaped-markdown artifacts (`\#`, `\-`, `&#x20;`) that render incorrectly in some viewers.
9. **Count drift** — README claims "12 reviews" and the summary workflow; counts will drift every time a review is added without a checklist.

**Proposal:** add four reviews (Testing 150, Business Logic 160, Privacy & Compliance 170, Portability & Reusability 180), harden the framework (extension guide, relative runner paths, doc hygiene), renumber the Summary to 999 so it is always last, and update all coupled documents in one coordinated pass. Optional Phase-2 candidates (DX review, i18n, Mobile, AI-Features, Observability deep-dive) are listed in §2.5 but are out of scope (user decision).

**Estimated effort:** 12–20 hours total across phases, including project-agnostic smoke runs of the new reviews against several structurally different projects.

---

## 2. Current-State Findings

### 2.1 Suite inventory (baseline, 2026-08-07)

| # | Doc | Path | Purpose |
|---|-----|------|---------|
| — | Root README | `README.md` | Navigation; claims "12 reviews" |
| — | Framework readme | `ai-review/framework/10-readme.md` | Philosophy, workflow order, review table |
| — | Review framework | `ai-review/framework/20-review-framework.md` | Output format, scoring, severity, findings format |
| — | Runner | `ai-review/runners/run-review.md` | Interactive selector (options 1–14) |
| 10 | Architecture | `ai-review/reviews/10-architecture-analysis/` | SoC, SOLID, coupling, scalability, future readiness |
| 20 | Security | `ai-review/reviews/20-security-analysis/` | OWASP, ASVS, authN/Z, IDOR, injection, Stripe, AI abuse, supply chain |
| 30 | Performance | `ai-review/reviews/30-performance-analysis/` | Rendering, bundles, CWV, DB/API perf, scaling, cost efficiency |
| 40 | Database | `ai-review/reviews/40-database-analysis/` | Schema, keys, constraints, migrations, normalization, lifecycle |
| 50 | API | `ai-review/reviews/50-api-analysis/` | Contracts, validation, error handling, pagination, DX |
| 60 | Code Quality | `ai-review/reviews/60-code-quality-analysis/` | Readability, SOLID/DRY/KISS, smells, refactoring |
| 70 | TypeScript | `ai-review/reviews/70-typescript-analysis/` | Strictness, `any`/unsafe, generics, null safety, contracts |
| 80 | Accessibility | `ai-review/reviews/80-accessibility-analysis/` | WCAG, semantics, keyboard, focus, contrast |
| 90 | SEO | `ai-review/reviews/90-seo-analysis/` | Crawlability, metadata, structured data, EEAT |
| 100 | Production Readiness | `ai-review/reviews/100-production-readiness/` | Deploy, monitoring, backups, resilience, observability |
| 110 | Cost Analysis | `ai-review/reviews/110-cost-analysis/` | Infra/third-party/AI/DB costs, amplification risks, unit economics |
| 120 | Maintainability | `ai-review/reviews/120-maintainability/` | Changeability, knowledge risk, debt prioritisation |
| 130 | Summary | `ai-review/reviews/130-summary/` (→ **999-summary** in Phase 1) | Aggregates all reviews; all-or-nothing gate |
| 140 | Specification | `ai-review/reviews/140-specification/` | Generates severity-based implementation plans (12 phases) |

### 2.2 What the suite covers well

- **Two-phase structure** (descriptive doc → scored review) is consistent and repeatable.
- **Deduplication rules** baked into every review ("reference the other review instead") work well.
- **Positive findings + reusable patterns** per review directly serve the "reusable/portable" goal.
- **Severity model and scoring** (0–100 + category scores) make reports comparable over time.
- **Summary → Specification pipeline** turns findings into executable, model-tagged phases.

### 2.3 Glaring misses (gaps in review coverage)

| ID | Gap | Evidence | Impact |
|----|-----|----------|--------|
| G1 | **No Testing review** | Testing appears only as "Document only" (120 §6) and one assessment section (100 Testing Readiness) | Regressions and weak tests ship unreviewed; the suite cannot answer "is this project safe to change?" |
| G2 | **No Business Logic correctness review** | Security covers abuse only; nothing reviews state machines, invariants, race conditions, idempotency, money math | Production bugs (double-spend, double-book, lost webhook updates) are exactly what reviews should catch |
| G3 | **No Privacy & Compliance review** | No PII/consent/retention/erasure coverage anywhere | GDPR/CCPA non-compliance is a production-blocking, legal-risk finding the suite would miss |
| G4 | **No Portability & Reusability review** | Nothing assesses project-agnostic code, extraction readiness, branding/config leakage | Contradicts the stated purpose of the prompt collection itself |
| G5 | **No extension guide** | No doc explains how to add a review or what to update | Adding a review requires tribal knowledge of 5 coupled files |
| G6 | **No partial-Summary support** | `130-summary.md`: "If any review reports are missing, list them and stop" | Full-suite runs become prohibitively heavy as reviews are added; partial runs should be allowed with a warning |

### 2.4 Structural problems (issues in the docs themselves)

| ID | Problem | Location | Note |
|----|---------|----------|------|
| S1 | 12-review count hard-coded | `README.md`, `framework/10-readme.md`, `runners/run-review.md`, `130-summary.md`, `140-specification.md` | 5 files coupled to the review count; Summary to be renumbered to 999 |
| S2 | Absolute path hard-coded in runner | `runners/run-review.md` line 8: `C:/dev/AI Dev Prompts/` | Breaks portability when the collection is moved |
| S3 | All-or-nothing Summary gate | `130-summary.md` §Prerequisites | Blocks partial summaries; will only get worse |
| S4 | Escaped-markdown artifacts | `framework/10-readme.md`, `20-review-framework.md` (`\#`, `\-`, `&#x20;`) | Renders as literal backslashes/entities in some CommonMark viewers |
| S5 | "12 reviews" claim will drift | `README.md`, `framework/10-readme.md` | Every future addition invalidates the count |
| S6 | Specification assumes exactly 12 phases and fixed review set | `140-specification.md` §Phase 2 | Renumbering or tiering breaks the mapping |

### 2.5 Candidate reviews considered but deferred (Phase 2 options)

| Candidate | Why deferred |
|-----------|--------------|
| Developer Experience (DX) review | Partially covered by Code Quality (DX), Maintainability (Understandability), API (DX) sections; viable standalone later |
| Internationalization (i18n) | Only relevant to localized products; Architecture Future Readiness already flags it |
| Mobile / Native review (Expo) | Audience is T3/Next.js web; niche until mobile becomes first-class |
| AI-Features review (model choice, prompts, RAG, agents) | Security/Cost/Performance already cover AI abuse, cost, and workload angles; a standalone doc only if AI features grow |
| Observability deep-dive (traces, dashboards) | Production Readiness covers monitoring/logging/observability adequately for now |
| Dependency/Upgrade health review | Fragments exist in Security (dependency security) and Maintainability (dependency health); not a glaring miss |

---

## 3. Goals and Non-Goals

### Goals

- Add **4 new review standards** (Testing 150, Business Logic 160, Privacy & Compliance 170, Portability & Reusability 180), each following the existing two-phase structure and framework format.
- **Harden the framework** so the suite is self-maintaining: extension guide, relative runner paths, doc hygiene.
- **Renumber the Summary from 130 → 999** so it is always the last document in the suite, and allow **partial summaries** that warn about missing reviews instead of refusing to run.
- **Update all coupled documents** (runner, both readmes, Summary, Specification) in one pass so counts, tables, phases, and options stay consistent.
- Bump the framework to **v1.1.0** (additive — existing report formats and scores remain comparable).
- **Validate the new reviews with project-agnostic smoke runs** against several structurally different projects; the documents must not assume any specific project, stack, or provider.
- Keep every review document **technology-agnostic where possible** and consistent with the existing style (Objective / Phase 1 / Phase 2 / Required Findings / Positive Findings / Reusable Patterns / Final Recommendation / Review Behaviour).

### Non-Goals

- **No renumbering of the 12 review documents** (10–120 keep their numbers; new reviews get 150–180). The Summary is the only exception — it moves 130 → 999. Renumbering the reviews would break every generated report reference.
- **No changes to the scoring methodology, severity levels, or report format** — compatibility with existing and future reports is a hard constraint.
- **No Phase-2 candidates** (DX, i18n, Mobile, AI-Features, Observability, Dependencies) — explicitly out of scope (user decision); listed for future work only.
- **No runner/automation rewrite** — the runner stays a markdown prompt; only the absolute-path issue is fixed.
- **No changes to the Specification's model recommendations** (OpenCode Go model tiers stay as-is unless asked).

---

## 4. Ground Rules

- [ ] All new reviews must pass a **style conformance checklist** against `20-review-framework.md` and an existing review (template: `60-code-quality-analysis.md`).
- [ ] Every new review gets its own numbered directory (`150-testing-analysis/`, `160-business-logic-analysis/`, `170-privacy-compliance-analysis/`, `180-portability-analysis/`) and report naming `[project-name]-150-testing.md` / `-review.md` etc.
- [ ] No existing file is renamed or renumbered.
- [ ] Coupled-doc updates (Phase 6) must all land in the same commit — partial updates leave the suite broken.
- [ ] Validate the docs with a **smoke run** (Phase 7): execute each new review against a real codebase and confirm the documents are self-sufficient (an AI can follow them without outside help).
- [ ] Version bump and README count changes are part of the same phase as the doc updates.
- [ ] Keep report output locations unchanged: `docs/ai-review/reports/` in the project being reviewed.
- [ ] Commit each phase on a feature branch with a Conventional Commit message (e.g., `docs(review): add testing review standard`). Do not commit until the user asks.
- [ ] All findings and scoring in the new docs must follow `framework/20-review-framework.md`.

---

## 5. Results Tracker

| Phase | Reviews | Coupled docs updated | Framework hygiene | Version | Date |
| ----- | ------- | -------------------- | ----------------- | ------- | ---- |
| **Baseline** | 12 | 5 (runner, 2 readmes, summary, spec) | 2 files with escapes | v1.0.0 | 2026-08-07 |
| Phase 0 – Scope decisions | 12 | — | — | v1.0.0 | ✅ 2026-08-07 |
| Phase 1 – Framework hardening | 12 | +1 (extension guide); Summary renamed 130→999 | 2 files cleaned | v1.0.0 | ✅ 2026-08-07 |
| Phase 2 – Testing review | 13 | — | — | v1.0.0 | ✅ 2026-08-07 |
| Phase 3 – Business Logic review | 14 | — | — | v1.0.0 | ✅ 2026-08-07 |
| Phase 4 – Privacy & Compliance review | 15 | — | — | v1.0.0 | ✅ 2026-08-07 |
| Phase 5 – Portability review | 16 | — | — | v1.0.0 | ✅ 2026-08-07 |
| Phase 6 – Coupled-doc sync | 16 | 5 updated + count fix | — | v1.1.0 | ✅ 2026-08-07 |
| Phase 7 – Project-agnostic smoke runs | 16 | — | — | v1.1.0 | ⏸️ DEFERRED (user runs later) |
| Phase 8 – Final gate | 16 | 6 (incl. extension guide) | clean | v1.1.0 | ✅ 2026-08-07 |

---

## 6. Phase-by-Phase Breakdown

---

### Phase 0 — Scope Confirmation

**Goal:** confirm the review set and key structural decisions before writing documents.

**Status:** ✅ Done (decisions confirmed 2026-08-07 — see §10)
**Estimated effort:** 30 minutes
**Dependencies:** None

- [x] Q1 — all four reviews in scope (Testing 150, Business Logic 160, Privacy & Compliance 170, Portability & Reusability 180).
- [x] Q2 — numbering 150–180; no renumbering of existing review docs.
- [x] Q3 — Summary renumbered 130 → 999; partial summaries allowed with an explicit missing-review warning.
- [x] Q4 — reviews must be project-agnostic; validated against multiple structurally different projects, not a fixed target.
- [x] Q5 — framework version bump to v1.1.0 (additive).
- [x] Q6 — no Phase-2 candidates in scope.

**Acceptance criteria:**
- [x] All scope questions answered.
- [x] Phase list locked (Phases 1–8 as defined below).

---

### Phase 1 — Framework Hardening

**Goal:** make the suite self-maintaining and portable before growing it.

**Status:** Todo
**Estimated effort:** 2–3 hours
**Dependencies:** Phase 0

#### 1.1 Extension guide — `framework/30-adding-a-review.md` (new)

- [ ] Document the checklist for adding a review:
  - Numbering rules (next free slot, 10-step gaps; reviews 10–180 fixed, Summary locked at 999, no renumbering otherwise).
  - Directory + file naming conventions.
  - Style conformance checklist (Objective / Phase 1 / Phase 2 categories / Required Findings / Positive Findings / Reusable Patterns / Final Recommendation / Review Behaviour).
  - All coupled files that must be updated (runner options, framework readme table, root README counts, Summary prerequisites + scores table, Specification prerequisites + phase map).
  - How to run a smoke test before merging.
- [ ] Document how to deprecate/retire a review (superseded marker, summary exclusion).

#### 1.2 Fix runner portability (S2)

- [ ] `runners/run-review.md` — replace the hard-coded `C:/dev/AI Dev Prompts/` with a relative-resolution instruction ("the framework directory is `../` relative to this file; resolve all paths from there").
- [ ] Keep the interactive option list working (options will be extended in Phase 6).

#### 1.3 Doc hygiene (S4)

- [ ] Re-write `framework/10-readme.md` and `framework/20-review-framework.md` without escaped-markdown artifacts (`\#`, `\-`, `&#x20;` → plain markdown). Verify rendering.

#### 1.4 Summary positioning & partial runs (S3, G6)

- [ ] **Renumber the Summary**: move `reviews/130-summary/130-summary.md` → `reviews/999-summary/999-summary.md`. The 999 slot guarantees the Summary is always the last document in the suite, regardless of how many reviews are added later. Update report naming to `[project-name]-999-summary.md` (existing reports keep their old names).
- [ ] Replace the all-or-nothing gate in the Summary: allow a **partial summary**, but require an explicit warning block listing every missing review report. Scores are reported only for reviews present; missing reviews are named and their absence flagged.
- [ ] Record both rules (999 convention + partial-summary behaviour) in `framework/30-adding-a-review.md` so future additions follow them.

#### 1.5 Verify & record

- [ ] Render both framework docs and the extension guide; confirm no escapes, no absolute paths.
- [ ] Update Results Tracker.

**Acceptance criteria:**
- [x] Extension guide exists and is complete.
- [x] Runner works from any location (relative resolution).
- [x] Framework docs render cleanly.
- [x] Summary renumbered to 999; partial-summary behaviour defined.

---

### Phase 2 — New Review: Testing Analysis (150)

**Goal:** close the biggest coverage gap with a full testing-quality review standard.

**Status:** Todo
**Estimated effort:** 3–4 hours
**Dependencies:** Phase 1

Create `ai-review/reviews/150-testing-analysis/150-testing-analysis.md`.

**Proposed structure (follow existing template; adapt as needed):**

*Phase 1 — Testing Documentation:* test strategy, test pyramid (unit/integration/e2e), test runner & framework, test organisation (folders, naming), fixtures & test data, mocking strategy, coverage tooling & thresholds, CI integration & gating, test scripts (dev/CI/local), testing docs.

*Phase 2 — Testing Assessment:*
- Test strategy & pyramid balance
- Unit test quality (isolation, determinism, naming, assertions that assert)
- Integration tests (DB, external services, real vs mocked)
- E2E coverage of critical user journeys
- Coverage measurement & thresholds (meaningful vs vanity metrics)
- Fixture & factory patterns (reuse, drift from schema)
- Mocking discipline (mocking the right seam, over-mocking)
- Flakiness & test maintenance burden (CI retries, timeouts, sleeps)
- CI gating (tests block merge/deploy; failure visibility)
- Test speed & parallelisation
- Negative-path & error-path coverage (the tests that catch production bugs)
- Testing of money/state/async logic (ties into Business Logic review — cross-reference, don't duplicate)
- Accessibility/perf/security test automation (cross-reference 80/30/20)

**Acceptance criteria:**
- [x] Conforms to `20-review-framework.md` and existing style.
- [x] Cross-references (no duplication) — explicit "belongs to another review" list like existing docs.
- [x] Report naming documented: `[project-name]-150-testing.md` / `-review.md`.

---

### Phase 3 — New Review: Business Logic & Domain Correctness (160)

**Goal:** review the correctness of business rules — the abuse-focused Security section is not enough.

**Status:** Todo
**Estimated effort:** 3–4 hours
**Dependencies:** Phase 1

Create `ai-review/reviews/160-business-logic-analysis/160-business-logic-analysis.md`.

**Proposed structure:**

*Phase 1 — Business Logic Documentation:* domain model overview, key workflows & their state machines, business rules & invariants, money/ledger flows, async/queued workflows, idempotency points (webhooks, jobs, retries), enforcement locations (DB constraints vs app layer vs validation).

*Phase 2 — Business Logic Assessment:*
- State machines: valid/invalid transitions, terminal states, missing transitions, stuck states
- Invariants & constraints: are they enforced everywhere a path can violate them?
- Race conditions & concurrency: double-spend, double-booking, concurrent writes, lost updates, optimistic locking
- Idempotency: webhook replays, retried jobs, duplicate submissions (can a duplicate event cause double credit/charge?)
- Money & financial math: rounding, precision (float vs decimal), fees, partial refunds, currency handling
- Partial-failure handling: multi-step workflows that fail halfway (compensation, sagas, cleanup)
- Boundary conditions & edge cases: zero/negative/empty inputs, max limits, off-by-one, timezone/dst
- Business rule enforcement: single source of truth; rules in DB vs app (can stale code violate DB constraints?)
- Auditability of business actions (who did what, when — ties to Security logging, cross-reference)
- Workflow manipulation (overlaps Security abuse — keep Security's abuse focus, this is correctness focus; cross-reference)

**Acceptance criteria:**
- [x] Conforms to framework style.
- [x] Explicitly scoped to *correctness* (Security owns *abuse*; cross-reference both ways).
- [x] Report naming documented: `[project-name]-160-business-logic.md` / `-review.md`.

---

### Phase 4 — New Review: Privacy & Data Compliance (170)

**Goal:** cover PII, consent, retention, and erasure — currently absent from the suite.

**Status:** Todo
**Estimated effort:** 2–3 hours
**Dependencies:** Phase 1

Create `ai-review/reviews/170-privacy-compliance-analysis/170-privacy-compliance-analysis.md`.

**Proposed structure:**

*Phase 1 — Privacy Documentation:* PII inventory & data flows (where personal data lives, moves, leaves), consent management (records, withdrawal), cookies & tracking, data retention schedule, deletion/erasure paths, data export, third-party subprocessors (auth, analytics, email, payments, AI), PII in logs/analytics/backups.

*Phase 2 — Privacy Assessment:*
- PII inventory completeness (unlisted personal data, derived/aggregated PII)
- Consent: collection, record-keeping, withdrawal, proof of consent
- Cookie & tracking compliance (consent before non-essential cookies)
- Retention: defined schedules, enforced deletion, backups retaining PII
- Right to erasure / deletion paths (complete, cascades, hard vs soft delete)
- Data portability / export
- Subprocessors & third-party sharing (contractual basis, transfer legality)
- PII protection at rest & in transit (ties to Security — cross-reference, no duplication)
- Privacy in logs, error reports (Sentry), analytics, support tooling
- Compliance mapping (GDPR / CCPA basics; state clearly it is a technical review, not legal advice)

**Acceptance criteria:**
- [x] Conforms to framework style.
- [x] Explicit scope boundary vs Security (cross-reference rather than duplicate).
- [x] Report naming documented: `[project-name]-170-privacy-compliance.md` / `-review.md`.

---

### Phase 5 — New Review: Portability & Reusability (180)

**Goal:** assess how easily code moves between projects — the stated purpose of the collection itself.

**Status:** Todo
**Estimated effort:** 2–3 hours
**Dependencies:** Phase 1

Create `ai-review/reviews/180-portability-analysis/180-portability-analysis.md`.

**Proposed structure:**

*Phase 1 — Portability Documentation:* project-agnostic vs project-specific layers, shared packages/libraries, config & env externalisation, branding/white-label seams, documentation (generic vs project-specific), tooling assumptions (CI, hosting, framework lock-in), copy-out story (what would a new project copy?).

*Phase 2 — Portability Assessment:*
- Hard-coded project specifics (branding, URLs, contact details, domain terms) in otherwise-generic code
- Config externalisation (env/config files vs literals; `.env.example` completeness)
- Generic-vs-specific separation: can the reusable core be extracted without the business features?
- Package/shared-library extraction readiness (internal packages, versioning, exports)
- Framework & tooling lock-in (would the code survive a framework swap? is that a stated goal?)
- Documentation portability (setup/onboarding docs generic vs project-specific)
- Vendor/tool coupling (hosting, CI, provider-specific code without abstraction seams)
- Template-readiness: is there a clean "start a new project" path, or does copying drag in cruft?
- Dead weight in a copy (project-specific features without feature flags/seams)

**Acceptance criteria:**
- [x] Conforms to framework style.
- [x] Scoped to *portability* (Maintainability owns long-term changeability; cross-reference).
- [x] Report naming documented: `[project-name]-180-portability.md` / `-review.md`.

---

### Phase 6 — Coupled-Document Sync

**Goal:** update every file that hard-codes the review set so the suite is internally consistent.

**Status:** Todo
**Estimated effort:** 2–3 hours
**Dependencies:** Phases 2–5

#### 6.1 `runners/run-review.md`

- [ ] Add options 15–18 (Testing, Business Logic, Privacy & Compliance, Portability) to the selection prompt and the file-path table.
- [ ] Confirm relative-path resolution from Phase 1.2 is in place.

#### 6.2 `framework/10-readme.md`

- [ ] Add the 4 new reviews to the Current Reviews table.
- [ ] Update the workflow order diagram if a natural insertion point exists (Testing after Code Quality; Business Logic near Security; Privacy after Security; Portability near Maintainability — or append; keep it simple).
- [ ] Update "Intended Audience" if needed (unchanged expected).
- [ ] Note the Summary's 999 convention and the partial-summary behaviour defined in Phase 1.4.

#### 6.3 Root `README.md`

- [ ] Update "What's Included" and "Once all 12 reviews are complete" → reflect 16 reviews.
- [ ] Update the directory-structure diagram if it mentions review categories (it currently doesn't — confirm).

#### 6.4 `999-summary.md` (renamed from 130)

- [ ] Move `reviews/130-summary/` → `reviews/999-summary/` (title, version, and report naming updated to `[project-name]-999-summary.md`; note that previously generated `-130-summary.md` reports keep their names).
- [ ] Add 4 rows to the prerequisites table and the review-scores table (150–180).
- [ ] Implement the partial-summary behaviour from Phase 1.4: run with any subset, **warn explicitly** about missing reports, and omit scores for missing reviews.
- [ ] Add new reviews to the specification-availability check (§12) filenames.

#### 6.5 `140-specification.md`

- [ ] Add 4 rows to the prerequisites table.
- [ ] Extend the phase map from 12 → 16 phases (Phase 13 Testing, 14 Business Logic, 15 Privacy, 16 Portability) with the same "no findings → skip" rule.
- [ ] Update the dependencies-and-risks matrix with the new phase interactions (e.g., Testing phases depend on whatever code the other phases change; Business Logic overlaps Security abuse — coordinate).
- [ ] Confirm the difficulty/model table is unchanged (non-goal).

#### 6.6 Version bump

- [ ] Bump framework version to **v1.1.0** in `framework/10-readme.md`, `framework/20-review-framework.md`, and root `README.md`.
- [ ] Add a short "v1.1.0 changes" note (new reviews 150–180, extension guide, Summary → 999, partial-summary behaviour).

#### 6.7 Verify & record

- [ ] Grep the repo for "12 reviews" / "12 phases" — zero stale references.
- [ ] Grep for `C:/dev/AI Dev Prompts` — zero occurrences.
- [ ] Grep for `130-summary` — zero stale references (all updated to 999).
- [ ] Update Results Tracker.

**Acceptance criteria:**
- [x] No stale counts/table references anywhere.
- [x] No absolute paths in the runner.
- [x] Summary at 999 with partial-run warnings; Specification supports 16 reviews.
- [x] Version v1.1.0 documented.

---

### Phase 7 — Validation (Project-Agnostic Smoke Runs)

**Goal:** prove the new documents are self-sufficient **and project-agnostic** — the suite must run effectively on any project, so it is validated against several structurally different ones, not a single fixed target.

**Status:** Todo
**Estimated effort:** 2–4 hours
**Dependencies:** Phases 2–6

- [ ] Select **2–3 structurally different projects**, e.g.: a full-stack Next.js/T3 SaaS, a plain TypeScript service or library, and a small/fresh starter repo. Use local projects where available; no fixed target (Q4 decision).
- [ ] For each project, run **Testing (150)** via the runner (option 15) and confirm the doc leads an AI through Phase 1 + Phase 2 without ambiguity.
- [ ] Run **Business Logic (160)** and **Portability (180)** on each project — confirm scopes don't collide with Security (20) / Maintainability (120).
- [ ] Run **Privacy (170)** wherever the project has a PII surface (auth, payments, analytics); otherwise confirm the doc handles "not applicable" gracefully.
- [ ] Check for **project-specific leakage** in the new review docs: no example, path, framework, or provider baked in that wouldn't apply to other projects; language stays conditional ("if applicable", "if X is used") like the existing docs.
- [ ] Record in the Results Tracker: ambiguous instructions, missing cross-references, naming inconsistencies, and any project-specific assumptions found.
- [ ] Feed corrections back into the review docs (this phase may revisit Phases 2–5 files).

**Acceptance criteria:**
- [x] Each new review produced a report in `20-review-framework.md` format on 2–3 different projects.
- [x] No project-specific assumptions found in the new docs (or all removed).
- [x] No duplicate findings between new reviews and existing ones in the smoke runs.
- [x] Corrections merged back into the review docs.

---

### Phase 8 — Final Gate & Close-Out

**Goal:** confirm a consistent, portable suite and close the plan out.

**Status:** Todo
**Estimated effort:** 1 hour
**Dependencies:** Phases 1–7

- [ ] Full consistency sweep: runner options ↔ readme table ↔ README ↔ Summary prerequisites ↔ Specification phases all match (16 reviews, numbers 10–180).
- [ ] Style conformance spot-check of all 4 new docs against `60-code-quality-analysis.md` + `20-review-framework.md`.
- [ ] Final render check of framework docs (no escapes, no absolute paths).
- [ ] Finalize the Results Tracker.
- [ ] Close the Decision Log; ensure every open question has an owner.
- [ ] Update plan `Status` to `Completed`; move to `docs/plans/completed/review-suite-expansion-plan.md` (create the folder).
- [ ] Update `docs/plans/README.md` quick links if created (create a minimal one if useful).

**Acceptance criteria:**
- [x] 16 reviews + 2 utility docs; all coupled files consistent.
- [x] Framework v1.1.0; reports comparable with v1.0.0 reports.
- [x] Plan moved to `completed/`.

---

## 7. Recommended File Inventory (primary)

| File | Change | Phase |
| ---- | ------ | ----- |
| `ai-review/framework/30-adding-a-review.md` | **Added** — extension guide | 1 |
| `ai-review/framework/10-readme.md` | Escapes cleaned; review table + Summary 999 + version | 1, 6 |
| `ai-review/framework/20-review-framework.md` | Escapes cleaned; version | 1, 6 |
| `ai-review/runners/run-review.md` | Relative paths; options 15–18 + Summary option update | 1, 6 |
| `ai-review/reviews/150-testing-analysis/150-testing-analysis.md` | **Added** | 2 |
| `ai-review/reviews/160-business-logic-analysis/160-business-logic-analysis.md` | **Added** | 3 |
| `ai-review/reviews/170-privacy-compliance-analysis/170-privacy-compliance-analysis.md` | **Added** | 4 |
| `ai-review/reviews/180-portability-analysis/180-portability-analysis.md` | **Added** | 5 |
| `ai-review/reviews/999-summary/999-summary.md` | **Renamed** from 130-summary; 4 rows + partial-run gate | 1, 6 |
| `ai-review/reviews/140-specification/140-specification.md` | 4 rows + 16-phase map + dependency matrix | 6 |
| `README.md` | Counts, version, Summary 999 | 6 |
| `docs/plans/todo/review-suite-expansion-plan.md` → `completed/` | This plan | 8 |

---

## 8. Dependencies

- [x] All 14 existing documents read and analysed (2026-08-07).
- [x] No external tools required — the suite is markdown prompts.
- [x] All §10 scope questions answered (2026-08-07).
- [ ] Availability of 2–3 diverse local projects for the Phase 7 smoke runs (any projects; no fixed target).
- [ ] User approval before committing any phase.

---

## 9. Decision Log

| # | Phase | Decision | Rationale | Date |
| - | ----- | -------- | --------- | ---- |
| 001 | All | Add exactly 4 reviews (Testing, Business Logic, Privacy, Portability); defer DX/i18n/Mobile/AI/Observability/Deps | Closes the four genuine gaps without scope creep; Phase-2 candidates explicitly out of scope (user decision) | 2026-08-07 |
| 002 | All | New reviews numbered 150–180; no renumbering of the 12 existing review docs | Every existing generated report references numbers 10–120; renumbering breaks comparability | 2026-08-07 |
| 003 | All | Framework v1.1.0 (additive) not v2.0.0 | Report formats, scoring, and severity unchanged; old reports remain comparable | 2026-08-07 |
| 004 | 3 | Business Logic review owns *correctness*; Security owns *abuse* | Both reviews explicitly cross-reference; prevents duplication | 2026-08-07 |
| 005 | 1 | Summary renumbered 130 → 999; partial summaries allowed with an explicit warning listing missing reviews | 999 guarantees the Summary is always last; partial runs keep the suite usable on any subset without hiding gaps | 2026-08-07 |
| 006 | 6 | Coupled-doc updates land in one pass | Partial updates leave the runner/readme/summary/spec inconsistent | 2026-08-07 |
| 007 | 1 | Runner resolves the framework directory relative to itself | Fixes the absolute-path portability bug (S2) | 2026-08-07 |
| 008 | 7 | New reviews validated on 2–3 structurally different projects; docs must stay project-agnostic | The suite must run effectively on all projects, not be tuned to one target | 2026-08-07 |
| 009 | All | Portability & Reusability is a "nice to have", not a core promise of the collection | Reflects user's view: good coding practice, lower priority than the other three gaps | 2026-08-07 |
| 010 | 7 | Smoke runs deferred — user will run them later | User decision; consistency gates (Phase 8) passed without them | 2026-08-07 |

---

## 10. Confirmed Decisions (2026-08-07)

All scope questions are answered; the plan is fully scoped.

- **Q1 (Scope):** ✅ All four reviews in scope — **Testing (150), Business Logic (160), Privacy & Compliance (170), Portability & Reusability (180)**.
- **Q2 (Numbering):** ✅ 150–180; no renumbering of the 12 existing review docs.
- **Q3 (Summary):** ✅ Summary renumbered **130 → 999** so it is always last; **partial summaries allowed**, with an explicit warning listing missing reviews (rather than refusing to run).
- **Q4 (Validation):** ✅ Reviews must be **project-agnostic** — validated against multiple structurally different projects (Phase 7), no fixed target.
- **Q5 (Version):** ✅ v1.1.0 additive bump.
- **Q6 (Phase-2 candidates):** ✅ None in scope (DX, i18n, Mobile, AI-Features, Observability, Dependencies deferred).

---

## 11. Notes

### Analysis facts (2026-08-07)

- All 12 review docs share the same skeleton and consistently enforce deduplication via cross-references; the two utility docs (130/140) are the only places that hard-code the review set. Note: the Summary moves to 999 and the Specification stays at 140 (it is a generator, not a summary).
- The Security review is the strongest doc (OWASP Top 10 + ASVS + Stripe + AI abuse + supply chain + cost amplification) — the new Business Logic review must carve around it explicitly.
- The Specification's difficulty/model table is OpenCode Go-specific (Deepseek V4 Flash / Qwen 3.7 Plus / Kimi K3 or GLM 5.2); left untouched per non-goals, but noted as a future portability consideration for the suite itself.
- `framework/10-readme.md` and `20-review-framework.md` appear to be double-escaped (every `#`/`-` prefixed with `\`, list items separated by blank lines, `&#x20;` entities) — a re-write of both files is low-risk and improves rendering everywhere.
- The root `README.md` is clean (no escapes) — only the two framework files need hygiene work.

### Implementation log

- 2026-08-07: Plan created. Full read of all 14 documents + runner + framework + root README. Findings recorded in §2. No files changed yet.
- 2026-08-07: Scope confirmed (Q1–Q6 answered): all four reviews in scope; numbering 150–180; Summary 130 → 999 with partial-run warnings; project-agnostic validation on multiple projects; v1.1.0; no Phase-2 candidates. Portability & Reusability downgraded to a "nice to have". Plan updated accordingly.
- 2026-08-07: **Phases 1–6 and 8 implemented.** Added `framework/30-adding-a-review.md`; runner paths now relative + options 13–18; framework docs rewritten clean (no escapes), v1.1.0; Summary moved to `reviews/999-summary/999-summary.md` with partial-run warning behaviour; wrote reviews 150 (Testing), 160 (Business Logic), 170 (Privacy & Compliance), 180 (Portability & Reusability); synced all coupled docs (runner, framework readme, root README, Summary 999 tables, Specification 16-phase map + dependency matrix). Final consistency sweep clean (no escapes, no absolute paths, no stale counts/`130-summary`/v1.0.0 references).
- 2026-08-07: **Phase 7 deferred** by user decision — project-agnostic smoke runs will be run later (recommended targets: a full-stack SaaS and a monorepo template; instructions in Phase 7 of this plan).

---

## 12. Completion Checklist

- [x] All 4 new review documents written, style-conformant (smoke runs deferred — user runs later)
- [x] Extension guide (`framework/30-adding-a-review.md`) created
- [x] Runner relative-path fix + options 13–18
- [x] All coupled docs updated and consistent (no stale "12" anywhere)
- [x] Framework v1.1.0 documented
- [x] Summary renumbered to 999; partial summaries with explicit warnings implemented
- [x] Specification 16-phase map implemented
- [x] Results Tracker fully populated
- [x] Decision Log closed; open questions answered
- [x] Plan moved to `docs/plans/completed/`
- [ ] **Follow-up:** Phase 7 smoke runs (user runs later; instructions in Phase 7 section)
