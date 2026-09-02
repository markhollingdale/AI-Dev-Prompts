# Pre-Live Gate — Single Night-Before Checklist

> **Owner:** on-call before launch. This is the **only** gate for `PRODUCTION READY: YES` in `ai-review/reviews/999-summary`. Every box must be checked or explicitly accepted as risk. References `100-production-readiness` (platform), `20-security` (abuse), `30-performance` (cache), `90-seo` (robots), `40-database` (PITR), `150-testing` (proof).

---

## 0. Code Checks (run `ai-review/runners/run-review.md` — the reviews prove in code)

- [ ] `run-review.md` → **20 Security** → 0 Critical, no unthrottled `search.search`/`venue.search` (High). Evidence: report at `docs/ai-review/reports/*-20-security-review.md`.
- [ ] **30 Performance** → ISR `export const revalidate` on `events/[slug]`, `venues/[slug]`, `whats-on` (P0 from previous pack) + `app/api/trpc/[trpc]/route.ts` not global `no-store` on public reads. Report at `*-30-performance-review.md`.
- [ ] **90 SEO** → `app/robots.ts` splits AI bots `Disallow: /`, sitemap canonical-only, canonical `noindex` on filtered `whats-on`. Report at `*-90-seo-review.md`.
- [ ] **40 Database** → PITR window documented + last restore drill date filed (not `NEVER`). Report at `*-40-database-review.md`.

If any review is not run, `999-summary` must emit **WARNING: partial summary** — provisional only.

---

## 1. Platform Toggles (dashboards — code cannot prove ON)

Vercel — Project → Settings:

- [ ] **Security / Firewall → Bot Management ON** (Managed Challenge — catches spoofed GPTBot). Drill: known.
- [ ] **Firewall → Custom Rules → Rate limit** `100 req/min` on `/api/trpc` (+ `60/min` on search if path rule supported) → `Challenge`/`429`.
- [ ] **Attack Challenge Mode** — locate toggle, name the 2am owner (add to on-call doc).
- [ ] **Billing → Spend Management → Alerts 50/75/100%** + Spend Limits/Pause if tier supports.
- [ ] Headers shipped: Deployment → Headers shows `public, s-maxage` not `no-store` on ISR pages.

Neon — Project → Settings:

- [ ] **Pooled `DATABASE_URL` (`-pooler.`)** in Vercel env, not direct. `echo $DATABASE_URL | grep pooler` passes.
- [ ] Compute auto-suspend tuned, `max_connections` vs Vercel concurrency noted.
- [ ] **PITR enabled** + **last restore drill date: ______** (branch restored, `SELECT COUNT(*) > 0` attached).

Supabase if used: Pooling `6543` ON + same drill date.

Detail steps: `docs/runbooks/vercel-neon-manual-setup.md`.

---

## 2. Live Curl Gate (5 min — paste output into `docs/ai-review/reports/*-100-production-readiness.md` Phase 1)

```bash
# robots split
curl -s https://<domain>/robots.txt
# expect GPTBot/ClaudeBot Disallow: /, Googlebot Allow, Sitemap: /sitemap.xml

# sitemap hygiene
curl -s https://<domain>/sitemap.xml | grep -E "filter|sort|page=" ; echo "count=$?"  # expect count 0

# ISR cache hit
curl -s -I https://<domain>/events/some-real-slug | grep -iE "cache-control|x-vercel-cache"
# expect public, s-maxage=300, stale-while-revalidate, second request HIT

# burst throttle
for i in $(seq 1 20); do curl -s -o /dev/null -w "%{http_code}\n" "https://<domain>/api/trpc/search.search?batch=1&input=%7B%22q%22%3A%22a%22%7D"; done | sort | uniq -c
# expect at least one 429

# spoofed UA still blocked
curl -s -o /dev/null -w "%{http_code}\n" -A "Mozilla/5.0 (compatible; GPTBot/1.0)" https://<domain>/events/some-real-slug
```

---

## 3. Testing Proof (required artifacts — `150-testing-analysis.md` gates)

- [ ] `tests/robots.test.ts` exists and CI passed (AI bots Disallow).
- [ ] `tests/rate-limit.test.ts` exists and CI passed (burst → `429`).
- [ ] `k6 run load/crawl-burst.js` (100 VUs * 30s PostGIS search) — attach p95 + 429 share.
- [ ] `k6 run load/baseline.js` (50 VUs steady) — attach p95 + `max_connections` headroom.
- [ ] Backup restore drill artifact: branch name + date + queries (see §1 Neon).
- [ ] Synthetic smoke job: `health → login → search → view → track` cron/canary URL + alert channel + last run date: ______.
- [ ] `axe` + `lighthouse` CI gate (accessibility + visual) last green: ______.
- [ ] Browser matrix manual date (Chrome/Firefox/Safari × mobile): ______.
- [ ] Seed scrub: `prisma/seed.ts` contains no plausible real PII in prod deploy.

Scripts: `docs/runbooks/abuse-red-team-playbook.md` (scenarios 1-6).

---

## 4. Resilience & Legal

- [ ] SLO written (p95 latency, error rate) + burn alert + on-call owner.
- [ ] Email deliverability: SPF/DKIM/DMARC `dig TXT` + `mail-tester.com` score > 9 + bounce/complaint handling documented.
- [ ] Analytics filters bots (PostHog/Amp crawler UA excluded — MAU not inflated).
- [ ] UGC (if present): moderation queue + `report` → triage SLA + block-user audit — per `20-security.md` §B conditional.
- [ ] Supply chain: `package-lock.json` present, `npm audit` gate in CI, GH Actions pinned SHA (Dependabot ON).
- [ ] Legal pages live and match behaviour: `/privacy`, `/terms`, cookie banner (consent-before-load, `170-privacy`).
- [ ] Incident runbook: who toggles Challenge, who restores DB, comms plan — rehearsed once.

---

## 5. Sign-Off

| Role | Name | Date | Gate OK? | Accepted risks |
|------|------|------|----------|----------------|
| On-call / launch owner |  |  |  |  |
| DB owner (Neon/Supabase) |  |  |  |  |
| Reviewer (signed `999-summary`) |  |  |  |  |

Attach this file + curl outputs + `k6` JSON + drill artifact to the release PR. Do not launch with unchecked boxes unless risk is explicitly listed in `Accepted risks` and approved.
