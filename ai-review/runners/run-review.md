# Run AI Review

You are reviewing the current project.

An AI Review Framework exists at:

```
C:/dev/AI Dev Prompts/ai-review
```

Read, in order:

1. `README.md`
2. `framework/10-readme.md`
3. `framework/20-review-framework.md`

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
13. All (full production review)
```

Based on the selection, read the corresponding review document from `reviews/` and follow its instructions exactly.

Produce all reports in:

```
docs/ai-review/reports/
```

Do not begin writing until all referenced documents have been read.
