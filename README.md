# eflip-power-pages — Claude Code Plugin

A Claude Code plugin that provides QA automation and testing plan generation for **Power Pages** portals (React/Vite SPA on Power Pages).

## What's included

| Artifact | Name | Purpose |
|---|---|---|
| Agent | `eflip-PowerPages-QA` | Executes QA checks against a testing plan — browser automation, network log verification, regression checks |
| Skill | `power-pages-create-testing-plan` | Generates a comprehensive testing plan (UAT + UX/UI + Unit Testing) by launching 4 agents in parallel |

## Install

```bash
# Add this repo as a marketplace
claude plugin marketplace add https://github.com/donaldngongang1/eflip-power-pages

# Install the plugin
claude plugin install eflip-power-pages@eflip-power-pages
```

## Usage

### Run QA on your portal
```
"run QA"
"post-deploy check"
"test the ticket flow"
"check network logs"
"regression check"
"did I break anything"
```
→ The `eflip-PowerPages-QA` agent activates, reads the testing plan, and executes targeted checks.

### Generate a testing plan for a new project
```
"create testing plan"
"generate QA docs"
"set up testing for my portal"
```
→ The `power-pages-create-testing-plan` skill activates, reads your project structure, launches 4 specialized agents in parallel, and synthesizes a combined plan file at `~/.claude/plans/{project-slug}-testing-plan.md`.

## How the QA agent works

1. **Finds the testing plan** — searches `.claude/plans/` for `*testing-plan*.md`
2. **If no plan** — auto-invokes `power-pages-create-testing-plan` to bootstrap one
3. **Maps your request** to the right plan section (UAT / UX/UI / Unit Testing / Regression)
4. **Executes QA** using Playwright MCP tools + code-level grep/read checks
5. **Reports** Pass ✓ / Fail ✗ / Skipped with evidence (network logs, screenshots, code locations)

## How the testing plan skill works

Launches 4 agents **in parallel**:
- **Explore** — researches Power Pages testing best practices for the specific project
- **UAT Plan** — writes numbered test cases with DevTools network log verification steps
- **UX/UI Plan** — covers design tokens, WCAG AA contrast, responsive breakpoints, Playwright scripts
- **Unit Testing Plan** — Vitest + RTL + MSW setup with concrete test code for critical paths

## Requirements

- Claude Code with Playwright MCP (`plugin:power-pages:playwright`)
- Power Pages project using React/Vite SPA pattern
- `powerPagesApi.ts` for OData/anti-forgery token patterns

## License

MIT
