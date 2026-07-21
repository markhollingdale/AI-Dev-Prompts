# Architectural Review & Improvement Opportunities

> This document captures findings from a comprehensive codebase review — items that could be improved, decisions that should be reconsidered, and patterns to carry forward. Use this when creating new T3 Stack templates.

---

## Decision Quality Rating

Each architectural decision is rated 1-10 based on its current state:

| Area | Rating | Key Issue |
|------|--------|-----------|
| Config-driven sections | 9/10 | Clean, flexible, builder-ready |
| Multi-tenancy resolution | 8/10 | Async/sync duality is confusing |
| Theming (CSS vars) | 9/10 | Elegant, performant, extensible |
| Route groups | 8/10 | Builder should document the pattern |
| tRPC API layer | 9/10 | Excellent middleware chain |
| Auth (BetterAuth) | 7/10 | No rate limiting on endpoints |
| Database (Drizzle) | 8/10 | Missing pagination everywhere |
| Security headers | **0/10** | None configured |
| Rate limiting | **2/10** | In-memory only, not distributed |
| Error handling (Sentry) | 9/10 | Consistent, well-structured |
| Event handling | 8/10 | Well-patterned, consistent usage |
| Testing coverage | **2/10** | Only 3 test files exist |
| File organization | 7/10 | `globals.css` is 1674 lines |
| Provider architecture | 5/10 | Over-engineered, fragile sync patterns |

---

## What to Keep (Patterns Worth Reusing)

### 1. Config-Driven Architecture

Every page section is driven by JSON config. Sections are reusable components that receive their data via `sectionConfig` props. The `SectionShell` wrapper handles layout, spacing, animations, and backgrounds uniformly.

**Reusable pattern**: `SectionShell` + `SectionIntro` + per-section components. This makes adding new sections trivial — just add the component, register it in the builder, and define the config schema.

### 2. CSS Custom Property Theming

The `data-theme` + `data-palette` + `data-font-pairing` system on `<html>` is:
- Zero runtime JS overhead (pure CSS)
- Server-side rendered
- Easy to extend (just add a new CSS `[data-theme="..."]` block)
- Custom palette injection via `<style>` tag

**Reusable pattern**: Theme definition files, palette definition files, font-pairing definitions — all independently versionable and swappable.

### 3. tRPC Middleware Chain

The middleware chain in `server/trpc.ts` (`sentryMiddleware` → `protectedProcedure` → `orgProcedure` → `requireRole` / `requirePermission`) provides composable, declarative access control.

**Reusable pattern**: Copy the entire `server/trpc.ts` setup — the context, middleware, and procedure factories are generic.

### 4. Graceful Degradation

- Auth returns 503 if no DB
- Config falls back through 3 tiers
- Email gracefully handles missing API keys
- Storage gracefully skips RLS check if DB unavailable

**Reusable pattern**: Every integration point should check if its dependency is available and degrade gracefully.

### 5. Dual Email Provider

No vendor lock-in. Automatic provider selection based on which API key is set. Both make direct HTTP calls — no SDK dependency.

---

## What to Fix (Issues to Address)

### Critical Priority

#### 1. Security Headers (HIGH)

**Problem**: No CSP, HSTS, X-Frame-Options, X-Content-Type-Options, Referrer-Policy, Permissions-Policy anywhere.

**Fix**: Create `middleware.ts`:

```ts
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(request: NextRequest) {
  const response = NextResponse.next();

  response.headers.set("X-Frame-Options", "DENY");
  response.headers.set("X-Content-Type-Options", "nosniff");
  response.headers.set("Referrer-Policy", "strict-origin-when-cross-origin");
  response.headers.set(
    "Content-Security-Policy",
    "default-src 'self'; script-src 'self' 'unsafe-eval' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; img-src 'self' data: blob: https:; font-src 'self' data:; connect-src 'self' https:; frame-src 'self' https://www.google.com;"
  );
  response.headers.set("Permissions-Policy", "camera=(), microphone=(), geolocation=()");
  response.headers.set("X-DNS-Prefetch-Control", "on");
  response.headers.set("Strict-Transport-Security", "max-age=63072000; includeSubDomains; preload");

  return response;
}

export const config = {
  matcher: ["/((?!api|_next/static|_next/image|favicon.ico|monitoring).*)"],
};
```

#### 2. Distributed Rate Limiting (HIGH)

**Problem**: In-memory `Map` in `contact.ts` doesn't work across Vercel serverless instances.

**Fix options**:
- **Upstash Redis**: `@upstash/ratelimit` + `@upstash/redis`
- **Database-backed**: Drizzle-based rate limit table
- **Vercel KV**: Built-in Redis-compatible store

**Recommended**: Upstash Ratelimit — minimal code change:

```ts
import { Ratelimit } from "@upstash/ratelimit";
import { Redis } from "@upstash/redis";

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(5, "1 h"),
  analytics: true,
});
```

#### 3. Sentry DSN Hardcoding (HIGH)

**Problem**: `sentry.server.config.ts` and `sentry.edge.config.ts` have hardcoded DSNs instead of using `NEXT_PUBLIC_SENTRY_DSN` like the client config.

**Fix**: Read from env var:

```ts
Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN || "https://...",
  // ...
});
```

#### 4. Next Config Hardcoding (HIGH)

**Problem**: `next.config.ts` has hardcoded `org: "aceware"` and `project: "acewebcraft"`.

**Fix**: Read from env vars:

```ts
const sentryOrg = process.env.SENTRY_ORG || "aceware";
const sentryProject = process.env.SENTRY_PROJECT || "acewebcraft";
```

### High Priority

#### 5. `globals.css` Monolith

**Problem**: 1674 lines in one file. Theme CSS (~800 lines), palette CSS (~500 lines), animations, base styles, layout styles all mixed.

**Fix**: Split into:
- `app/base.css` — :root variables, dark mode, base styles
- `app/themes.css` — All `[data-theme="..."]` blocks (the largest chunk)
- `app/palettes.css` — All `[data-palette="..."]` blocks
- `app/animations.css` — Keyframe animations, scroll-triggered system
- `app/builder.css` — Builder-specific styles
- `app/globals.css` — Just `@import` directives

```css
@import "tailwindcss";
@import "tw-animate-css";
@import "shadcn/tailwind.css";
@import "./base.css";
@import "./themes.css";
@import "./palettes.css";
@import "./animations.css";
@import "./builder.css";
```

#### 6. Provider Architecture Fragility

**Problem**: `ConfigProvider` and `ThemeProvider` use complex patterns:
- `useEffect` + `useRef` to detect initial config changes
- tRPC queries fetch data, then `useEffect` copies to local state
- State mutations via `useState` + tRPC mutations in parallel

**Fix**: Consider:
- Using `useSyncExternalStore` for tRPC data reactivity
- Moving all CRUD logic to custom hooks (`useSiteConfigs`, `usePalettes`, etc.)
- Making `ThemeProvider` a consumer of `ConfigProvider` for theme/palette keys

#### 7. No Pagination

**Problem**: tRPC queries for leads, sites, users, media return all records.

**Fix**: Add cursor-based pagination to all list procedures:

```ts
list: publicProcedure
  .input(z.object({
    cursor: z.string().optional(),
    limit: z.number().min(1).max(100).default(20),
  }))
  .query(async ({ ctx, input }) => {
    const items = await ctx.db.query.site.findMany({
      limit: input.limit + 1,
      offset: input.cursor ? parseInt(input.cursor) : 0,
    });
    const nextCursor = items.length > input.limit ? String(input.limit + (parseInt(input.cursor || "0"))) : undefined;
    return { items: items.slice(0, input.limit), nextCursor };
  });
```

#### 8. No Loading States

**Problem**: No `loading.tsx` files anywhere. Users see blank pages or layout shifts during navigation.

**Fix**: Add `app/(site)/loading.tsx`, `app/(platform)/loading.tsx`, etc. with skeleton components.

#### 9. TypeScript Config Spread

**Problem**: `include: ["**/*.ts"]` picks up everything. Have to manually exclude `db-utils.ts`, `backups`, `sql`, `scripts`.

**Fix**: Scope the include to actual app files:

```json
"include": [
  "next-env.d.ts",
  "app/**/*.ts",
  "app/**/*.tsx",
  "components/**/*.ts",
  "components/**/*.tsx",
  "config/**/*.ts",
  "config/**/*.tsx",
  "lib/**/*.ts",
  "lib/**/*.tsx",
  "server/**/*.ts",
  "themes/**/*.ts",
  "palettes/**/*.ts",
  ".next/types/**/*.ts",
  "**/*.mts"
]
```

Or keep the broad include but add more excludes.

### Medium Priority

#### 10. Duplicate SiteConfig Fields

**Problem**: `privacyPolicyEnabled`, `termsEnabled`, `analyticsEnabled`, `companyRegistrationNumber`, `vatNumber` exist at both top level and in `legal: LegalConfig`.

**Fix**: Deprecate top-level fields. `lib/legal.ts` already handles the fallback. Eventually remove the top-level fields from `SiteConfig`.

#### 11. No Cookie Consent Gating for Analytics

**Problem**: `UmamiAnalytics` component renders based on `isAnalyticsEnabled()` config flag, but doesn't check if user has consented to analytics cookies.

**Fix**: Pass consent state from `CookieBanner` to `UmamiAnalytics`. Create a consent context or use local storage.

#### 12. ALTCHA Dev Key Warning

**Problem**: Falls back to a hardcoded dev key with no console warning.

**Fix**: Add warning in dev:

```ts
if (process.env.NODE_ENV !== "production" && !process.env.ALTCHA_HMAC_KEY) {
  console.warn("[ALTCHA] Using dev HMAC key — set ALTCHA_HMAC_KEY in production");
}
```

#### 13. Hardcoded `max-w-2xl` in SectionIntro

**Problem**: All section intros are capped at `max-w-2xl`. This should respect the section's container style override.

**Fix**: Make the max-width configurable via `SectionStyleOverrides.container`.

#### 14. `example` Table in Schema

**Problem**: An `exampleTable` exists in the Drizzle schema. Appears to be a leftover.

**Fix**: Either remove it (with a migration) or document it as a reference pattern.

#### 15. Image Optimization

**Problem**: Gallery and section images are plain `<img>` tags with URL sources. No Next.js Image optimization, no blur placeholders, no responsive srcsets.

**Fix**: Use `next/image` with remote pattern configuration in `next.config.ts` for external image URLs.

#### 16. No Auth Rate Limiting

**Problem**: BetterAuth endpoints (`/api/auth/[...all]`) have no rate limiting. Password reset and login are vulnerable.

**Fix**: Add rate limiting middleware to the auth handler or use BetterAuth's rate limiting plugin if available.

#### 17. Email Template Maintainability

**Problem**: Auth email templates (`lib/auth-email.ts`) use template literals with inline CSS. Hard to maintain and customize.

**Fix**: Consider:
- MJML for responsive email templates
- A simple React-to-email renderer like `@react-email/components`
- Separate template files per email type

#### 18. Server Config Hardcoded `sendDefaultPii: true`

**Problem**: Both `sentry.server.config.ts` and `sentry.edge.config.ts` have `sendDefaultPii: true` and `tracesSampleRate: 1`. This sends PII and 100% traces in production.

**Fix**: Use env vars to control these:

```ts
Sentry.init({
  tracesSampleRate: process.env.NODE_ENV === "production" ? 0.1 : 1.0,
  sendDefaultPii: process.env.NODE_ENV !== "production",
});
```

#### 19. `db-utils.ts` Not in package.json Dependencies

**Problem**: `db-utils.ts` imports `dotenv` and `dotenv-expand` which are not in `package.json`. They might be globally installed or transitive dependencies.

**Fix**: Add them as devDependencies:

```json
"devDependencies": {
  "dotenv": "^16.4.7",
  "dotenv-expand": "^11.0.7"
}
```

#### 20. No Config Caching

**Problem**: `getSiteConfigByHostnameAsync` queries the database on every request. No memoization or caching layer.

**Fix**: Add caching. Options:
```ts
import { unstable_cache } from "next/cache";

const getCachedConfig = unstable_cache(
  async (hostname) => resolveFromDb(hostname),
  ["site-config"],
  { revalidate: 60, tags: ["site-config"] }
);
```

Or use Vercel KV / Upstash Redis for cross-region caching.

---

## Consistent Patterns Noted

1. **Sentinel values**: `isDbConfigured`, `NEXT_PUBLIC_DEMO_MODE` — consistent pattern for gating features
2. **Sentry in catch blocks**: Every `catch` block includes a `Sentry.captureException()` call with `tags` and `extra`
3. **`cn()` utility**: Used everywhere for conditional classnames — never raw `clsx` or template literals
4. **Exported functions over classes**: Pure functions exported from modules, no class-based utilities
5. **UUID primary keys**: All DB tables use UUID with `defaultRandom()`
6. **Timestamps**: `createdAt` with `defaultNow()`, `updatedAt` with `$onUpdate()`
7. **Optional dependencies**: Every external service (DB, auth, email, analytics) is optional and degrades gracefully
8. **Zod for validation**: Used consistently in tRPC inputs, env vars, and form patterns
9. **`use client` directive**: Section components are client components due to `useConfig()` usage
10. **`ReactNode` typed children**: Consistent pattern in layout components

---

## Migration Guide: From This Project to a New T3 App

If creating a new T3 Stack template from this codebase:

### Take (Reusable Modules)
1. `lib/env.ts` — t3-env setup (adapt validation schemas)
2. `server/trpc.ts` — tRPC initialization + middleware chain
3. `lib/utils.ts` — `cn()` utility
4. `lib/db/index.ts` — Drizzle + postgres.js connection (adapt schema)
5. `lib/storage.ts` — Supabase file storage pattern
6. `lib/auth-email.ts` — Dual email provider pattern
7. `lib/legal.ts` — Legal compliance helpers
8. `lib/altcha.ts` — ALTCHA challenge/verification
9. `server/routers/contact.ts` — Contact form pattern (rate limiting, ALTCHA, email, lead persist)
10. `components/providers/TRPCProvider.tsx` — tRPC client wrapper

### Skip (Project-Specific)
1. All config files/providers/themes/palettes — replace with app-specific schema
2. Section components — app-specific
3. Builder — app-specific
4. Layout components — app-specific
5. `globals.css` — app-specific styling
6. All section config types — replace with domain model
7. DB schema tables (site, lead, etc.) — app-specific
8. tRPC routers (site, lead, admin, etc.) — app-specific
9. Auth/permissions setup — adapt to app's auth model
10. SEO/analytics code — app-specific

### Fix (Don't Repeat Mistakes)
1. Add `middleware.ts` with security headers from day one
2. Use distributed rate limiting (Upstash) from the start
3. Keep Sentry DSNs in env vars
4. Split `globals.css` into multiple files
5. Add `loading.tsx` files for each route group
6. Add pagination to all list queries
7. Scope TypeScript includes to app directories
8. Add tests alongside new features
9. Never hardcode org/project names in config files
