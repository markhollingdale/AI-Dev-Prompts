# Performance Analysis

## Objective

Perform a comprehensive performance assessment of this application.

The goal is to identify bottlenecks, inefficient implementations, scalability concerns and opportunities to improve responsiveness, throughput and resource utilisation.

Consider both the current implementation and how performance is likely to evolve as the application grows.

This review focuses **only on performance**.

Do **not** perform detailed reviews of architecture, security, accessibility, SEO, maintainability or production readiness except where they directly impact performance.

All findings and scoring must follow the standards defined in:

```
framework/20-review-framework.md
```

---

# Phase 1 - Performance Documentation

Create:

```
reports/performance.md
```

Document the application's performance architecture.

This document should explain how performance has been considered throughout the application.

Do not assess quality yet.

---

## 1. Rendering Strategy

Document:

- Server Components
- Client Components
- Static rendering
- Dynamic rendering
- Streaming
- Suspense boundaries
- Partial prerendering (if applicable)
- Hydration strategy

---

## 2. Data Fetching

Document:

- Server-side fetching
- Client-side fetching
- Parallel requests
- Sequential requests
- Prefetching
- Caching strategy
- Revalidation strategy

---

## 3. Database Access

Document:

- ORM
- Query patterns
- Transactions
- Connection management
- Pagination
- Background processing

Do not assess indexing or schema quality.

---

## 4. API Performance

Document:

- API architecture
- Request flow
- Response flow
- Validation
- Caching
- Streaming
- Batching

---

## 5. Frontend Assets

Document:

- JavaScript bundles
- CSS strategy
- Fonts
- Images
- Icons
- Third-party scripts

---

## 6. Caching

Document:

- Browser caching
- HTTP caching
- CDN caching
- Next.js cache
- Database caching
- In-memory caching

---

## 7. Background Processing

Document:

- Scheduled jobs
- Queues
- Workers
- Cron jobs
- Long-running tasks

---

## 8. External Services

Document:

- Database
- Authentication
- Payments
- Email
- AI providers
- Storage
- Monitoring

Explain how external dependencies affect performance.

---

# Phase 2 - Performance Assessment

Create:

```
reports/performance-review.md
```

Follow the format defined in:

```
framework/20-review-framework.md
```

---

# Rendering Performance

Review:

- React rendering
- Component hierarchy
- Re-render frequency
- Hydration
- Suspense usage
- Lazy loading
- Memoisation
- Server vs Client Components

Attempt to identify unnecessary rendering work.

---

# Frontend Performance

Review:

- JavaScript bundle size
- Code splitting
- Dynamic imports
- CSS loading
- Font loading
- Image optimisation
- Layout shifts
- Interaction latency

Review Core Web Vitals where possible.

---

# Data Fetching

Review:

- Duplicate requests
- Sequential requests
- Parallelisation
- Request waterfalls
- Over-fetching
- Under-fetching
- Cache effectiveness

Attempt to minimise unnecessary network traffic.

---

# Database Performance

Review:

- Query efficiency
- N+1 queries
- Transaction scope
- Query duplication
- Pagination strategy
- Connection usage

Do not review schema design or indexing unless they directly impact performance.

---

# API Performance

Review:

- Request validation overhead
- Response sizes
- Serialization
- Compression
- Streaming
- Batching
- Pagination

Attempt to identify unnecessary processing.

---

# Caching Strategy

Review:

- Browser caching
- HTTP cache headers
- Next.js cache
- Server cache
- CDN usage
- Cache invalidation

Verify caching is used appropriately.

---

# Network Efficiency

Review:

- Request count
- Payload sizes
- Compression
- Duplicate downloads
- Third-party requests
- Resource prioritisation
- Prefetching

Attempt to identify unnecessary network overhead.

---

# Resource Utilisation

Review:

- CPU intensive operations
- Memory usage
- Background processing
- Event listeners
- Timers
- Cleanup
- Resource leaks

Attempt to identify avoidable resource consumption.

---

# Scalability Assessment

Evaluate how the application is likely to perform as usage increases.

Consider:

- Concurrent users
- Database growth
- Large datasets
- Background jobs
- Queue growth
- External API limits
- AI workloads
- Storage growth

Estimate likely bottlenecks.

---

# External Service Performance

Review the impact of:

- Database latency
- Authentication providers
- Email providers
- AI providers
- Payment providers
- Storage providers

Identify opportunities to reduce external dependency latency.

---

# Cost Efficiency

Identify performance issues that may unnecessarily increase operational costs.

Examples:

- Excessive database queries
- Uncached requests
- Expensive AI calls
- Large bandwidth usage
- Repeated API calls
- Inefficient polling

Estimate potential cost impact where practical.

---

# Load Readiness

Assess how well the application would perform under increasing load.

Consider:

- Burst traffic
- Sustained traffic
- Background processing
- Queue congestion
- Database contention
- Memory pressure

Identify likely scaling limits.

---

# Required Findings

Every issue must include:

- Severity
- Explanation
- Business impact
- Technical impact
- Recommendation
- Example implementation (where appropriate)
- Estimated effort

Do not duplicate findings that belong in other reviews.

For example:

- Missing authentication → Security Audit
- Poor folder structure → Architecture Analysis
- Missing indexes (design issue) → Database Review
- Poor TypeScript usage → TypeScript Review

Reference the appropriate review instead.

---

# Positive Findings

Identify performance decisions worth keeping.

Explain why they improve performance or scalability.

---

# Reusable Performance Patterns

Highlight reusable patterns that could be applied in future projects.

Examples:

- Efficient data fetching
- Cache architecture
- Background processing
- Rendering strategies
- API optimisation
- Resource loading

---

# Final Recommendation

Provide:

- Overall Performance Score
- Category Scores
- Production Readiness (Performance only)
- Highest Priority Improvements
- Estimated Remediation Effort
- Overall Recommendation

Follow the structure defined in:

```
framework/20-review-framework.md
```

---

# Review Behaviour

Read the implementation before making conclusions.

Inspect the source code rather than making assumptions.

Prioritise evidence over opinion.

Focus on realistic bottlenecks rather than micro-optimisations.

Consider both current performance and future scalability.

Provide pragmatic recommendations with measurable benefit.

Recognise good performance practices as well as weaknesses.

Avoid duplicate findings across review standards.

When uncertain, clearly state assumptions.