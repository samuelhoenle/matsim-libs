---
description: "Pre-PR checklist: verify that staged changes meet MATSim contribution standards before submitting a pull request."
agent: "agent"
---
Review my current changes and verify contribution readiness for the MATSim repository.
Follow `.agents/skills/preparing-upstream-pr/SKILL.md`; the checklist below is its summary.

Check each item and report pass/fail with specifics:

1. **Conventional commit title**: Suggest a PR title in `<type>[scope]: <description>` format based on the changes, using only the types allowed by `.github/workflows/pr-title.yml` (no `style`).
2. **Diff scope**: Flag any files that appear unrelated to the main change.
3. **AI files guard**: Confirm that `AGENTS.md`, `.agents/`, `.claude/`, `.github/copilot-instructions.md`, `.github/instructions/`, and `.github/prompts/` are NOT included in the diff against `main` (`git diff --name-only main...pr/<name> -- AGENTS.md .agents/ .claude/ .github/copilot-instructions.md .github/instructions/ .github/prompts/` prints nothing). These must never appear in an upstream PR. If they are present, the PR branch was not prepared with `git rebase --onto main dev pr/<name>`.
4. **License headers**: Verify new Java files have the GPL v2+ MATSim license header.
5. **Test coverage**: Check that behavior changes are covered by tests. Flag untested public methods.
6. **API compatibility**: Flag any removed or signature-changed public methods that lack `@Deprecated` predecessors.
7. **Dependency changes**: If POM files changed, verify versions use root POM properties and enforcer rules are respected.
8. **Build verification**: Suggest the exact `mvn` commands to compile and test the affected modules.

Finish with a summary: ready to submit, or list the items that need attention.
