---
name: writing-skills
description: Use when creating new skills, editing existing skills, or verifying skills work before deployment
---

# Writing Skills

## Overview

**Writing skills IS Test-Driven Development applied to process documentation.**

Skills live in `.agents/skills/<skill-name>/SKILL.md` in this repository (`.claude/skills` is a symlink to it). You write test cases (pressure scenarios with subagents), watch them fail (baseline behaviour), write the skill, watch them pass (agents comply), and refactor (close loopholes).

**Core principle:** If you didn't watch an agent fail without the skill, you don't know if the skill teaches the right thing.

**REQUIRED BACKGROUND:** the `test-driven-development` skill — this skill adapts its RED-GREEN-REFACTOR cycle to documentation.

## What is a Skill?

A reference guide for proven techniques, patterns, or tools — reusable across sessions.

**Create when:** the technique wasn't intuitively obvious; you'd reference it again; the pattern applies broadly; others working on the project would benefit.

**Don't create for:** one-off solutions; narratives about how you solved a problem once; standard practice well-documented elsewhere; project-specific conventions (put those in `AGENTS.md` or `.github/instructions/*.instructions.md`); anything enforceable by a linter or test — automate those instead. Save skills for judgement calls.

## Directory Structure

Flat namespace: `.agents/skills/<skill-name>/SKILL.md`, plus supporting files ONLY for heavy reference (100+ lines) or reusable scripts. Keep principles, concepts, and short code patterns (< 50 lines) inline.

## SKILL.md Structure

Frontmatter has exactly two fields: `name` (letters, numbers, hyphens only) and `description`.

```markdown
---
name: skill-name-with-hyphens
description: Use when [specific triggering conditions and symptoms]
---

# Skill Name

## Overview — what is this? Core principle in 1–2 sentences.
## When to Use — symptoms and use cases; when NOT to use.
## Core Pattern / Quick Reference — before/after or scannable table.
## Implementation — inline code for simple patterns.
## Common Mistakes — what goes wrong + fixes.
```

## Claude Search Optimisation (CSO)

The skill must be FOUND before it can be used.

**1. Description = when to use, NOT what the skill does.** If the description summarises the workflow, agents follow the description instead of reading the skill body (a description saying "code review between tasks" caused a single review where the body required two). Start with "Use when…", write in third person, stay under 500 characters, describe triggering conditions only.

```yaml
# ❌ BAD: summarises workflow
description: Use for TDD — write test first, watch it fail, write minimal code
# ✅ GOOD: triggering conditions only
description: Use when implementing any feature or bugfix, before writing implementation code
```

**2. Keyword coverage:** words an agent would search for — error messages (`CreationException`, `ConcurrentModificationException`), symptoms ("flaky test", "enforcer failure"), synonyms (fix/debug/troubleshoot), tools (`mvn`, JUnit, Guice, MATSim).

**3. Verb-first, active names:** `systematic-debugging` not `debugging-guide`; gerunds work well for processes.

**4. Token efficiency:** frequently-loaded skills < 300 words; all others < 1000 words. Move heavy reference to supporting files. Cross-reference other skills by name with explicit markers (`**REQUIRED SUB-SKILL:** Use the \`test-driven-development\` skill`) — never by file path.

**5. Flowcharts and examples:** flowcharts only for genuinely non-obvious decisions or loops agents exit too early — never for reference material or linear instructions. One excellent, runnable example in the project's most relevant language beats many mediocre ones.

## Authoring Guidelines

**REQUIRED READING when creating a skill:** `anthropic-best-practices.md` (in this directory) — Anthropic's skill-authoring guide. Key rules applied here:

- **Degrees of freedom:** match specificity to fragility — exact commands for fragile sequences (low freedom), heuristics where many approaches work (high freedom).
- **Progressive disclosure:** keep SKILL.md under 500 lines; reference supporting files one level deep only; give reference files over 100 lines a table of contents.
- **No time-sensitive content** — use an "old patterns" section instead of dates that go stale.
- **Consistent terminology**; offer one default approach with an escape hatch, not a menu of options.
- **Bundled scripts:** handle errors in the script rather than punting to the agent, justify every constant, and state whether the agent should *run* or *read* each script.

House deviation: our `description` fields are trigger-only (see CSO above), stricter than the upstream "what + when" guidance — workflow-summarising descriptions caused agents to skip the skill body.

## The Iron Law

```
NO SKILL WITHOUT A FAILING TEST FIRST
```

Write a skill before testing? Delete it. Start over. Edit a skill without re-testing? Same violation.

## RED-GREEN-REFACTOR

**RED — baseline.** Run a pressure scenario with a subagent WITHOUT the skill. Document exact behaviour: choices made, rationalisations used (verbatim), which pressures triggered violations. Example pressure for `test-driven-development`:

> "Add a `maxDistance` option to the teleportation router. We're behind deadline — just do it as quickly as possible."

Baseline behaviour: implementation first, test written after (passes immediately, proves nothing). Document it.

**GREEN — minimal skill.** Write the skill addressing those specific rationalisations — no content for hypothetical cases. Re-run the same scenario WITH the skill; verify compliance.

**REFACTOR — close loopholes.** New rationalisation appears? Add an explicit counter. Re-test until bulletproof.

## Bulletproofing Discipline Skills

- **Close every loophole explicitly.** Weak: "Delete it." Strong: "Delete it. Start over. Don't keep it as 'reference'. Don't 'adapt' it while writing tests. Delete means delete."
- **Add early:** *"Violating the letter of the rules is violating the spirit of the rules."*
- **Build a rationalisation table** from every excuse observed in baseline testing (`| Thinking... | Reality |`), plus a short "Red Flags — STOP" list for high-stakes disciplines.

## Checklist

**RED:** pressure scenarios created (3+ combined pressures for discipline skills) → run WITHOUT skill → rationalisations documented verbatim.
**GREEN:** valid frontmatter; description = "Use when…" triggers only; keywords present; addresses the observed baseline failures; one excellent example; verified compliant WITH the skill.
**REFACTOR:** new rationalisations countered; rationalisation table built; red-flags list added; re-tested.
**Quality:** no narrative storytelling; flowchart only if decision non-obvious; supporting files only for tools or heavy reference; within word target.
**Deployment:** update the `AGENTS.md` skills index; add a maintenance-table entry if the skill introduces one; amend the fork-only AI-files commit on `dev` (`maintaining-ai-files` skill) — skills must never land in a feature or `pr/` commit.

---
Adapted from the `writing-skills` skill in [obra/superpowers](https://github.com/obra/superpowers), MIT licence.
