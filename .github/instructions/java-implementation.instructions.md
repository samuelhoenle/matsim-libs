---
description: "Use when writing or modifying Java source files in MATSim. Covers code style, license headers, Guice patterns, API conventions, and deprecation rules."
applyTo: "**/*.java"
---
# Java Implementation Guidelines

<!-- Scope: all *.java files. Loaded automatically by Copilot via applyTo. Other agents: read this file when creating or editing Java source files. -->

## License Header

Every new Java file must start with the MATSim GPL v2+ license header:
```java
/* *********************************************************************** *
 * project: org.matsim.*
 * ClassName.java
 *                                                                         *
 * *********************************************************************** *
 *                                                                         *
 * copyright       : (C) YYYY by the members listed in the COPYING,        *
 *                   LICENSE and WARRANTY file.                            *
 * email           : info at matsim dot org                                *
 *                                                                         *
 * *********************************************************************** *
 *                                                                         *
 *   This program is free software; you can redistribute it and/or modify  *
 *   it under the terms of the GNU General Public License as published by  *
 *   the Free Software Foundation; either version 2 of the License, or     *
 *   (at your option) any later version.                                   *
 *   See also COPYING, LICENSE and WARRANTY file                            *
 *                                                                         *
 * *********************************************************************** */
```
Replace `YYYY` with the current year and `ClassName.java` with the actual file name.
Do not add or modify license headers on existing files unless explicitly asked.

## Formatting and Code Style — Existing-File-First Rule

**When modifying an existing file, match the coding style already present in that file.** This takes priority over any general preference listed below. MATSim has no enforced repo-wide formatter yet, so individual files and modules may differ in brace placement, blank-line usage, comment style, `final` usage, `var` vs explicit types, etc. Consistency within a file matters more than cross-file uniformity.

Concretely:
- Read the surrounding 20–50 lines before writing new code. Mirror the patterns you see.
- Do not re-style existing code as part of a functional change — keep the diff focused.
- If a file mixes styles, follow the style of the immediate surrounding context.

Baseline rules that apply when creating **new files** or when the existing file has no clear convention:
- Use **tabs** for indentation (not spaces).
- Max **150 characters** per line.
- Final newline at end of file.
- Trim trailing whitespace.
- Prefer `var` for local variables when the type is obvious from the right-hand side.
- Use `final` on local variables and parameters only when it adds clarity, not as a blanket rule.
- Prefer early returns over deeply nested if-else chains.
- Keep methods short and focused.

> **Standing instruction for AI agents:** Periodically check the repository and recent commit history for the introduction of a consistent, enforced coding style (e.g., a formatter config such as `.editorconfig` changes, Spotless/Checkstyle plugin, or an `STYLE.md`). If one is found, update these agent instruction files to reference it instead of the baseline rules above.

## Dependency Injection (Guice)

- MATSim modules extend `org.matsim.core.controler.AbstractModule`, not `com.google.inject.AbstractModule` directly.
- Use `install()` to compose modules; use `bind()` for specific bindings.
- Avoid `@Singleton` unless the class is truly stateless or explicitly designed as a singleton.
- Follow existing module patterns in the same package.

## API and Deprecation

- Do not remove or change the signature of public methods without deprecation first.
- `@Deprecated` must include a Javadoc comment explaining the alternative:
  ```java
  /**
   * @deprecated Use {@link NewClass#newMethod()} instead.
   */
  @Deprecated
  public void oldMethod() { ... }
  ```
- Mark experimental APIs with `@Beta` and document that they may change.

## Do

- Follow the naming and package conventions of the surrounding code.
- Reuse existing utility classes (e.g., `IOUtils`, `ObjectAttributes`, `Id<T>`).
- Use Log4j 2 (`LogManager.getLogger()`) — not `System.out.println` or SLF4J directly.
- Use `Id<T>` typed identifiers for links, nodes, persons, vehicles — not raw strings.

## Avoid

- Adding new dependencies without checking root POM `dependencyManagement` and discussing.
- Importing `*` (wildcard imports).
- Broad refactors or style-only changes in files you are not otherwise modifying.
- Creating utility classes for one-off operations.
- Suppressing warnings without documented justification.
