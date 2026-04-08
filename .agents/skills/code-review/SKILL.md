---
name: code-review
description: Use when requesting, performing, or receiving code review — covers dispatching a review, applying review criteria, and handling feedback
---

# Code Review

## Overview

Code review in this project prioritises **correctness over style**. Reviews only surface issues that genuinely matter: real bugs, logic errors, thread-safety problems, MATSim API misuse, backward-compatibility breaks, and security issues. Style, formatting, naming, and refactoring opportunities are out of scope unless they introduce a correctness risk.

## When to Use

**Requesting review (mandatory):**
- After implementing a feature or bugfix, before opening an upstream PR (`preparing-upstream-pr` skill)
- After any change to a public API, a `ConfigGroup`, or code that runs in parallel (QSim, replanning, routing)

**Requesting review (optional):**
- When stuck (fresh perspective)
- Before refactoring (baseline check)

**Receiving review:**
- When feedback arrives on an upstream PR or from a subagent review
- Especially when feedback seems unclear or technically questionable

## Review Criteria — ONLY These

**Correctness bugs:**
- Off-by-one errors in iteration, time bins, or array indexing
- Wrong units (seconds vs hours, metres vs kilometres, utils/h vs utils/s)
- Undefined times (`OptionalTime.undefined()`, or `Double.NEGATIVE_INFINITY` in older code) treated as a real time
- String manipulation of file paths that also hits directory names (e.g. `String.replace(".xml", …)` on a full path)

**MATSim API misuse:**
- Custom `ConfigGroup` added after the config file was read, so its values are silently ignored
- Using `config.getModules().get(name)` and casting instead of `ConfigUtils.addOrGetModule(config, X.class)`
- Guice bindings that silently override a core binding instead of `addOverridingModule`, or duplicate `Multibinder` entries
- Mutating plans, routes, or the network from inside a mobsim/event handler where other threads read them
- Core code depending on a contrib

**Thread safety and determinism:**
- Shared mutable state in classes instantiated once but used by several QSim, replanning, or routing threads
- Iteration over `HashMap`/`HashSet` where order affects output (use `IdMap`, `LinkedHashMap`, or sort)
- Randomness not drawn from `MatsimRandom` (or a seeded local `Random`)

**Resource and data handling:**
- Readers/writers not closed (use try-with-resources; `IOUtils.getBufferedWriter` handles `.gz`)
- Output paths not obtained from `OutputDirectoryHierarchy` (`getControllerIO()`)
- Tests writing outside `utils.getOutputDirectory()`

**Numerical issues:**
- Integer division where a double is needed
- Comparing doubles with `==` (use `MatsimTestUtils.EPSILON` or a tolerance)
- Accumulating floating-point error in inner loops over large populations

**Backward compatibility:**
- Removed or changed public method, class, or config parameter without a `@Deprecated` predecessor pointing to the replacement
- Changed default behaviour without a config switch or PR-body note

**Security / data integrity:**
- Hardcoded absolute or machine-specific paths
- Credentials or tokens in config files or tests

**Do NOT comment on:** code style, formatting, variable naming (unless confusing enough to cause bugs), refactoring opportunities, missing Javadoc on non-public code, performance (unless O(n²)+ on population- or network-sized data).

## Requesting Review

### 1. Run tests first

Run the affected modules' suites (`AGENTS.md § Build and Test`) before requesting review.

### 2. Dispatch the Code Reviewer subagent

> *[Copilot]* Use `runSubagent` with `agentName: "Code Reviewer"`.  
> *[Claude Code]* Use the `Agent` tool with `subagent_type: "general-purpose"` (no dedicated reviewer agent exists in Claude Code).

Provide:

```
Review the changes between {BASE_SHA} and {HEAD_SHA}.

What was implemented: {DESCRIPTION}
Requirements: {PLAN_OR_REQUIREMENTS}

First read .agents/skills/code-review/SKILL.md and apply ONLY its review criteria.
Do NOT comment on style, formatting, or refactoring unless it introduces a correctness risk.
```

On a feature branch, `{BASE_SHA}` is `dev` — this excludes the fork-only AI-files commit from the review.

### 3. Act on feedback

- Fix **critical** issues immediately
- Fix **important** issues before proceeding
- Note **minor** issues for later
- Push back if the reviewer is wrong (see below)

## Receiving Review Feedback

**Response pattern:**

1. **Read** complete feedback without reacting
2. **Understand** — restate requirement in own words (or ask)
3. **Verify** — check against codebase reality
4. **Evaluate** — technically sound for THIS codebase?
5. **Respond** — technical acknowledgement or reasoned pushback
6. **Implement** — one item at a time, test each

**Forbidden responses:** "You're absolutely right!", "Great point!", "Let me implement that now" (before verification). Just start working — actions over words.

**If any item is unclear:** STOP. Do not implement anything yet. Ask for clarification on unclear items before proceeding — items may be related and partial understanding leads to wrong implementation.

### Handling external reviewer feedback

Before implementing, verify:
1. Technically correct for THIS codebase?
2. Breaks existing functionality (core *and* contribs that use it)?
3. Reason for current implementation (`git log -L`, `git blame`)?
4. Conflicts with the user's prior decisions?

**Project-specific verification:**
- Does the suggestion rename or remove a public method or config parameter? Deprecate first — never break users' configs silently.
- Does it widen the PR scope? Upstream prefers small focused PRs — propose a follow-up PR instead.
- On an upstream PR, add fixes as new commits (upstream squash-merges); do not force-push over reviewed commits without saying so in a comment.

---
Adapted from the `requesting-code-review` and `receiving-code-review` skills in [obra/superpowers](https://github.com/obra/superpowers), MIT licence.
