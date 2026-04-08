# MATSim-Libs Project Guidelines

This file is the entry point for AI coding agents (Claude Code, Cursor, Windsurf, Aider, etc.).
The same rules are also in `.github/copilot-instructions.md` for GitHub Copilot.
Detailed, topic-specific rules live in `.github/instructions/` — read the relevant file when working on that type of code.

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

Run tests for a single module (run from within the module directory):
```
mvn verify --batch-mode -Dmaven.test.redirectTestOutputToFile -Dmatsim.preferLocalDtds=true --fail-at-end -Dsource.skip
```

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
- **GPL v2+ license header** required on all new source files
- **Conventional Commits** for PR titles: `<type>[optional scope]: <description>`
  - Types (enforced by `.github/workflows/pr-title.yml`): feat, fix, docs, chore, refactor, test, ci, perf, build, revert, maintenance — **not** `style`
  - Upstream squash-merges PRs, so the PR title becomes the upstream commit message
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

This repo is a fork of `matsim-org/matsim-libs`. The AI files in this fork (`AGENTS.md`, `.agents/`, `.claude/`, `.github/copilot-instructions.md`, `.github/instructions/`, `.github/prompts/`) must never reach upstream.

- **`main`** — pure mirror of `upstream/main`. Never commit here.
- **`dev`** — one commit ahead of `main` (the AI files commit). Base all feature branches from `dev`.
- **`feature/<name>`** — branch from `dev`; AI files are present during development.
- **`pr/<name>`** — temporary branch used only to push a clean PR to upstream (`git rebase --onto main dev pr/<name>` strips the AI files commit).

To change AI files, amend the single AI-files commit on `dev` and rebase feature branches onto it (`maintaining-ai-files` skill).

See `.github/instructions/contribution-hygiene.instructions.md` for the full step-by-step workflow and the `preparing-upstream-pr` skill for opening a PR.

## What Not to Do

- Do not add dependencies without checking the root `dependencyManagement` section first.
- Do not modify code outside the scope of your task — keep diffs minimal.
- Do not introduce new modules or restructure packages without discussion.
- Do not bypass Maven enforcer rules.

## Skills

Skills live in `.agents/skills/<name>/SKILL.md` (`.claude/skills` is a symlink to it). Read the skill before doing the task it covers.

| Skill | Use when |
|-------|----------|
| `brainstorming` | Before implementing a new feature, config option, API change, or skill — design approval gate |
| `writing-implementation-plans` | Writing a plan to hand off to an implementation agent |
| `test-driven-development` | Implementing any feature or bug fix, before writing production code |
| `systematic-debugging` | Any bug, test/build failure, or unexpected simulation behaviour, before proposing fixes |
| `ponytail` | Writing, refactoring, or reviewing code; choosing dependencies — simplest working solution |
| `dispatching-parallel-agents` | 2+ independent tasks (e.g. failures in unrelated modules) |
| `code-review` | Requesting, performing, or receiving code review |
| `git-committing` | Staging, grouping changes into commits, writing commit messages or PR titles |
| `preparing-upstream-pr` | Checking readiness, creating `pr/<name>`, opening or updating a PR to `matsim-org/matsim-libs` |
| `maintaining-ai-files` | Changes to structure, build, conventions or CI that agents rely on; editing AI files or skills |
| `writing-skills` | Creating or editing a skill |

## Maintaining AI Files

| Trigger | File(s) to update |
|---------|-------------------|
| Build/test commands, Java version, Maven plugins | `AGENTS.md`, `.github/copilot-instructions.md`, affected instruction files and skills (`test-driven-development`, `preparing-upstream-pr`) |
| Allowed PR-title types (`.github/workflows/pr-title.yml`) | `AGENTS.md`, `.github/copilot-instructions.md`, `contribution-hygiene.instructions.md`, `git-committing` skill |
| Test framework or `MatsimTestUtils` changes | `java-tests.instructions.md`, `test-driven-development` skill |
| Fork branch workflow or AI-file locations | `AGENTS.md`, `.github/copilot-instructions.md`, `contribution-hygiene.instructions.md`, `preparing-upstream-pr` and `maintaining-ai-files` skills, `.github/prompts/prepare-contribution.prompt.md` |
| Skill added, renamed, or removed | Skills table above, `.github/copilot-instructions.md` |

## Detailed Guidelines

Read these files when working on the corresponding code:

- `.github/instructions/java-implementation.instructions.md` — Java code style, license headers, Guice patterns, API conventions (applies to `**/*.java`)
- `.github/instructions/java-tests.instructions.md` — Test conventions, MatsimTestUtils, naming, assertions (applies to `**/src/test/**/*.java`)
- `.github/instructions/maven-pom.instructions.md` — POM editing, dependency management, contrib structure (applies to `**/pom.xml`)
- `.github/instructions/contribution-hygiene.instructions.md` — PR readiness, conventional commits, diff review, quality checklist
