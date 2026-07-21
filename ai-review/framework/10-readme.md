\# AI Engineering Review Framework



> A comprehensive collection of AI-powered engineering review standards for modern TypeScript SaaS applications.



\## Overview



This repository contains a set of structured engineering review documents designed to help AI assistants perform consistent, repeatable and production-grade reviews of software projects.



Unlike traditional prompts, these documents define engineering standards. Each review focuses on a specific area of software development while following a shared review framework and scoring methodology.



The goal is to produce objective, repeatable reviews that identify risks, technical debt, performance issues and opportunities for improvement before software reaches production.



\---



\# Philosophy



Every review should be:



\- Thorough rather than fast

\- Evidence-based

\- Technology agnostic where possible

\- Opinionated but justified

\- Focused on long-term maintainability

\- Suitable for production systems

\- Consistent across every project



The AI should read the actual implementation rather than making assumptions.



Whenever possible it should inspect source code, configuration files, dependencies and project structure before making recommendations.



\---



\# Review Workflow



Reviews should normally be completed in the following order.



```

Architecture

&#x20;       ↓

Security

&#x20;       ↓

Performance

&#x20;       ↓

Database

&#x20;       ↓

API

&#x20;       ↓

Code Quality

&#x20;       ↓

TypeScript

&#x20;       ↓

Accessibility

&#x20;       ↓

SEO

&#x20;       ↓

Production Readiness

&#x20;       ↓

Cost Analysis

&#x20;       ↓

Maintainability



&#x20;       ↓



Overall Engineering Score

```



Some reviews may overlap, but each has a clearly defined primary responsibility.



\---



\# Current Reviews



| Review | Purpose |

|---------|----------|

| Architecture | Overall project architecture, design patterns and scalability |

| Security | Application security, OWASP, authentication and authorisation |

| Performance | Runtime, rendering, database and infrastructure performance |

| Database | Schema design, indexing, migrations and data modelling |

| API | API quality, contracts, validation and scalability |

| Code Quality | Maintainability, complexity and engineering practices |

| TypeScript | Type safety and language best practices |

| Accessibility | WCAG compliance and usability |

| SEO | Search engine optimisation and metadata |

| Production Readiness | Deployment readiness checklist |

| Cost Analysis | Infrastructure efficiency and operational costs |

| Maintainability | Technical debt and long-term sustainability |



\---



\# Review Framework



Every review must follow the common format defined in:



```

review-framework.md

```



This ensures every report has:



\- Consistent scoring

\- Consistent severity levels

\- Consistent recommendations

\- Comparable results across projects



\---



\# Severity Levels



\## Critical



A serious issue that could result in:



\- Security compromise

\- Data loss

\- Financial loss

\- Complete service failure



Must be fixed before production.



\---



\## High



Major issue affecting:



\- Reliability

\- Security

\- Performance

\- Scalability



Should normally be resolved before production.



\---



\## Medium



Important improvement that reduces technical debt or future risk.



Should be planned for the next development cycle.



\---



\## Low



Minor improvement or best practice recommendation.



\---



\# Principles



Every recommendation should explain:



\- What is wrong

\- Why it matters

\- The impact

\- How to fix it

\- Estimated effort

\- Priority



Recommendations should be actionable.



Avoid vague statements.



\---



\# Project Scoring



Every review produces:



\- Category scores

\- Overall review score

\- Severity summary

\- Estimated remediation effort

\- Production readiness assessment



The scores should make it possible to compare projects over time.



\---



\# Intended Audience



This framework is intended for:



\- SaaS applications

\- Modern TypeScript projects

\- Next.js applications

\- T3 Stack projects

\- AI-assisted software development

\- Production engineering reviews



Although optimised for the T3 Stack, most reviews should be applicable to other modern web frameworks.



\---



\# Contributing



When creating a new review:



1\. Follow the Review Framework.

2\. Keep the scope focused.

3\. Avoid overlap with existing reviews.

4\. Provide evidence-based recommendations.

5\. Maintain consistent terminology and scoring.



\---



\# Versioning



The review framework should evolve over time.



Changes should be versioned so previous review reports remain comparable.



Current Version:



\*\*v1.0.0\*\*

