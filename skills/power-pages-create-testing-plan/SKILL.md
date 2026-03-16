---
name: power-pages-create-testing-plan
description: |
  Generates a comprehensive testing plan for a Power Pages code site (SPA) by launching
  4 specialized agents in parallel (Explore + UAT + UX/UI + Unit Testing) and synthesizing
  their output into a single combined plan file.

  USE THIS SKILL whenever:
  - User asks to "create testing plan", "generate test plan", "make a QA plan", "create qa documentation"
  - The eflip-PowerPages-QA agent finds no testing plan file and needs to bootstrap one
  - Starting QA setup for a new Power Pages portal project for the first time
  - User says "set up testing for my portal", "prepare QA docs", "document my test cases"
  - Any Power Pages project that needs a UAT plan, UX/UI plan, or unit testing plan

  Do NOT use for: executing tests (use eflip-PowerPages-QA agent), deploying the site
  (use power-pages:deploy-site), or reviewing code (use code-review:code-review).
---

# Power Pages — Create Testing Plan

You are generating a comprehensive, multi-part testing plan for a Power Pages code site
by running 4 specialized research/planning agents **in parallel** and synthesizing their
output into a single authoritative plan file.

This is the same proven method used to produce the `assist-portal-testing-plan.md` —
follow it exactly.

---

## Step 1 — Read Project Context

Before doing anything else, gather project-specific facts. Read these files:

1. `package.json` — project name, deploy script (to find the site URL), scripts
2. `src/App.tsx` — all routes and auth guards (`RequireAuth` usage)
3. `src/shared/powerPagesApi.ts` — entity prefix, fetch patterns, error codes
4. All files in: `src/shared/services/`, `src/shared/hooks/`, `src/pages/`, `src/components/`
5. All files in: `.powerpages-site/table-permissions/` — scope, entity, web role info
6. `src/styles/theme.css` or equivalent — design tokens

Extract and note:
- **Project name** (from `package.json` → `name` field)
- **Site URL** (from deploy script in `package.json`, or README)
- **Entity prefix** (e.g. `cr69c_`, `new_`, found in service files)
- **Entity/table names** (from service files and table permission YAMLs)
- **Routes** (from `App.tsx`)
- **Auth-protected routes** (wrapped in `RequireAuth`)
- **Stack** (React version, router, icon library — from `package.json`)
- **Known issues** (look for comments like `// DEF-`, `// KNOWN ISSUE`, `// TODO`, `// WORKAROUND`)
- **Table permission scopes** (from YAML files: Self, Authenticated, Global, Anonymous)

---

## Step 2 — Determine Output Path

1. Slugify the project name: lowercase, replace spaces and special chars with `-`
2. Output path: `C:\Users\XavierNGONGANGACHOUN\.claude\plans\{project-slug}-testing-plan.md`
3. Check if the file already exists with `Glob`
4. If it exists: tell the user and ask "Regenerate? (yes/no)" before continuing
5. If not: proceed to Step 3

---

## Step 3 — Launch 4 Agents IN PARALLEL

Send a **single message** with all 4 `Agent` tool calls at once. All must have
`run_in_background: true`. Do not launch them sequentially.

Use the project context gathered in Step 1 to write specific, detailed prompts for each agent.
The more project-specific the prompts, the better the output.

### Agent 1 — Explore (subagent_type: "Explore")

Prompt template:
```
You are researching how to test the {PROJECT_NAME} Power Pages portal.

Read these project files:
- src/shared/powerPagesApi.ts
- src/shared/services/*.ts (all service files)
- src/shared/hooks/*.ts (all hook files)
- .powerpages-site/table-permissions/*.yml (all permission files)

Produce a structured summary covering:
1. OData endpoint patterns (entity set names, $select fields, $filter patterns)
2. Anti-forgery token behavior (TTL, fetch endpoint, retry on 403)
3. Known project-specific quirks (look for TODO/DEF/WORKAROUND comments)
4. Table permission scopes and what they allow (Self, Authenticated, Global, Anonymous)
5. MSW mock patterns for unit testing (what endpoints need mocking, response shapes)
6. Key things to verify in browser network logs (headers, response shape, $count usage)
7. Auth states to test (anonymous, authenticated, admin/elevated role)
8. Playwright testing patterns specific to Power Pages (token flow, OData response shape)
```

### Agent 2 — UAT Plan (subagent_type: "Plan")

Prompt template:
```
Write a detailed UAT (User Acceptance Testing) plan for the {PROJECT_NAME} Power Pages portal.

Project context:
- Site URL: {SITE_URL}
- Stack: {STACK}
- Routes: {ROUTES} (auth-protected: {AUTH_ROUTES})
- Tables: {ENTITY_NAMES}
- Known issues: {KNOWN_ISSUES}
- Table permission scopes: {PERMISSION_SCOPES}

Write test cases for every page/route. Each test case must include:
- Test Case ID (TC-A-001, TC-B-001, etc. — module letter per page)
- Description
- Preconditions
- Steps
- Expected Result
- Network Log Verification: exact URL pattern, HTTP method, status code, request headers
  to check (__RequestVerificationToken, OData headers), response shape (OData value array,
  @odata.count, etc.)
- Pass/Fail criteria

Also write:
- Auth & Permissions test cases (anonymous vs authenticated, cross-user isolation,
  anti-forgery token 403 retry, session expiry 401 no-retry)
- API Integration test cases (OData headers present, $count behavior, If-Match on PATCH,
  Location header ID extraction, exponential backoff)
- Post-Deploy Regression Checklist (numbered steps to verify after every PAC CLI deploy)
- Known Defects & Workarounds table
- UAT Sign-off Criteria (mandatory blocking vs non-blocking)

Be specific about browser DevTools Network tab verification: exact URL patterns,
which headers to check, exact response JSON shape.
```

### Agent 3 — UX/UI Plan (subagent_type: "Plan")

Prompt template:
```
Write a comprehensive UX/UI testing plan for the {PROJECT_NAME} Power Pages portal.

Project context:
- Site URL: {SITE_URL}
- Stack: {STACK}
- Routes: {ROUTES}
- Components: {COMPONENT_LIST}
- Design system: {DESIGN_TOKEN_NOTES}

Include all of:

1. UX Test Strategy: tools (Playwright, axe-core, DevTools), viewport sizes, auth states,
   Playwright config (playwright.config.ts with desktop/mobile/tablet projects + storageState
   for authenticated tests)

2. Visual/Design Checklist: design token compliance (CSS custom properties), typography,
   spacing, brand consistency, color contrast WCAG AA table (foreground/background/expected ratio)

3. Component-Level UX Tests: for each component — aria attributes, keyboard behavior,
   loading states, error states, visual variants

4. Page-Level UX Flows: for each route — step-by-step user journeys, auth state differences,
   responsive behavior at each breakpoint

5. Responsive Design Tests: breakpoint matrix table, specific overflow checks

6. Accessibility Checklist: ARIA landmarks, keyboard nav, screen reader, focus management,
   reduced-motion

7. Network-Aware UX Tests: loading states, error states, optimistic updates, revert behavior

8. Browser Network Log Verification Steps: per page — exact URL, headers to check
   (__RequestVerificationToken, OData-Version, Prefer), response shape, what to look
   for on mutating requests (If-Match, 204 vs 201, Location header)

9. Playwright Automation Scripts: concrete TypeScript code for:
   - Public page navigation tests
   - Responsive navbar breakpoint tests (hamburger toggle at exact breakpoint)
   - Network header verification (OData headers on all requests)
   - No horizontal overflow test (check scrollWidth > clientWidth)
   - StatusConfirmDialog or equivalent modal flow
   - File upload tests
   - Accessibility audit with @axe-core/playwright

10. Known UX Issues: document anything found while reading the code
```

### Agent 4 — Unit Testing Plan (subagent_type: "Plan")

Prompt template:
```
Write a full unit testing plan for the {PROJECT_NAME} Power Pages portal (React/TypeScript/Vite).

Project context:
- Stack: {STACK}
- API layer: src/shared/powerPagesApi.ts
- Services: {SERVICE_FILES}
- Hooks: {HOOK_FILES}
- Components: {COMPONENT_LIST}

The codebase has no existing tests. Recommend and plan for: Vitest + React Testing Library + MSW.

Include:

1. Test Setup & Toolchain
   - Package installation command
   - vitest.config.ts (using mergeConfig to extend vite.config.ts)
   - Global setup file (MSW server, token cache reset, portal user helper)
   - MSW server structure and handler files
   - Test factory files (one per entity)
   - Important: explain the module-level token cache leak issue and mitigation

2. Unit Tests: powerPagesApi.ts
   - Token caching (fresh, cached hit, expiry) using vi.useFakeTimers()
   - 403 retry: CONCRETE test code showing token is invalidated + refreshed + different token used
   - 429/5xx exponential backoff: CONCRETE test with vi.advanceTimersByTimeAsync
   - 401 no-retry: CONCRETE test verifying callCount === 1
   - fetchPage always adds $count=true
   - escapeODataString edge cases (empty, multiple quotes)
   - buildODataQuery skips undefined/empty
   - extractRecordId from OData-EntityId and Location headers
   - isPermissionError for all error codes + HTTP 403 catch-all
   - fetchCurrentContactId fallback chain: CONCRETE tests for each of the 3 paths + exhaustion

3. Unit Tests: Services
   - For EACH service file: list test cases covering CRUD operations
   - CRITICAL: test that timeline service (if present) does NOT use $count=true
     (look for a service that uses powerPagesFetch directly instead of fetchPage)
   - Annotation/file service: formatBytes pure function, upload base64 flow

4. Unit Tests: Hooks
   - Each hook: loading state, success state, error state, permission error message
   - Optimistic patch: CONCRETE test showing immediate UI update before await
   - Revert-on-failure: CONCRETE test showing refetch() restores server state

5. Unit Tests: Components
   - renderWithProviders wrapper with MemoryRouter + LanguageProvider (if applicable)
   - Each component: aria attributes, keyboard behavior, variant rendering

6. Integration Tests (full page + MSW)
   - Form submission flow: fill → submit → success state
   - Error flow: API failure → error banner shown
   - Auth-gated page: unauthenticated → SignInPrompt rendered

7. Coverage Targets table (per module: lines %, rationale)

8. CI Integration (GitHub Actions workflow YAML)

9. Known Tricky Areas: module-level state leaks, jsdom quirks, authService localhost check,
   any component-specific gotchas found in the code
```

---

## Step 4 — Wait for All 4 Agents

After launching, wait for all 4 background agents to complete.
Use `TaskOutput` with `block: true` for each agent ID.

Collect all 4 outputs before proceeding to synthesis.

---

## Step 5 — Synthesize the Combined Plan

Write the plan to the output path determined in Step 2.

Use this exact structure:

```markdown
# {PROJECT_NAME} — Comprehensive Testing Plan

**Site:** {SITE_URL}
**Stack:** {STACK}
**Date:** {TODAY}
**Covers:** UAT · UX/UI · Unit Testing · Browser Network Log Verification

---

## Quick Reference — Known Defects / Issues

| ID | Status | Description |
|---|---|---|
| ... | ... | ... |

---

# PART 1 — UAT PLAN
{UAT plan content from Agent 2}

---

# PART 2 — UX/UI TESTING PLAN
{UX/UI plan content from Agent 3}

---

# PART 3 — UNIT TESTING PLAN
{Unit testing plan content from Agent 4}

---

# PART 4 — KEY PLAYWRIGHT E2E SCRIPTS
{Extract the most critical Playwright scripts from Agent 3:
- Network regression test (most critical API contract)
- OData header verification
- Responsive nav breakpoint test
- No horizontal overflow
- Accessibility audit}
```

When synthesizing, do not truncate. Include ALL test cases, ALL code examples, ALL checklists.
The plan should be complete enough that a developer or QA engineer can execute it without
referring back to the codebase.

---

## Step 6 — Return Result

After writing the file, tell the user:

1. **Plan file path**: the full path to the saved plan
2. **Coverage summary**: number of test cases per module, total
3. **Project-specific findings**: any quirks or risks discovered by the Explore agent
   (e.g. known bugs in comments, unusual API patterns, missing permissions)
4. **Next step**: "You can now invoke the `eflip-PowerPages-QA` agent to execute
   any section of this plan. Just say 'run QA on [feature]' or 'post-deploy check'."

---

## Notes on Agent Prompt Customization

The templates in Step 3 use placeholders like `{PROJECT_NAME}`, `{SITE_URL}`, etc.
Replace ALL placeholders with real values extracted in Step 1 before launching agents.

The more specific the agent prompts, the better the plan output:
- Use actual entity names (not generic `entity_set`)
- Use actual route paths (not `/page-1`, `/page-2`)
- Use actual component names from the codebase
- Include actual known issues from code comments

If Step 1 couldn't determine the site URL (not in package.json or README), ask the user
before proceeding: "What is the deployed URL of your Power Pages site?"
