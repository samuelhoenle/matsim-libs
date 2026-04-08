---
name: git-committing
description: Use when staging files, writing commit messages or PR titles, or grouping codebase changes into Git commits
---

# Git Committing

## Overview
This skill governs how changes are grouped, staged, and committed. Commits must be atomic, semantic, and use "recipe-style" Conventional Commit messages written in the present tense.

Upstream (`matsim-org/matsim-libs`) **squash-merges** every PR: the PR title becomes the single commit on upstream `main`, and the individual commits on the PR branch are discarded. So the **PR title** is the message that matters most and is the one CI lints; commits on the branch are for reviewers and should still follow the same format.

## When to Use
- Determining which files or hunks to stage
- Splitting up large workspace changes into independent commits
- Generating and formatting a commit message or PR title

## Process: Staging & Committing

### 1. Assess Scope — One Commit or Multiple?

**Default assumption: all current changes belong to one commit.** You are usually working on a single task, and all modified files serve that task. Do NOT split changes per-file or per-module by default.

**Check for outliers:** Scan `git status` and `git diff --stat` for changes that clearly serve a *different* purpose than the main task. Signs of an outlier:
- A bug fix mixed with a feature addition
- An unrelated formatting or refactoring change
- Changes to a module with no connection to the task
- **AI files** (`AGENTS.md`, `.agents/`, `.claude/`, `.github/copilot-instructions.md`, `.github/instructions/`, `.github/prompts/`) — these never go into a feature commit. They belong in the fork-only AI-files commit on `dev` (see the `maintaining-ai-files` skill).

**If all changes serve the same task → one commit.** Proceed to step 2.

**If outliers exist → propose a split to the user.** Present:
- Which files go into which commit
- A draft commit message for each
- Ask for confirmation before executing any commit

If an outlier would make a good upstream PR on its own, say so — upstream prefers small, focused PRs over bundles.

### 2. Propose the Commit to the User

**Always present the plan before committing.** Show:
- The list of files to be staged (or "all changes" if everything goes together)
- The proposed commit message

Wait for the user's confirmation. Accept feedback on scope or message wording and adjust before committing. Only execute `git add` + `git commit` after approval. Never commit on `main` (pure mirror of upstream).

### 3. Write the Commit Message
Commit messages and PR titles follow the [Conventional Commits v1.0.0 specification](references/conventional-commits-v1.0.0.md) (bundled locally — see that file for the full spec), restricted to the types upstream's PR-title check (`.github/workflows/pr-title.yml`) accepts.

- **Format:** `<type>[optional scope]: <description>`
- **Allowed types (exactly these):** `feat`, `fix`, `docs`, `chore`, `refactor`, `test`, `ci`, `perf`, `build`, `revert`, `maintenance`. **`style` is not allowed** — use `chore` (or `refactor` if code structure changes) for formatting-only changes. (`.github/pull_request_template.md` still lists `style` and omits `maintenance`; the workflow is authoritative.)
- **Scope:** optional; use the module or contrib name as seen in upstream history, e.g. `core`, `drt`, `router`, `config`, `scoring`, `dsim`, `carriers`, `application`.
- **Breaking changes:** add `!` before the `:` (e.g. `feat(core)!: …`) and describe the migration in a `BREAKING CHANGE:` footer / the PR body.
- **One-line summary:** the description follows the colon and a space. Keep the first line under ~100 characters.
- **Present tense (CRITICAL):** imperative, present tense. Describe what happens if the commit is applied. The log should read like a recipe.
- **Optional body:** if the change cannot be summarised in one line, add a blank line and explain *why* (root cause for fixes), wrapped at ~72 characters.

## Good vs Bad Commit Messages

| Status | Example | Reason |
|--------|---------|--------|
| ❌ Bad | `Added caching to the router` | Missing type; past tense |
| ❌ Bad | `fix: Fixes NPE in facilities reader` | Third-person singular ("Fixes" instead of "fix") |
| ❌ Bad | `feat :add caching` | Space before colon; no space after colon |
| ❌ Bad | `style: reformat QSim engine` | `style` is rejected by the PR-title check |
| ✅ Good | `feat(drt): add rebalancing strategy option` | Correct type and scope, present tense |
| ✅ Good | `fix: do not rewrite ".xml" in the output path when writing output_config_initial.xml` | Present tense, says what changes |
| ✅ Good | `maintenance: remove deprecated config group accessors` | Allowed upstream type |
| ✅ Good | `refactor(core)!: replace String ids in scoring with Id<Person>` | Breaking change indicator |

## Red Flags — STOP and Correct
- **Splitting without reason:** you are about to create multiple commits but all changes serve the same task. One commit is correct.
- **Committing without approval:** you are about to run `git commit` without showing the user the proposed message and scope first. Present the plan.
- **Bundled unrelated changes:** the diff contains a bug fix *and* an unrelated feature. Flag the outlier and propose a split.
- **AI files in a feature commit:** stage them separately into the `dev` AI-files commit, never into feature or `pr/` branches.
- **Past tense:** your message contains "-ed" verbs ("removed", "updated"). Rewrite to "remove", "update".
- **`style` type:** rewrite to `chore` or `refactor`.
