---
description: "Use when editing Maven POM files in MATSim. Covers dependency management, version properties, contrib structure, and enforcer rules."
applyTo: "**/pom.xml"
---
# Maven POM Guidelines

<!-- Scope: all pom.xml files. Loaded automatically by Copilot via applyTo. Other agents: read this file when creating or editing Maven POM files. -->

## Dependency Versions

- All shared dependency versions are defined as **properties** in the root `pom.xml`.
- Use `${property.version}` references — never hardcode versions in child POMs.
- Before adding a dependency, check if it already exists in the root `dependencyManagement` section.
- New dependencies to the root POM require explicit justification.

## Contrib POM Structure

A contrib POM inherits from the `contrib` parent:
```xml
<parent>
    <groupId>org.matsim</groupId>
    <artifactId>contrib</artifactId>
    <version>2026.0-SNAPSHOT</version>
</parent>
<groupId>org.matsim.contrib</groupId>
<artifactId>contrib-name</artifactId>
```

Cross-contrib dependencies use `${project.parent.version}`:
```xml
<dependency>
    <groupId>org.matsim.contrib</groupId>
    <artifactId>dvrp</artifactId>
    <version>${project.parent.version}</version>
</dependency>
```

## Enforcer Rules (Active)

The Maven enforcer plugin checks:
- **requireUpperBoundDeps** — transitive dependency versions must not exceed direct ones
- **banDuplicatePomDependencyVersions** — no duplicate version declarations
- **requireReleaseDeps** — only release deps allowed (except `org.matsim:*`)
- **requireMavenVersion** — Maven ≥ 3.8

Validate enforcer compliance: `mvn validate -pl <module>`

## Do

- Keep POM changes minimal — only what is needed for your task.
- Use the managed version from the root POM when possible.
- Add `<scope>test</scope>` for test-only dependencies.
- Sort dependencies logically: core matsim first, then contribs, then third-party.

## Avoid

- Overriding plugin versions in child POMs without strong reason.
- Adding repositories — use only the repositories declared in the root POM.
- Duplicating `dependencyManagement` entries that the root POM already provides.
- Adding `-SNAPSHOT` dependencies to third-party libraries.
