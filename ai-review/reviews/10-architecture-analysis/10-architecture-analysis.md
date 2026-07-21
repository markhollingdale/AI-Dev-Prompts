Create a comprehensive architecture document for this project as [docs/architecture.md](docs/architecture.md).

Do NOT describe what the project does or its business purpose. Focus purely on:

1. **Technology Stack** — Every framework, library, package with exact versions and what it's used for
2. **Project Structure** — Every directory, every file, what lives where
3. **Routing & Layouts** — Route groups, layouts, how pages are organized
4. **Configuration System** — How config is loaded, resolved, overridden at runtime
5. **Component Architecture** — How components are organized, key patterns, reusable shells/wrappers
6. **API Layer** — tRPC setup, routers, middleware chain, context creation
7. **Authentication** — Auth library choice, setup, session management, RBAC/permissions model
8. **Database** — ORM, schema (table counts and purposes), migrations, connection setup
9. **Security** — What security measures exist AND what's missing (CSP, rate limiting, input validation, etc.)
10. **External Integrations** — Email, storage, analytics, monitoring — how each connects, what env vars they need, setup instructions with URLs
11. **Environment Variables** — Every variable, what it does, whether it's required, validation schema
12. **Reusable Code** — Any cross-cutting utilities, hooks, patterns worth extracting

Then create a second file [docs/architecture-review.md] that is a critical audit:

1. Rate each architectural area 0-10 with a one-line reason
2. List every issue found, ranked by priority (Critical > High > Medium)
3. For each issue, explain the problem, why it matters, and provide concrete fix code/solution
4. Call out what patterns are worth keeping for future projects
5. Call out what should be skipped as project-specific
6. Notes on consistency patterns observed (good or bad)

Be thorough. Read every key file: package.json, tsconfig, next.config, all layouts, all providers, the main lib files, the server/trpc setup, the schema, the env validation. Do not summarize — read the actual code.