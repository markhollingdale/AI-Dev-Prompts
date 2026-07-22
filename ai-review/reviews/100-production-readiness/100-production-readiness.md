# Production Readiness Analysis

## Objective

Perform a comprehensive production readiness assessment of this application.

The goal is to determine whether the application is ready for deployment to a production environment.

Assess operational maturity, deployment readiness, reliability, observability and resilience.

This review focuses **only on production readiness**.

Do **not** perform detailed reviews of architecture, security, performance, accessibility or code quality except where they directly impact production deployment.

All findings and scoring must follow the standards defined in:

```
framework/20-review-framework.md
```

---

# Phase 1 - Production Documentation

Create:

```
docs/ai-review/reports/production-readiness.md
```

Document how the application is prepared for production.

Do not assess quality yet.

---

## 1. Deployment Strategy

Document:

- Hosting platform
- Deployment process
- CI/CD pipeline
- Rollback strategy
- Deployment frequency

---

## 2. Environment Configuration

Document:

- Development
- Staging
- Production
- Environment variables
- Configuration management

---

## 3. Monitoring

Document:

- Application monitoring
- Error monitoring
- Performance monitoring
- Health monitoring
- Alerting

---

## 4. Logging

Document:

- Structured logging
- Log aggregation
- Log retention
- Audit logging

---

## 5. Health Checks

Document:

- Health endpoints
- Dependency checks
- Readiness checks
- Liveness checks

---

## 6. Background Processing

Document:

- Scheduled jobs
- Queues
- Workers
- Retry strategy
- Failure handling

---

## 7. Backups & Recovery

Document:

- Backup strategy
- Recovery procedures
- Disaster recovery
- Data retention

---

## 8. Third-Party Dependencies

Document:

- External APIs
- AI providers
- Payment providers
- Email providers
- Storage providers

Explain operational dependencies.

---

## 9. Operations

Document:

- Maintenance procedures
- Operational runbooks
- Incident handling
- Release process

---

## 10. Documentation

Document:

- Deployment documentation
- Operational documentation
- Recovery documentation
- Developer documentation

---

# Phase 2 - Production Assessment

Create:

```
docs/ai-review/reports/production-readiness-review.md
```

Follow the format defined in:

```
framework/20-review-framework.md
```

---

# Deployment Readiness

Review:

- Build process
- Deployment automation
- Environment separation
- Rollback capability
- Repeatability

Evaluate deployment reliability.

---

# Configuration Management

Review:

- Environment variables
- Secret management
- Configuration consistency
- Environment separation
- Feature flags

Evaluate configuration quality.

---

# Monitoring

Review:

- Error monitoring
- Application monitoring
- Performance monitoring
- Infrastructure monitoring
- Alerting

Determine whether production issues would be detected quickly.

---

# Logging

Review:

- Structured logging
- Log consistency
- Correlation IDs
- Error logging
- Audit logging
- Sensitive data handling

Evaluate operational usefulness.

---

# Health & Reliability

Review:

- Health endpoints
- Readiness probes
- Dependency checks
- Failure detection
- Self-healing capabilities

Assess operational resilience.

---

# Error Recovery

Review:

- Retry logic
- Timeouts
- Circuit breakers
- Graceful degradation
- Failure recovery

Evaluate resilience to failures.

---

# Background Processing

Review:

- Queue handling
- Scheduled jobs
- Retry policies
- Idempotency
- Dead-letter handling

Assess reliability of asynchronous processing.

---

# Backup & Disaster Recovery

Review:

- Backup procedures
- Recovery procedures
- Restore testing
- Data retention
- Recovery objectives

Evaluate disaster recovery readiness.

---

# Dependency Management

Review:

- External services
- Service dependencies
- Failure handling
- Timeouts
- Fallback behaviour

Assess resilience to third-party outages.

---

# Scalability Readiness

Review:

- Horizontal scaling
- Stateless services
- Session management
- Resource limits
- Autoscaling compatibility

Evaluate readiness for increased demand.

---

# Operational Readiness

Review:

- Runbooks
- Incident response
- Maintenance procedures
- Release process
- Operational documentation

Determine whether the application can be effectively supported in production.

---

# Testing Readiness

Review:

- Automated tests
- Integration tests
- End-to-end tests
- Smoke tests
- Deployment validation

Evaluate confidence in production deployments.

---

# Observability

Review:

- Metrics
- Logs
- Traces
- Dashboards
- Alerting

Assess whether production behaviour can be understood and diagnosed.

---

# Business Continuity

Review:

- Single points of failure
- Disaster recovery
- Data recovery
- Service continuity
- Operational resilience

Identify risks to business continuity.

---

# Technical Debt

Identify:

- Manual deployment steps
- Missing monitoring
- Missing documentation
- Missing health checks
- Operational workarounds
- Incomplete automation

Estimate long-term operational impact.

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

- Authentication weaknesses → Security Audit
- Slow rendering → Performance Analysis
- Database schema issues → Database Analysis
- Code organisation → Code Quality Analysis

Reference the appropriate review instead.

---

# Positive Findings

Identify production practices worth keeping.

Explain why they improve reliability, resilience or operational maturity.

---

# Reusable Production Patterns

Highlight reusable operational patterns.

Examples:

- Deployment pipelines
- Health checks
- Monitoring
- Logging
- Background workers
- Recovery procedures

---

# Final Recommendation

Provide:

- Overall Production Readiness Score
- Category Scores
- Deployment Recommendation
- Go / No-Go Recommendation
- Highest Priority Improvements
- Estimated Remediation Effort
- Overall Recommendation

Clearly state whether the application is suitable for production deployment.

Follow the structure defined in:

```
framework/20-review-framework.md
```

---

# Review Behaviour

Read the implementation before making conclusions.

Inspect deployment configuration, operational tooling, monitoring and production infrastructure before making recommendations.

Prioritise operational reliability over feature completeness.

Avoid recommending unnecessary operational complexity.

Recognise mature operational practices as well as weaknesses.

Avoid duplicate findings across review standards.

When uncertain, clearly state assumptions.