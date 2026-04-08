# MATSim-Libs Project Guidelines

<!-- This file is the entry point for GitHub Copilot. The same rules are mirrored in /AGENTS.md for other AI coding agents. Detailed topic-specific instructions live in .github/instructions/ and apply to all AI coding agents. -->

## Project Overview

MATSim is a large-scale agent-based transport simulation framework.
This is a multi-module Maven project licensed under GPL v2+.

### Module Layout

- `matsim/` — Core library (API, config, controler, events, mobsim, network, population, routing, scoring)
- `contribs/` — 40+ extension modules under `org.matsim.contrib.*`, each with its own POM
- `examples/` — Example scenarios and usage
- `benchmark/` — Performance benchmarks

### Package Namespace

- Core: `org.matsim.core.*`, `org.matsim.api.*`, `org.matsim.pt.*`, `org.matsim.run.*`
- Contribs: `org.matsim.contrib.<contrib-name>.*`

## Build and Test

Build everything (skip tests):
```
mvn package -DskipTests
```

Install core only:
```
mvn install --also-make --projects matsim
```

Run tests for a single module:
```
mvn verify --batch-mode -Dmaven.test.redirectTestOutputToFile -Dmatsim.preferLocalDtds=true --fail-at-end -Dsource.skip
```
Run this from within the module directory (e.g., `contribs/drt/`).

Build a contrib with dependencies:
```
mvn install --batch-mode --also-make --projects contribs/<name> -DskipTests -Dsource.skip
```

CI runs a **per-module matrix build** — each module is built and tested independently.

## Key Conventions

- **Java 25** — `maven.compiler.release=25`
- **Tabs** for indentation, not spaces (but when editing an existing file, match the style already in that file)
- **UTF-8** encoding everywhere
- **150-character** max line length for Java
- **GPL v2+ license header** required on all source files
- **Conventional Commits** for PR titles: `<type>[optional scope]: <description>`
  - Types: feat, fix, docs, style, refactor, perf, test, build, ci, chore, revert
- **Dependency versions** are managed in the root POM via properties — do not duplicate versions in child POMs
- **Guice 7.0.0** for dependency injection — modules extend `org.matsim.core.controler.AbstractModule`
- **Maven enforcer** is active: upper-bound deps, no duplicate POM versions, Maven ≥ 3.8

## Architecture Principles

- Contribs depend on core `matsim` but core must not depend on contribs.
- Cross-contrib dependencies are explicit in the contrib's own POM.
- Public API changes require deprecation-first approach unless explicitly approved.
- `@Deprecated` annotations should include Javadoc pointing to the replacement.
- Mark experimental APIs with `@Beta` where appropriate.

## Branch Structure (Fork-Specific)

This repo is a fork of `matsim-org/matsim-libs`. The AI files in this fork must never reach upstream.

- **`main`** — pure mirror of `upstream/main`. Never commit here.
- **`dev`** — one commit ahead of `main` (the AI files commit). Base all feature branches from `dev`.
- **`feature/<name>`** — branch from `dev`; AI files are present during development.
- **`pr/<name>`** — temporary branch used only to push a clean PR to upstream (`git rebase --onto main dev pr/<name>` strips the AI files commit).

See `.github/instructions/contribution-hygiene.instructions.md` for the full step-by-step workflow.

## What Not to Do

- Do not add dependencies without checking the root `dependencyManagement` section first.
- Do not modify code outside the scope of your task — keep diffs minimal.
- Do not introduce new modules or restructure packages without discussion.
- Do not bypass Maven enforcer rules.
