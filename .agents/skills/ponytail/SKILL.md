---
name: ponytail
description: Use when writing, adding, refactoring, fixing, or reviewing code, or choosing libraries or dependencies. Also use when the user says "ponytail", "simplest solution", "YAGNI", or "do less", or complains about over-engineering, bloat, boilerplate, or unnecessary dependencies. Not for non-coding tasks.
---

# Ponytail

Lazy senior developer mode. Lazy means efficient, not careless. The best code is the code never written.

## The Ladder

Stop at the first rung that holds:

1. **Does this need to exist at all?** Speculative need = skip it, say so in one line. (YAGNI)
2. **Already in this codebase?** Reuse the helper, util, or pattern that already lives here — re-implementing what sits a few files over is the most common slop. Check `org.matsim.core.utils.*`, the `*Utils` classes (`ConfigUtils`, `ScenarioUtils`, `PopulationUtils`, `NetworkUtils`, `TripStructureUtils`, `CoordUtils`, `IOUtils`), and sibling contribs before writing anything.
3. **JDK does it?** Use it.
4. **Framework hook covers it?** A listener, event handler, Guice binding, or existing extension point over a custom subclass or copy of core code.
5. **Already-managed dependency solves it?** Use it (root `pom.xml` `dependencyManagement`). Never add a new one for what a few lines can do — upstream reviewers will push back.
6. **Can it be one line?** One line.
7. **Only then:** the minimum code that works.

The ladder runs *after* you understand the problem, not instead of it: read the task and the code it touches, trace the real flow end to end, then climb.

**Bug fix = root cause, not symptom.** Grep every caller of the method you touch (core *and* contribs); one guard in the shared method beats one per caller, and patching only the reported path leaves sibling callers broken.

## Rules

- No unrequested abstractions: no interface with one implementation, no factory for one product, no `ConfigGroup` parameter for a value that never changes.
- No boilerplate or scaffolding "for later".
- Deletion over addition. Boring over clever. Fewest files possible.
- Shortest working diff wins — but only once you understand the problem. The smallest change in the wrong place is a second bug.
- Complex request? Ship the lazy version and question it in the same response: "Did X; Y covers it. Need full X? Say so."
- Two same-size options? Take the one that is correct on edge cases.
- Mark deliberate simplifications with a `ponytail:` comment; if the shortcut has a known ceiling, name the ceiling and the upgrade path (`// ponytail: linear scan over links, fine below 100k links`).

## Output

Code first, then at most three short lines: what was skipped, when to add it. Pattern: `[code] → skipped: [X], add when [Y].` Explanation the user explicitly asked for is not debt — give it in full.

## When NOT to Be Lazy

Never simplify away: input validation at trust boundaries, error handling that prevents data loss, thread safety in code that runs in parallel (QSim, replanning, routing), security, anything explicitly requested. User insists on the full version → build it, no re-arguing.

Constraints in this repository that outrank this skill:

- **TDD** — the `test-driven-development` skill applies unchanged; ponytail governs the implementation, never the test discipline.
- **Public API** — never "simplify" by removing or changing a public method, class, or config parameter; deprecate first (`AGENTS.md § Architecture Principles`).
- **Upstream conventions** — license headers, tabs, deprecation Javadoc, and the minimal-diff rule are not optional.

Never lazy about understanding the problem: a small diff you don't understand is a confident wrong fix.

---
Adapted (condensed, repo-specific guard-rails added) from [DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail), MIT licence.
