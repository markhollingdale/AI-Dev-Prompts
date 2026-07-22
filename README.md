# AI Development Prompts

A collection of structured prompts and frameworks for AI-assisted software development.

## Overview

This repository contains reusable prompts and frameworks designed to help AI assistants perform consistent, high-quality work across different aspects of software development.

The collection is organized into three main areas:

- **AI Review Framework** — Structured engineering review standards for production-grade code reviews
- **Skills** — Prompts for building expertise and creating learning plans
- **Miscellaneous** — One-off prompts for specific development tasks

## Directory Structure

```
AI Dev Prompts/
├── README.md
├── ai-review/          ← Engineering review framework
├── skills/             ← Skill-building and learning prompts
└── miscellaneous/      ← One-off development prompts
```

## AI Review Framework

The largest and most fully implemented component. A comprehensive collection of AI-powered engineering review standards designed to help AI assistants perform consistent, repeatable and production-grade reviews of software projects.

Unlike traditional prompts, these documents define engineering standards. Each review focuses on a specific area of software development while following a shared review framework and scoring methodology.

### What's Included

- **Framework documents** — Define the review philosophy, output format, severity levels, and scoring methodology
- **Review documents** — Individual review standards for different areas (architecture, security, performance, database, API, code quality, TypeScript, accessibility, SEO, production readiness, cost analysis, maintainability, summary, and specification)
- **Runner** — An orchestrator that asks which review to run and guides the AI through the process

### How It Works

From the project you want to review, point your AI assistant at:

```
ai-review/runners/run-review.md
```

The runner will ask which review to run, load the appropriate review document, and guide the AI through the process.

Each review follows a two-phase structure:

1. **Phase 1 — Documentation**: The AI creates a descriptive document of how the area is implemented
2. **Phase 2 — Assessment**: The AI creates a scored review with findings, severity ratings, and recommendations

Reports are generated in the project being reviewed at:

```
docs/ai-review/reports/
```

Once all 12 reviews are complete, the Summary review aggregates all findings into a single engineering health report.

See the `ai-review/framework/` directory for detailed documentation on the review philosophy and methodology.

## Skills

Prompts for building expertise and creating structured learning plans. These are general-purpose prompts that can be adapted for any skill or domain.

## Miscellaneous

One-off prompts for specific development tasks that don't fit into the other categories. These are typically task-specific prompts that may be reused across similar projects.

## How to Use

1. **Pick a prompt** from the appropriate directory
2. **Give it to an AI assistant** (Claude, GPT, etc.) along with relevant context
3. **Follow the instructions** the prompt provides
4. **Review the output** and iterate as needed

Each prompt is designed to be self-contained and reusable across different projects.

## Philosophy

These prompts are designed to be:

- **Structured** — Clear steps and expected outputs
- **Repeatable** — Consistent results across different projects
- **Evidence-based** — Focused on reading actual implementation, not assumptions
- **Production-focused** — Geared toward real-world software development

## Versioning

Current version: **v1.0.0**
