# Abuse Red-Team Playbook — 6 Scripted Scenarios

> Purpose: prove bot/spam/cost controls actually work before launch. Each scenario is `curl` + `k6` runnable now, and codifiable as `vitest` integration tests later. Referenced by `ai-review/reviews/150-testing-analysis` (missing coverage = High) and `20-security` (missing control = Critical/High).

Pre-req: deploy to preview or production with real `robots.txt`, `revalidate`, rate limits, and Vercel Firewall toggles from `vercel-neon-manual-setup.md` ON.

---

## Scenario 1 — AI Crawler Respects robots.txt (Passive)

**What it proves:** GPTBot/ClaudeBot will not crawl your training data.

```bash
curl -s https://<domain>/robots.txt
# expect:
# User-agent: GPTBot
# Disallow: /
# User-agent: ChatGPT-User
# Disallow: /
# User-agent: ClaudeBot
# Disallow: /
# (plus Bytespider, CCBot, PerplexityBot, Google-Extended)

# negative check: Googlebot still allowed on canonical pages
curl -s https://<domain>/robots.txt | grep -A2 "Googlebot"
# expect Allow: /  (not Disallow)
```

- **Pass:** all 7 AI UAs show `Disallow: /`, sitemap directive present, Googlebot/Bingbot allowed.
- **Fail (Critical):** single `User-agent: *` only, or `app/robots.ts` missing so `public/robots.txt` is stale — crawl trap remains open.

Codify: `tests/robots.test.ts` → fetch `/robots.txt` → regex per-UA.

---

## Scenario 2 — Infinite URL Space Is Collapsed (Crawl Trap)

**What it proves:** `/whats-on?filter=&sort=&page=` permutations don't each cost a DB render.

```bash
# sitemap must not list permutations
curl -s https://<domain>/sitemap.xml | grep -E "filter|sort|page=" && echo "FAIL: sitemap leaks permutations" || echo "PASS"

# filtered page must canonicalise to clean base or be noindex
curl -s "https://<domain>/whats-on?category=music&sort=price&page=2" | grep -i 'rel="canonical"'
# expect: <link rel="canonical" href="https://<domain>/whats-on" />
# (or href with only meaningful canonical params, not sort/page)
```

- **Pass:** sitemap clean, canonical points to base URL, `noindex` on filtered variant.
- **Fail (High):** each `?sort`/`?filter` permutation canonicalises to itself → engines crawl N! duplicates.

---

## Scenario 3 — Spoofed UA Still Blocked/Challenged (Edge)

```bash
# real GPTBot UA
curl -s -o /dev/null -w "%{http_code}\n" -A "GPTBot/1.0" https://<domain>/events/some-slug
# expect: 200 if legitimate allow, OR Challenge page (Vercel Firewall) — but never 200 with full DB + no rate limit

# spoofed UA (scraper faking GPTBot to bypass naive rules)
curl -s -o /dev/null -w "%{http_code}\n" -A "Mozilla/5.0 (compatible; GPTBot/1.0)" https://<domain>/events/some-slug
# expect: same as above — middleware Firewall blocks UA regardless of spoof; if only robots.txt, spoofed passes and hits DB = fail

# anon burst on expensive search (the bill driver)
for i in $(seq 1 30); do
  curl -s -A "GPTBot/1.0" "https://<domain>/api/trpc/search.search?batch=1&input=%7B%220%22%3A%7B%22q%22%3A%22a%22%7D%7D" -w "%{http_code}\n" -o /dev/null
done | sort | uniq -c
# expect: at least one 429 after window (e.g., 20/10s sliding window)
```

- **Pass:** spoofed UA still challenged/429'd or CDN-cached without DB; search burst returns `429` + `Retry-After`.
- **Fail (Critical):** spoofed `GPTBot` gets `200` with full data, no limit — scraper just changes UA to bypass everything.

---

## Scenario 4 — Expensive Read Is Rate-Limited + Cached (The P0)

This is your real money. Pick the most expensive public read: `search.search` (PostGIS) or `venue.search`.

```bash
# 60 rapid searches from one IP — should throttle
SEQ=$(seq 1 60); for i in $SEQ; do
  curl -s "https://<domain>/api/trpc/search.search?batch=1&input=%7B%220%22%3A%7B%22q%22%3A%22london+music%22%7D%7D" -w " %{http_code}\n" -o /dev/null
done | tail -n 20

# check cache header on detail page (needs ISR)
curl -s -I https://<domain>/events/some-slug | grep -i "cache-control\|x-vercel-cache"
# expect: cache-control: public, s-maxage=300, stale-while-revalidate=86400
#       + x-vercel-cache: HIT (on second request)
# fail: cache-control: no-store  OR  x-vercel-cache: MISS every time
```

`k6` variant (`load/crawl-burst.js`):

```js
import http from 'k6/http';
import { check, sleep } from 'k6';
export const options = { vus: 50, duration: '30s' };
export default function () {
  const r = http.get('https://<domain>/api/trpc/search.search?batch=1&input=%7B%220%22%3A%7B%22q%22%3A%22e%22%7D%7D', { headers: { 'User-Agent': 'GPTBot/1.0' } });
  check(r, { '429 when abusive': (x) => x.status === 200 || x.status === 429 });
  sleep(0.2);
}
```

- **Pass:** burst → `429` after threshold, detail page → `HIT` after first render, `100k` hits would be ~`300` DB queries (one per `revalidate` window), not `100k`.
- **Fail (Critical):** `no-store` on all tRPC + no `revalidate` on pages → `100k` hits = `100k` PostGIS queries = bill + DB queue + possible Neon `max_connections` exhaustion.

---

## Scenario 5 — Spam / Fake Account

```bash
# registration spam: 10 rapid registrations from one IP
for i in $(seq 1 10); do
  curl -s -X POST https://<domain>/api/trpc/auth.register \
    -H "content-type: application/json" \
    -d "{\"email\":\"spam+$i@test.invalid\",\"password\":\"Test1234!\"}" \
    -w " %{http_code}\n" -o /dev/null
done | sort | uniq -c
# expect: 429 after N (e.g., 5/min) OR ALTCHA challenge required — not 10x 200

# ALTCHA bypass attempt: POST without ALTCHA token
curl -s -X POST https://<domain>/api/contact -d '{"message":"spam"}' -H "content-type: application/json" -w " %{http_code}\n"
# expect: 400/403 challenge required, not 200
```

- **Pass:** burst `429`, ALTCHA verified server-side (client-only = fail).
- **Fail (High):** unlimited registrations/emails — used for mail-bombing and cost.

---

## Scenario 6 — Engagement Inflation

If `lib/engagement.ts` or equivalent counts views/impressions:

```bash
# 20 views from same IP+UA on same event — should dedupe to ~1
for i in $(seq 1 20); do curl -s "https://<domain>/api/trpc/event.trackView?input=%7B%22slug%22%3A%22some-slug%22%7D"; done
# then check DB: SELECT count(*) FROM engagement WHERE ip='...' — expect 1-2, not 20

# same with crawler UA — should be ignored entirely
curl -s -A "GPTBot/1.0" "https://<domain>/api/trpc/event.trackView?input=%7B%22slug%22%3A%22some-slug%22%7D"
# check DB count unchanged if crawler UAs are filtered
```

- **Pass:** IP+UA dedup + crawler UA ignored.
- **Fail (Medium):** bots running JS inflate engagement → ranking/title gaming.

---

## How to Run & Codify

1. Manual once before live: copy-paste `curl` blocks against preview deploy.
2. Codify as CI (recommended):
   - `tests/robots.test.ts` (Scenario 1+2) — `fetch('/robots.txt')` assertions.
   - `tests/rate-limit.test.ts` (Scenario 3-5) — `vitest` integration hitting real or seeded DB with `x-forwarded-for` mock, asserting `TRPCError { code: 'TOO_MANY_REQUESTS' }`.
   - `load/crawl-burst.js` (Scenario 4) — run `k6 run load/crawl-burst.js` in nightly CI, gate on `http_req_failed < 5%` and `checks.passes > 95%`.
3. Log evidence: Vercel → Logs → filter `User-Agent: GPTBot` and `status:429` — screenshot for the `docs/ai-review/reports/` Phase 1 docs.
