# Architecture Document

> This document describes the complete architecture of the project for use as a reference when creating new T3 Stack applications. It covers structure, key libraries, patterns, reusable code, security considerations, environment variables, and integration details for external services.

---

## Table of Contents

1. [Technology Stack](#1-technology-stack)
2. [Project Structure](#2-project-structure)
3. [Route Groups Architecture](#3-route-groups-architecture)
4. [Configuration & Multi-Tenancy](#4-configuration--multi-tenancy)
5. [Theming System](#5-theming-system)
6. [Component Architecture](#6-component-architecture)
7. [API Layer (tRPC)](#7-api-layer-trpc)
8. [Authentication (BetterAuth)](#8-authentication-betterauth)
9. [Database (Drizzle ORM + Postgres)](#9-database-drizzle-orm--postgres)
10. [Security](#10-security)
11. [Email Integration](#11-email-integration)
12. [File Storage](#12-file-storage)
13. [Analytics](#13-analytics)
14. [SEO & Structured Data](#14-seo--structured-data)
15. [Legal Compliance](#15-legal-compliance)
16. [Builder (Visual Site Editor)](#16-builder-visual-site-editor)
17. [Error Handling & Monitoring (Sentry)](#17-error-handling--monitoring-sentry)
18. [Environment Variables](#18-environment-variables)
19. [Design Tokens System](#19-design-tokens-system)
20. [Animation System](#20-animation-system)
21. [Utility Libraries](#21-utility-libraries)
22. [Architectural Decisions & Improvement Opportunities](#22-architectural-decisions--improvement-opportunities)
23. [External Service References](#23-external-service-references)

---

## 1. Technology Stack

| Category | Technology | Version | Purpose |
|----------|-----------|---------|---------|
| Framework | Next.js | 16.2.4 | React framework with App Router |
| UI Library | React | 19.2.4 | Component library |
| Language | TypeScript | 5.x | Type safety |
| Styling | Tailwind CSS | 4.2.4 | Utility-first CSS |
| UI Components | shadcn/ui | 4.6.0 | Accessible React primitives |
| UI Primitives | base-ui/react | 1.4.1 | Low-level UI primitives |
| Icons | Lucide React | 1.14.0 | Icon library |
| API | tRPC | 11.17.0 | End-to-end typesafe APIs |
| Validation | Zod | 4.4.3 | Runtime validation |
| Serialization | superjson | 2.2.6 | Type-safe serialization |
| State Mgmt | TanStack React Query | 5.100.10 | Server state via tRPC |
| ORM | Drizzle ORM | 0.45.2 | Database access |
| Database Driver | postgres (via postgres.js) | 3.4.9 | PostgreSQL driver |
| Auth | BetterAuth | 1.6.14 | Authentication + RBAC |
| Email (Resend) | resend | 6.12.3 | Transactional email |
| Email (Brevo) | (direct API) | — | Alternative email |
| Spam Protection | ALTCHA | 3.0.8 | CAPTCHA-free PoW |
| Monitoring | Sentry | 10.x | Error tracking/performance |
| Theming | next-themes | 0.4.6 | Dark/light mode |
| CSS Animations | tw-animate-css | 1.4.0 | Animation utilities |
| Notifications | sonner | 2.0.7 | Toast notifications |
| Classnames | clsx + tailwind-merge | — | `cn()` utility |
| Variants | class-variance-authority | 0.7.1 | Component variants |
| Color | culori | 4.0.2 | Color manipulation |
| Env Validation | @t3-oss/env-nextjs | 0.13.11 | Runtime env validation |
| Database (external) | Supabase | — | Postgres + Storage |

**Key architectural decisions:**

- **No `src/` directory**: All code lives at the project root (Next.js default layout)
- **No middleware.ts**: Security headers and CSP are not configured
- **5 route groups** for code splitting and layout isolation
- **Config-driven sections**: Every page section is defined in JSON config
- **Dual email provider**: Resend or Brevo based on env vars — no hard dependency
- **Drizzle ORM + Postgres.js**: Minimal driver, no Prisma-style client overhead

---

## 2. Project Structure

```
├── app/                          # Next.js App Router pages & API routes
│   ├── (site)/                   # Public-facing website route group
│   ├── (auth)/                   # Authentication pages (signin, signup, etc.)
│   ├── (platform)/               # Client portal (dashboard, leads, settings)
│   ├── (admin)/                  # Platform admin (users, sites)
│   ├── builder/                  # Visual site editor
│   ├── api/                      # API routes
│   │   ├── altcha/               # ALTCHA challenge endpoint
│   │   ├── auth/                 # BetterAuth handler
│   │   ├── trpc/                 # tRPC handler
│   │   └── sentry-example-api/   # Sentry test endpoint
│   ├── actions/                  # Server Actions (auth, sign-out)
│   ├── layout.tsx                # Root layout (static, minimal)
│   ├── not-found.tsx             # 404 page
│   ├── error.tsx                 # Error boundary
│   ├── global-error.tsx          # Global error boundary
│   ├── robots.ts                 # Dynamic robots.txt
│   ├── sitemap.ts                # Dynamic sitemap.xml
│   └── globals.css               # Global styles + theme/palette CSS
├── components/                   # React components
│   ├── analytics/                # Analytics integrations (Umami)
│   ├── builder/                  # Builder UI components
│   │   ├── canvas/               # Drag-and-drop canvas
│   │   ├── inline-editor/        # Inline text editing
│   │   ├── property-panel/       # Section property editor
│   │   ├── providers/            # Builder context providers
│   │   ├── registry/             # Section component registry
│   │   ├── sidebar/              # Builder sidebar
│   │   └── toolbar/              # Builder toolbar
│   ├── layout/                   # Layout components (Navbar, Footer, Logo, etc.)
│   ├── legal/                    # Legal compliance (CookieBanner, LegalPage)
│   ├── platform/                 # Platform UI (Sidebar, FAB, RoleGuard, MediaPicker)
│   ├── providers/                # Context providers (Config, Theme, TRPC)
│   ├── sections/                 # Page sections (Hero, Services, Gallery, etc.)
│   ├── seo/                      # SEO components (JsonLd)
│   └── ui/                       # shadcn/ui primitives (button, card, input, etc.)
├── config/                       # Site configuration system
│   ├── defaults/                 # Built-in platform site configs (TypeScript)
│   ├── sites/                    # JSON site configs
│   │   ├── defaults/             # Template business configs (carpenter, plumber, etc.)
│   │   └── custom-*.json         # User-created site configs
│   ├── types.ts                  # SiteConfig TypeScript types
│   ├── index.ts                  # Client-side config utilities
│   └── server.ts                 # Server-side config resolution by hostname
├── lib/                          # Shared utilities & core logic
│   ├── builder/                  # Builder utilities (section-props, normalization)
│   ├── db/                       # Database (schema, migrations, connection)
│   ├── seo/                      # JSON-LD generators
│   ├── supabase/                 # Supabase client (server + client)
│   ├── tokens/                   # Design token system
│   ├── themes/                   # Template presets
│   ├── altcha.ts                 # ALTCHA challenge/verification
│   ├── auth.ts                   # BetterAuth server config
│   ├── auth-client.ts            # BetterAuth client config
│   ├── auth-server.ts            # Server-side session helper
│   ├── auth-email.ts             # Email sending (Resend/Brevo) + templates
│   ├── auth-redirect.ts          # Auth redirect helpers
│   ├── env.ts                    # Environment variable validation
│   ├── getTheme.ts               # Theme/palette cache utility
│   ├── legal.ts                  # Legal compliance helpers
│   ├── permissions.ts            # RBAC permissions (BetterAuth access control)
│   ├── permissions-data.ts       # Role definitions & permission data
│   ├── portal-nav.ts             # Portal navigation structure
│   ├── storage.ts                # Supabase file storage
│   └── utils.ts                  # cn() utility
├── server/                       # tRPC server
│   ├── routers/                  # tRPC route handlers
│   │   ├── admin.ts              # Admin procedures
│   │   ├── altcha.ts             # ALTCHA procedures
│   │   ├── contact.ts            # Contact form submission
│   │   ├── dashboard.ts          # Dashboard stats
│   │   ├── font-pairing.ts       # Font pairing CRUD
│   │   ├── lead.ts               # Lead management
│   │   ├── media.ts              # Media asset management
│   │   ├── organization.ts       # Organization settings
│   │   ├── palette.ts            # Palette CRUD
│   │   ├── site.ts               # Site CRUD + publish/rollback
│   │   ├── site-config.ts        # Site config management
│   │   ├── team.ts               # Team management
│   │   └── theme.ts              # Theme CRUD
│   ├── index.ts                  # App router (merges all routers)
│   └── trpc.ts                   # tRPC initialization + middleware
├── themes/                       # Theme definitions
│   ├── default.ts                # Theme type interface
│   ├── index.ts                  # All built-in theme presets (20 themes)
│   └── custom.json               # Custom theme storage
├── palettes/                     # Palette definitions
│   ├── index.ts                  # All built-in palettes (19 palettes)
│   └── custom.json               # Custom palette storage
├── public/images/                # Static images
├── sql/                          # SQL scripts (seeds, policies)
├── docs/                         # Documentation
├── db-utils.ts                   # Database management CLI script
├── instrumentation.ts            # Next.js instrumentation (Sentry init, storage checks)
├── drizzle.config.ts             # Drizzle Kit configuration
├── sentry.client.config.ts       # Sentry client config
├── sentry.server.config.ts       # Sentry server config
├── sentry.edge.config.ts         # Sentry edge config
└── next.config.ts                # Next.js + Sentry config
```

---

## 3. Route Groups Architecture

The project uses **5 route groups** for optimal code splitting and layout isolation. There is no `(main)` route group (despite what older docs say) — the public site layout lives in `(site)`.

### Root Layout (`app/layout.tsx`)

- **Minimal static layout** — only fonts, HTML structure, global CSS
- Sets `data-theme`, `data-palette`, `data-font-pairing` attributes on `<html>`
- Loads 5 Google Fonts (Syne, Playfair Display, JetBrains Mono, Bodoni Moda, Nunito)
- Injects custom palette CSS for `custom-*` palettes
- Purpose: enables static prerendering for error pages, 404, and static content

### `(site)` — Public Website

- Full application shell with all providers (tRPC, Config, Theme)
- Navbar/Header, Footer/FooterSection, StickyCTA, CookieBanner, PortalFAB, UmamiAnalytics
- JSON-LD structured data injection (LocalBusiness, FAQ, Service, Review)
- Dynamic metadata generation per hostname
- Dynamic `robots.ts` and `sitemap.ts` per hostname
- Auth-protected routes redirect to signin

### `(auth)` — Authentication Pages

- Minimal centered card layout
- TRPCProvider + ConfigProvider only
- Routes: signin, signup, forgot-password, reset-password, callback

### `(platform)` — Client Portal

- Auth-protected (redirects to signin if no session)
- Sidebar layout (PlatformSidebar + SidebarInset)
- Routes: dashboard, leads, settings, team

### `(admin)` — Platform Admin

- Auth-protected (redirects to signin if no session)
- Admin sidebar layout (AdminSidebar)
- Routes: admin, admin/sites, admin/users

### `builder` — Visual Site Editor

- Separate layout with TRPCProvider + BuilderLayoutWrapper
- Neutral theme (builder chrome unaffected by user site theme)
- Canvas, sidebar, property panel, toolbar

### Error Pages

- `app/not-found.tsx` — 404 for unconfigured hostnames
- `app/(site)/not-found.tsx` — 404 for configured sites with missing pages
- `app/error.tsx` — App error boundary (captures to Sentry)
- `app/global-error.tsx` — Root error boundary (captures to Sentry)

---

## 4. Configuration & Multi-Tenancy

### Config Resolution

The project supports **hostname-based multi-tenancy** with a 3-tier config resolution:

```
getSiteConfigByHostnameAsync(hostname)
  → 1. Database (site table, by hostname)
  → 2. JSON files (config/sites/*.json)
  → 3. Built-in defaults (config/defaults/ace-webcraft.ts)
```

### Config Sources

1. **Database** (`lib/db/schema.ts` — `site` table): Sites created through the platform. Tried first because DB-backed sites can be published/rolled back. Contains `data` (current working) and `publishedData` (live version).

2. **JSON files** (`config/sites/`): Custom JSON configs for quick deployment without DB. Files named `custom-{timestamp}.json`. Template defaults in `config/sites/defaults/` (carpenter, electrician, plumber, hairdresser).

3. **Built-in** (`config/defaults/ace-webcraft.ts`): The platform's own site config in TypeScript. Used as fallback for localhost/unmatched hostnames.

### Sync Functions

There are two separate concepts for "importing" JSON configs:
- **`syncFromDisk`**: Import a single JSON config into the DB, creating/updating a site record
- **`syncAllFromDisk`**: Import all JSON configs from `config/sites/` into the DB

Both are available via tRPC admin procedures.

### SiteConfig Type (`config/types.ts`)

The `SiteConfig` type defines every aspect of a tenant site:

- `businessName`, `tagline`, `description`, `phone`, `email`, `whatsapp`, `address`, `town`
- `areas` (service areas), `services` (list), `serviceDescriptions` (map)
- `sections` (enabled sections with props, animation configs, background images, side images)
- `features` (whatsapp, booking, gallery booleans)
- `testimonials`, `faq`, `process`, `galleryImages`
- `themeKey`, `paletteKey`, `fontPairingKey`
- `hostnames` (array of hostnames this config responds to)
- `seo` (title, description, canonical, ogImage, noIndex)
- `legal` (privacy, terms, analytics, company reg, VAT)
- `logo`, `businessHours`, `socialLinks`, `priceRange`
- `authNav` (signin/signup/dashboard labels)
- `animations` (per-section animation configs)

### Path Aliases (`tsconfig.json`)

```json
{
  "@/components/*": ["./components/*", "./app/components/*"],
  "@/config/*": ["./config/*"],
  "@/*": ["./*"]
}
```

---

## 5. Theming System

The theming system uses **CSS custom properties** applied via `data-*` attributes on `<html>`. It has 3 independent axes:

### Axes

1. **Theme** (`data-theme`): Visual style (rounded corners, shadows, layouts, background effects)
   - 20 built-in themes across 5 categories (Modern, Artistic, Retro, Interactions, Builder)
   - Themes define: `radius`, `spacing`, `typography` classes, `button` styles
   - Stored in `themes/` directory as TypeScript + JSON

2. **Palette** (`data-palette`): Color scheme (background, foreground, primary, accent, secondary)
   - 19 built-in palettes using OKLCH color space
   - Each palette defines 5 swatches: background, foreground, primary, accent, secondary
   - CSS renders all derived colors (card, muted, border, ring, etc.) via `color-mix()`
   - Custom palettes inject inline `<style>` via JS at runtime
   - Stored in `palettes/` directory as TypeScript + JSON

3. **Font Pairing** (`data-font-pairing`): Typography combination (heading + body + accent fonts)
   - 8 built-in font pairings using 5 Google Fonts
   - Stored in `lib/tokens/font-pairings.ts`

### CSS Architecture

- Base variables declared in `:root` (light) and `.dark` (dark mode)
- Theme-specific CSS lives in `app/globals.css` using attribute selectors:
  ```css
  html[data-theme="minimalist"] { ... }
  [data-theme="bento-grid"] [data-slot="card"] { ... }
  ```
- Palette colors override `:root` variables via `[data-palette="..."]` selectors
- Custom palettes inject `<style>` tags at runtime
- `globals.css` is ~1674 lines — the single largest file in the project

### Theme/Palette/FontPairing CRUD

All three are managed via tRPC procedures with corresponding `custom.json` files:
- `theme`: create, delete (read from file system)
- `palette`: create, delete
- `fontPairing`: create, delete

### ThemeProvider

- React context providing `themeKey`, `paletteKey`, `fontPairingKey` + setters
- Loads custom items from tRPC on mount
- Injects custom palette CSS via `useEffect`
- All state is client-side (no server persistence of live edits)

---

## 6. Component Architecture

### Layout Components (`components/layout/`)

| Component | Purpose |
|-----------|---------|
| `Navbar` | Sticky top navigation with mobile menu, phone button, auth nav |
| `Footer` | Business info, contact, areas, legal links, copyright |
| `Logo` | Config-based logo (image or text fallback) |
| `AuthNav` | Sign in/sign up/dashboard links, conditional on auth state |
| `AuthLogo` | Logo variant for auth pages |
| `StickyCTA` | Fixed bottom CTA bar (phone/email buttons) |

### Section Components (`components/sections/`)

Each section component receives its config via `sectionConfig` prop and renders based on the config:

| Component | Config Type | Purpose |
|-----------|-------------|---------|
| `Header` | section with navItems, ctas | Configurable header (overrides Navbar) |
| `Hero` | section with kicker, ctas, backgroundImages | Hero section with slideshow background |
| `Services` | section with serviceItems, sideImage | Services grid/carousel |
| `Gallery` | section with galleryImages | Image gallery |
| `Testimonials` | section with testimonials | Testimonial carousel |
| `FAQ` | section with faq items | Accordion FAQ |
| `Areas` | section with areas | Service area pills |
| `Process` | section with process steps | Step-by-step process |
| `CTA` | section with ctas | Call-to-action banner |
| `Contact` | section with contact props | Contact form with ALTCHA protection |
| `Map` | section with googleMapsEmbedUrl | Google Maps embed |
| `About` | section with about content | About text section |
| `Stats` | section with stats | Statistics display |
| `LogoCloud` | section with logos | Client logo cloud |
| `AnnouncementBar` | section with announcement text | Top announcement bar |
| `Footer` | section with footer columns | Configurable footer (overrides Footer) |

### Shell Components

- **`SectionShell`**: Consistent padding/container wrapper for all sections. Handles background images, side images, animations, and style overrides.
- **`SectionIntro`**: Standardized section title/eyebrow/description layout.
- **`SectionBackground`**: Background image slideshow with arrow/dot navigation.
- **`SectionWithImage`**: Side-by-side layout (image left or right).
- **`CtaButton`**: Renders CTA items with proper action type handling (call, whatsapp, email, scroll-to, page-link, external-link, form-submit).

### UI Components (`components/ui/`)

All shadcn/ui primitives — 20 components total:

accordion, animated-element, avatar, badge, button, card, dialog, dropdown-menu, input, label, link-button, select, separator, sheet, sidebar, skeleton, sonner, table, textarea, tooltip

### Providers (`components/providers/`)

| Provider | Purpose |
|----------|---------|
| `ConfigProvider` | Site config context — exposes current config, all configs, CRUD methods |
| `ThemeProvider` | Theme/palette/font-pairing context — exposes current selection + CRUD |
| `TRPCProvider` | tRPC React Query client — wraps children with QueryClientProvider |

### Provider Architecture Notes

- `ConfigProvider` is the **most complex provider** — manages config state, loads from tRPC, provides CRUD via mutations
- `ThemeProvider` loads custom items from tRPC `getAll` queries on mount
- Both use `useEffect` to sync fetched data into local state
- Both providers use `useRef` to track previous values for change detection

---

## 7. API Layer (tRPC)

### Architecture

tRPC is used for all API communication. It runs as a Next.js route handler at `app/api/trpc/[trpc]/route.ts`.

### Setup (`server/trpc.ts`)

- Initializes tRPC with superjson transformer
- Creates context from request (session, db, ip)
- Provides middleware chain:
  - **`sentryMiddleware`**: Wraps all procedures with Sentry performance monitoring
  - **`publicProcedure`**: Base procedure with Sentry middleware
  - **`protectedProcedure`**: Requires authenticated session, throws UNAUTHORIZED
  - **`orgProcedure`**: Requires org membership, resolves orgId
  - **`requireRole(...roles)`**: RBAC — restricts to specific roles
  - **`requirePermission(resource, action)`**: Fine-grained permission check

### Routers (13 total)

| Router | Procedures | Auth | Purpose |
|--------|------------|------|---------|
| `contact` | `submit` | Public | Contact form submission |
| `altcha` | `getChallenge` | Public | ALTCHA challenge |
| `theme` | `getAll`, `create`, `delete` | Public | Theme management |
| `palette` | `getAll`, `create`, `delete` | Public | Palette management |
| `fontPairing` | `getAll`, `create`, `delete` | Public | Font pairing mgmt |
| `siteConfig` | `getAll`, `create`, `duplicate`, `delete` | Public | Site config CRUD |
| `site` | `list`, `getById`, `create`, `createForOrg`, `update`, `publish`, `getVersions`, `rollback`, `delete`, `syncFromDisk`, `syncAllFromDisk`, `assignToOrg`, `setAsTemplate`, `importFromDisk` | Protected | Full site lifecycle |
| `lead` | `list`, `getById`, `updateStatus`, `assign`, `addNote`, `delete` | Protected | Lead management |
| `dashboard` | `getStats`, `getActivityFeed` | Protected | Dashboard data |
| `admin` | `getStats`, `listSites`, `listOrganizations`, `listTemplates`, `listUsers`, `checkRlsStatus`, `listUnsyncedFiles` | Protected | Admin dashboard |
| `organization` | `getMembership`, `getSettings`, `updateSettings` | Protected | Org settings |
| `team` | `getMembers`, `getPendingInvitations`, `changeMemberRole`, `updateMemberPermissions` | Protected | Team management |
| `media` | `list`, `getById`, `upload`, `update`, `delete` | Protected | Media assets |

### Context Creation

```ts
createContext = async (req: Request) => ({
  db: isDbConfigured ? db : undefined,  // Drizzle ORM instance
  session,                              // BetterAuth session
  headers: req.headers,
  ip: resolved from x-forwarded-for / cf-connecting-ip / x-real-ip
})
```

---

## 8. Authentication (BetterAuth)

### Configuration (`lib/auth.ts`)

BetterAuth is initialized with:

- **Drizzle adapter** (Postgres via postgres.js)
- **Email/Password** authentication with email verification
- **Google OAuth** (conditional on env vars)
- **Organization plugin** (multi-org support with RBAC)
- **oAuthProxy plugin** (OAuth proxy support)
- Session settings: 7-day expiry, 1-day update age

### Gated Initialization

```ts
const isDbConfigured = db !== undefined;
export const auth = isDbConfigured ? betterAuth({...}) : null;
```

If `DATABASE_URL` is not set, auth returns `null` and the auth API returns 503.

### RBAC Permissions (`lib/permissions.ts`)

BetterAuth's access control system defines 5 roles:

| Role | Permissions |
|------|-------------|
| **owner** | Full access — org CRUD, member mgmt, site publish, settings |
| **admin** | Org update, member mgmt, site publish, settings |
| **manager** | Member mgmt, site edit/publish, team mgmt |
| **editor** | Site edit only, lead read |
| **member** | Lead read only |

Statements (resources): `organization`, `member`, `invitation`, `ac`, `leads`, `site`, `team`, `settings`

The `requirePermission(resource, action)` middleware checks the user's role + optional custom permissions against the requested resource/action combination.

### Auth Routes

- `app/api/auth/[...all]/route.ts` — Catch-all BetterAuth handler
- `app/actions/auth.ts` — Server actions (`checkUserProvider`, `getUserEmailVerificationState`)
- `app/actions/sign-out.ts` — Sign out action
- `lib/auth-server.ts` — `getSession()` helper for server components
- `lib/auth-client.ts` — `authClient` for client-side auth operations

### Auth UI Components

- `components/sections/auth/signin-form.tsx` — Signin form component
- `components/sections/auth/signup-form.tsx` — Signup form component
- `components/sections/auth/forgot-password-form.tsx` — Forgot password form
- `components/sections/auth/reset-password-form.tsx` — Reset password form

---

## 9. Database (Drizzle ORM + Postgres)

### Connection (`lib/db/index.ts`)

```ts
import postgres from "postgres";
import { drizzle } from "drizzle-orm/postgres-js";

const conn = postgres(DATABASE_URL);   // postgres.js client
const db = drizzle(conn, { schema });  // Drizzle ORM wrapper
```

- Uses global caching in development (prevents hot-reload connection leaks)
- Graceful failure: if `DATABASE_URL` is not set, `db` is `undefined`
- Sentry captures connection errors with diagnostic context

### Schema (`lib/db/schema.ts`)

**BetterAuth Core Tables:**

| Table | Purpose |
|-------|---------|
| `user` | Users with email, name, emailVerified |
| `session` | Auth sessions with optional activeOrganizationId |
| `account` | OAuth/provider accounts |
| `verification` | Email verification tokens |

**BetterAuth Organization Plugin Tables:**

| Table | Purpose |
|-------|---------|
| `organization` | Organizations with name, slug, logo, metadata |
| `member` | Org membership with role + customPermissions (JSONB) |
| `invitation` | Org invitations with status tracking |

**Platform Business Tables:**

| Table | Purpose |
|-------|---------|
| `site` | Tenant sites with hostname, data (JSONB), publishedData (JSONB), status (draft/published), isTemplate flag, filePath |
| `site_version` | Version history for publish/rollback |
| `lead` | Contact form submissions with source, status tracking |
| `lead_note` | Internal notes on leads |
| `media_asset` | Uploaded media files with storage path tracking |
| `activity_event` | Audit trail for org activity |

### Migrations

- Drizzle Kit manages migrations in `lib/db/migrations/`
- 5 migrations currently exist
- The `db-utils.ts` CLI provides: backup, restore, migrate, generate migration, seed, reset, enable RLS

### Schema Design Patterns

- All IDs are UUIDs with `defaultRandom()`
- Timestamps use `defaultNow()` + `$onUpdate()` for `updatedAt`
- `site.data` and `site.publishedData` are JSONB — the full `SiteConfig` object is stored as JSON
- `activity_event.metadata` is JSONB for flexible event payloads

---

## 10. Security

### Implemented Security Measures

1. **ALTCHA Proof-of-Work** (`lib/altcha.ts`)
   - Challenge created server-side, verified server-side
   - HMAC-signed with `ALTCHA_HMAC_KEY`
   - Configurable cost (default: 5000)
   - 10-minute TTL on challenges
   - Dev fallback key when `NODE_ENV !== "production"`
   - Rendered as `<altcha-widget>` in Contact form

2. **Honeypot Fields** (Contact form)
   - Hidden fields (`company`, `_honeypot`) that bots fill in
   - Returns fake success on detection (bots can't learn they're blocked)

3. **Rate Limiting** (in-memory, `server/routers/contact.ts`)
   - IP-based rate limiting: 5 requests per hour per IP
   - In-memory `Map<string, { count, resetAt }>` — does NOT work across multiple Vercel serverless instances
   - **Must migrate to Upstash Redis or database-backed solution before production**

4. **Email Validation** (server-side)
   - Length checks (max 254 chars)
   - Disallowed character rejection
   - Domain format validation

5. **HTML Escaping** (client name/phone/email/message)
   - `escapeHtml()` function for safe email content rendering
   - Prevents XSS in email previews

6. **File Upload Validation** (`lib/storage.ts`)
   - MIME type whitelist (jpeg, png, webp, avif, gif, svg)
   - Max file size: 3MB
   - UUID-based storage paths — no path traversal risk

7. **Supabase RLS Policy Checking** (`lib/storage.ts`)
   - `checkStoragePolicies()` runs at server startup
   - Verifies 4 expected policies exist for the `site-images` bucket
   - Sends warning to Sentry if policies are missing

8. **tRPC Auth Middleware Chain**
   - `protectedProcedure`: Rejects unauthenticated requests
   - `orgProcedure`: Verifies org membership
   - `requireRole()`: Role-based access control
   - `requirePermission()`: Fine-grained permission checks

9. **Sentry Error Tracking**
   - All errors captured with context tags
   - `onRequestError` handler for unhandled request errors

10. **Graceful Auth Degradation**
    - Auth returns 503 if database is not configured
    - Auth procedures check `auth !== null` before handling

### Missing Security Measures

| Measure | Status | Priority |
|---------|--------|----------|
| Content Security Policy (CSP) | **Not configured** | HIGH |
| HSTS headers | **Not configured** | HIGH |
| X-Frame-Options | **Not configured** | HIGH |
| X-Content-Type-Options | **Not configured** | HIGH |
| Referrer-Policy | **Not configured** | HIGH |
| Permissions-Policy | **Not configured** | HIGH |
| Security headers middleware | **Not configured** | HIGH |
| Rate limiting (production-grade) | In-memory only, not distributed | HIGH |

---

## 11. Email Integration

### Dual Provider Architecture (`lib/auth-email.ts`)

The email system supports two providers with automatic selection:

```ts
getEmailProvider() → "resend" | "brevo" | null
```

- **Resend** is tried first (if `RESEND_API_KEY` is set)
- **Brevo** is used as fallback (if `BREVO_API_KEY` is set)
- Both use direct HTTP API calls (no SDK wrapper, except Resend in contact form)

### Email Types

| Type | Function | Trigger |
|------|----------|---------|
| Verification | `sendVerificationEmail()` | User signup |
| Password Reset | `sendPasswordResetEmail()` | Password reset request |
| Invitation | `sendInvitationEmail()` | Org invitation |
| Contact Enquiry | Inline in `contact` router | Contact form submission |

### Contact Form Email Flow

1. Form submitted with ALTCHA payload, honeypot check
2. Server verifies ALTCHA, rate limits IP
3. Email sent via configured provider (Resend or Brevo)
4. Lead persisted to DB (outside email transaction — notification failures don't roll back business state)
5. Attachments uploaded to Supabase storage before lead persistence
6. Activity event created

### Email Template

- `baseTemplate()` in `lib/auth-email.ts` provides a styled HTML wrapper
- Templated with: header, content body, footer
- Blue (#2563eb) primary color scheme
- Responsive inline CSS

---

## 12. File Storage

### Supabase Storage (`lib/storage.ts`)

- **Bucket**: `site-images`
- **Allowed types**: jpeg, png, webp, avif, gif, svg+xml
- **Max size**: 3MB
- **Path pattern**: `{orgId or prefix/orgId}/{uuid}.{ext}`
- **Functions**: `uploadFile()`, `deleteFile()`, `listFilesByOrg()`

### Supabase Client (`lib/supabase/`)

| File | Client Type | Key Used |
|------|-------------|----------|
| `server.ts` | `createClient()` | Service role key (server-side only) |
| `client.ts` | `createClient()` | Anon key (browser-safe) |

### Storage RLS

At server startup, `checkStoragePolicies()` runs to verify that 4 RLS policies exist on the `storage.objects` table:
- `site_images_select`, `site_images_insert`, `site_images_update`, `site_images_delete`

Missing policies are reported to Sentry as warnings.

---

## 13. Analytics

### Umami Integration (`components/analytics/UmamiAnalytics.tsx`)

- Self-hosted privacy-focused analytics
- Activated when `NEXT_PUBLIC_UMAMI_WEBSITE_ID` and `NEXT_PUBLIC_UMAMI_TRACKER_URL` are set
- Renders the Umami tracker `<script>` tag
- Gated behind `isAnalyticsEnabled()` from `lib/legal.ts`

### Environment Variables

- `NEXT_PUBLIC_UMAMI_WEBSITE_ID` — Umami site ID
- `NEXT_PUBLIC_UMAMI_TRACKER_URL` — Tracker script URL
- `UMAMI_DATABASE_URL` — Umami's own database (separate from app database)

---

## 14. SEO & Structured Data

### JSON-LD (`lib/seo/json-ld.ts`)

4 generators that create structured data for search engines:

| Generator | Schema Type | When |
|-----------|-------------|------|
| `generateLocalBusinessJsonLd` | `LocalBusiness` | Always |
| `generateFaqPageJsonLd` | `FAQPage` | When FAQ items exist |
| `generateServiceJsonLd` | `Service` + `OfferCatalog` | Always |
| `generateReviewJsonLd` | `LocalBusiness` + `AggregateRating` | When testimonials exist |

These are injected in the `(site)/layout.tsx` via `<JsonLdScript>` component.

### Dynamic SEO Files

- `app/robots.ts` — Dynamic per hostname, respects `seo.noIndex` flag
- `app/sitemap.ts` — Dynamic per hostname, includes sections, privacy/terms pages
- `app/(site)/layout.tsx` — Dynamic `generateMetadata()` per hostname with OpenGraph + Twitter cards

### SeoConfig Type

```ts
interface SEOConfig {
  title?: string;
  description?: string;
  canonical?: string;
  ogImage?: string;
  noIndex?: boolean;
}
```

---

## 15. Legal Compliance

### Components

- **`CookieBanner`** (`components/legal/CookieBanner.tsx`): Cookie consent banner with 3 categories (essential, analytics, marketing). Uses the corresponding `lib/legal.ts` helpers.
- **`LegalPage`** (`components/legal/LegalPage.tsx`): Reusable legal page component with proper HTML content styling.

### Routes

- `/privacy` — Privacy policy page (enabled via `legal.privacyPolicyEnabled`)
- `/terms` — Terms of service page (enabled via `legal.termsEnabled`)

### Helpers (`lib/legal.ts`)

| Function | Purpose |
|----------|---------|
| `isPrivacyPolicyEnabled()` | Checks config for privacy policy toggle |
| `isTermsEnabled()` | Checks config for terms toggle |
| `isAnalyticsEnabled()` | Checks config for analytics toggle |
| `getCompanyRegistrationNumber()` | Returns company reg number from legal config |
| `getVatNumber()` | Returns VAT number from legal config |
| `getBusinessAddress()` | Formats address + town |

### Compliance Features

- Cookie consent with granular categories
- Privacy policy and terms pages per tenant
- Company registration and VAT number display in footer
- Analytics opt-in via cookie consent

---

## 16. Builder (Visual Site Editor)

The builder is a full visual site editor located at `/builder` with its own route group and layout.

### Builder Structure (`components/builder/`)

| Module | Purpose |
|--------|---------|
| `canvas/` | Drag-and-drop section canvas |
| `inline-editor/` | In-place text editing for section content |
| `property-panel/` | Right-side property editor for selected section |
| `providers/` | Builder-specific context (BuilderLayoutWrapper) |
| `registry/` | Section component registry — maps section types to components |
| `sidebar/` | Left-side panel for adding sections |
| `toolbar/` | Top toolbar (save, publish, undo/redo) |

### Builder Theme

The builder uses a neutral "builder" theme (`data-theme="builder"`) that's unaffected by the user's site theme selections. This ensures the builder UI stays readable regardless of the site's design.

### Key Capabilities

- Add/reorder/remove sections
- Edit section content inline or via property panel
- Change theme, palette, font pairing
- Save drafts and publish live versions
- Version history and rollback

---

## 17. Error Handling & Monitoring (Sentry)

### Configuration Files

| File | Runtime | Config |
|------|---------|--------|
| `sentry.client.config.ts` | Browser | DSN from env, 10% traces in prod, Replay + Feedback integrations |
| `sentry.server.config.ts` | Node.js | DSN hardcoded, 100% traces, sendDefaultPii |
| `sentry.edge.config.ts` | Edge | Same as server config |
| `instrumentation.ts` | Server start | Initializes Sentry + checks storage policies |
| `next.config.ts` | Build | Source map upload, tunnelRoute at `/monitoring` |

### Sentry Usage Patterns

Throughout the codebase, Sentry is used consistently:

- **`Sentry.captureException(error, { tags, extra })`** — All catch blocks
- **`Sentry.captureMessage("Description", "warning", { extra })`** — Business logic warnings
- **`Sentry.trpcMiddleware()`** — Automatic tRPC performance monitoring
- **`Sentry.captureRequestError`** — Global error handler
- **`Sentry.replayIntegration()`** — Session replays
- **`Sentry.feedbackIntegration()`** — User feedback widget

### Error Boundaries

- `app/error.tsx` — Captures errors to Sentry with `app-error-boundary` tag
- `app/global-error.tsx` — Captures critical errors with `global-error-boundary` tag

---

## 18. Environment Variables

### Validation (`lib/env.ts`)

Uses `@t3-oss/env-nextjs` (`createEnv`) for runtime validation with Zod schemas.

**Server-side variables:**

| Variable | Required | Purpose |
|----------|----------|---------|
| `NODE_ENV` | No (default: development) | Environment |
| `SENTRY_ORG` | No | Sentry organization slug |
| `SENTRY_PROJECT` | No | Sentry project slug |
| `SENTRY_AUTH_TOKEN` | No | Sentry auth token for source maps |
| `BREVO_API_KEY` | No | Brevo email API key |
| `BREVO_FROM_EMAIL` | No | Brevo sender email |
| `BREVO_FROM_NAME` | No | Brevo sender name |
| `RESEND_API_KEY` | No | Resend email API key |
| `RESEND_FROM_EMAIL` | No | Resend sender email |
| `ALTCHA_HMAC_KEY` | No | ALTCHA HMAC secret |
| `ALTCHA_CHALLENGE_COST` | No (default: 5000) | PoW difficulty |
| `SUPABASE_SERVICE_ROLE_KEY` | No | Supabase admin key |
| `DATABASE_URL` | No | Primary database URL |
| `DATABASE_URL_PROD` | No | Production DB (db-utils) |
| `DATABASE_URL_STAGING` | No | Staging DB (db-utils) |
| `DATABASE_URL_DEV` | No | Dev DB (db-utils) |
| `UMAMI_DATABASE_URL` | No | Umami analytics DB |
| `BETTER_AUTH_SECRET` | **Yes** | Session signing secret |
| `BETTER_AUTH_URL` | **Yes** | Auth base URL |
| `AUTH_GOOGLE_ID` | No | Google OAuth client ID |
| `AUTH_GOOGLE_SECRET` | No | Google OAuth secret |

**Client-side variables (prefixed with `NEXT_PUBLIC_`):**

| Variable | Required | Purpose |
|----------|----------|---------|
| `NEXT_PUBLIC_DEMO_MODE` | No (default: true) | Demo mode toggle |
| `NEXT_PUBLIC_SENTRY_DSN` | No | Sentry client DSN |
| `NEXT_PUBLIC_SUPABASE_URL` | No | Supabase project URL |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | No | Supabase anon key |
| `NEXT_PUBLIC_UMAMI_WEBSITE_ID` | No | Umami site ID |
| `NEXT_PUBLIC_UMAMI_TRACKER_URL` | No | Umami tracker URL |
| `NEXT_PUBLIC_APP_URL` | No | Public app URL |

**Key behaviors:**
- `skipValidation: !!process.env.SKIP_ENV_VALIDATION` — allows bypassing validation
- `emptyStringAsUndefined: true` — empty env vars are treated as unset
- All optional vars are `.optional()` in the Zod schema — the app degrades gracefully
- `BETTER_AUTH_SECRET` and `BETTER_AUTH_URL` are the only required variables

---

## 19. Design Tokens System

The design tokens system (`lib/tokens/`) provides a React hook for consuming theme/palette/font-pairing values.

### Structure

| File | Purpose |
|------|---------|
| `types.ts` | `DesignTokens` interface |
| `resolve-tokens.ts` | Resolves theme, palette, font-pairing into flat token object |
| `use-design-tokens.ts` | React hook — `useDesignTokens()` returns resolved tokens |
| `section-styles.ts` | Section style helpers — spacing, container, background, height |
| `font-pairings.ts` | Font pairing definitions |

### DesignTokens Type

```ts
interface DesignTokens {
  spacing: { section, sectionCompact, sectionSpacious }
  container: { default, narrow, wide, full }
  colors: { muted, primary, accent }
  typography: { hero, heading, subheading, body }
  radius: string
}
```

### Usage Pattern

```tsx
const tokens = useDesignTokens();
// tokens.typography.heading → "text-2xl md:text-3xl font-bold"
// tokens.spacing.section → "py-16 md:py-24"
```

---

## 20. Animation System

### Animation Types

```ts
type AnimationType =
  | "none" | "fade-in" | "fade-in-up" | "fade-in-down"
  | "slide-in-left" | "slide-in-right"
  | "scale-in" | "float";
```

### Config Structure (`SiteAnimations`)

Per-section animation configs for: hero, services, testimonials, faq, contact, cta, gallery, areas, process, map.

Each section can have:
- `intro` (ElementAnimationConfig) — animation for the section intro
- `items` (ElementAnimationConfig) — animation for individual items

Each element config:
```ts
interface ElementAnimationConfig {
  type: AnimationType;
  duration?: number;    // default 0.6s
  delay?: number;       // stagger delay
  staggerDelay?: number; // delay between items
  loop?: boolean;       // infinite loop (for float)
}
```

### Implementation

- CSS keyframe animations defined in `globals.css`
- `AnimatedElement` component applies animation classes via IntersectionObserver
- Classes: `animate-on-scroll`, `is-visible`, `anim-{type}`
- CSS custom properties: `--anim-duration`, `--anim-delay`
- Float animations loop infinitely via `anim-loop` class

---

## 21. Utility Libraries

### `lib/utils.ts` — `cn()` function

```ts
import { clsx, type ClassValue } from "clsx";
import { twMerge } from "tailwind-merge";
export function cn(...inputs: ClassValue[]) { return twMerge(clsx(inputs)); }
```

### `lib/getTheme.ts` — Theme caching

Simple Map-based cache for theme/palette lookups with `setTheme(key)` and `getTheme(key)`.

### `lib/builder/section-props.ts`

Type-safe prop extraction from section configs:
- `getStringProp(props, key, fallback)`
- `getBooleanProp(props, key, fallback)`
- `getNumberProp(props, key, fallback)`

### `lib/builder/sections.ts`

Section normalization utilities with tests (`sections.test.ts`).

### `lib/permissions-data.ts`

Raw role/permission data used by BetterAuth and tRPC middleware. Contains `ALL_PERMISSIONS`, `ROLE_DEFINITIONS`, `getRolePermissions()`, `getMemberPermissions()`, `hasPermission()`, `getAssignableRoles()`, `getGrantablePermissions()`, `isRoleSubset()`.

### Database Utility (`db-utils.ts`)

A CLI script for database management operations:
- Backup (SQL or dump, full/data/schema)
- Restore (with schema wipe + RLS re-enable)
- Migration generation + execution
- Database reset (wipe public schema)
- Seed database
- Enable RLS on all tables
- Connection check

Uses `dotenv` and `dotenv-expand` for environment loading (not in `package.json` — must be installed globally or handled separately).

---

## 22. Architectural Decisions & Improvement Opportunities

### Decisions That Went Well

1. **Config-driven sections**: Every page section is defined by JSON config, not hardcoded. This enables the builder to add/reorder/remove sections without code changes.

2. **Hostname-based multi-tenancy**: Simple and effective. No subdomain routing complexity. Works with any domain setup.

3. **3-tier config resolution** (DB → JSON → built-in): Graceful degradation means the app works even without a database configured.

4. **Dual email provider** (Resend/Brevo): No vendor lock-in. Automatic fallback based on which API key is set.

5. **CSS custom properties for theming**: The `data-theme` + `data-palette` + `data-font-pairing` system is elegant and performant. No runtime CSS-in-JS overhead. Themes are pure CSS.

6. **OKLCH color space**: Future-proof color model. Better gamut than sRGB, perceptual uniformity. All palette definitions use OKLCH.

7. **tRPC for API**: Full type safety end-to-end. No manual API client generation. SuperJSON handles Date/Map/Set serialization.

8. **Graceful auth degradation**: Auth returns 503 if no database — the rest of the app still works. This is excellent for development and demo scenarios.

9. **Route groups for code splitting**: 5 route groups mean auth pages don't load the builder's heavy components, and the public site doesn't load portal/admin components.

### Areas for Improvement

#### Critical

1. **No security headers**: No CSP, HSTS, X-Frame-Options, X-Content-Type-Options, Referrer-Policy, or Permissions-Policy. A `middleware.ts` should be added with Helmet or manual headers. This is a **high-severity** issue for production.

2. **In-memory rate limiting**: The `Map`-based rate limiter in `contact.ts` doesn't scale across Vercel serverless instances. Must migrate to Upstash Redis or a database-backed solution. Consider using Upstash's `@upstash/ratelimit` for simplicity.

3. **No middleware.ts**: Besides security headers, this also means no request logging, no redirect logic, and no path-based rewrites. Consider adding middleware for: security headers, request ID generation, geolocation detection.

#### High

4. **No middleware.ts**: Besides security headers, this also means no request logging, no redirect logic, and no path-based rewrites. Consider adding middleware for: security headers, request ID generation, geolocation detection.

5. **`globals.css` is 1674 lines**: This file contains all theme CSS, palette CSS, layout CSS, animations, and base styles. This should be split into multiple CSS files or CSS modules. At minimum: `base.css`, `themes.css`, `palettes.css`, `animations.css`, `builder.css`. Tailwind's `@import` directive makes this trivial.

6. **Provider complexity**: `ConfigProvider` and `ThemeProvider` both use `useEffect` + `useRef` patterns for syncing server data to local state. This is fragile and can cause stale closures. Consider using `useSyncExternalStore` or a simpler data flow.

7. **Hardcoded Sentry DSN**: `sentry.server.config.ts` has a hardcoded DSN. This should come from `NEXT_PUBLIC_SENTRY_DSN` like the client config does. Currently both server configs have the same hardcoded DSN, which is a deployment concern.

8. **No tests for most code**: Only 3 test files exist (`sections.test.ts`, `template-presets.test.ts`) — both for utility functions. No tests for components, tRPC routers, or page-level behavior.

#### Medium

9. **Multiple config formats**: Configs exist as TypeScript (`config/defaults/ace-webcraft.ts`), JSON (`config/sites/*.json`), and DB (JSONB). This is flexible but confusing. The TypeScript config is the most maintainable (type-checked), JSON is good for non-developer tooling, and DB is good for the platform. Document which to use when.

10. **`config/server.ts` has sync and async versions**: `getSiteConfigByHostname` (sync, file-only) and `getSiteConfigByHostnameAsync` (async, tries DB first). The sync version is used in layouts that can't be async (root layout, auth layout). This is a Next.js App Router quirk. Keep both but document clearly.

11. **`lib/auth-email.ts` uses inline CSS for email templates**: This is fine for email clients, but the template strings are hard to maintain. Consider a simple template builder or MJML integration for production.

12. **ALTCHA HMAC key dev fallback**: Falls back to `"dev-altcha-hmac-key-change-for-production"` in development. This is fine for dev but should have a console warning.

13. **No rate limiting for auth endpoints**: BetterAuth doesn't have built-in rate limiting. Password reset and login endpoints are vulnerable to brute force. Add rate limiting to the auth handler.

14. **`db-utils.ts` is included in the TypeScript project**: Must be excluded in tsconfig (or use `tsx --skip-project`). Current approach of excluding specific files in tsconfig is fine but fragile — consider naming the file with a `.cts` extension or moving to a `scripts/` directory.

15. **No database migration for the `example` table**: The schema has an `exampleTable` that appears to be a leftover from scaffolding. Either remove it or document its purpose.

16. **Duplicate theme/palette/font-pairing state**: Both `ConfigProvider` (via `config.themeKey`, `config.paletteKey`) and `ThemeProvider` independently manage theme/palette state. They can get out of sync. Consider making `ThemeProvider` read from `ConfigProvider` as the source of truth.

17. **No loading states for pages**: No `loading.tsx` files in any route group. Users may see blank pages or layout shifts during navigation.

18. **UmamiAnalytics component doesn't check cookie consent**: The analytics script should be gated behind cookie consent, not just the `isAnalyticsEnabled()` config flag. The `CookieBanner` component doesn't appear to gate this either.

19. **`SectionIntro` has a `max-w-2xl` hardcoded**: Section intros always have max width 2xl. This should respect the `container` style override from `SectionStyleOverrides`.

20. **No pagination for leads, users, sites, media**: tRPC queries return all records. For production with hundreds of leads/sites, this will be slow. Add cursor-based pagination.

21. **`SiteConfig` type has duplicate fields**: `privacyPolicyEnabled`, `termsEnabled`, `analyticsEnabled`, `companyRegistrationNumber`, `vatNumber` exist both at the top level and inside `legal: LegalConfig`. The `lib/legal.ts` helpers handle this with `??` fallbacks, but the type should be cleaned up.

22. **Animation system uses CSS classes + IntersectionObserver**: This works well but makes server-side rendering of animations impossible. For SSRed animations, consider using CSS `@starting-style` or `animation-delay` with server-known viewport positions.

23. **No image optimization for gallery images**: Gallery images are referenced as URLs in config. No lazy loading config, no responsive image sets, no blur placeholder. Next.js `<Image>` component should be considered for production sites.

24. **`next.config.ts` hardcodes Sentry org/project**: `org: "aceware"`, `project: "acewebcraft"` — these should come from env vars.

25. **No Redis or distributed cache**: Config loading from DB happens on every request. For high-traffic sites, this should be cached. Consider using `unstable_cache` from Next.js or Upstash Redis.

---

## 23. External Service References

### Sentry
- **Website**: https://sentry.io
- **Next.js SDK Docs**: https://docs.sentry.io/platforms/javascript/guides/nextjs/
- **Sign up**: https://sentry.io/signup/
- **Setup**: Create project → Copy DSN → Set `NEXT_PUBLIC_SENTRY_DSN`, `SENTRY_ORG`, `SENTRY_PROJECT`, `SENTRY_AUTH_TOKEN`
- **Auth Token**: https://sentry.io/settings/account/api/auth-tokens/ (scopes: org:read, project:read, project:releases, project:write)
- **Config files**: `sentry.client.config.ts`, `sentry.server.config.ts`, `sentry.edge.config.ts`, `instrumentation.ts`

### Supabase
- **Website**: https://supabase.com
- **Docs**: https://supabase.com/docs
- **Sign up**: https://supabase.com/dashboard/sign-up
- **Setup**: Create project → Get from Settings > API: URL, anon key, service role key
- **Database URL format**: `postgresql://postgres.{project-ref}:{password}@{host}:{port}/postgres`
- **Storage**: Bucket `site-images` with RLS policies (see `sql/policies/storage/index.sql`)
- **Pooler**: Use port 6543 for connection pooling, 5432 for direct connection

### BetterAuth
- **Website**: https://www.better-auth.com
- **Docs**: https://www.better-auth.com/docs
- **Setup**:
  1. Set `BETTER_AUTH_SECRET` (generate: `openssl rand -hex 32`)
  2. Set `BETTER_AUTH_URL` (your app URL)
  3. Configure social providers (Google OAuth shown in `.env.example`)
  4. The Drizzle adapter picks up your schema automatically
- **Auth handler**: Route at `app/api/auth/[...all]/route.ts`
- **Client setup**: `lib/auth-client.ts`

### Resend
- **Website**: https://resend.com
- **Docs**: https://resend.com/docs/introduction
- **Setup**: Create account → Verify domain → Create API key → Set `RESEND_API_KEY`, `RESEND_FROM_EMAIL`
- **Unverified sender**: Default `onboarding@resend.dev` works for testing

### Brevo (formerly Sendinblue)
- **Website**: https://www.brevo.com
- **Docs**: https://developers.brevo.com/docs/getting-started
- **Setup**: Create account → SMTP & API → API Keys → Set `BREVO_API_KEY`, `BREVO_FROM_EMAIL`, `BREVO_FROM_NAME`

### ALTCHA
- **Website**: https://altcha.org
- **Docs**: https://altcha.org/docs/
- **Setup**:
  1. Set `ALTCHA_HMAC_KEY` (generate: `openssl rand -hex 32`)
  2. Set `ALTCHA_CHALLENGE_COST` (default: 5000)
  3. Widget auto-renders in contact form at `components/sections/Contact.tsx`
  4. Web component: `<altcha-widget challenge="/api/altcha" name="altcha" type="checkbox" />`
- **How it works**: Client computes a proof-of-work, server verifies the HMAC-signed solution. No CAPTCHA, no user interaction needed.

### Umami
- **Website**: https://umami.is
- **Docs**: https://umami.is/docs
- **Setup**: Self-host on Supabase → Add website → Get tracker script URL + website ID → Set `NEXT_PUBLIC_UMAMI_WEBSITE_ID`, `NEXT_PUBLIC_UMAMI_TRACKER_URL`
- **Note**: Requires self-hosting — no cloud version

### Vercel
- **Website**: https://vercel.com
- **Docs**: https://nextjs.org/docs/app/building-your-application/deploying
- **Setup**:
  1. Connect Git repository
  2. Set all env variables in Vercel dashboard
  3. Configure custom domains
  4. Deploy
- **Troubleshooting**: If `db-utils.ts` causes build errors, ensure `tsconfig.json` excludes it properly

### Drizzle ORM
- **Website**: https://orm.drizzle.team
- **Docs**: https://orm.drizzle.team/docs/overview
- **Kit Docs**: https://orm.drizzle.team/kit-docs/overview
- **Commands**:
  - `pnpm db:generate` — Generate migration from schema changes
  - `pnpm db:migrate` — Apply pending migrations
  - `pnpm db:push` — Push schema directly (dev only)
  - `pnpm db:studio` — Launch Drizzle Studio (GUI)
  - `pnpm db:utils` — Launch db-utils.ts CLI
- **Schema**: `lib/db/schema.ts` — defines all tables with relations

### Postgres.js
- **Website**: https://github.com/porsager/postgres
- **Docs**: https://github.com/porsager/postgres#usage
- **Usage**: Raw SQL queries when Drizzle abstractions aren't enough. Used for storage RLS checks and pg_isready connection tests.

### TanStack React Query
- **Website**: https://tanstack.com/query/latest
- **Docs**: https://tanstack.com/query/latest/docs/framework/react/overview
- **Usage**: Via tRPC client — wraps all tRPC queries with React Query caching, refetching, and mutation handling.

### shadcn/ui
- **Website**: https://ui.shadcn.com
- **Docs**: https://ui.shadcn.com/docs
- **Setup**: `pnpm ui-add` to add new components (not currently configured — components are directly in `components/ui/`)
- **Note**: The project uses Tailwind CSS v4, which has different syntax from v3. shadcn/ui components may need manual adaptation.

### base-ui
- **Website**: https://base-ui.com
- **Docs**: https://base-ui.com/docs
- **Usage**: Low-level UI primitives for accessibility (complementing shadcn/ui)

### Lucide React
- **Website**: https://lucide.dev
- **Docs**: https://lucide.dev/docs/lucide-react
- **Usage**: Icons throughout the app — over 1000 available icons

### Sonner
- **Website**: https://sonner.emilkowal.ski
- **Docs**: https://sonner.emilkowal.ski/getting-started
- **Usage**: Toast notifications via `<Toaster />` in layouts, `toast()` to trigger

### Zod
- **Website**: https://zod.dev
- **Docs**: https://zod.dev/?id=basic-usage
- **Usage**: tRPC input validation, env var validation via t3-env, form validation patterns

### class-variance-authority (cva)
- **Website**: https://cva.style
- **Docs**: https://cva.style/docs
- **Usage**: Component variant definitions (used in shadcn/ui components)

### next-themes
- **Website**: https://github.com/pacocoursey/next-themes
- **Docs**: https://github.com/pacocoursey/next-themes#readme
- **Usage**: Dark/light mode toggle (applied alongside custom theme system)

### Google Fonts
- **Fonts loaded**: Syne, Playfair Display, JetBrains Mono, Bodoni Moda, Nunito
- **Implementation**: `next/font/google` in root layout with CSS variable injection
- **Font pairings**: 8 preset combinations using these 5 fonts

### clsx + tailwind-merge
- **clsx**: Conditional classname joining
- **tailwind-merge**: Smart merging of Tailwind classes (resolves conflicts)
- **Combined**: `cn()` utility function used throughout
