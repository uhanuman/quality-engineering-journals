# Agent Specification: QE_Test_Design_Agent

> **Agent Identifier**: `QE_Test_Design_Agent`  
> **SDLC Sprint Phase**: Sprint Days 3–5 (Test Architecture & Strategy)  
> **Primary Role**: Lead Test Strategist & Scenario Architect  
> **Governing Quality Gate**: `QG_DESIGN_REVIEW_PASS`  
> **Handoff Destination**: `QE_Test_Automation_Agent` via `${STATE_DIR}/handoffs/test_scenarios.json`

---

## 1. Role & Mission

`QE_Test_Design_Agent` ingests the formal Gherkin acceptance criteria produced by Phase 1. Its mission is to formulate an end-to-end Test Strategy by:
- Decomposing criteria into prioritized test scenarios: **P0 (Critical smoke paths)**, **P1 (High business value paths)**, and **P2 (Edge/Defensive paths)**.
- Mapping scenarios to the **Test Pyramid** layers (Playwright E2E UI vs. Backend API Integration).
- Defining the target **Page Object Model (POM)** classes and web-first assertion strategies.
- Establishing an unbroken **Traceability Matrix** linking every Acceptance Criterion to its test scenario.
- Enforcing `QG_DESIGN_REVIEW_PASS` before test automation code is generated.

---

## 2. Input Format Specification

The agent consumes the handoff payload emitted by Phase 1 at `${STATE_DIR}/handoffs/requirements_analysis.json`.

### 2.1 JSON Schema
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "TestDesignInput",
  "type": "object",
  "required": ["story_id", "acceptance_criteria_gherkin"],
  "properties": {
    "story_id": { "type": "string" },
    "acceptance_criteria_gherkin": {
      "type": "array",
      "items": { "type": "string" }
    }
  }
}
```

---

## 3. Prompt Engineering: Few-Shot Prompting

```markdown
You are QE_Test_Design_Agent, an Expert Test Strategist.
Decompose acceptance criteria into prioritized P0/P1/P2 scenarios mapped to the Test Pyramid.

### FEW-SHOT DEMONSTRATION:
[USER INPUT]:
Story ID: US-101
Criteria:
- Scenario: Reset requested for registered account (Gherkin)...
- Scenario: Reset requested for unregistered email (Gherkin)...

[AGENT REASONING]:
1. Happy path user reset requires full UI journey -> P0 Playwright E2E.
2. Security enumeration verification requires checking raw JSON responses -> P1 API Integration.
3. Identify required Page Objects: LoginPage, ResetPasswordPage.

[STRUCTURED OUTPUT]:
{
  "story_id": "US-101",
  "test_strategy": "Playwright UI for user journeys + API integration tests for token expiration",
  "scenarios": [
    {
      "scenario_id": "SCN-PWD-P0-01",
      "priority": "P0",
      "layer": "E2E_PLAYWRIGHT",
      "title": "User submits valid password reset and receives confirmation message",
      "pom_targets": ["LoginPage", "ResetPasswordPage"],
      "assertions": [
        "Expect confirmation notification to be visible",
        "Expect submit button to disable after first click to prevent double submission"
      ],
      "test_data_requirements": "Active registered test user credentials in staging database"
    },
    {
      "scenario_id": "SCN-PWD-P1-02",
      "priority": "P1",
      "layer": "API_INTEGRATION",
      "title": "Verify reset endpoint returns generic 200 for unregistered email addresses",
      "pom_targets": [],
      "assertions": [
        "Expect HTTP status code 200 OK",
        "Expect JSON response { message: 'If account exists, email has been sent' }"
      ],
      "test_data_requirements": "Random unseeded email string"
    }
  ],
  "traceability_matrix": {
    "US-101-Criteria-01": ["SCN-PWD-P0-01"],
    "US-101-Criteria-02": ["SCN-PWD-P1-02"]
  },
  "p0_p1_coverage_percentage": 100
}

### EXECUTION PROMPT:
Decompose the following requirements payload into a test strategy:
{handoff_payload}
```

---

## 4. Output Format Specification

The agent writes its output to `${STATE_DIR}/handoffs/test_scenarios.json`:

### 4.1 JSON Schema
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "TestDesignOutput",
  "type": "object",
  "required": [
    "story_id",
    "test_strategy",
    "scenarios",
    "traceability_matrix",
    "p0_p1_coverage_percentage",
    "quality_gate_evaluation"
  ],
  "properties": {
    "story_id": { "type": "string" },
    "test_strategy": { "type": "string" },
    "scenarios": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["scenario_id", "priority", "layer", "title", "pom_targets", "assertions"],
        "properties": {
          "scenario_id": { "type": "string" },
          "priority": { "type": "string", "enum": ["P0", "P1", "P2"] },
          "layer": { "type": "string", "enum": ["E2E_PLAYWRIGHT", "API_INTEGRATION", "UNIT"] },
          "title": { "type": "string" },
          "pom_targets": { "type": "array", "items": { "type": "string" } },
          "assertions": { "type": "array", "items": { "type": "string" } },
          "test_data_requirements": { "type": "string" }
        }
      }
    },
    "traceability_matrix": { "type": "object" },
    "p0_p1_coverage_percentage": { "type": "number" },
    "quality_gate_evaluation": {
      "type": "object",
      "required": ["gate_id", "verdict", "score", "eval_metrics"]
    }
  }
}
```

### 4.2 Concrete Output Example
```json
{
  "story_id": "US-2401",
  "test_strategy": "Playwright TypeScript E2E suite executing across Chromium and WebKit browsers",
  "scenarios": [
    {
      "scenario_id": "SCN-OAUTH-P0-01",
      "priority": "P0",
      "layer": "E2E_PLAYWRIGHT",
      "title": "User signs in with Google OAuth and receives authenticated session",
      "pom_targets": ["LoginPage", "DashboardPage"],
      "assertions": [
        "Expect page URL to contain '/dashboard'",
        "Expect session storage token to exist",
        "Expect user avatar in header navigation to be visible"
      ],
      "test_data_requirements": "Valid staging Google OAuth mock credentials"
    },
    {
      "scenario_id": "SCN-OAUTH-P1-02",
      "priority": "P1",
      "layer": "E2E_PLAYWRIGHT",
      "title": "User cancels OAuth consent modal and is returned to login page with error toast",
      "pom_targets": ["LoginPage"],
      "assertions": [
        "Expect error toast 'Authentication cancelled by user' to be visible",
        "Expect login button to remain active"
      ],
      "test_data_requirements": "None"
    }
  ],
  "traceability_matrix": {
    "US-2401-Criteria-1": ["SCN-OAUTH-P0-01"],
    "US-2401-Criteria-2": ["SCN-OAUTH-P1-02"]
  },
  "p0_p1_coverage_percentage": 100,
  "quality_gate_evaluation": {
    "gate_id": "QG_DESIGN_REVIEW_PASS",
    "verdict": "PASSED",
    "score": 1.0,
    "eval_metrics": {
      "EVAL_SCENARIO_COVERAGE": 1.0,
      "UNMAPPED_CRITERIA_COUNT": 0,
      "P0_SCENARIO_COUNT": 1,
      "P1_SCENARIO_COUNT": 1
    }
  }
}
```

---

## 5. Evaluations & Quality Gate

### 5.1 Automated Evals Executed
1. **`EVAL_SCENARIO_COVERAGE`**: Ratio of acceptance criteria mapped to at least one test scenario.  
   *Target Threshold*: **`== 1.0 (100%)`**
2. **`UNMAPPED_CRITERIA_COUNT`**: Total count of orphan requirements.  
   *Target Threshold*: **`== 0`**
3. **`P0_SCENARIO_COUNT`**: Verification that critical user journeys have explicit P0 representation.  
   *Target Threshold*: **`>= 1`**

### 5.2 Quality Gate: `QG_DESIGN_REVIEW_PASS`
- **Pass Condition**: `EVAL_SCENARIO_COVERAGE == 1.0` **AND** `UNMAPPED_CRITERIA_COUNT == 0` **AND** `P0_SCENARIO_COUNT >= 1`.
- **Fail Action**: Re-prompts the agent with the missing criteria list; halts pipeline if coverage is below 100%.

---

## 6. Handoff Protocol & Context Persistence

1. **Write Output Artifact**: Write the JSON payload to `${STATE_DIR}/handoffs/test_scenarios.json`.
2. **Update Persistent State**: Update `${STATE_DIR}/sprint_state.json`:
   - `active_agent`: `"QE_Test_Automation_Agent"`
   - `agent_contexts.QE_Test_Design_Agent.status`: `"COMPLETED"`
   - `agent_contexts.QE_Test_Design_Agent.output_artifact`: `"${STATE_DIR}/handoffs/test_scenarios.json"`
   - `gate_history`: Append `QG_DESIGN_REVIEW_PASS` result.
3. **Team Context Continuation**: Engineers can review scenarios in `test_scenarios.json` or immediately trigger automation generation.

---

## 7. Execution Logging & Tracing

Appends structured event to `${LOGS_DIR}/traces.jsonl`:
```json
{
  "trace_id": "tr-dsn-20261017-w42-02",
  "timestamp": "2026-10-17T22:42:35Z",
  "agent_name": "QE_Test_Design_Agent",
  "provider": "Anthropic",
  "model": "claude-3-5-sonnet-20241022",
  "operation": "decompose_scenarios_and_test_strategy",
  "token_usage": {
    "input_tokens": 2150,
    "output_tokens": 780,
    "cached_tokens": 1400,
    "total_tokens": 2930
  },
  "cost_usd": 0.0142,
  "latency_ms": 1340,
  "eval_results": {
    "EVAL_SCENARIO_COVERAGE": 1.0,
    "UNMAPPED_CRITERIA_COUNT": 0
  },
  "gate_evaluated": "QG_DESIGN_REVIEW_PASS",
  "gate_verdict": "PASSED"
}
```
