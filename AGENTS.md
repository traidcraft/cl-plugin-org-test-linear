# CodeLantern project context

This project uses CodeLantern for agentic development workflows. This file is the
single source of instructions; it is read natively by Codex, Copilot, and other
AGENTS.md-aware agents, and by Claude Code via the one-line `CLAUDE.md` that
imports it (`@AGENTS.md`).

## Configuration

- **Repository:** `traidcraft/cl-plugin-org-test-linear`
- **Config file:** `.codelantern/config.json`
- **Knowledge base:** `.codelantern/knowledge-base/` (`project.md`, `conventions.md`, `recipes.md`, `learnings.md`, `glossary.md`, `adr/`, and `components.md` if the project has a UI)

## Skills

CodeLantern skills are activated by description — name the task and the matching
skill runs (on Claude Code you can also invoke it as a slash command, e.g.
`/spec`). The suite:

| Skill | What it does |
|-------|--------------|
| `spec` | Interview a request into a spec'd issue with acceptance criteria |
| `size` | Score an issue's complexity and set its Size field |
| `backlog` | Triage and prioritize open issues |
| `claim-issue` | Claim an issue: branch, placeholder plan, draft PR |
| `design-solution` | Produce/refine a solution design (re-runnable; grills it) |
| `reduce-complexity` | Interactive Ousterhout complexity grill on a design, plan, or code |
| `review-design` | Written structural + security + privacy review of a design or plan |
| `create-impl-plan` | Turn a claimed issue into a phased implementation plan |
| `approve-plan` | Approve a drafted plan so implementation can start |
| `implement` | Execute an approved plan phase by phase |
| `consolidate` | Update the knowledge base from the work (learnings, recipes, conventions, components) |
| `finalize` | Author the summary, assemble the PR body, mark it ready |
| `review-code` | Systematic review of a PR diff |
| `grill` | Challenge any subject — surface hidden assumptions and fuzzy terms |
| `set-context` | Load a work item's state to resume |
| `discover` | Populate the KB from a codebase scan |
| `extract` | Interview a developer to seed the KB |
| `define` | Sharpen a domain term into the glossary |
| `adr` | Record an architecture decision (gated) |
| `doctor` | Health-check the CodeLantern setup |
| `handoff` | Delegate a command to the cloud agent |
| `cl-init` | Initialize a repository for CodeLantern |

## Workflow

```
spec → claim-issue → [ design-solution → review-design / reduce-complexity ] → create-impl-plan → approve-plan → implement → consolidate → finalize → review-code → merge
```

Each workflow skill ends by telling you the next step. `design-solution` and the
two design checks are for larger or riskier work; small issues can go straight
from `claim-issue` to `create-impl-plan`. Seed a fresh repo with `discover`, then
`extract`.

## MCP servers

Configured in `.mcp.json` at the project root.

- **CodeLantern** — use for **all** issue, PR, and project-board operations
  (`get_issue`, `search_issues`, `update_issue`, `create_pull_request`,
  `add_pull_request_comment`, `get_execution_context`, …). Always use these;
  never fall back to another MCP for PM/SCM operations.
- **Context7** — library documentation (`resolve-library-id`, `get-library-docs`).

## Key files

| File | Purpose |
|------|---------|
| `.codelantern/config.json` | Project configuration (repo + PM/SCM providers) |
| `.codelantern/knowledge-base/` | Project context, conventions, recipes, learnings, glossary, ADRs |
| `.codelantern/work-items/` | Per-work-item artifacts: solution designs, impl-plans, sessions, decisions, summaries |
