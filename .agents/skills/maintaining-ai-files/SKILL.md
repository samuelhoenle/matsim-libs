---
name: maintaining-ai-files
description: Use when modifying project structure, build commands, conventions, CI checks, or dependencies that AI agents rely on, or when editing AI files themselves, before completing the task
---

# Maintaining AI Files

## Overview
AI customisation files must always reflect the current state of the project. When you make or observe a change that affects the project's architecture, tools, or conventions, update the relevant AI files as part of the same task — do not leave it for later.

The AI files in this fork are:

- `AGENTS.md` — entry point, skills index, maintenance table
- `.github/copilot-instructions.md` — Copilot mirror of `AGENTS.md`
- `.github/instructions/*.instructions.md` — topic-specific rules
- `.github/prompts/*.prompt.md` — Copilot prompt files
- `.agents/skills/*/` — skills (`.claude/skills` is a symlink to `.agents/skills`)

## When to Use

- Changes to build or test commands, Java version, or Maven plugins
- Changes to upstream CI checks (e.g. allowed PR-title types in `.github/workflows/pr-title.yml`)
- Bumping core dependencies (Guice, JUnit, AssertJ, Mockito)
- Adding, renaming, or removing modules or contribs
- Changes to the fork branch workflow
- Adding, renaming, or removing a skill
- Experiencing agent hallucinations due to outdated context

**When NOT to use:**
- Localised logic changes that don't affect project structure or conventions.
- Minor bug fixes that don't change expected behaviour or workflows.

## Workflow: Trigger, Identify, Update, Amend

1. **Trigger:** you made or observed a structural or conventional change.
2. **Identify:** check the `AGENTS.md` maintenance table ("Trigger → file(s) to update").
3. **Update:** edit the target files before concluding the task. Keep `AGENTS.md` and `.github/copilot-instructions.md` in sync.
4. **Amend the AI-files commit on `dev`:** AI files live only in the single fork-only commit on top of `main`. Never commit them on a feature, `fix/` or `pr/` branch.

   ```bash
   git status                                   # clean working tree first
   git checkout dev
   # edit AI files
   git add AGENTS.md .agents .claude .github/copilot-instructions.md .github/instructions .github/prompts
   git commit --amend --no-edit
   git log --oneline main..dev                  # must show exactly one commit
   git diff --stat main dev                     # must list only AI files
   # rebase every local feature/fix branch onto the new dev:
   git rebase --onto dev <old-dev-sha> <branch>
   ```

   Pushing the amended `dev` (and rebased branches) needs `--force-with-lease` — only when the user asks.

## Rules for Updating

- **Immediate action:** update the relevant file as part of the same task.
- **Factual and verifiable:** descriptions must match the code. No aspirational, planned, or speculative content.
- **DRY:** do not duplicate content across files. Put details in the most specific file (instruction file or skill) and cross-reference from `AGENTS.md`.
- **Language:** all prose in British English.
- **Version accuracy:** cross-check versions against the root `pom.xml` (`maven.compiler.release`, dependency properties).

## Red Flags — STOP and Fix

- Completing a major change without updating agent instructions.
- Stating "I will update the documentation later." Update it *now*.
- Adding speculative functionality to agent instructions before it exists.
- AI files appearing in `git diff main...<feature-or-pr-branch>` — they leaked out of the `dev` commit.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Duplicating an instruction in `AGENTS.md` and an instruction file or skill | Keep it in the most specific file; link from `AGENTS.md`. |
| Updating `AGENTS.md` but not `.github/copilot-instructions.md` | Edit both in the same change. |
| Adding a skill without updating the `AGENTS.md` skills index | Add the index line in the same amend. |
| Committing AI files as a new commit on `dev` | `dev` must stay exactly one commit ahead of `main` — amend instead. |
| Forgetting to rebase feature branches after amending `dev` | `git rebase --onto dev <old-dev-sha> <branch>` for each. |
