---
description: "Use when writing or modifying test classes in MATSim. Covers JUnit Jupiter patterns, MatsimTestUtils, naming conventions, assertions, and test resource layout."
applyTo: "**/src/test/**/*.java"
---
# Test Guidelines

<!-- Scope: test source files (**/src/test/**/*.java). Loaded automatically by Copilot via applyTo. Other agents: read this file when creating or editing test classes. -->

## Framework

- **JUnit Jupiter** (JUnit 6.x) — use `org.junit.jupiter.api.*` imports.
- **AssertJ** for fluent assertions (`org.assertj.core.api.Assertions`).
- **Mockito** with `mockito-junit-jupiter` extension when mocking is needed.
- Do not use JUnit 4 (`org.junit.Test`, `@RunWith`) in new tests.

## MatsimTestUtils

Register as a JUnit 5 extension for per-test output directories and test resources:
```java
@RegisterExtension
private MatsimTestUtils utils = new MatsimTestUtils();
```

Key methods:
- `utils.getOutputDirectory()` — unique output dir for the current test method
- `utils.getInputDirectory()` — finds test resources at method, class, or package level
- `utils.createConfigWithInputResourcePathAsContext()` — creates a Config pointing to test resources
- `MatsimTestUtils.EPSILON` (`1e-10`) — use for floating-point comparisons

## Naming

- Unit tests: `*Test.java` (e.g., `DefaultPlansCacheTest.java`)
- Integration tests: `*IT.java` (e.g., `RoutingIT.java`) — run by `maven-failsafe-plugin`
- Test methods: descriptive names, optionally prefixed with `test` (no strict rule)

## Test Resources

Place test input files in:
```
src/test/resources/test/input/org/matsim/.../ClassName/methodName/
```

Hierarchy (MatsimTestUtils searches from most to least specific):
1. Method-level: `.../ClassName/methodName/`
2. Class-level: `.../ClassName/`
3. Package-level: `.../package/`

## Do

- Write at least one test for every new public method or behavior change.
- Use `@RegisterExtension` with `MatsimTestUtils` — it also resets `MatsimRandom`.
- Compare floating-point values with `MatsimTestUtils.EPSILON` or AssertJ's `isCloseTo()`.
- Keep tests independent — no shared mutable state between test methods.
- Use `@Disabled("reason")` instead of commenting out tests.
- Set `@Tag("slow")` on tests that take more than a few seconds.

## Avoid

- Writing tests that depend on execution order.
- Using `Thread.sleep()` — prefer latches or test-specific synchronization.
- Creating temporary files outside of `utils.getOutputDirectory()`.
- Testing implementation details — test observable behavior.
- Adding test-only production code (e.g., package-private accessors just for tests).

## Quality Gate

Before finishing test changes:
1. The test compiles: `mvn compile test-compile -pl <module>`
2. The test passes: `mvn verify -pl <module> -Dmaven.test.redirectTestOutputToFile`
3. No unrelated test files were modified.
