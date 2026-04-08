---
name: writing-implementation-plans
description: Use when writing or finalising an implementation plan that will be handed off to an implementation agent.
---

# Writing Implementation Plans

Write so a less capable implementation model can implement without any clarifying questions. All conceptual decisions must be resolved before handoff — leave only mechanical implementation work.

## Required plan sections

Every plan must contain all of the following:

**1. Context** (2–4 sentences)
What is being built, why, and which issue (upstream issue/PR number) or user requirement it addresses.

**2. Files to change**
An explicit list of every file to create or modify, with exact repo-relative paths (e.g. `matsim/src/main/java/org/matsim/core/controler/ControllerListenerManagerImpl.java`).

**3. Per-file changes**
For each file: what exactly changes, where (class/method/line number if known), and the precise new behaviour. Do not write "update X" — write "in `ControllerListenerManagerImpl#fireControllerStartupEvent`, apply the `_initial` suffix to `DefaultFiles.config.getFilename()` and pass the result to `getOutputFilenameWithOutputPrefix`". New source files: state the package and that the GPL v2+ license header is required.

**4. Test plan**
For each new or changed behaviour: the test class (`*Test` or `*IT`), the test method name, what it asserts, and what resource files (if any) it needs under `src/test/resources/test/input/...`. If TDD is not applicable, state the reason explicitly.

**5. Step order**
Number the steps and note any dependency (e.g. "step 3 requires core to be installed before the contrib test runs"). Linear if no dependencies.

**6. Verification**
The exact `mvn` commands per affected module (`AGENTS.md § Build and Test`).

**7. Out of scope**
A short list of what this plan explicitly does NOT cover, so the implementer does not gold-plate or touch unrelated files.

## Quality checks before handing off

- No vague verbs: "update", "improve", "handle" — replace with exact operations.
- Every class, method, or config parameter referred to by name must exist in the codebase (verify with grep) or be explicitly marked as new.
- Public API changes: the plan names the deprecated predecessor and its `@Deprecated` Javadoc pointer to the replacement.
- Core must not gain a dependency on a contrib; new cross-contrib dependencies are declared in the contrib's own POM.
- New dependencies: confirm the version property exists in the root `dependencyManagement`.

---
Adapted (heavily condensed) from the `writing-plans` skill in [obra/superpowers](https://github.com/obra/superpowers), MIT licence.
