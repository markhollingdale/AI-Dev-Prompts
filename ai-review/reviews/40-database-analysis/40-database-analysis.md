# Database Analysis

## Objective

Perform a comprehensive analysis of the application's database design.

The goal is to evaluate the quality of the data model, schema design, relationships, migrations, integrity and long-term maintainability.

This review focuses **only on database design**.

Do **not** perform detailed reviews of application architecture, security, performance, APIs or production readiness except where they directly impact the database design.

All findings and scoring must follow the standards defined in:

```
framework/20-review-framework.md
```

---

# Phase 1 - Database Documentation

Create:

```
reports/database.md
```

Document the complete database architecture.

This document should explain how the application's data is organised.

Do not assess quality yet.

---

## 1. Database Overview

Document:

- Database platform
- ORM
- Migration tool
- Schema organisation
- Naming conventions
- General design philosophy

---

## 2. Schema Structure

Document:

- Schemas
- Tables
- Views
- Materialised views
- Functions
- Triggers
- Enums
- Composite types

Explain how responsibilities are divided.

---

## 3. Entity Relationships

Document:

- One-to-One
- One-to-Many
- Many-to-Many
- Junction tables
- Optional relationships
- Required relationships

Produce an overview of the domain model.

---

## 4. Primary Keys

Document:

- Primary key strategy
- UUID usage
- Integer usage
- Composite keys
- Natural keys

Explain why each strategy has been chosen.

---

## 5. Foreign Keys

Document:

- Relationships
- Cascade rules
- Restrict rules
- Null handling
- Orphan prevention

---

## 6. Constraints

Document:

- Unique constraints
- Check constraints
- Default values
- Not-null constraints
- Business constraints

---

## 7. Migrations

Document:

- Migration framework
- Migration strategy
- Rollback strategy
- Seeding
- Initialisation

---

## 8. Transactions

Document:

- Transaction usage
- Atomic operations
- Rollback behaviour
- Long-running transactions

---

## 9. Data Lifecycle

Document:

- Record creation
- Updates
- Soft deletes
- Hard deletes
- Archiving
- Retention

---

## 10. External Dependencies

Document:

- External databases
- Replication
- Third-party storage
- Synchronisation

---

# Phase 2 - Database Assessment

Create:

```
reports/database-review.md
```

Follow the format defined in:

```
framework/20-review-framework.md
```

---

# Data Model

Review:

- Entity modelling
- Domain modelling
- Table responsibilities
- Normalisation
- Denormalisation
- Data duplication

Evaluate whether the schema accurately represents the business domain.

---

# Relationships

Review:

- Relationship correctness
- Cardinality
- Optional relationships
- Cascade behaviour
- Referential integrity

Attempt to identify inconsistent relationships.

---

# Naming Conventions

Review:

- Table names
- Column names
- Foreign keys
- Junction tables
- Constraints
- Migration names

Evaluate consistency across the project.

---

# Keys

Review:

- Primary keys
- Foreign keys
- Composite keys
- Surrogate keys
- Natural keys

Evaluate whether appropriate key strategies have been chosen.

---

# Constraints

Review:

- NOT NULL
- UNIQUE
- CHECK
- DEFAULT
- Foreign key constraints

Identify missing integrity protections.

---

# Normalisation

Review:

- First Normal Form
- Second Normal Form
- Third Normal Form

Identify unnecessary duplication.

Identify over-normalisation where it unnecessarily increases complexity.

---

# Migrations

Review:

- Migration quality
- Ordering
- Naming
- Safety
- Repeatability
- Rollback capability

Attempt to identify migration risks.

---

# Transactions

Review:

- Atomicity
- Transaction boundaries
- Error handling
- Consistency

Identify operations that should be transactional.

---

# Data Lifecycle

Review:

- Record ownership
- Audit timestamps
- Soft delete strategy
- Hard delete strategy
- Archiving
- Data retention

Evaluate long-term maintainability.

---

# Multi-Tenant Data Model

If the application is multi-tenant, review:

- Tenant ownership
- Tenant boundaries
- Shared tables
- Shared data
- Tenant identifiers

Focus on the quality of the data model rather than security enforcement.

---

# ORM Usage

Review:

- Schema organisation
- Model definitions
- Relationships
- Reusable types
- Generated code
- Validation integration

Evaluate maintainability of the ORM layer.

---

# Database Evolution

Assess how well the schema supports future development.

Consider:

- New features
- Additional entities
- Changing relationships
- Historical data
- Reporting
- Analytics

Identify areas likely to become difficult to maintain.

---

# Data Integrity

Review:

- Referential integrity
- Duplicate prevention
- Invalid state prevention
- Required relationships
- Business rule enforcement

Attempt to identify opportunities for inconsistent data.

---

# Technical Debt

Identify:

- Duplicate tables
- Duplicate columns
- Legacy schema
- Obsolete relationships
- Poor naming
- Missing constraints
- Temporary workarounds

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

- Missing RLS → Security Audit
- Slow queries → Performance Analysis
- Poor repository design → Architecture Analysis
- API validation → API Review

Reference the appropriate review instead.

---

# Positive Findings

Identify database design decisions worth keeping.

Explain why they improve maintainability, integrity or future scalability.

---

# Reusable Database Patterns

Highlight reusable database design patterns.

Examples:

- Audit tables
- Soft delete strategy
- Junction table design
- Migration structure
- Naming conventions
- Transaction patterns

---

# Final Recommendation

Provide:

- Overall Database Score
- Category Scores
- Production Readiness (Database only)
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

Inspect the schema, migrations, ORM definitions and database structure before making recommendations.

Prioritise correctness and maintainability over theoretical perfection.

Avoid recommending unnecessary complexity.

Recognise well-designed schemas as well as weaknesses.

Avoid duplicate findings across review standards.

When uncertain, clearly state assumptions.