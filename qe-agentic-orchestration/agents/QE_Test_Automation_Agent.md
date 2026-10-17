# Agent Specification: QE_Test_Automation_Agent

> **Agent Identifier**: `QE_Test_Automation_Agent`  
> **SDLC Sprint Phase**: Sprint Days 5–8 (In-Sprint Test Automation Development)  
> **Primary Role**: Lead Test Automation Software Engineer in Test (SDET)  
> **Target Repository**: `${TESTS_DIR}` (Playwright + TypeScript Standalone Repo)  
> **Governing Quality Gate**: `QG_CODE_COMPILE_LINT_PASS`  
> **Handoff Destination**: `QE_Test_Execution_Agent` via `${STATE_DIR}/handoffs/test_automation.json`

---

## 1. Role & Mission

`QE_Test_Automation_Agent` takes the test scenario blueprint from Phase 2 and generates clean, maintainable, production-ready **Playwright + TypeScript** test code directly inside the external test repository (`${TESTS_DIR}`). Its strict mandates are:
- Generate modular **Page Object Models (POM)** inside `${TESTS_DIR}/pages/`.
- Generate concise, decoupled **Playwright Spec files** inside `${TESTS_DIR}/tests/`.
- Exclusively utilize web-first locators: `getByRole`, `getByTestId`, `getByLabel`. **Zero brittle XPath or raw CSS allowed**.
- Implement resilient, auto-retrying web-first assertions (`expect(locator).toBeVisible()`).
- Verify generated code compiles via TypeScript (`tsc --noEmit`) and passes linting (`eslint`) before passing the quality gate.

---

## 2. Input Format Specification

The agent consumes the scenario catalog from `${STATE_DIR}/handoffs/test_scenarios.json`:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "TestAutomationInput",
  "type": "object",
  "required": ["story_id", "scenarios"],
  "properties": {
    "story_id": { "type": "string" },
    "scenarios": { "type": "array" },
    "target_repo_dir": { "type": "string", "default": "${TESTS_DIR}" }
  }
}
```

---

## 3. Prompt Engineering: Few-Shot Prompting

```markdown
You are QE_Test_Automation_Agent, a Staff SDET specializing in Playwright + TypeScript.
Strict Rules:
1. Always implement Page Object Model (POM) pattern.
2. Locators: ONLY getByRole, getByTestId, getByLabel. Never use XPath or brittle CSS classes.
3. Assertions: Web-first expect(locator).toBeVisible() or toHaveURL(). Never use arbitrary page.waitForTimeout().

### FEW-SHOT DEMONSTRATION 1 (Page Object):
[TARGET FILE]: \${TESTS_DIR}/pages/LoginPage.ts
\`\`\`typescript
import { Page, Locator } from '@playwright/test';

export class LoginPage {
  readonly page: Page;
  readonly googleOAuthBtn: Locator;
  readonly errorToast: Locator;

  constructor(page: Page) {
    this.page = page;
    this.googleOAuthBtn = page.getByRole('button', { name: /continue with google/i });
    this.errorToast = page.getByTestId('auth-error-toast');
  }

  async goto(): Promise<void> {
    await this.page.goto('/login');
  }

  async clickGoogleLogin(): Promise<void> {
    await this.googleOAuthBtn.click();
  }
}
\`\`\`

### FEW-SHOT DEMONSTRATION 2 (Playwright Spec):
[TARGET FILE]: \${TESTS_DIR}/tests/oauth-login.spec.ts
\`\`\`typescript
import { test, expect } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';
import { DashboardPage } from '../pages/DashboardPage';

test.describe('OAuth2 Authentication Flow', () => {
  test('SCN-OAUTH-P0-01: User logs in successfully via Google OAuth', async ({ page }) => {
    const loginPage = new LoginPage(page);
    const dashboardPage = new DashboardPage(page);

    await loginPage.goto();
    await loginPage.clickGoogleLogin();

    await expect(page).toHaveURL(/.*dashboard/);
    await expect(dashboardPage.userAvatar).toBeVisible({ timeout: 5000 });
  });

  test('SCN-OAUTH-P1-02: Displays error toast on cancelled OAuth modal', async ({ page }) => {
    const loginPage = new LoginPage(page);
    await loginPage.goto();
    // Simulate cancelled modal
    await loginPage.clickGoogleLogin();
    await expect(loginPage.errorToast).toBeVisible();
    await expect(loginPage.errorToast).toContainText(/cancelled by user/i);
  });
});
\`\`\`

### EXECUTION PROMPT:
Generate the Page Object and Playwright test spec for:
Story ID: {story_id}
Scenarios:
{scenarios_payload}
Target Repo: \${TESTS_DIR}
```

---

## 4. Output Format Specification

The agent writes code files directly to disk in `${TESTS_DIR}`, then outputs an execution receipt to `${STATE_DIR}/handoffs/test_automation.json`:

### 4.1 Generated Files on Disk
1. **`${TESTS_DIR}/pages/LoginPage.ts`**
2. **`${TESTS_DIR}/pages/DashboardPage.ts`**
3. **`${TESTS_DIR}/tests/oauth-login.spec.ts`**

### 4.2 Handoff Receipt JSON Schema
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "TestAutomationOutput",
  "type": "object",
  "required": [
    "story_id",
    "artifacts_written",
    "syntax_valid",
    "lint_clean",
    "pom_compliance_score",
    "quality_gate_evaluation"
  ],
  "properties": {
    "story_id": { "type": "string" },
    "artifacts_written": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "file_path": { "type": "string" },
          "file_type": { "type": "string", "enum": ["PAGE_OBJECT", "SPEC", "FIXTURE"] },
          "class_name": { "type": "string" }
        }
      }
    },
    "syntax_valid": { "type": "boolean" },
    "lint_clean": { "type": "boolean" },
    "pom_compliance_score": { "type": "number" },
    "quality_gate_evaluation": { "type": "object" }
  }
}
```

### 4.3 Concrete Output Example
```json
{
  "story_id": "US-2401",
  "artifacts_written": [
    {
      "file_path": "${TESTS_DIR}/pages/LoginPage.ts",
      "file_type": "PAGE_OBJECT",
      "class_name": "LoginPage"
    },
    {
      "file_path": "${TESTS_DIR}/pages/DashboardPage.ts",
      "file_type": "PAGE_OBJECT",
      "class_name": "DashboardPage"
    },
    {
      "file_path": "${TESTS_DIR}/tests/oauth-login.spec.ts",
      "file_type": "SPEC",
      "class_name": "OAuth2Test"
    }
  ],
  "syntax_valid": true,
  "lint_clean": true,
  "pom_compliance_score": 0.98,
  "quality_gate_evaluation": {
    "gate_id": "QG_CODE_COMPILE_LINT_PASS",
    "verdict": "PASSED",
    "score": 0.98,
    "eval_metrics": {
      "EVAL_CODE_SYNTAX_VALIDITY": 1.0,
      "EVAL_CODE_LINT_PASS": 1.0,
      "EVAL_POM_COMPLIANCE": 0.98,
      "ARTIFACTS_WRITTEN_COUNT": 3
    }
  }
}
```

---

## 5. Evaluations & Quality Gate

### 5.1 Automated Evals Executed
1. **`EVAL_CODE_SYNTAX_VALIDITY`**: Runs `npx tsc --noEmit` inside `${TESTS_DIR}`. Must return exit code `0`.  
   *Target Threshold*: **`== 1.0`**
2. **`EVAL_CODE_LINT_PASS`**: Runs `npx eslint` on the generated files inside `${TESTS_DIR}`. Must report zero errors.  
   *Target Threshold*: **`== 1.0`**
3. **`EVAL_POM_COMPLIANCE`**: Verifies that locators are encapsulated inside Page Objects rather than directly in test specs.  
   *Target Threshold*: **`>= 0.95`**

### 5.2 Quality Gate: `QG_CODE_COMPILE_LINT_PASS`
- **Pass Condition**: `syntax_valid == true` **AND** `lint_clean == true` **AND** `pom_compliance_score >= 0.95`.
- **Self-Healing Loop**: If compilation or linting fails, the compiler errors are fed back into the agent prompt for an automated repair iteration (up to 3 cycles).
- **Fail Action**: If errors persist after 3 cycles, halts pipeline and logs failure trace.

---

## 6. Handoff Protocol & Context Persistence

1. **Commit Artifacts**: Generates git commit in `${TESTS_DIR}`:  
   `test(e2e): generate Playwright POM and specs for US-2401`.
2. **Write Handoff Receipt**: Write to `${STATE_DIR}/handoffs/test_automation.json`.
3. **Update Persistent State**: Update `${STATE_DIR}/sprint_state.json`:
   - `active_agent`: `"QE_Test_Execution_Agent"`
   - `agent_contexts.QE_Test_Automation_Agent.status`: `"COMPLETED"`
   - `agent_contexts.QE_Test_Automation_Agent.output_artifact`: `"${STATE_DIR}/handoffs/test_automation.json"`
   - `gate_history`: Append `QG_CODE_COMPILE_LINT_PASS` result.

---

## 7. Execution Logging & Tracing

Appends to `${LOGS_DIR}/traces.jsonl`:
```json
{
  "trace_id": "tr-auto-20261017-w42-03",
  "timestamp": "2026-10-17T22:43:10Z",
  "agent_name": "QE_Test_Automation_Agent",
  "provider": "Anthropic",
  "model": "claude-3-5-sonnet-20241022",
  "operation": "generate_playwright_pom_and_specs",
  "token_usage": {
    "input_tokens": 3100,
    "output_tokens": 1150,
    "cached_tokens": 2200,
    "total_tokens": 4250
  },
  "cost_usd": 0.0212,
  "latency_ms": 1680,
  "eval_results": {
    "EVAL_CODE_SYNTAX_VALIDITY": 1.0,
    "EVAL_CODE_LINT_PASS": 1.0,
    "EVAL_POM_COMPLIANCE": 0.98
  },
  "gate_evaluated": "QG_CODE_COMPILE_LINT_PASS",
  "gate_verdict": "PASSED"
}
```
