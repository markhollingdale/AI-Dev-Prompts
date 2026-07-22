# Engineering Review Summary

## Objective

Aggregate and synthesise findings from all completed engineering reviews into a single summary report.

This document does **not** perform any new analysis.

It reads existing review reports and produces a consolidated view of the project's engineering health.

All findings and scoring must follow the standards defined in:

```
framework/20-review-framework.md
```

---

# Prerequisites

Before producing the summary, verify that the following reports exist in:

```
docs/ai-review/reports/
```

## Required Review Reports

| Review | Expected Files |
|--------|----------------|
| Architecture | `[project-name]-10-architecture.md`, `[project-name]-10-architecture-review.md` |
| Security | `[project-name]-20-security.md`, `[project-name]-20-security-review.md` |
| Performance | `[project-name]-30-performance.md`, `[project-name]-30-performance-review.md` |
| Database | `[project-name]-40-database.md`, `[project-name]-40-database-review.md` |
| API | `[project-name]-50-api.md`, `[project-name]-50-api-review.md` |
| Code Quality | `[project-name]-60-code-quality.md`, `[project-name]-60-code-quality-review.md` |
| TypeScript | `[project-name]-70-typescript.md`, `[project-name]-70-typescript-review.md` |
| Accessibility | `[project-name]-80-accessibility.md`, `[project-name]-80-accessibility-review.md` |
| SEO | `[project-name]-90-seo.md`, `[project-name]-90-seo-review.md` |
| Production Readiness | `[project-name]-100-production-readiness.md`, `[project-name]-100-production-readiness-review.md` |
| Cost Analysis | `[project-name]-110-cost-analysis.md`, `[project-name]-110-cost-analysis-review.md` |
| Maintainability | `[project-name]-120-maintainability.md`, `[project-name]-120-maintainability-review.md` |

If any review reports are missing, list them and stop.

Do not produce a partial summary.

---

# Phase 1 - Read All Reviews

Read every review report listed above.

Extract from each:

- Overall score
- Category scores
- Severity summary (Critical, High, Medium, Low)
- Production readiness verdict
- Top priority improvements
- Estimated remediation effort

---

# Phase 2 - Summary Report

Create:

```
docs/ai-review/reports/[project-name]-130-summary.md
```

Follow the format defined in:

```
framework/20-review-framework.md
```

---

## 1. Executive Summary

Provide a concise overview of the project's engineering health.

Include:

- Overall assessment
- Key strengths across all reviews
- Highest risks across all reviews
- Production recommendation

---

## 2. Overall Engineering Score

Calculate an overall score based on all review scores.

Explain the weighting used.

---

## 3. Review Scores Summary

Present a table of all review scores:

| Review | Score | Production Ready |
|--------|-------|------------------|
| Architecture | X/100 | YES/NO |
| Security | X/100 | YES/NO |
| Performance | X/100 | YES/NO |
| Database | X/100 | YES/NO |
| API | X/100 | YES/NO |
| Code Quality | X/100 | YES/NO |
| TypeScript | X/100 | YES/NO |
| Accessibility | X/100 | YES/NO |
| SEO | X/100 | YES/NO |
| Production Readiness | X/100 | YES/NO |
| Cost Analysis | X/100 | YES/NO |
| Maintainability | X/100 | YES/NO |

---

## 4. Combined Severity Summary

Aggregate severity counts across all reviews:

| Severity | Count |
|----------|-------|
| Critical | X |
| High | X |
| Medium | X |
| Low | X |

---

## 5. Production Readiness Assessment

Provide an overall production readiness verdict.

Consider:

- Are there any Critical findings?
- Are there any High findings that block production?
- Which reviews are not production ready?
- What is the overall confidence level?

State clearly:

```
Production Ready: YES / NO
```

Explain why.

---

## 6. Top Priority Improvements

List the highest priority improvements across all reviews.

Rank by:

1. Severity
2. Business impact
3. Effort required

Limit to the top 10-15 items.

---

## 7. Remediation Roadmap

Suggest a phased approach to addressing findings.

### Immediate (Before Production)

Critical and blocking High items.

### Short-Term (First Sprint)

High priority items that don't block production.

### Medium-Term (Next 1-3 Months)

Medium priority items and technical debt.

### Long-Term (Strategic)

Low priority items and future improvements.

---

## 8. Estimated Total Remediation Effort

Sum the estimated effort across all reviews.

Break down by:

- Quick Wins (< 1 day each)
- Medium Improvements (1-5 days each)
- Major Refactoring (> 5 days each)
- Total Estimated Effort

---

## 9. Strengths Worth Preserving

Highlight the strongest engineering decisions across all reviews.

These are patterns and practices worth keeping.

---

## 10. Patterns Worth Reusing

Identify reusable engineering patterns observed across multiple reviews.

These become references for future projects.

---

## 11. Recommendations for Next Review Cycle

Suggest:

- Which areas to review first in the next cycle
- What to focus on
- When to schedule the next review

---

## 12. Implementation Specifications

Check for implementation specifications at:
- `[project-name]-140-specification-critical.md`
- `[project-name]-140-specification-high.md`
- `[project-name]-140-specification-medium.md`
- `[project-name]-140-specification-low.md`

For each specification that exists, summarise:

- Severity level
- Total phases
- Total findings
- Estimated total effort
- Progress (if any phases have been completed)

If no specifications exist, recommend generating them using option 14 from the runner. Suggest starting with Critical, then High, then Medium. Low can be skipped if not needed.

---

## 13. Final Recommendation

Provide a concise engineering conclusion.

Include:

- Overall confidence
- Production recommendation
- Highest priority actions
- Suggested next steps

---

# Review Behaviour

Read all existing review reports before producing the summary.

Do not perform new analysis.

Do not duplicate findings.

Aggregate and synthesise.

When scores conflict between reviews, note the discrepancy.

Prioritise clarity and actionability.

The summary should give a decision-maker everything needed to understand the project's engineering health and make informed decisions.
