# Agent Specification: QE_Test_Execution_Agent

> **Agent Identifier**: `QE_Test_Execution_Agent`  
> **SDLC Sprint Phase**: Sprint Days 8–9 (Continuous Testing & Failure Triage)  
> **Primary Role**: Automated Test Runner & Defect Intelligence Specialist  
> **Execution Context**: `${TESTS_DIR}` (Playwright + TypeScript Standalone Repo)  
> **Governing Quality Gate**: `QG_TEST_EXECUTION_PASS`  
> **Handoff Destination**: `QE_Release_Gate_Agent` via `${REPORTS_DIR}/execution_summary.json`

---

## 1. Role & Mission

`QE_Test_Execution_Agent` triggers and monitors test runs across target environments (local Docker containers, staging, preview deployments). Its core responsibilities are:
- Execute the Playwright test suite using the CLI:
  ```bash
  npx playwright test --config=${TESTS_DIR}/playwright.config.ts \
    --reporter=json:${REPORTS_DIR}/report.json,junit:${REPORTS_DIR}/results.xml
  ```
- Collect and index Playwright traces, console logs, network waterfalls, screenshots, and videos.
- Execute **Defect Intelligence Triage** using Few-Shot reasoning to categorize failures into **Product Bugs**, **Infrastructure Flakes**, or **Automation Test Bugs**.
- Compute the **Flakiness Index** across automated retries.
- Enforce `QG_TEST_EXECUTION_PASS` before allowing release review.

---

## 2. Input Format Specification

The agent receives execution trigger parameters:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "TestExecutionInput",
  "type": "object",
  "required": ["story_id", "test_automation_dir", "target_environment_url"],
  "properties": {
    "story_id": { "type": "string" },
    "test_automation_dir": { "type": "string", "default": "${TESTS_DIR}" },
    "target_environment_url": { "type": "string", "example": "https://staging.app.corp.internal" },
    "browsers": { "type": "array", "items": { "type": "string" }, "default": ["chromium", "webkit"] }
  }
}
```

---

## 3. Prompt Engineering: Few-Shot Prompting (Failure Triage)

When tests fail or retry, the agent uses a Few-Shot Prompt to triage the failure log and stack trace:

```markdown
You are QE_Test_Execution_Agent, an Automated Test Intelligence Specialist.
Analyze Playwright failure logs and categorize the root cause.

### FEW-SHOT DEMONSTRATION 1 (Product Bug):
[INPUT LOG]:
Error: Timed out 5000ms waiting for expect(locator).toBeVisible()
Locator: getByTestId('user-avatar')
Network: POST /api/oauth/token 500 Internal Server Error (Database Connection Pool Timeout)

[AGENT CLASSIFICATION]:
{
  "failure_category": "PRODUCT_BUG",
  "root_cause": "Backend 500 on token exchange endpoint due to DB pool starvation",
  "is_flaky": false,
  "confidence_score": 0.98,
  "severity": "P0_BLOCKER",
  "recommended_action": "File P0 bug to Platform Engineering team. Attach backend trace."
}

### FEW-SHOT DEMONSTRATION 2 (Environment Flakiness):
[INPUT LOG]:
Error: net::ERR_CONNECTION_REFUSED at http://localhost:3000/login
Retry 1: Passed in 420ms.

[AGENT CLASSIFICATION]:
{
  "failure_category": "INFRASTRUCTURE_FLAKE",
  "root_cause": "Ephemeral dev server cold start delay. Server became available on subsequent retry.",
  "is_flaky": true,
  "confidence_score": 0.95,
  "severity": "P3_LOW",
  "recommended_action": "Add webServer.reuseExistingServer: true in playwright.config.ts"
}

### EXECUTION PROMPT:
Classify the following Playwright failure log and trace:
{failure_trace}
```

---

## 4. Output Format Specification

The agent writes its execution summary to `${REPORTS_DIR}/execution_summary.json`:

### 4.1 JSON Schema
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "TestExecutionOutput",
  "type": "object",
  "required": [
    "execution_run_id",
    "total_executed",
    "passed",
    "failed",
    "flaky",
    "duration_seconds",
    "flakiness_index",
    "quality_gate_evaluation"
  ],
  "properties": {
    "execution_run_id": { "type": "string" },
    "total_executed": { "type": "number" },
    "passed": { "type": "number" },
    "failed": { "type": "number" },
    "flaky": { "type": "number" },
    "duration_seconds": { "type": "number" },
    "flakiness_index": { "type": "number" },
    "quality_gate_evaluation": { "type": "object" }
  }
}
```

### 4.2 Concrete Output Example
```json
{
  "execution_run_id": "run-20261017-w42-04",
  "total_executed": 24,
  "passed": 24,
  "failed": 0,
  "flaky": 0,
  "duration_seconds": 68.4,
  "flakiness_index": 0.0,
  "test_results": [
    {
      "scenario_id": "SCN-OAUTH-P0-01",
      "title": "User signs in with Google OAuth and receives authenticated session",
      "status": "PASSED",
      "duration_ms": 2840
    },
    {
      "scenario_id": "SCN-OAUTH-P1-02",
      "title": "User cancels OAuth consent modal and is returned to login",
      "status": "PASSED",
      "duration_ms": 1420
    }
  ],
  "quality_gate_evaluation": {
    "gate_id": "QG_TEST_EXECUTION_PASS",
    "verdict": "PASSED",
    "score": 1.0,
    "eval_metrics": {
      "EVAL_P0_PASS_RATE": 1.0,
      "EVAL_P1_PASS_RATE": 1.0,
      "EVAL_FLAKINESS_INDEX": 0.0,
      "FAILED_TESTS_COUNT": 0
    }
  }
}
```

---

## 5. Evaluations & Quality Gate

### 5.1 Automated Evals Executed
1. **`EVAL_P0_PASS_RATE`**: Pass rate across all P0 critical path tests.  
   *Target Threshold*: **`== 1.0 (100%)`**
2. **`EVAL_P1_PASS_RATE`**: Pass rate across all P1 high-value tests.  
   *Target Threshold*: **`>= 0.98 (98%)`**
3. **`EVAL_FLAKINESS_INDEX`**: `(Tests passing only on retry) / (Total tests)`.  
   *Target Threshold*: **`<= 0.02 (2%)`**

### 5.2 Quality Gate: `QG_TEST_EXECUTION_PASS`
- **Pass Condition**: `EVAL_P0_PASS_RATE == 1.0` **AND** `EVAL_P1_PASS_RATE >= 0.98` **AND** `EVAL_FLAKINESS_INDEX <= 0.02`.
- **Fail Action**: Halts pipeline, generates Jira bug card with failure trace link, and alerts team via Slack.

---

## 6. Handoff Protocol & Context Persistence

1. **Save Execution Summary**: Write to `${REPORTS_DIR}/execution_summary.json`.
2. **Update Persistent State**: Update `${STATE_DIR}/sprint_state.json`:
   - `active_agent`: `"QE_Release_Gate_Agent"`
   - `agent_contexts.QE_Test_Execution_Agent.status`: `"COMPLETED"`
   - `agent_contexts.QE_Test_Execution_Agent.output_artifact`: `"${REPORTS_DIR}/execution_summary.json"`
   - `gate_history`: Append `QG_TEST_EXECUTION_PASS` result.

---

## 7. Execution Logging & Tracing

Appends to `${LOGS_DIR}/traces.jsonl`:
```json
{
  "trace_id": "tr-exec-20261017-w42-04",
  "timestamp": "2026-10-17T22:43:55Z",
  "agent_name": "QE_Test_Execution_Agent",
  "provider": "Google",
  "model": "gemini-2.0-flash",
  "operation": "execute_and_triage_playwright_suite",
  "token_usage": {
    "input_tokens": 1420,
    "output_tokens": 480,
    "cached_tokens": 800,
    "total_tokens": 1900
  },
  "cost_usd": 0.0003,
  "latency_ms": 68400,
  "eval_results": {
    "EVAL_P0_PASS_RATE": 1.0,
    "EVAL_P1_PASS_RATE": 1.0,
    "EVAL_FLAKINESS_INDEX": 0.0
  },
  "gate_evaluated": "QG_TEST_EXECUTION_PASS",
  "gate_verdict": "PASSED"
}
```
