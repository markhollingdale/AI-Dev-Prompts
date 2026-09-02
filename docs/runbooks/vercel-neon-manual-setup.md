# Vercel + Neon/Supabase Manual Setup — Pre-Launch Click Checklist

> Code reviews (`ai-review/reviews/100-production-readiness`, `20-security`, `30-performance`) can verify **headers in code** but cannot verify **toggles in dashboards**. Run this once before going live, then re-verify after any Vercel/Neon plan change.

---

## 1. Vercel — Firewall & Spend Safeguards (~10 min)

Project → **Settings → Security / Firewall**

- [ ] **Bot Management** → **ON** (Managed Challenge for unverified bots). This is the only thing that catches spoofed `GPTBot` by ASN verification — a `robots.txt` alone does not.
- [ ] **Custom Rules** → Create rule:
  - Name: `rate-limit-api-trpc`
  - If: `Path` `starts with` `/api/trpc`  (add second rule for `/api/` if you have REST routes)
  - Then: `Rate Limit` `100 requests per minute per IP` → Action `Challenge` (or `Block` returning `429`). For expensive search paths, create tighter `60/min` rule on `/api/trpc/search.search` or `venue.search` if Vercel supports path suffix matching; otherwise rely on code-level `@upstash/ratelimit`.
  - Test: `curl -i https://<domain>/api/trpc/search.search?batch=1&input=%7B%220%22%3A%7B%22q%22%3A%22test%22%7D%7D` burst 60/min from one IP → expect `429` + `Retry-After`.
- [ ] **Attack Challenge Mode** — locate the single toggle in Firewall tab. Document **who** can toggle at 2am (Vercel RBAC) and drill once: toggle ON → verify site shows challenge page → toggle OFF. This is your kill-switch for a spike; knowing where it is saves hours.
- [ ] **Headers shipped:** Deployment → Headers tab → verify `Cache-Control: public, s-maxage=..., stale-while-revalidate=...` on ISR pages, `no-store` only on mutations/auth. `vercel.json` + `next.config.ts` headers must appear here.

Project → **Settings → Billing → Spend Management** (Team-level if on Pro)

- [ ] **Spend Alerts** at `50%`, `75%`, `100%` of monthly budget — add Slack/email recipient that is actually monitored.
- [ ] **Spend Limits / Pause Project** if tier supports — set to `150%` of expected so a crawl cannot 10x you silently.
- [ ] Verify **Notifications** go to the same channel as on-call.

Project → **Settings → Environment Variables**

- [ ] `CRON_SECRET` set and **not** `NEXT_PUBLIC_*`; cron routes gate on it (`x-cron-secret` header check).
- [ ] Upstash `UPSTASH_REDIS_REST_URL` / `UPSTASH_REDIS_REST_TOKEN` set if using Edge middleware rate limiting.

---

## 2. Vercel — Robots & Sitemap (code + verify live)

- [ ] `app/robots.ts` exists (Next.js: this **ignores** `public/robots.txt` when present). Verify it renders per-UA:
  ```
  User-agent: GPTBot, ChatGPT-User, ClaudeBot, Bytespider, CCBot, PerplexityBot, Google-Extended
  Disallow: /
  User-agent: Googlebot, Bingbot
  Allow: /
  User-agent: *
  Disallow: /api/
  Disallow: /search
  Disallow: /*?*sort=
  Disallow: /*?*filter=
  Disallow: /*?*page=
  Disallow: /favorites
  Disallow: /dashboard
  Sitemap: https://<domain>/sitemap.xml
  ```
- [ ] Live check: `curl -s https://<domain>/robots.txt | head -n 40`
- [ ] `app/sitemap.ts` `lastModified` is real entity `updatedAt` (not `new Date()` now), lists canonical URLs only, no `?page`/`?sort` variants.
- [ ] Live check: `curl -s https://<domain>/sitemap.xml | head -n 60`

---

## 3. Neon — DB Pool & Recovery (~5 min)

> If you use Neon, Serverless Functions open **one connection per invocation** — a `no-store` crawl storm exhausts `max_connections` in seconds. Pooling is not optional.

- [ ] **Pooled connection string:** In Vercel env vars + locally, `DATABASE_URL` must be the **pooled (PgBouncer)** URL (`...-pooler.`), not the direct `postgres://` URL. Verify: `echo $DATABASE_URL | grep pooler`.
- [ ] **Compute → Auto-suspend** tuned (e.g., 5 min) — not `0` (never) for cost, not too aggressive for latency.
- [ ] **Max connections** noted vs Vercel concurrency (e.g., Neon free 100 conns vs Vercel 100 concurrent fn → tight). Load test with `no-store` off (post-ISR) should stay well under limit.
- [ ] **PITR / Branching:** Project → Branches → verify Point-In-Time Recovery window (e.g., 7 days) and that a branch-from-restore has been tested once.
- [ ] **Billing alerts** ON in Neon dashboard.

---

## 4. Supabase — If Used Instead of Neon

- [ ] Database → **Connection pooling (PgBouncer)** ON, `DATABASE_URL` uses pooler port `6543`, `directUrl` on `5432` only for migrations if using Prisma.
- [ ] **Pool size** vs Vercel concurrency modelled (Supabase pool 15-20 vs 100 fn concurrency → queue).
- [ ] Auth → Rate limits reviewed (email send caps).
- [ ] **RLS** is ON for every table (Security review owns policy correctness — this checklist just confirms toggle, not logic).
- [ ] Billing alerts ON.

---

## 5. Post-Setup Verification (5 min)

```bash
# should list AI bots as Disallow
curl -s https://<domain>/robots.txt

# should be canonical only, no ?filter
curl -s https://<domain>/sitemap.xml | grep -c "filter\|sort\|page="  # expect 0

# burst test: 20 rapid searches from one IP -> at least one 429
for i in $(seq 1 20); do curl -s -o /dev/null -w "%{http_code}\n" "https://<domain>/api/trpc/search.search?batch=1&input=%7B%7D"; done | sort | uniq -c

# canary log check: Vercel Logs → filter "GPTBot" — should show Challenge or 429, not 200 with DB query
```

Record date + who verified in `docs/runbooks/vercel-neon-manual-setup.md` footer.

---

## When to Re-Run

- After any Vercel plan/region change, after `app/robots.ts` / `middleware.ts` / `revalidate` edits, before any marketing push, and monthly as part of Production Readiness review.
