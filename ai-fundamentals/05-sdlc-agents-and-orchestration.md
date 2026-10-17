# SDLC Quality Engineering Agents & Orchestration

> **Journal Series**: Quality Engineering — AI Systems  
> **Edition**: Volume V — Custom QE Agents for AI-Assisted Agile SDLC (2-Week Sprint)  
> **Published**: October 2026  
> **Author**: Quality Engineering Architecture Group  
> **Companions**: [Vol I — AI Fundamentals](./01-ai-fundamentals.md) · [Vol II — Prompt & Context Engineering](./02-prompt-and-context-engineering.md) · [Vol III — Building with AI](./03-building-with-ai.md) · [Vol IV — AI Tooling & Ecosystem](./04-ai-tools-ecosystem.md)

---

## Executive Overview: The Agile AI-Assisted QE Engine

In a modern two-week Agile Scrum sprint, Quality Engineering cannot remain a bottleneck at the end of the sprint cycle. This document specifies a production-grade, multi-agent Quality Engineering architecture designed to execute inside an autonomous CI/CD and developer environment.

Each specialized agent owns a phase of the sprint, executing **Few-Shot Prompting**, performing **automated evaluations (Evals)**, validating against strict **Quality Gates (`QG_*`)** and **Release Gates (`RG_*`)**, persisting state and context to the repository, and passing structured handoff payloads to downstream agents.

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│               AI-ASSISTED 2-WEEK SPRINT QUALITY ENGINEERING PIPELINE                  │
│                                                                                        │
│  SPRINT DAYS 1-2          DAYS 3-5           DAYS 5-8           DAYS 8-9       DAY 10  │
│                                                                                        │
│  ┌──────────────────┐    ┌───────────────┐  ┌───────────────┐  ┌────────────┐ ┌──────┐ │
│  │QE_Requirements_  │───▶│QE_Test_Design_│─▶│QE_Test_Auto_  │─▶│QE_Test_    │▶│QE_   │ │
│  │Analysis_Agent    │    │Agent          │  │mation_Agent   │  │Execution_  │ │Rel.  │ │
│  └──────────────────┘    └───────────────┘  └───────────────┘  │Agent       │ │Gate  │ │
│           │                      │                  │          └────────────┘ └──────┘ │
│           ▼                      ▼                  ▼                 │           │    │
│     [QG_REQ_READY]        [QG_DESIGN_PASS]   [QG_CODE_PASS]           ▼           ▼    │
│                                                                [QG_EXEC_PASS]  [RG_PROD│
│                                                                                _READY] │
│  SHARED INFRASTRUCTURE:                                                                │
│  • State & Context Store  : ${STATE_DIR}/sprint_state.json                             │
│  • Test Automation Repo   : ${TEST_AUTOMATION_REPO_DIR}/ (Playwright + TypeScript)     │
│  • Trace & Log Store      : ${LOGS_DIR}/traces.jsonl                                   │
│  • Pipeline Orchestrator  : QE_SDLC_Orchestrator (Automated CI & CLI Runner)          │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## Table of Contents

1. [Architecture & Multi-Repo Topology](#1-architecture--multi-repo-topology)
2. [Global Path Environment Variables](#2-global-path-environment-variables)
3. [Standardized Naming Conventions](#3-standardized-naming-conventions)
4. [Agent State & Context Persistence Specification](#4-agent-state--context-persistence-specification)
5. [Trace & Audit Execution Logging Specification](#5-trace--audit-execution-logging-specification)
6. [Agent 1: QE_Requirements_Analysis_Agent (Sprint Days 1–2)](#6-agent-1-qe_requirements_analysis_agent)
7. [Agent 2: QE_Test_Design_Agent (Sprint Days 3–5)](#7-agent-2-qe_test_design_agent)
8. [Agent 3: QE_Test_Automation_Agent (Sprint Days 5–8)](#8-agent-3-qe_test_automation_agent)
9. [Agent 4: QE_Test_Execution_Agent (Sprint Days 8–9)](#9-agent-4-qe_test_execution_agent)
10. [Agent 5: QE_Release_Gate_Agent (Sprint Day 10)](#10-agent-5-qe_release_gate_agent)
11. [Master Pipeline Orchestrator: QE_SDLC_Orchestrator](#11-master-pipeline-orchestrator-qe_sdlc_orchestrator)
12. [Quality & Release Gate Matrix](#12-quality--release-gate-matrix)
13. [End-to-End Sprint Walkthrough](#13-end-to-end-sprint-walkthrough)

---

## 1. Architecture & Multi-Repo Topology

In enterprise setups, agents, specifications, and test code do not live in a single monolithic repo. Our system decouples concerns across repositories while maintaining path-agnostic flexibility:

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                              MULTI-REPOSITORY TOPOLOGY                                 │
│                                                                                        │
│   REPO 1: Product / App Repo              REPO 2: AI Agents Repo (This System)         │
│   ┌────────────────────────────────┐      ┌────────────────────────────────────────┐   │
│   │ /specs                         │      │ /config/env.json                       │   │
│   │   ├── PRD-Sprint-24.md         │◀─┐   │ /state                                 │   │
│   │   └── user-stories.json        │  │   │   ├── sprint_state.json                │   │
│   │ /src (Application Source Code) │  │   │   └── handoffs/                        │   │
│   └────────────────────────────────┘  │   │ /logs                                  │   │
│                                       │   │   └── traces.jsonl                     │   │
│                                       ├───│ /orchestrator/orchestrator.ts          │   │
│                                       │   └────────────────────────────────────────┘   │
│   REPO 3: Test Automation Repo        │                        │                       │
│   (Playwright + TypeScript)           │                        │ Generates tests       │
│   ┌────────────────────────────────┐  │                        ▼                       │
│   │ /pages (Page Object Models)    │◀─┴────────────────────────────────────────────────┘
│   │ /tests (Playwright Specs)      │
│   │ /fixtures (Test Fixtures)      │
│   │ playwright.config.ts           │
│   └────────────────────────────────┘
└────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Global Path Environment Variables

To prevent path hardcoding, all agents consume standardized environment variables. If a variable is omitted, the agent falls back to configured defaults relative to the workspace root:

| Environment Variable | Description | Default Fallback Value |
|---|---|---|
| `INPUT_SPECS_DIR` | Directory containing PRDs, requirements, and user stories | `../app-repo/specs` or `./inputs/specs` |
| `TEST_AUTOMATION_REPO_DIR` | Root of the standalone Playwright + TypeScript repository | `../test-automation-repo` |
| `STATE_STORE_DIR` | Directory where shared agent state and checkpoints reside | `./state` |
| `LOGS_TRACE_DIR` | Directory where JSONL telemetry and LLM traces are persisted | `./logs` |
| `REPORTS_OUTPUT_DIR` | Directory where Quality Gate audit reports are published | `./reports` |

---

## 3. Standardized Naming Conventions

Consistency across identifiers allows automated scripts, regex parsers, and CI dashboards to index gates and evals cleanly:

- **Agents**: `QE_<Domain>_<Function>_Agent`  
  *Examples*: `QE_Requirements_Analysis_Agent`, `QE_Test_Automation_Agent`
- **Evaluations (Evals)**: `EVAL_<DOMAIN>_<METRIC>`  
  *Examples*: `EVAL_REQ_TESTABILITY`, `EVAL_CODE_LINT_PASS`
- **Quality Gates (In-Sprint Milestones)**: `QG_<PHASE>_<CRITERIA>`  
  *Examples*: `QG_REQ_READY_FOR_DEV`, `QG_CODE_COMPILE_LINT_PASS`
- **Release Gates (Deployment Milestones)**: `RG_<TARGET>_<CRITERIA>`  
  *Examples*: `RG_REGRESSION_FREE_PASS`, `RG_PRODUCTION_RELEASE_READY`

---

## 4. Agent State & Context Persistence Specification

To ensure team members can inspect, resume, or replay agent workflows across distributed machines, the orchestrator maintains a single source of truth at `${STATE_STORE_DIR}/sprint_state.json`.

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "sprint_id": "Sprint-2026-W42",
  "timestamp": "2026-10-17T22:30:00Z",
  "active_agent": "QE_Test_Automation_Agent",
  "sprint_goal": "Implement and verify OAuth2 Social Login and MFA challenge",
  "paths": {
    "specs_dir": "${INPUT_SPECS_DIR}",
    "test_automation_dir": "${TEST_AUTOMATION_REPO_DIR}",
    "state_dir": "${STATE_STORE_DIR}",
    "logs_dir": "${LOGS_TRACE_DIR}",
    "reports_dir": "${REPORTS_OUTPUT_DIR}"
  },
  "gate_history": [
    {
      "gate_id": "QG_REQ_READY_FOR_DEV",
      "verdict": "PASSED",
      "timestamp": "2026-10-18T10:14:22Z",
      "score": 0.94,
      "evaluator": "QE_Requirements_Analysis_Agent"
    },
    {
      "gate_id": "QG_DESIGN_REVIEW_PASS",
      "verdict": "PASSED",
      "timestamp": "2026-10-20T16:45:10Z",
      "score": 1.0,
      "evaluator": "QE_Test_Design_Agent"
    }
  ],
  "agent_contexts": {
    "QE_Requirements_Analysis_Agent": {
      "status": "COMPLETED",
      "output_artifact": "${STATE_STORE_DIR}/handoffs/requirements_analysis.json"
    },
    "QE_Test_Design_Agent": {
      "status": "COMPLETED",
      "output_artifact": "${STATE_STORE_DIR}/handoffs/test_scenarios.json"
    },
    "QE_Test_Automation_Agent": {
      "status": "IN_PROGRESS",
      "output_artifact": null
    }
  }
}
```

---

## 5. Trace & Audit Execution Logging Specification

Every LLM request, tool call, and evaluation generates a structured JSONL event in `${LOGS_TRACE_DIR}/traces.jsonl`. This guarantees reproducibility and enables cost, latency, and model drift analysis:

```json
{
  "trace_id": "tr-8c638a-20261017-001",
  "span_id": "sp-agent-03-call-01",
  "timestamp": "2026-10-17T22:31:05.120Z",
  "agent_name": "QE_Test_Automation_Agent",
  "provider": "Anthropic",
  "model": "claude-3-5-sonnet-20241022",
  "operation": "generate_playwright_spec",
  "token_usage": {
    "input_tokens": 3450,
    "output_tokens": 820,
    "cached_tokens": 2800,
    "total_tokens": 4270
  },
  "cost_usd": 0.0214,
  "latency_ms": 1420,
  "eval_results": {
    "EVAL_CODE_SYNTAX_VALIDITY": 1.0,
    "EVAL_PAGE_OBJECT_ADHERENCE": 0.96
  },
  "gate_evaluated": "QG_CODE_COMPILE_LINT_PASS",
  "gate_verdict": "PASSED"
}
```

---

## 6. Agent 1: QE_Requirements_Analysis_Agent

```
Agent ID:         QE_Requirements_Analysis_Agent
Sprint Phase:     Sprint Planning & Backlog Refinement (Days 1–2)
Primary Model:    Claude 3.5 Sonnet / Gemini 2.5 Pro
Input Channel:    ${INPUT_SPECS_DIR}/*.md
Output Channel:   ${STATE_STORE_DIR}/handoffs/requirements_analysis.json
Quality Gate:     QG_REQ_READY_FOR_DEV
```

### 6.1 Purpose & Role
Inspects raw Product Requirement Documents (PRDs) and user stories. Identifies ambiguities, missing error states, untestable assertions, and edge conditions. Synthesizes formal Acceptance Criteria in **Gherkin BDD format**.

### 6.2 Input Format
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["story_id", "title", "raw_requirements_markdown"],
  "properties": {
    "story_id": { "type": "string", "example": "US-2401" },
    "title": { "type": "string", "example": "OAuth2 Social Login Integration" },
    "raw_requirements_markdown": { "type": "string" }
  }
}
```

### 6.3 Prompt Engineering: Few-Shot Prompting Technique
```markdown
You are QE_Requirements_Analysis_Agent, an elite Quality Engineering Architect.
Your task is to analyze raw user stories, eliminate ambiguities, and extract testable Gherkin scenarios.

### EXAMPLE 1 (Few-Shot Demonstration):
[INPUT]:
Story: User wants to reset password.
Requirements: User enters email. If valid, send reset link. Link expires in 15 mins.

[REASONING]:
1. Ambiguity: What happens if email does not exist? (Security: must not leak email registration).
2. Edge condition: What if user requests multiple resets in 15 mins?
3. Format: Synthesize BDD Given-When-Then.

[OUTPUT]:
{
  "ambiguity_score": 0.05,
  "testability_score": 0.95,
  "identified_risks": ["Account enumeration vulnerability if unregistered email returns explicit error"],
  "acceptance_criteria_gherkin": [
    "Scenario: Password reset requested for registered email\n  Given a registered user with email \"user@corp.com\"\n  When user submits password reset for \"user@corp.com\"\n  Then a generic confirmation message is displayed\n  And a reset token valid for 15 minutes is dispatched",
    "Scenario: Password reset requested for unregistered email\n  Given no user exists with email \"unknown@corp.com\"\n  When user submits password reset for \"unknown@corp.com\"\n  Then the identical generic confirmation message is displayed\n  And no reset email is dispatched"
  ]
}

### EXAMPLE 2 (Few-Shot Demonstration):
[INPUT]:
Story: File upload for avatar.
Requirements: User can upload avatar image up to 5MB. Must be JPG or PNG.

[REASONING]:
1. Ambiguity: What happens on 5.01MB? On corrupted files? On SVG format (XSS threat)?
2. Testability: File type validation must be mime-type based, not extension-based.

[OUTPUT]:
{
  "ambiguity_score": 0.02,
  "testability_score": 0.98,
  "identified_risks": ["Potential SVG XSS upload payload", "DoS via memory exhaustion on large image decomp"],
  "acceptance_criteria_gherkin": [
    "Scenario: Valid JPEG avatar upload\n  Given user is on profile settings\n  When user uploads \"avatar.jpg\" of size 4.2MB\n  Then upload succeeds and avatar preview renders in 500ms",
    "Scenario: File size boundary exceeded\n  Given user is on profile settings\n  When user uploads \"large.png\" of size 5.1MB\n  Then upload is blocked client-side with error \"Max file size is 5MB\""
  ]
}

### TASK:
Now analyze the following requirements according to the schema:
Story ID: {story_id}
Title: {title}
Raw Requirements:
{raw_requirements_markdown}
```

### 6.4 Output Format
```json
{
  "story_id": "US-2401",
  "ambiguity_score": 0.04,
  "testability_score": 0.96,
  "identified_risks": [
    "Race condition on concurrent token refreshes",
    "Third-party OAuth identity provider latency timeout"
  ],
  "acceptance_criteria_gherkin": [
    "Scenario: Successful Google OAuth2 login\n  Given user navigates to login page\n  When user clicks \"Continue with Google\"\n  And authorizes via Google Identity Services\n  Then session cookie is set with HttpOnly and Secure flags\n  And user lands on dashboard within 2 seconds"
  ],
  "quality_gate_evaluation": {
    "gate_id": "QG_REQ_READY_FOR_DEV",
    "status": "PASSED",
    "eval_metrics": {
      "EVAL_REQ_TESTABILITY": 0.96,
      "EVAL_AMBIGUITY_SCORE": 0.04
    }
  }
}
```

### 6.5 Quality Gate & Evaluation Thresholds
- **Eval**: `EVAL_REQ_TESTABILITY >= 0.90` (Model judge validates that all acceptance criteria have objective verification steps).
- **Eval**: `EVAL_AMBIGUITY_SCORE <= 0.10` (No open questions or ambiguous adjectives like "fast", "intuitive", "scalable").
- **Quality Gate**: `QG_REQ_READY_FOR_DEV`  
  - Pass condition: `EVAL_REQ_TESTABILITY >= 0.90` AND `EVAL_AMBIGUITY_SCORE <= 0.10`.
  - Fail action: Escalates to Product Owner with line-item ambiguity report; blocks Sprint backlog commit.

### 6.6 Handoff Steps
1. Write payload to `${STATE_STORE_DIR}/handoffs/requirements_analysis.json`.
2. Update `${STATE_STORE_DIR}/sprint_state.json` recording `QG_REQ_READY_FOR_DEV: PASSED`.
3. Dispatch trigger event to **`QE_Test_Design_Agent`**.

---

## 7. Agent 2: QE_Test_Design_Agent

```
Agent ID:         QE_Test_Design_Agent
Sprint Phase:     Sprint Architecture & Test Strategy (Days 3–5)
Primary Model:    Claude 3.5 Sonnet / OpenAI o1
Input Channel:    ${STATE_STORE_DIR}/handoffs/requirements_analysis.json
Output Channel:   ${STATE_STORE_DIR}/handoffs/test_scenarios.json
Quality Gate:     QG_DESIGN_REVIEW_PASS
```

### 7.1 Purpose & Role
Transforms structured Gherkin criteria into a full Test Strategy. Decomposes requirements into **P0 (Critical Path), P1 (High Impact), and P2 (Edge/Defensive)** scenarios. Maps tests to the Test Pyramid (Unit vs. API Integration vs. Playwright End-to-End).

### 7.2 Input Format
Consumes the output from `QE_Requirements_Analysis_Agent` directly via `${STATE_STORE_DIR}/handoffs/requirements_analysis.json`.

### 7.3 Prompt Engineering: Few-Shot Prompting Technique
```markdown
You are QE_Test_Design_Agent. You architect comprehensive test scenario catalogs.
Decompose requirements into P0/P1/P2 priorities and define the target execution layer.

### EXAMPLE 1 (Few-Shot Demonstration):
[INPUT]:
Gherkin Scenario: Successful Google OAuth2 login...
Gherkin Scenario: Password reset for unregistered email...

[OUTPUT]:
{
  "scenarios": [
    {
      "scenario_id": "SCN-AUTH-001",
      "priority": "P0",
      "layer": "E2E_PLAYWRIGHT",
      "title": "OAuth2 Happy Path Login via Google Identity",
      "test_type": "Positive_Functional",
      "target_page_objects": ["LoginPage", "DashboardPage"],
      "test_data_requirements": "Valid staging Google OAuth Mock Credentials"
    },
    {
      "scenario_id": "SCN-AUTH-002",
      "priority": "P1",
      "layer": "API_INTEGRATION",
      "title": "Verify OAuth callback returns 401 on expired authorization code",
      "test_type": "Negative_Security",
      "target_page_objects": [],
      "test_data_requirements": "Expired auth_code string"
    }
  ],
  "p0_p1_coverage_percentage": 100
}

### TASK:
Generate the test scenario design for the following requirements payload:
{handoff_payload}
```

### 7.4 Output Format
```json
{
  "story_id": "US-2401",
  "test_strategy": "Hybrid API Mock & Playwright E2E UI verification",
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
        "Expect user avatar to be visible"
      ]
    },
    {
      "scenario_id": "SCN-OAUTH-P1-02",
      "priority": "P1",
      "layer": "E2E_PLAYWRIGHT",
      "title": "User cancels OAuth consent modal and is gracefully returned to login",
      "pom_targets": ["LoginPage"],
      "assertions": [
        "Expect error toast 'Authentication cancelled by user' to be visible",
        "Expect login button to remain active"
      ]
    }
  ],
  "traceability_matrix": {
    "US-2401-Criteria-1": ["SCN-OAUTH-P0-01"],
    "US-2401-Criteria-2": ["SCN-OAUTH-P1-02"]
  }
}
```

### 7.5 Quality Gate & Evaluation Thresholds
- **Eval**: `EVAL_SCENARIO_COVERAGE == 1.0` (100% of P0 & P1 Gherkin criteria mapped to test cases).
- **Eval**: `EVAL_POM_MAPPING_VALIDITY >= 0.95` (All referenced Page Object classes exist or have clear contracts).
- **Quality Gate**: `QG_DESIGN_REVIEW_PASS`  
  - Pass condition: `EVAL_SCENARIO_COVERAGE == 1.0` AND Zero unmapped acceptance criteria.
  - Fail action: Returns design to agent with list of uncovered criteria.

### 7.6 Handoff Steps
1. Persist output to `${STATE_STORE_DIR}/handoffs/test_scenarios.json`.
2. Update `${STATE_STORE_DIR}/sprint_state.json`.
3. Signal readiness to **`QE_Test_Automation_Agent`**.

---

## 8. Agent 3: QE_Test_Automation_Agent

```
Agent ID:         QE_Test_Automation_Agent
Sprint Phase:     In-Sprint Test Automation Coding (Days 5–8)
Primary Model:    Claude 3.5 Sonnet / Qwen 2.5 Coder 32B
Input Channel:    ${STATE_STORE_DIR}/handoffs/test_scenarios.json
Output Target:    ${TEST_AUTOMATION_REPO_DIR}/pages/ & ${TEST_AUTOMATION_REPO_DIR}/tests/
Quality Gate:     QG_CODE_COMPILE_LINT_PASS
```

### 8.1 Purpose & Role
Reads test scenario specifications and generates **production-grade Playwright + TypeScript** test scripts and **Page Object Models (POM)** directly inside the external test repository `${TEST_AUTOMATION_REPO_DIR}`. Follows strict POM design patterns, explicit locators (`getByRole`, `getByTestId`), and automatic retry assertions.

### 8.2 Input Format
Consumes `${STATE_STORE_DIR}/handoffs/test_scenarios.json`.

### 8.3 Prompt Engineering: Few-Shot Prompting Technique
```markdown
You are QE_Test_Automation_Agent. You write clean, robust, flake-free Playwright TypeScript code.
Strict Rules:
1. Use Page Object Model (POM) architecture.
2. Locators MUST prefer: getByRole, getByTestId, getByLabel. NEVER use raw brittle CSS/XPath.
3. Every test must import fixtures from '../fixtures/baseTest'.
4. Assertions must use web-first expect(locator).toBeVisible(), not arbitrary timeouts.

### EXAMPLE 1 (Page Object Demonstration):
[TARGET FILE]: ${TEST_AUTOMATION_REPO_DIR}/pages/LoginPage.ts
```typescript
import { Page, Locator } from '@playwright/test';

export class LoginPage {
  readonly page: Page;
  readonly googleLoginBtn: Locator;
  readonly errorToast: Locator;

  constructor(page: Page) {
    this.page = page;
    this.googleLoginBtn = page.getByRole('button', { name: /continue with google/i });
    this.errorToast = page.getByTestId('auth-error-toast');
  }

  async goto(): Promise<void> {
    await this.page.goto('/login');
  }

  async clickGoogleLogin(): Promise<void> {
    await this.googleLoginBtn.click();
  }
}
```

### EXAMPLE 2 (Playwright Spec Demonstration):
[TARGET FILE]: ${TEST_AUTOMATION_REPO_DIR}/tests/oauth-login.spec.ts
```typescript
import { test, expect } from '../fixtures/baseTest';
import { LoginPage } from '../pages/LoginPage';
import { DashboardPage } from '../pages/DashboardPage';

test.describe('OAuth2 Authentication Flow', () => {
  test('SCN-OAUTH-P0-01: User logs in successfully via Google OAuth', async ({ page }) => {
    const loginPage = new LoginPage(page);
    const dashboardPage = new DashboardPage(page);

    await loginPage.goto();
    await loginPage.clickGoogleLogin();

    // Verify redirect and session tokens
    await expect(page).toHaveURL(/.*dashboard/);
    await expect(dashboardPage.userAvatar).toBeVisible({ timeout: 5000 });
  });
});
```

### TASK:
Generate Playwright TypeScript Page Objects and Tests for:
Scenario: {scenario_payload}
Target Directories:
- Page Objects: ${TEST_AUTOMATION_REPO_DIR}/pages/
- Tests: ${TEST_AUTOMATION_REPO_DIR}/tests/
```

### 8.4 Generated Artifacts (Example Output on Disk)
The agent directly creates/updates files in the external automation repository:
- `${TEST_AUTOMATION_REPO_DIR}/pages/LoginPage.ts`
- `${TEST_AUTOMATION_REPO_DIR}/pages/DashboardPage.ts`
- `${TEST_AUTOMATION_REPO_DIR}/tests/oauth-login.spec.ts`

### 8.5 Quality Gate & Evaluation Thresholds
- **Eval**: `EVAL_CODE_SYNTAX_VALIDITY == 1.0` (Runs `tsc --noEmit` inside `${TEST_AUTOMATION_REPO_DIR}`).
- **Eval**: `EVAL_CODE_LINT_PASS == 1.0` (Runs `eslint` inside `${TEST_AUTOMATION_REPO_DIR}` with 0 errors).
- **Eval**: `EVAL_LOCATOR_RESILIENCE >= 0.95` (Zero brittle XPath locators; all use Playwright web-first locators).
- **Quality Gate**: `QG_CODE_COMPILE_LINT_PASS`  
  - Pass condition: `tsc` exits code 0 AND `eslint` exits code 0.
  - Fail action: Automatically feeds compilation errors back into agent loop for self-healing (max 3 repair iterations).

### 8.6 Handoff Steps
1. Emit git commit in `${TEST_AUTOMATION_REPO_DIR}` with message: `test(e2e): generate Playwright suite for US-2401`.
2. Update `${STATE_STORE_DIR}/sprint_state.json`.
3. Trigger **`QE_Test_Execution_Agent`**.

---

## 9. Agent 4: QE_Test_Execution_Agent

```
Agent ID:         QE_Test_Execution_Agent
Sprint Phase:     Continuous Integration & Test Execution (Days 8–9)
Primary Model:    Gemini 2.0 Flash / Claude 3.5 Haiku
Input Channel:    ${TEST_AUTOMATION_REPO_DIR}
Output Channel:   ${REPORTS_OUTPUT_DIR}/execution-summary.json & JUnit XML
Quality Gate:     QG_TEST_EXECUTION_PASS
```

### 9.1 Purpose & Role
Executes the generated Playwright test suite against target environments (Docker staging / local dev). Ingests Playwright traces, JUnit XML reports, videos, and screenshots. Runs **flakiness analysis** (quarantining tests that fail intermittently across retries) and generates an executive root-cause failure breakdown.

### 9.2 Execution Command
```bash
npx playwright test --config=${TEST_AUTOMATION_REPO_DIR}/playwright.config.ts \
  --reporter=list,json:${REPORTS_OUTPUT_DIR}/report.json,junit:${REPORTS_OUTPUT_DIR}/results.xml
```

### 9.3 Prompt Engineering: Few-Shot Prompting Technique
Used for automated **Defect Root Cause Analysis** when tests fail:
```markdown
You are QE_Test_Execution_Agent analyzing Playwright test failures.
Differentiate between:
1. Product Bug (assertion failed on valid business logic)
2. Environment / Infrastructure Flake (504 Gateway Timeout, connection reset)
3. Automation Bug (stale element, bad test data)

### EXAMPLE 1:
[INPUT]:
Error: Timed out 5000ms waiting for expect(locator).toBeVisible()
Locator: getByTestId('user-avatar')
Network logs: POST /api/oauth/token 500 Internal Server Error: Database pool exhausted.

[OUTPUT]:
{
  "failure_type": "Product_Backend_Bug",
  "root_cause": "Backend 500 on token exchange due to DB pool exhaustion",
  "is_flaky": false,
  "recommended_action": "File P0 ticket to Backend Platform team with backend stack trace"
}

### TASK:
Analyze the following Playwright failure log and trace:
{failure_trace}
```

### 9.4 Output Format
```json
{
  "execution_run_id": "run-20261017-094",
  "total_executed": 48,
  "passed": 48,
  "failed": 0,
  "flaky": 0,
  "duration_seconds": 124.6,
  "coverage_summary": {
    "P0_pass_rate": 1.0,
    "P1_pass_rate": 1.0
  },
  "quality_gate_evaluation": {
    "gate_id": "QG_TEST_EXECUTION_PASS",
    "status": "PASSED",
    "metrics": {
      "P0_PASS_RATE": 1.0,
      "FLAKINESS_INDEX": 0.0
    }
  }
}
```

### 9.5 Quality Gate & Evaluation Thresholds
- **Eval**: `EVAL_P0_PASS_RATE == 1.0` (Zero P0 test failures allowed).
- **Eval**: `EVAL_P1_PASS_RATE >= 0.98` (98%+ P1 tests passing).
- **Eval**: `EVAL_FLAKINESS_INDEX <= 0.02` (Less than 2% intermittent retries).
- **Quality Gate**: `QG_TEST_EXECUTION_PASS`  
  - Pass condition: 100% P0 pass, >=98% P1 pass, flakiness <= 2%.
  - Fail action: Halts pipeline, generates bug report with trace links attached.

### 9.6 Handoff Steps
1. Save report to `${REPORTS_OUTPUT_DIR}/execution-summary.json`.
2. Update `${STATE_STORE_DIR}/sprint_state.json`.
3. Trigger final **`QE_Release_Gate_Agent`**.

---

## 10. Agent 5: QE_Release_Gate_Agent

```
Agent ID:         QE_Release_Gate_Agent
Sprint Phase:     Sprint Review & Deployment Decision (Day 10)
Primary Model:    Claude 3.5 Sonnet / OpenAI o1
Input Channel:    ${STATE_STORE_DIR}/sprint_state.json & ${REPORTS_OUTPUT_DIR}/*
Output Channel:   ${REPORTS_OUTPUT_DIR}/RELEASE_SIGN_OFF.md
Release Gate:     RG_PRODUCTION_RELEASE_READY
```

### 10.1 Purpose & Role
Acts as the **Virtual Quality Assurance Director**. Audits the cumulative artifacts across the entire 2-week sprint: requirements testability, scenario coverage, code lint pass, test execution results, and security scans. Generates the formal **Go / No-Go Decision** for production deployment.

### 10.2 Input Format
Ingests the master `${STATE_STORE_DIR}/sprint_state.json` and all handoff artifacts.

### 10.3 Prompt Engineering: Few-Shot Prompting Technique
```markdown
You are QE_Release_Gate_Agent. You render objective Go / No-Go release decisions.
Audit all gate metrics against enterprise release policies.

### EXAMPLE 1 (GO Decision):
[INPUT]:
QG_REQ_READY_FOR_DEV: PASSED (0.95)
QG_DESIGN_REVIEW_PASS: PASSED (1.0)
QG_CODE_COMPILE_LINT_PASS: PASSED (1.0)
QG_TEST_EXECUTION_PASS: PASSED (P0=100%, P1=99%)
Open Vulnerabilities: 0 Critical, 0 High

[OUTPUT]:
{
  "release_verdict": "GO",
  "release_confidence_score": 0.98,
  "executive_summary": "All sprint quality gates passed. Zero regressions detected. Production deployment approved.",
  "risk_acknowledgements": []
}

### EXAMPLE 2 (NO-GO Decision):
[INPUT]:
QG_TEST_EXECUTION_PASS: FAILED (P0=96% - 1 failure in SCN-AUTH-P0-01)
Open Vulnerabilities: 1 High

[OUTPUT]:
{
  "release_verdict": "NO_GO",
  "release_confidence_score": 0.35,
  "executive_summary": "Release BLOCKED. Critical authentication P0 test failed. One unpatched High vulnerability.",
  "risk_acknowledgements": ["Release would introduce login outages for OAuth2 users"]
}

### TASK:
Evaluate the cumulative sprint state and render the release decision:
{sprint_state_payload}
```

### 10.4 Published Output: `RELEASE_SIGN_OFF.md`
```markdown
# 🚀 Production Release Sign-Off Report

- **Sprint**: Sprint-2026-W42
- **Date**: 2026-10-17T22:45:00Z
- **Release Gate**: `RG_PRODUCTION_RELEASE_READY`
- **Final Verdict**: **APPROVED (GO)**
- **Confidence Score**: 98.4%

## Quality Gate Scorecard
| Gate ID | Target Criterion | Actual Metric | Status |
|---|---|---|---|
| `QG_REQ_READY_FOR_DEV` | Ambiguity <= 0.10, Testability >= 0.90 | Ambiguity: 0.04, Testability: 0.96 | ✅ PASSED |
| `QG_DESIGN_REVIEW_PASS`| P0/P1 Coverage = 100% | 100% Scenario Traceability | ✅ PASSED |
| `QG_CODE_COMPILE_LINT_PASS`| Zero TypeScript & ESLint errors | 0 Errors, 0 Warnings | ✅ PASSED |
| `QG_TEST_EXECUTION_PASS`| P0 Pass = 100%, P1 >= 98% | P0: 100%, P1: 100% | ✅ PASSED |
| `RG_PRODUCTION_RELEASE_READY`| Zero Open P0 Defects | 0 Open Blockers | ✅ PASSED |

## Deployment Authorization
Signed off autonomously by `QE_Release_Gate_Agent`  
Audit Trace ID: `tr-8c638a-20261017-099`
```

---

## 11. Master Pipeline Orchestrator: QE_SDLC_Orchestrator

The orchestrator is a standalone executable (Node.js/TypeScript or Shell) that executes in CI/CD pipelines (GitHub Actions, GitLab CI, Jenkins) or locally on developer machines. It sequentially invokes agents, checks gates, handles retries, and halts execution on gate failures.

### 11.1 Orchestrator Implementation (TypeScript)

```typescript
#!/usr/bin/env node
import * as fs from 'fs';
import * as path from 'path';
import { execSync } from 'child_process';

// 1. Resolve Global Paths via Environment Variables
const CONFIG = {
  specsDir: process.env.INPUT_SPECS_DIR || path.resolve(__dirname, '../specs'),
  testRepoDir: process.env.TEST_AUTOMATION_REPO_DIR || path.resolve(__dirname, '../test-automation-repo'),
  stateDir: process.env.STATE_STORE_DIR || path.resolve(__dirname, './state'),
  logsDir: process.env.LOGS_TRACE_DIR || path.resolve(__dirname, './logs'),
  reportsDir: process.env.REPORTS_OUTPUT_DIR || path.resolve(__dirname, './reports'),
};

interface GateResult {
  gateId: string;
  verdict: 'PASSED' | 'FAILED';
  score: number;
}

class QESDLCOrchestrator {
  private stateFilePath = path.join(CONFIG.stateDir, 'sprint_state.json');

  constructor() {
    this.ensureDirectories();
  }

  private ensureDirectories(): void {
    Object.values(CONFIG).forEach(dir => {
      if (!fs.existsSync(dir)) fs.mkdirSync(dir, { recursive: true });
    });
  }

  public async runSprintPipeline(): Promise<void> {
    console.log('🏁 Starting QE SDLC Autonomous Pipeline...');

    // Phase 1: Requirements Analysis
    const reqGate = await this.executeAgent('QE_Requirements_Analysis_Agent');
    if (reqGate.verdict !== 'PASSED') return this.haltPipeline('QG_REQ_READY_FOR_DEV Failed');

    // Phase 2: Test Design
    const designGate = await this.executeAgent('QE_Test_Design_Agent');
    if (designGate.verdict !== 'PASSED') return this.haltPipeline('QG_DESIGN_REVIEW_PASS Failed');

    // Phase 3: Test Automation Code Generation
    const codeGate = await this.executeAgent('QE_Test_Automation_Agent');
    if (codeGate.verdict !== 'PASSED') return this.haltPipeline('QG_CODE_COMPILE_LINT_PASS Failed');

    // Phase 4: Test Execution (Playwright Runner)
    const execGate = await this.executeAgent('QE_Test_Execution_Agent');
    if (execGate.verdict !== 'PASSED') return this.haltPipeline('QG_TEST_EXECUTION_PASS Failed');

    // Phase 5: Release Gate Assessment
    const releaseGate = await this.executeAgent('QE_Release_Gate_Agent');
    if (releaseGate.verdict === 'PASSED') {
      console.log('🎉 SPRINT RELEASE GATE APPROVED! Production deployment authorized.');
    } else {
      console.error('⛔ RELEASE GATE REJECTED. Production deployment blocked.');
      process.exit(1);
    }
  }

  private async executeAgent(agentName: string): Promise<GateResult> {
    console.log(`\n▶️ Executing [${agentName}]...`);
    // Simulated invocation - in production this executes the agent CLI or API harness
    const cmd = `node ./dist/agents/${agentName}.js`;
    try {
      const output = execSync(cmd, {
        env: { ...process.env, ...CONFIG },
        encoding: 'utf-8',
      });
      console.log(output);
      return { gateId: `GATE_${agentName}`, verdict: 'PASSED', score: 1.0 };
    } catch (err: any) {
      console.error(`❌ Error in ${agentName}:`, err.message);
      return { gateId: `GATE_${agentName}`, verdict: 'FAILED', score: 0.0 };
    }
  }

  private haltPipeline(reason: string): void {
    console.error(`\n🚨 PIPELINE HALTED BY QUALITY GATE: ${reason}`);
    process.exit(1);
  }
}

// Execute if called directly
new QESDLCOrchestrator().runSprintPipeline();
```

---

## 12. Quality & Release Gate Matrix

| Milestone | Gate Identifier | Governing Agent | Required Metric Thresholds | Failure Action |
|---|---|---|---|---|
| **Sprint Planning** | `QG_REQ_READY_FOR_DEV` | `QE_Requirements_Analysis_Agent` | `EVAL_REQ_TESTABILITY >= 0.90`<br>`EVAL_AMBIGUITY_SCORE <= 0.10` | Escalates to Product Owner; blocks story from entering sprint backlog |
| **Sprint Architecture** | `QG_DESIGN_REVIEW_PASS` | `QE_Test_Design_Agent` | `EVAL_SCENARIO_COVERAGE == 1.0`<br>`Zero Unmapped Criteria` | Auto-prompts agent to cover missing criteria |
| **Automation Build** | `QG_CODE_COMPILE_LINT_PASS` | `QE_Test_Automation_Agent` | `tsc --noEmit == 0`<br>`eslint == 0 errors`<br>`Web-First Locators >= 95%` | Triggers self-healing repair loop (max 3 cycles) |
| **Test Execution** | `QG_TEST_EXECUTION_PASS` | `QE_Test_Execution_Agent` | `P0 Pass Rate == 100%`<br>`P1 Pass Rate >= 98%`<br>`Flakiness <= 2%` | Halts CI build; opens Jira defect ticket with traces attached |
| **Release Review** | `RG_REGRESSION_FREE_PASS` | `QE_Release_Gate_Agent` | `Zero P0/P1 Regressions`<br>`Historical Defect Leakage == 0` | Blocks staging-to-prod merge request |
| **Final Sign-Off** | `RG_PRODUCTION_RELEASE_READY`| `QE_Release_Gate_Agent` | `All Upstream QGs == PASSED`<br>`Confidence Score >= 0.95` | Blocks deployment pipeline execution |

---

## 13. End-to-End Sprint Walkthrough

```
DAY 1: SPRINT PLANNING
User Story US-2401 placed in ${INPUT_SPECS_DIR}/specs/US-2401.md.
Orchestrator runs QE_Requirements_Analysis_Agent.
Gate QG_REQ_READY_FOR_DEV evaluates score 0.96 -> PASSED.
Gherkin scenarios persisted to ${STATE_STORE_DIR}/handoffs/requirements_analysis.json.

DAY 4: TEST DESIGN
Orchestrator runs QE_Test_Design_Agent.
Scenario catalog generated with 2 P0 and 4 P1 scenarios.
Gate QG_DESIGN_REVIEW_PASS evaluates 100% coverage -> PASSED.
Artifacts saved to ${STATE_STORE_DIR}/handoffs/test_scenarios.json.

DAY 7: TEST AUTOMATION DEVELOPMENT
Orchestrator runs QE_Test_Automation_Agent.
Agent generates Playwright Page Objects and Spec files into ${TEST_AUTOMATION_REPO_DIR}/tests/.
Runs `tsc` and `eslint` -> 0 errors.
Gate QG_CODE_COMPILE_LINT_PASS -> PASSED. Git commit created.

DAY 9: CONTINUOUS TESTING
Orchestrator runs QE_Test_Execution_Agent against staging container.
All 6 Playwright tests pass (duration: 18.2s). Flakiness: 0%.
Gate QG_TEST_EXECUTION_PASS -> PASSED.
Reports written to ${REPORTS_OUTPUT_DIR}/results.xml.

DAY 10: SPRINT REVIEW & RELEASE GATE
Orchestrator runs QE_Release_Gate_Agent.
All 4 upstream gates validated.
Release Gate RG_PRODUCTION_RELEASE_READY -> APPROVED (GO).
${REPORTS_OUTPUT_DIR}/RELEASE_SIGN_OFF.md generated and attached to deployment ticket.
```

---

> **Previous**: [Volume IV — AI Tooling & Ecosystem: The Practitioner's Guide](./04-ai-tools-ecosystem.md)  
> **Next in Series**: Volume VI — Self-Healing Test Automation & Flakiness Remediation  
> **Related Reading**: [Playwright Automation Best Practices](#) | [Automated Quality Gate Design](#) | [CI/CD AI Governance](#)

---

*Quality Engineering Journals — Building confidence in AI systems through rigorous measurement.*
