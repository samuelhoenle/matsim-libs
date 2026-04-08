---
name: preparing-upstream-pr
description: Use when preparing, checking, opening, or updating a pull request to matsim-org/matsim-libs, creating a pr/ branch, or checking contribution readiness of a feature branch
---

# Preparing an Upstream PR

## Overview

This repository is a fork of `matsim-org/matsim-libs`. Feature work happens on branches that contain the fork-only AI-files commit; upstream PRs are pushed from a clean `pr/<name>` branch that does not.

**Core principle:** nothing from the AI-files commit may ever reach upstream. Verify it, don't assume it.

```
main ──────────── (pure mirror of upstream/main, never commit here)
  └─ dev ──────── (+1 commit: AI files only)
       └─ feature/<name> or fix/<name>   (your work, AI files present)
main
  └─ pr/<name> ── (your work replayed onto main, no AI files) → upstream PR
```

AI files: `AGENTS.md`, `.agents/`, `.claude/`, `.github/copilot-instructions.md`, `.github/instructions/`, `.github/prompts/`.

## Workflow

### 1. Sync first

```bash
git status                                    # must be clean
git fetch upstream && git fetch origin
git rev-parse main upstream/main              # must be equal; else sync via GitHub "Sync fork" + `git merge --ff-only origin/main`
git log --oneline main..dev                   # exactly one commit: the AI-files commit
git rebase main dev                           # only if main moved
git rebase dev <feature-branch>               # only if dev moved; feature must sit on top of current dev
```

### 2. Readiness checklist (report pass/fail with specifics for each)

Run against the feature branch, using `dev` as base so the AI-files commit is excluded:

1. **PR title** — suggest one in `<type>[scope]: <description>` form; only the types allowed by `.github/workflows/pr-title.yml` (`git-committing` skill). No `style`.
2. **Diff scope** — `git diff --stat dev...<feature-branch>`. Flag files unrelated to the main change, formatting-only hunks in files not functionally modified, IDE artefacts, build outputs.
3. **License headers** — every new `.java` file carries the GPL v2+ MATSim header (`git diff --name-only --diff-filter=A dev...<feature-branch> -- '*.java'`).
4. **Test coverage** — every behaviour change has a test; bug fixes have a regression test that fails without the fix (`test-driven-development` skill). Flag untested public methods.
5. **API compatibility** — flag removed or signature-changed public methods, classes, or config parameters lacking a `@Deprecated` predecessor with Javadoc pointing to the replacement.
6. **Dependencies** — if POMs changed: versions come from root properties, no duplicated versions in child POMs, `mvn validate` (enforcer) passes.
7. **Build** — for each affected module, run (core first if it changed):
   ```bash
   mvn install --also-make --projects matsim -DskipTests -Dsource.skip     # only if core changed or a contrib is tested
   cd <module> && mvn verify --batch-mode -Dmaven.test.redirectTestOutputToFile -Dmatsim.preferLocalDtds=true --fail-at-end -Dsource.skip
   ```
   Report the actual result — do not claim it passes without running it.
8. **Code review** — run the `code-review` skill on `dev..<feature-branch>` if not done yet.

Finish with: **ready to submit**, or the list of items that need attention. Stop here if anything fails.

### 3. Create the clean PR branch

```bash
git checkout -b pr/<name> <feature-branch>
git rebase --onto main dev pr/<name>          # replays only the commits above dev
```

### 4. AI-files guard (MANDATORY, before every push)

```bash
git log --oneline main..pr/<name>             # only your commits; no "add AI coding agent instruction files"
git diff --name-only main...pr/<name> -- AGENTS.md .agents/ .claude/ .github/copilot-instructions.md .github/instructions/ .github/prompts/
```

The second command **must print nothing**. If it prints anything:
- STOP. Do not push.
- Find the offending commit (`git log --oneline main..pr/<name> -- <path>`), move the AI-file change into the `dev` AI-files commit (`maintaining-ai-files` skill), drop it from the feature branch, and recreate `pr/<name>`.

Also re-check that the PR branch still compiles on its own: `mvn compile test-compile -pl <module>` (feature code must not depend on anything from the AI-files commit).

### 5. Push and open as Draft

Only with the user's explicit go-ahead:

```bash
git push -u origin pr/<name>
gh pr create --repo matsim-org/matsim-libs --base main --head samuelhoenle:pr/<name> --draft \
  --title "<type>[scope]: <description>" --body-file <body.md>
```

PR body: what and why, root cause for fixes, behaviour changes and migration notes, `BREAKING CHANGE` at the bottom if applicable (`.github/pull_request_template.md`). Convert Draft → Ready only after CI is green.

### 6. During review

- Make changes on the feature branch, then cherry-pick the new commits onto `pr/<name>` (`git cherry-pick <sha>…` — `pr/` no longer contains `dev`, so `rebase --onto` cannot be reused), rerun the guard (step 4), and push.
- Add fixes as new commits; upstream squash-merges, so they do not clutter upstream history.
- After any force-push (e.g. rebase onto newer `main`), rerun the guard and leave a short PR comment: *"rebased onto current main"*.

### 7. After merge

```bash
# sync fork's main via GitHub "Sync fork", then:
git fetch origin && git checkout main && git merge --ff-only origin/main
git rebase main dev
git branch -d <feature-branch> pr/<name>
git push origin --delete pr/<name>
```

Pushing the rebased `dev` needs `--force-with-lease` — ask the user first.

## Red Flags — STOP

| Thinking... | Reality |
|-------------|---------|
| "`git rebase --onto` always strips the AI files" | Only if they live solely in the `dev` commit. Run the guard. |
| "I'll branch `pr/` straight from the feature branch and push" | That pushes the AI-files commit upstream. Always `rebase --onto main dev`. |
| "Tests passed on the feature branch, so `pr/` is fine" | Different base. Compile `pr/` on its own. |
| "Small PR, skip the checklist" | Upstream reviewers will find what you skipped. |
| "Push now, open the PR later" | Push only with explicit user approval. |
