# SEO Analysis

## Objective

Perform a comprehensive SEO assessment of this application.

The goal is to evaluate the application's technical SEO, content structure, discoverability and search engine readiness.

Assess whether the application follows modern SEO best practices and provides search engines with sufficient information to accurately crawl, index and understand its content.

This review focuses **only on SEO**.

Do **not** perform detailed reviews of accessibility, performance, marketing, branding or user experience except where they directly impact SEO.

All findings and scoring must follow the standards defined in:

```
framework/20-review-framework.md
```

---

# Phase 1 - SEO Documentation

Create:

```
reports/seo.md
```

Document the application's SEO implementation.

Do not assess quality yet.

---

## 1. SEO Strategy

Document:

- SEO objectives
- Target audience
- Target search intent
- Content strategy
- Indexing strategy

---

## 2. Metadata

Document:

- Title strategy
- Meta descriptions
- Canonical URLs
- Robots directives
- Open Graph
- Twitter Cards

---

## 3. URL Structure

Document:

- URL conventions
- Slugs
- Dynamic routes
- Pagination URLs
- Canonicalisation

---

## 4. Structured Data

Document:

- Schema.org usage
- JSON-LD
- Rich snippets
- Business schema
- Product schema
- Article schema

---

## 5. Internal Linking

Document:

- Navigation
- Breadcrumbs
- Related content
- Internal links
- Footer links

---

## 6. Content Structure

Document:

- Heading hierarchy
- Landing pages
- Category pages
- Service pages
- Blog structure

---

## 7. Media

Document:

- Images
- Alt text strategy
- Image optimisation
- Video
- File naming

---

## 8. Technical SEO

Document:

- Sitemap
- Robots.txt
- Canonical strategy
- Redirect strategy
- HTTPS
- Internationalisation (if applicable)

---

## 9. Performance Signals

Document:

- Core Web Vitals
- Rendering strategy
- JavaScript usage
- SSR / SSG
- Lazy loading

Document only.

Do not assess performance.

---

## 10. Analytics

Document:

- Analytics platform
- Search Console integration
- Event tracking
- Conversion tracking

---

# Phase 2 - SEO Assessment

Create:

```
reports/seo-review.md
```

Follow the format defined in:

```
framework/20-review-framework.md
```

---

# Crawlability

Review:

- Robots.txt
- XML sitemap
- Crawl depth
- Internal links
- Redirects
- Broken links

Evaluate whether search engines can efficiently crawl the site.

---

# Indexability

Review:

- Meta robots
- Canonical tags
- Duplicate pages
- Parameter handling
- Noindex usage

Identify pages that may not index correctly.

---

# Metadata

Review:

- Page titles
- Meta descriptions
- Canonical URLs
- Open Graph
- Twitter Cards

Evaluate quality, consistency and uniqueness.

---

# URL Structure

Review:

- Readability
- Keywords
- Slug quality
- Hierarchy
- Consistency

Identify unnecessarily complex URLs.

---

# Content Structure

Review:

- Heading hierarchy
- Semantic structure
- Page hierarchy
- Content organisation
- Landing pages

Evaluate whether content is easy for search engines to understand.

---

# Structured Data

Review:

- JSON-LD
- Schema.org
- Rich Results
- Business information
- Breadcrumb schema
- FAQ schema

Identify opportunities to improve search engine understanding.

---

# Internal Linking

Review:

- Navigation
- Contextual links
- Breadcrumbs
- Related content
- Orphan pages

Evaluate the effectiveness of the internal linking strategy.

---

# Images & Media

Review:

- Alt text
- File names
- Captions
- Image formats
- Lazy loading

Evaluate media optimisation for search engines.

---

# Mobile SEO

Review:

- Responsive layouts
- Mobile usability
- Viewport configuration
- Touch targets
- Mobile rendering

Assess mobile search readiness.

---

# Local SEO

If applicable, review:

- Business information
- Address consistency
- Contact information
- Local structured data
- Location pages

Evaluate local search optimisation.

---

# International SEO

If applicable, review:

- hreflang
- Language handling
- Regional targeting
- Canonicalisation
- Duplicate content

---

# JavaScript SEO

Review:

- Server-side rendering
- Static rendering
- Hydration
- Search engine rendering
- Dynamic content

Evaluate whether search engines can fully understand rendered content.

---

# Content Quality

Review:

- Originality
- Relevance
- Search intent
- Duplicate content
- Thin content

Evaluate whether content satisfies likely user intent.

---

# EEAT Signals

Review evidence supporting:

- Experience
- Expertise
- Authoritativeness
- Trustworthiness

Examples:

- Author information
- Business details
- Contact information
- Policies
- Reviews
- Citations

Evaluate credibility signals where applicable.

---

# Technical Debt

Identify:

- Duplicate metadata
- Missing metadata
- Broken links
- Thin pages
- Duplicate content
- Redirect chains
- Missing structured data

Estimate long-term SEO impact.

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

- Slow rendering → Performance Analysis
- Poor accessibility → Accessibility Analysis
- Missing security headers → Security Audit

Reference the appropriate review instead.

---

# Positive Findings

Identify SEO practices worth keeping.

Explain why they improve discoverability or search performance.

---

# Reusable SEO Patterns

Highlight reusable SEO patterns.

Examples:

- Metadata generation
- Structured data
- Sitemap generation
- URL conventions
- Content hierarchy
- Internal linking

---

# Final Recommendation

Provide:

- Overall SEO Score
- Category Scores
- Search Engine Readiness
- Production Readiness (SEO only)
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

Inspect routing, metadata generation, structured data and rendered HTML before making recommendations.

Prioritise long-term discoverability over short-term ranking tactics.

Avoid recommending keyword stuffing or manipulative SEO techniques.

Recognise excellent technical SEO practices as well as weaknesses.

Avoid duplicate findings across review standards.

When uncertain, clearly state assumptions.