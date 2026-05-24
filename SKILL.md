---
name: testcase-script-generator
description: "Automatically generates executable Playwright test scripts from structured test cases (Steps + Expected Result), with automatic Step alignment self-check. Supports @web/@Server/@API script types."
---

# Test Case Script Generator + Step Alignment

Automatically converts structured test cases (Steps + Expected Result) into executable Playwright test scripts (`*.spec.ts`), reusing existing project POM/Helper/fixture patterns for consistency and maintainability. Includes mandatory Step alignment self-check after generation.

## When to Use

Invoke this skill when users ask for any of:

- "Generate automation script from test case / turn testcase into spec"
- "Create new script following @web/@Server/@API tagged reference scripts"
- "Need executable, project-compliant, maintainable script (locators/interactions/assertions)"

## Required Inputs

Collect or extract from conversation:

1) **Test Case:**
   - Case ID/Name (e.g., T1230)
   - Precondition
   - Steps + Expected Result (line by line)
   - Test data (file name, sample values, account/permission, product/config, etc.)

2) **Runtime Environment:**
   - Target tag: `@web` / `@Server` / `@API` (infer from directory/content if not provided)
   - Must reuse Default portal (if Web)
   - Language locale (e.g., English UK)

3) **Reference Script:**
   - At least 1 similar `*.spec.ts` (same module/page/flow)
   - POM/Helper file paths involved (主动检索 if user doesn't know)

## Deliverables

1) New executable `*.spec.ts`
2) Minimal POM/Helper incremental changes (maximize reuse)
3) Generation self-check report (via `testcase-step-alignment`): Step alignment table + Blockers/Major/Minor

## Generation Workflow

### Step 0: Classification

Map the test case to one of three script types:

- **@web**: Portal/Shop frontend/admin Web
- **@Server**: Server UI config/JobList/Workflow
- **@API**: Pure API

Output spec must land in corresponding directory, using project's existing `@setup/testSetup` or `@playwright/test`.

### Step 1: Reference Script Search (Key to ≥95% Success Rate)

**Reference-centered generation strategy:**
1. Find scripts for same page/module/popup/table in target directory
2. If none, find scripts with same action (create/delete/import/drag-right-click/pre-check)
3. Reuse their:
   - `test.describe` tag combinations
   - `beforeEach/afterEach` login/cleanup patterns
   - POM naming, method naming, wait strategies, assertion message conventions

### Step 2: Action-Assertion Pairing

Convert each test case step into an "action block + assertion block":
- Action block: Call existing POM/Helper methods; if missing, add to POM first (never write complex selectors in spec)
- Assertion block: Every assertion must have readable message, format: `StepX: ...`
- Wait: Prefer POM's `waitForShowByCss/waitForHidden/waitForMessageDisappear`; avoid fixed waits

### Step 3: POM Incremental Strategy

When adding/modifying POM methods:
- **One method, one semantic**: e.g., `waitForFTPConnectionInList(name)`, `clickDuplicateInRightOption()`
- **Wait methods first**: encapsulate "wait for element show/hide/list refresh/popup close" as `waitForXxx`
- **Locator stability priority**: `data-testid` > role/label > stable text > css层级; avoid `nth-child`/long xpath

### Step 4: Template & Style Consistency

Scripts must have:
- File header comment block (consistent with same-directory scripts: testcase id, author, environment, etc.)
- `test.setTimeout(...)` usage consistent with same directory
- `try/catch/finally` only if普遍存在 in that directory; otherwise follow reference script structure
- Expect message format: `StepX: <what> <expected> <context>`
- Step comments must be at step action start, format: `// StepX <action from testcase>`
- Cleanup: prefer `afterEach` or `finally`, ensure cleanup runs even on failure

### Step 5: Executability Check

After generation, must verify and fix:
- No `test.only/describe.only/skip`
- No commented critical assertions/steps
- No excessive fixed waits (unless reference directory has clear established convention)
- No obvious anomalies (non-existent enum/method, typos, type definition conflicts); ensure Playwright can match and load

### Step 6: Self-Check Loop (Mandatory)

After generation, MUST call `testcase-step-alignment` for step-by-step review:
- If "partial coverage/fail": immediately fill missing actions or assertions
- If Blocker/Major found: fix per suggestions and re-check
- Repeat until all Step 1..N show "Pass"

## Output Structure Template

### Spec Skeleton
1) `test.describe('<tags>', () => { ... })`
2) `beforeEach`: login/page entry/cleanup (per reference script)
3) `test('<TestNameTxxxx>', async ({ ...fixtures }) => { ... })`
4) Each Step:
   - `// StepX <action from testcase>` (comment at step action start)
   - `await <POM action>`
   - `expect(<actual>, 'StepX: <assertion message>').toBeTruthy()/toBe()/toEqual()`
5) `afterEach`: cleanup/logout (per reference script)

### POM Method Naming Conventions
- Click: `clickXxxButton`, `clickXxxTab`
- Type: `typeXxx`, `setXxx`
- Select: `selectXxx`
- Wait: `waitForXxxShown`, `waitForXxxClosed`, `waitForXxxInList`, `waitForXxxNotInList`
- Assert helpers (optional): `isXxxShown`, `getXxxValue`

## Failure/Exception Strategy
- Network/backend sporadic: prioritize "wait for stable point", not try-catch swallowing errors
- Dialogs: open/close must have explicit wait or verifiable anchor
- List changes: add/delete must wait for list refresh before asserting (use waitForVisible/Hidden or short polling with cap)

## Final Acceptance Criteria
- Generated script is executable (Playwright can match and run it)
- Step alignment self-check passes (`testcase-step-alignment` outputs Step 1..N all "Pass")
- Code style consistent with same-directory reference scripts (imports, structure, naming, assertion messages, cleanup strategy)