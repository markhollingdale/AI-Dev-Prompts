# Cost Analysis

## Objective

Perform a comprehensive cost analysis of this application.

The goal is to evaluate whether the application's technical implementation is economically sustainable and whether operational costs are likely to remain under control as the application grows.

Assess infrastructure costs, third-party services, AI usage, storage, bandwidth, database usage and cost amplification risks.

This review focuses **only on operational cost efficiency**.

Do **not** perform detailed reviews of performance, architecture or production readiness except where they directly impact operational cost.

All findings and scoring must follow the standards defined in:

```
framework/20-review-framework.md
```

---

# Phase 1 - Cost Documentation

Create:

```
docs/ai-review/reports/[project-name]-110-cost-analysis.md
```

Document how the application incurs operational costs.

Do not assess quality yet.

---

## 1. Infrastructure Overview

Document:

- Hosting platform
- Database
- Storage
- CDN
- Edge Functions
- Serverless Functions
- Regions

Describe the primary infrastructure components contributing to operational cost.

---

## 2. Third-Party Services

Document:

- Authentication
- Payments
- Email
- AI providers
- Analytics
- Monitoring
- Logging
- Search
- Maps
- SMS
- Other external services

Summarise how each service contributes to recurring costs.

---

## 3. AI Usage

If applicable, document:

- AI providers
- Models used
- Prompt sizes
- Token usage
- Embeddings
- Image generation
- Audio generation
- Batch processing

---

## 4. Database Usage

Document:

- Reads
- Writes
- Storage
- Backups
- Replication
- Scheduled jobs

Focus on cost drivers rather than performance.

---

## 5. Storage

Document:

- File storage
- Image storage
- Video storage
- Object storage
- Backups

---

## 6. Bandwidth

Document:

- Static assets
- Dynamic responses
- Media
- API traffic
- CDN usage

---

## 7. Background Processing

Document:

- Scheduled jobs
- Queues
- Workers
- AI processing
- Batch operations

---

## 8. Billing Model

Document:

Where practical, identify how major services are billed.

Examples:

- Per request
- Per user
- Per GB
- Per million operations
- Per token
- Per execution
- Per seat

---

## 9. Cost Controls

Document existing controls such as:

- Rate limiting
- Usage limits
- Quotas
- Budget alerts
- Spend monitoring
- Caching
- Request batching

---

## 10. Growth Assumptions

Estimate the primary cost drivers as usage increases.

Examples:

- 100 users
- 1,000 users
- 10,000 users
- 100,000 users

---

# Phase 2 - Cost Assessment

Create:

```
docs/ai-review/reports/[project-name]-110-cost-analysis-review.md
```

Follow the format defined in:

```
framework/20-review-framework.md
```

---

# Infrastructure Efficiency

Review:

- Hosting choices
- Serverless usage
- Compute utilisation
- Regions
- Resource allocation

Identify unnecessary infrastructure costs.

---

# Third-Party Services

Review:

- Service selection
- Pricing tiers
- Vendor lock-in
- Duplicate services
- Under-utilised services

Identify opportunities to reduce recurring costs.

---

# AI Cost Efficiency

If applicable, review:

- Model selection
- Prompt efficiency
- Context size
- Response size
- Embeddings
- Caching
- Batch processing
- Token optimisation

Estimate avoidable AI expenditure.

---

# Database Cost Efficiency

Review:

- Read amplification
- Write amplification
- Storage growth
- Backup costs
- Replication
- Query frequency

Focus on financial impact rather than performance.

---

# Storage Efficiency

Review:

- Object storage
- Media optimisation
- Compression
- Retention
- Lifecycle policies

Identify unnecessary storage costs.

---

# Bandwidth Efficiency

Review:

- Static assets
- Images
- Video
- Downloads
- CDN usage
- Compression

Estimate unnecessary bandwidth expenditure.

---

# Cost Amplification Risks

Identify technical implementations that could rapidly increase costs.

Examples:

- Uncached AI requests
- Infinite polling
- Duplicate processing
- Excessive logging
- Repeated API calls
- Inefficient retries
- Unbounded background jobs

Estimate potential impact.

---

# Scaling Economics

Assess how costs are likely to scale as usage grows.

Consider:

- Infrastructure
- Database
- AI
- Storage
- Bandwidth
- Monitoring
- Third-party services

Identify likely cost bottlenecks.

---

# Unit Economics

Where practical, estimate:

- Cost per request
- Cost per active user
- Cost per tenant
- Cost per AI interaction
- Cost per file upload
- Cost per scheduled job

If exact figures are unavailable, provide reasoned estimates based on the implementation and service pricing models.

---

# Cost Controls

Review:

- Rate limiting
- Usage quotas
- Soft limits
- Hard limits
- Billing alerts
- Spend monitoring
- Administrative controls

Evaluate how effectively the application prevents unexpected expenditure.

---

# Cost Optimisation Opportunities

Identify practical improvements such as:

- Better caching
- Request batching
- Lower-cost services
- Reduced AI usage
- Storage lifecycle policies
- Image optimisation
- Scheduled processing
- Infrastructure consolidation

Estimate potential savings where practical.

---

# Technical Debt

Identify:

- Temporary infrastructure
- Legacy services
- Duplicate subscriptions
- Over-provisioned resources
- Missing cost controls
- Expensive workarounds

Estimate long-term financial impact.

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
- Estimated financial impact (where practical)

Do not duplicate findings that belong in other reviews.

For example:

- Slow queries → Performance Analysis
- Missing monitoring → Production Readiness
- Poor architecture → Architecture Analysis
- Security vulnerabilities → Security Audit

Reference the appropriate review instead.

---

# Positive Findings

Identify engineering decisions that reduce operational costs.

Explain why they improve long-term financial sustainability.

---

# Reusable Cost Optimisation Patterns

Highlight reusable cost-saving patterns.

Examples:

- AI request caching
- Tiered storage
- Background batching
- CDN optimisation
- Usage quotas
- Cost-aware architecture

---

# Final Recommendation

Provide:

- Overall Cost Efficiency Score
- Category Scores
- Cost Scalability Assessment
- Highest Priority Savings
- Estimated Remediation Effort
- Estimated Annual Savings (where practical)
- Overall Recommendation

Follow the structure defined in:

```
framework/20-review-framework.md
```

---

# Review Behaviour

Read the implementation before making conclusions.

Inspect infrastructure, third-party services, deployment configuration and application behaviour before estimating costs.

Prioritise sustainable operational economics over premature micro-optimisations.

Avoid recommending changes that significantly increase complexity for negligible savings.

Recognise cost-efficient engineering practices as well as unnecessary expenditure.

Avoid duplicate findings across review standards.

When uncertain, clearly state assumptions and explain the basis of any cost estimates.