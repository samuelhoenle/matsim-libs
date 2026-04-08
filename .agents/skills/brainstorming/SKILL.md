---
name: brainstorming
description: >
  Use before implementing any new feature, config option, contrib extension, public API change,
  or agent/skill customisation. Also use when asked to brainstorm, plan, or think through an idea before coding.
---

# Brainstorming

<HARD-GATE>
Do NOT write code, create files, run builds, or take any implementation action until the user has explicitly approved a design. No exceptions, regardless of apparent simplicity — in a framework used by many downstream projects, a wrong default or an unplanned API change is expensive to undo once merged upstream.
</HARD-GATE>

Precedence: this gate outranks the `ponytail` skill; once the design is approved, `ponytail` governs the implementation.

## Process (execute in order, no skipping)

**1. Read context** — the code the change touches and its callers (core *and* contribs), the relevant module/contrib `README.md`, related upstream issues/PRs if mentioned, `git log --oneline -10 -- <path>`.

**2. Classify domain** and read the corresponding instruction file before proposing anything:

| Domain | Read before proposing |
|--------|----------------------|
| Java production code (core or contrib) | `.github/instructions/java-implementation.instructions.md` |
| Tests | `.github/instructions/java-tests.instructions.md` |
| POM / dependencies / new contrib | `.github/instructions/maven-pom.instructions.md` |
| Anything heading for an upstream PR | `.github/instructions/contribution-hygiene.instructions.md` |
| Agent / skill change | `maintaining-ai-files` and `writing-skills` skills |

If the request spans domains or would make a large upstream PR, flag it and decompose into sequential sub-tasks — each gets its own design→approval cycle and ideally its own PR.

**3. Ask clarifying questions** — one at a time, multiple-choice preferred. Cover: purpose, constraints, success criteria, scope boundary, whether it is meant for upstream or fork-only.

**4. Propose 2–3 approaches** with trade-offs and a recommendation. Project-specific axes: core vs contrib; new `ConfigGroup` parameter vs Guice binding/override; extend an existing extension point vs new interface; default behaviour unchanged (opt-in) vs changed default; additive API vs deprecate-and-replace.

**5. Present design section by section** — get approval after each section before continuing. Cover as applicable:

| Domain | Required design sections |
|--------|--------------------------|
| Core / contrib feature | Affected packages and classes; public API additions/changes (+ deprecations); Guice wiring (`AbstractModule`, bindings, overrides); thread-safety (QSim/replanning/routing threads); test classes and coverage |
| Config option | `ConfigGroup` and parameter name, type, default, `@StringSetter`/`@StringGetter` and comment text; behaviour when unset; consistency checks in `checkConsistency` |
| Bug fix | Root cause (`systematic-debugging` skill); minimal fix location; regression test |
| Dependency / POM | Version property in root `dependencyManagement`; enforcer impact; affected modules |
| Agent / skill change | Trigger phrases; tools needed; domain knowledge encoded; `AGENTS.md` skills index / maintenance table entries affected |

**6. Save design doc** — write to a session temp directory (or the scratchpad) so it persists across checkpoints. Do not commit design docs to feature branches — they would end up in the upstream PR.

**7. Self-review the spec:**
- No "TBD" or vague requirements — fill in or flag explicitly
- Respects `AGENTS.md § Architecture Principles`: core never depends on contribs, deprecation-first for public API, `@Beta` for experimental API
- Default behaviour unchanged unless *explicitly agreed BY THE HUMAN USER*
- Single coherent task — if it has grown into multiple independent changes, flag for decomposition
- No ambiguous requirement — pick one interpretation and make it explicit

**8. Present spec path to user.** Wait for explicit approval. Update and repeat self-review if changes are requested. Only proceed once approved.

**9. Transition to implementation:**
- Create the branch from `dev`: `git checkout -b feature/<name> dev` (or `fix/<name>`).
- Small, well-defined tasks: proceed directly, state which file you'll edit first and why.
- Larger tasks: build a structured implementation plan first (`writing-implementation-plans` skill).
- Implement with the `test-driven-development` skill; validate with `AGENTS.md § Build and Test`.

## Key principles

- One question at a time — multiple questions signal unclear thinking.
- YAGNI ruthlessly — simplest change that achieves the goal.
- No silent behaviour changes — if existing scenarios would produce different output, call it out and make it opt-in or document it for the PR body.
- Respect upstream — minimal diffs, backward compatibility, and conventions are what get a PR merged.

---
Adapted from the `brainstorming` skill in [obra/superpowers](https://github.com/obra/superpowers), MIT licence.
