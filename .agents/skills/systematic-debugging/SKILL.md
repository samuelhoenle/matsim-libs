---
name: systematic-debugging
description: Use when encountering any bug, test failure, build failure, or unexpected simulation behaviour, before proposing fixes
---

# Systematic Debugging

## Overview

Random fixes waste time and create new bugs.

**Core principle:** ALWAYS find the root cause before attempting fixes. Symptom fixes are failure.

**Violating the letter of this process is violating the spirit of debugging.**

## The Iron Law

```
NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST
```

If you haven't completed Phase 1, you cannot propose fixes.

## When to Use

ANY technical issue: unit or integration test failures, flaky tests, Maven build or enforcer failures, Guice injection errors, config loading errors, `Controler` aborting at startup, wrong scores or plans, nondeterministic output, CI failures on one matrix module only.

Use it ESPECIALLY under time pressure, when "one quick fix" seems obvious, when previous fixes didn't work, or when the issue "seems simple" — simple bugs have root causes too.

## The Four Phases (complete each before the next)

### Phase 1: Root Cause Investigation

1. **Read error messages completely.** Stack traces name the exact class and line; for Guice `CreationException`/`ProvisionException` read every listed error, not just the first; MATSim startup `WARN`s often reveal a mis-named or unregistered config parameter; the cause is usually not the last line.
2. **Reproduce consistently.** Run the single failing test (`-Dtest=Class#method` or `-Dit.test=Class#method`, see the `test-driven-development` skill). Test output is redirected to `target/surefire-reports/` or `target/failsafe-reports/` when `-Dmaven.test.redirectTestOutputToFile` is set — read the `*-output.txt`. Not reproducible → gather more data (seed, thread count, test order), don't guess.
3. **Check recent changes.** `git diff`, `git log --oneline -10`, `git log --oneline main..HEAD`. Did upstream `main` move (`git log --oneline <old-main>..main -- <path>`)? Dependency bump in the root `pom.xml`? Stale core jar in `~/.m2` while testing a contrib?
4. **Gather evidence at component boundaries.** The pipeline is: config XML → `ConfigUtils.loadConfig` (custom `ConfigGroup`s must be passed or added before reading) → `ScenarioUtils.loadScenario` (network, population, facilities, transit) → `Controler` + Guice module wiring (`AbstractModule.install`, bindings, overrides) → mobsim (QSim engines, departure handlers) → events → scoring / replanning → output writers. Instrument each boundary BEFORE proposing fixes: log the config after loading, count persons/links after scenario loading, print Guice bindings, attach a temporary `EventHandler`, run with `setLastIteration(0)` on the `equil` test scenario.

   This reveals which layer fails (config ✓, scenario ✓, injection ✗ = missing binding).
5. **Trace data flow.** Wrong value (score `NaN`, missing leg, empty output file)? Trace backwards from the observed value to where it was first wrong. Fix at the source, not the symptom.

### Phase 2: Pattern Analysis

- **Find a working example:** another contrib, test, or `examples/` scenario doing the same thing on the same code path.
- **Read the reference completely:** the `ConfigGroup` setters (`@StringSetter`), the Javadoc, and the code you call — not just the signature.
- **List every difference** between working and broken, including "that can't matter" ones.
- **Understand dependencies:** which `ConfigGroup`s must exist before loading, which Guice modules the `Controler` installs by default, which listeners run in which priority order, what is shared across QSim threads.

### Phase 3: Hypothesis and Testing

- Form ONE specific hypothesis and write it down: "X is the root cause because Y".
- Test it with the SMALLEST possible change — one variable at a time. Don't fix multiple things at once.
- Confirmed → Phase 4. Not confirmed → new hypothesis. Do NOT stack more fixes on top.
- Don't know? Say so, read the MATSim source further, check upstream issues/PRs, or ask for help.

### Phase 4: Implementation

1. **Create the simplest failing test case first** (`test-driven-development` skill).
2. **Implement a single fix** for the identified root cause — no "while I'm here" improvements, no bundled refactoring (upstream reviewers will ask you to split it anyway).
3. **Verify** with the module's suite (`AGENTS.md § Build and Test`).
4. **Fix didn't work? STOP.** Fewer than 3 attempts → return to Phase 1 with the new information. **3+ failed fixes → question the architecture:** fixes that reveal new coupling or create symptoms elsewhere indicate a structural problem — typically shared mutable state across QSim threads, a listener ordering assumption, or a binding overridden by another module. **Discuss with the user before attempting more fixes.**

## MATSim-Specific Debugging Patterns

| Symptom | Likely cause | Where to look |
|---------|-------------|---------------|
| `Could not find module` / unknown parameter warning | Module or param name mis-spelled, or `ConfigGroup` not passed to `ConfigUtils.loadConfig` | `ConfigGroup` name constant and `@StringSetter` names |
| Guice `CreationException`: no implementation bound | Module not installed, or binding only in a contrib module | `AbstractModule.install` chain, `Controler.addOverridingModule` |
| Guice: binding already configured | Same type bound in two modules | Use `addOverridingModule` or a `MapBinder`/`Multibinder` |
| `ConcurrentModificationException` / flaky results | Shared mutable state across QSim or replanning threads | `global.numberOfThreads`, `qsim.numberOfThreads`; make state per-thread or immutable |
| Results differ between runs | Unseeded randomness, `HashMap` iteration order, thread scheduling | `MatsimRandom`, `IdMap`/`LinkedHashMap`, thread counts in test config |
| Test passes alone, fails in suite | Static state leaking between tests | Statics, singletons; `MatsimTestUtils` resets `MatsimRandom` only |
| Contrib test sees old core behaviour | Stale `matsim` jar in `~/.m2` | `mvn install --also-make --projects matsim -DskipTests` |
| Run aborts at startup writing output | Output path manipulation, overwrite policy | `OutputDirectoryHierarchy`, `controller.overwriteFiles` |
| `NaN` / huge scores | Unit error (seconds vs hours), sign of a marginal utility | `ScoringConfigGroup` mode params, scoring function factories |
| Enforcer failure | Dependency upper-bound conflict or duplicate version | Root `dependencyManagement`; `mvn dependency:tree -pl <module>` |

## Stop and return to Phase 1 if:

| Thinking... | Reality |
|-------------|---------|
| "Quick fix, investigate later" / "Just try X" | First fix sets the pattern. Diagnose first. |
| "Multiple fixes at once saves time" | Can't isolate what worked. Creates new bugs. |
| "Just bump the tolerance / add a retry / `@Disabled` it" | Masks the bug. Find out why it fails. |
| "Issue is simple, no need for process" | Simple issues have root causes too. The process is fast. |
| "Emergency, no time for process" | Systematic debugging beats guess-and-check thrashing. |
| "I see the problem, let me fix it" | Seeing symptoms ≠ understanding the root cause. |
| "One more fix attempt" (after 3+ failures) | Architectural problem. Discuss before more fixes. |
| Each fix reveals a new problem elsewhere | Structural issue. STOP. Question the architecture. |

---
Adapted from the `systematic-debugging` skill in [obra/superpowers](https://github.com/obra/superpowers), MIT licence.
