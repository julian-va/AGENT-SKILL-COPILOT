# AGENT-SKILL-COPILOT

A repository to hold "skills" describing how an AI agent should write, review and maintain backend and frontend services.

This project collects machine-readable and human-readable skills that guide automated agents and contributors on conventions, architecture and CI for Java, Kotlin backends and Angular frontend.

## Contents

- `.github/skills/java-backend/SKILL.md` — Java backend skill: Hexagonal architecture, style rules, use cases/gateways, examples, CI hints.
- `.github/skills/kotlin-backend/SKILL.md` — Kotlin backend skill: Kotlin idioms, coroutines, Hexagonal architecture examples and CI suggestions.
- `.github/skills/angular-frontend/SKILL.md` — Angular frontend skill: Standalone components, signals, Clean Architecture for frontend, Angular Material with Material Design 3.
- `.github/skills/python-backend/SKILL.md` — Python backend skill: FastAPI, Pydantic, MCPs, testing, observability and CI examples.

## Purpose

These SKILL files are intended for automated agents (and humans) to:

- Understand repository conventions and architecture rules.
- Follow consistent coding, testing and CI practices.
- Validate and automate work (lint, build, ArchUnit checks, integration tests).

## Quick Start

### For Backend (Java/Kotlin)

1. Read the relevant skill: `.github/skills/java-backend/SKILL.md` or `.github/skills/kotlin-backend/SKILL.md`.
2. When asking the agent to make a change, require the agent to return the Reasoning Checklist before producing code (see SKILL.md).
3. Follow package conventions:
   - `domain.model` — entities, value objects, domain exceptions
   - `domain.usecase` — use-case interfaces and implementations
   - `domain.gateway` — interfaces for external resources (repositories, services)
   - `infrastructure.drivenAdapters` — gateway implementations
   - `infrastructure.providers` — technical providers (DB, SMTP, security)
   - `infrastructure.entryPoints` — REST controllers and DTOs
   - `infrastructure.config` — bean configuration

### For Frontend (Angular)

1. Read the Angular skill: `.github/skills/angular-frontend/SKILL.md`.
2. Follow Clean Architecture for frontend:
   - `core/` — domain models, business logic services, guards
   - `data/` — repositories, API clients, mappers
   - `features/` — feature-specific components and services
   - `shared/` — reusable components, directives, pipes
   - `layout/` — layout components (header, footer, sidebar)
3. Use modern Angular: standalone components, signals, `@if/@for/@switch`, `input()/output()` functions
4. Use Angular Material with Material Design 3 for UI components and theming

## CI

This repo suggests GitHub Actions workflows for both backend and frontend:

- **Backend**: JDK 21 with Gradle/Maven (see Java/Kotlin SKILL.md for snippets)
- **Frontend**: Node.js 20 with pnpm and Angular CLI (see Angular SKILL.md for snippets)

## Contributing

- Edit or extend SKILL.md files to refine rules.
- When changing architecture rules, document rationale and update examples and ArchUnit checks.
- Open PRs, include the Reasoning Checklist for code-generation tasks.

## Licensing

Specify a license in this repository (e.g. MIT) by adding a `LICENSE` file.

---

If you want, I can:

- Add a `LICENSE` file (MIT) and commit it.
- Add `.github/workflows/java-ci.yml` and `.github/workflows/kotlin-ci.yml` examples.
