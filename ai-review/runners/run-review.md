# Run AI Review

You are reviewing the current project.

An AI Review Framework exists at:

```
C:/dev/AI Dev Prompts/ai-review
```

Read, in order:

1. `C:/dev/AI Dev Prompts/README.md` (the parent project README)
2. `C:/dev/AI Dev Prompts/ai-review/framework/10-readme.md`
3. `C:/dev/AI Dev Prompts/ai-review/framework/20-review-framework.md`

Then ask the user:

```
What review would you like to run?

1. Architecture
2. Security
3. Performance
4. Database
5. API
6. Code Quality
7. TypeScript
8. Accessibility
9. SEO
10. Production Readiness
11. Cost Analysis
12. Maintainability
13. Summary (aggregates existing reviews)
```

Based on the selection, read the corresponding review document from the paths below and follow its instructions exactly.

## Review File Paths

| Number | Review | File Path |
|--------|--------|-----------|
| 1 | Architecture | `C:/dev/AI Dev Prompts/ai-review/reviews/10-architecture-analysis/10-architecture-analysis.md` |
| 2 | Security | `C:/dev/AI Dev Prompts/ai-review/reviews/20-security-analysis/20-security-analysis.md` |
| 3 | Performance | `C:/dev/AI Dev Prompts/ai-review/reviews/30-performance-analysis/30-performance-analysis.md` |
| 4 | Database | `C:/dev/AI Dev Prompts/ai-review/reviews/40-database-analysis/40-database-analysis.md` |
| 5 | API | `C:/dev/AI Dev Prompts/ai-review/reviews/50-api-analysis/50-api-analysis.md` |
| 6 | Code Quality | `C:/dev/AI Dev Prompts/ai-review/reviews/60-code-quality-analysis/60-code-quality-analysis.md` |
| 7 | TypeScript | `C:/dev/AI Dev Prompts/ai-review/reviews/70-typescript-analysis/70-typescript-analysis.md` |
| 8 | Accessibility | `C:/dev/AI Dev Prompts/ai-review/reviews/80-accessibility-analysis/80-accessibility-analysis.md` |
| 9 | SEO | `C:/dev/AI Dev Prompts/ai-review/reviews/90-seo-analysis/90-seo-analysis.md` |
| 10 | Production Readiness | `C:/dev/AI Dev Prompts/ai-review/reviews/100-production-readiness/100-production-readiness.md` |
| 11 | Cost Analysis | `C:/dev/AI Dev Prompts/ai-review/reviews/110-cost-analysis/110-cost-analysis.md` |
| 12 | Maintainability | `C:/dev/AI Dev Prompts/ai-review/reviews/120-maintainability/120-maintainability.md` |
| 13 | Summary | `C:/dev/AI Dev Prompts/ai-review/reviews/130-summary/130-summary.md` |

## Report Output

Create the output folder if it does not exist:

```
docs/ai-review/reports/
```

Produce all reports in this folder.

Do not begin writing until all referenced documents have been read.
