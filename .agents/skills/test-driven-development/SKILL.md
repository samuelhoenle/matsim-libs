---
name: test-driven-development
description: Use when implementing any feature or bugfix, before writing implementation code
---

# Test-Driven Development (TDD)

## Overview

Write the test first. Watch it fail. Write minimal code to pass.

**Core principle:** If you didn't watch the test fail, you don't know if it tests the right thing.

**Violating the letter of the rules is violating the spirit of the rules.**

## When to Use

**Always:** new features, bug fixes, refactoring, behaviour changes in `matsim/`, `contribs/`, `examples/`. Upstream also expects it: every behaviour change needs a test, every bug fix a regression test that fails without the fix (`.github/instructions/contribution-hygiene.instructions.md`).

**Exceptions (ask the user):** throwaway prototypes, generated code, POM/dependency bumps, documentation-only changes.

Thinking "skip TDD just this once"? Stop. That's rationalisation.

## The Iron Law

```
NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST
```

Write code before the test? Delete it. Start over. **No exceptions:** don't keep it as "reference", don't "adapt" it while writing tests, don't look at it. Delete means delete. Implement fresh from tests.

## The Cycle: RED → GREEN → REFACTOR

Test conventions (JUnit Jupiter, AssertJ, `MatsimTestUtils`, resource layout, `*Test` vs `*IT`): `.github/instructions/java-tests.instructions.md`.

### RED — Write One Failing Test

One minimal test showing what should happen. Requirements: tests ONE behaviour, name describes that behaviour, exercises real code — real `Config`, `Scenario`, `Network`, `Population` objects built with `ConfigUtils`/`ScenarioUtils`/`PopulationUtils`/`NetworkUtils`, not mocks.

Example — regression test for `output_config_initial.xml` being written to a wrong path when the output directory contains `.xml`:

```java
// matsim/src/test/java/org/matsim/core/controler/ControlerIT.java

@Test
void testInitialConfigWithXmlInOutputDirectory() {
	final Config config = utils.loadConfig(IOUtils.extendUrl(ExamplesUtils.getTestScenarioURL("equil"), "config.xml"));
	String outputDirectory = new File(utils.getOutputDirectory(), "run.xml").getPath();
	config.controller().setOutputDirectory(outputDirectory);
	config.controller().setLastIteration(0);
	new Controler(config).run();

	assertTrue(new File(outputDirectory, "output_config_initial.xml").exists());
}
```

Bad: `Controler c = mock(Controler.class); ...; verify(c).getConfig()` — tests the mock, not the behaviour.

Prefer the smallest scope that shows the behaviour: a unit test on the class beats a full `Controler` run. Only reach for a `Controler` run (`*IT`, `equil` scenario, `setLastIteration(0)`) when the behaviour only exists inside the control loop.

### Verify RED — Watch It Fail (MANDATORY, never skip)

```bash
# Unit test (surefire), core module
mvn test -pl matsim -Dtest='SomeClassTest#methodName' -Dsurefire.failIfNoSpecifiedTests=false

# Integration test (*IT, failsafe)
mvn verify -pl matsim -Dit.test='ControlerIT#testInitialConfigWithXmlInOutputDirectory' -Dtest=skip -Dsurefire.failIfNoSpecifiedTests=false

# Contrib: install core first (mvn install --also-make --projects matsim -DskipTests), then run with -pl contribs/<name>
```

Confirm: the test **fails** (not errors), for the expected reason (behaviour missing — `AssertionFailedError`, not a compile error, `NullPointerException` in setup or a missing test resource).

**Test passes?** You're testing existing behaviour — fix the test. **Test errors?** Fix the setup, re-run until it fails correctly.

### GREEN — Minimal Code

The simplest code that makes the test pass. Don't add parameters, config options, or refactoring beyond the test. Adding a new `ConfigGroup` parameter "in case someone needs it" before a test exists for it is over-engineering — one test at a time.

### Verify GREEN — Watch It Pass (MANDATORY)

Run the target test, then the module's suite for regressions (`AGENTS.md § Build and Test`). Confirm: target test passes, all other tests still pass, no new compiler warnings.

**Test fails?** Fix the code, not the test. **Other tests fail?** Fix now — don't accumulate failures.

### REFACTOR — After Green Only

Remove duplication, improve names, extract helpers. Keep tests green. Don't add behaviour. Then write the next failing test for the next behaviour.

## Good Tests

| Quality | Good | Bad |
|---------|------|-----|
| **Minimal** | One thing; "and" in the name? Split it. | `testRoutingAndScoringAndOutput` |
| **Clear** | Name describes behaviour | `test1`, `testWorks` |
| **Shows intent** | Demonstrates the desired API | Obscures what code should do |
| **Real code** | Real MATSim objects, `MatsimTestUtils.EPSILON` for doubles | Asserting a mock's call count |
| **Deterministic** | Relies on `MatsimTestUtils` resetting `MatsimRandom` | Depends on test order or wall-clock time |

## Stop and start over if:

| Thinking... | Reality |
|-------------|---------|
| "Too simple to test" | Simple code breaks. The test takes a minute. |
| "I'll test after" / "tests after achieve the same" | Tests written after code pass immediately — they prove nothing, test the implementation rather than the behaviour, and you never saw them catch anything. |
| "Already manually tested with a run" | Ad-hoc ≠ systematic. No record, can't re-run, upstream reviewers can't see it. |
| "Deleting work is wasteful" / "keep as reference" | Sunk cost. Unverified code is debt. You'll adapt it otherwise — delete means delete. |
| "Need to explore first" | Fine. Throw the exploration away, then start with TDD. |
| "Test is hard to write" / "needs a whole scenario" | Hard to test = hard to use. Simplify the design, or use the `equil` test scenario with 0 iterations. |
| "The build is too slow to run the test twice" | Run the single test with `-Dtest=`/`-Dit.test=`, not the module suite. |
| "This is different because…" / "just this once" | It isn't. Delete the code. Start over. |

## Example: Bug Fix

Bug: `output_config_initial.xml` missing when the output directory is named `run.xml`. **RED:** the test above → fails (file not found; startup aborted writing to `run_initial.xml/`). **GREEN:** apply the `_initial` suffix to the file name only, not to the full path → passes, module suite green. **REFACTOR:** drop the now-unused import.

## When Stuck

| Problem | Solution |
|---------|----------|
| Don't know how to test | Write the wished-for API; write the assertion first; ask the user. |
| Test too complicated | Design too complicated. Simplify the interface. |
| Must mock everything | Code too coupled. Use Guice/constructor injection. |
| Test setup is huge | Extract helpers or put inputs under `src/test/resources/test/input/...`. Still complex? Simplify the design. |

## Final Rule

```
Production code → test exists and failed first
Otherwise → not TDD
```

Before marking work complete: every new behaviour has a test; you watched each fail for the right reason; minimal code; module suite green; edge cases covered. Can't tick all boxes? You skipped TDD — start over. No exceptions without the user's explicit permission.

---
Adapted from the `test-driven-development` skill in [obra/superpowers](https://github.com/obra/superpowers), MIT licence.
