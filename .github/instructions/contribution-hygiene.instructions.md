---
description: "Use when preparing code for a pull request, reviewing contribution readiness, writing commit messages, or ensuring changes are mergeable upstream. Covers conventional commits, diff hygiene, PR templates, open-source contribution rules, and the fork branch workflow."
---
# Contribution and PR Hygiene

<!-- Scope: PR preparation and contribution review tasks. Loaded on-demand by Copilot via description matching. Other agents: read this file before submitting or reviewing pull requests. -->

## Fork Branch Structure

This repository is a fork of `matsim-org/matsim-libs`. The AI instruction files (this file and its siblings) must never be submitted to upstream. This is maintained using two permanent branches:

- **`main`** — pure mirror of `upstream/main`. Never commit directly to this branch.
- **`dev`** — always exactly one commit ahead of `main`, containing only the AI instruction files commit. This is the base for all feature work.

Feature branches are created from `dev`, not from `main`.

## Day-to-Day Workflow

### Start a new feature
```bash
git checkout -b feature/<name> dev
```

### Sync after upstream merges (replaces using GitHub's Sync Fork button on main)
```bash
# 1. Sync fork's main via GitHub UI ("Sync fork" button), then:
git fetch origin
git checkout main && git merge --ff-only origin/main   # fast-forward only; safe since main is a pure mirror
git rebase main dev                                     # rebase the AI files commit onto new main
git rebase dev feature/<name>                           # rebase feature work on top
```

### Open a PR to upstream
Upstream squash-merges all PRs — individual working commits inside the PR are fine and help reviewers.

```bash
# 1. Create a clean PR branch without the AI files commit:
git checkout -b pr/<name>
git rebase --onto main dev pr/<name>
# (This replays only commits above dev, leaving the AI files commit behind)

# 2. Push and open PR as a Draft first:
git push origin pr/<name>
# Open PR: fork:pr/<name> → upstream:main
# Before converting Draft → Ready: verify mvn compile test-compile -pl <module> passes locally.
```

### During review
- Add fix-up commits un-squashed so reviewers can diff each iteration.
- After any force-push (rebase during review), leave a short comment: *"rebased onto current main"* or *"rebased + addressed #N"*.
- Upstream will squash-merge; individual commits do not land in upstream history.

### After PR is merged upstream
```bash
# Sync fork's main via GitHub UI, then:
git fetch origin
git checkout main && git merge --ff-only origin/main
git rebase main dev
git branch -d feature/<name>
git push origin --delete pr/<name>
```

## What Must Never Appear in an Upstream PR

- `AGENTS.md`
- `.github/copilot-instructions.md`
- `.github/instructions/*.instructions.md`
- `.github/prompts/*.prompt.md`

When preparing a PR branch with `git rebase --onto main dev`, these files are automatically excluded because they live only in the `dev` base commit. Verify with `git diff main...pr/<name> -- AGENTS.md .github/` before pushing.

## Conventional Commits

PR titles and commit messages **must** follow [Conventional Commits](https://www.conventionalcommits.org/):
```
<type>[optional scope]: <description>
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`

Examples:
- `fix(QSim): Correctly calculate travel times on links`
- `feat(drt): Add rebalancing strategy option`
- `refactor(core): Simplify event handling pipeline`

For breaking changes, add `BREAKING CHANGE` in the PR body.

## Diff Hygiene

- Touch only files relevant to the change.
- Do not mix formatting/style changes with functional changes.
- Do not reformat files you did not functionally modify.
- Keep the diff as small as possible — reviewers trust small, focused changes.
- If a refactor is needed to enable a feature, split it into a separate commit.

## Backward Compatibility

- Preserve public method signatures unless explicitly approved to break them.
- Deprecate before removing — provide at least one release cycle.
- If a breaking change is unavoidable, document migration steps in the PR body.
- Ensure existing tests still pass after your change.

## Test Requirements

- Every behavior change must be covered by a test.
- New features need at least one happy-path and one edge-case test.
- Bug fixes need a regression test that fails without the fix.
- Run the module's full test suite before submitting: `mvn verify -pl <module>`

## Documentation

- Update Javadoc for changed public APIs.
- Update README files if user-facing behavior changes.
- Note changelog-relevant changes in the PR body (CI generates changelogs from conventional commits).

## Quality Checklist (Before Submitting)

1. PR title follows conventional commit format.
2. All modified modules compile: `mvn compile -pl <module>`
3. All tests pass: `mvn verify -pl <module> -Dmaven.test.redirectTestOutputToFile`
4. Maven enforcer passes: `mvn validate`
5. No unrelated file changes in the diff.
6. New files include the GPL v2+ license header.
7. No new compiler warnings introduced.

## Avoid

- Force-pushing over reviewed commits without discussion.
- Submitting generated code (IDE artifacts, build outputs) in the diff.
- Adding new dependencies without checking the root POM first.
- Making changes to `package-info.java` files marked with maintainer restrictions without approval.
