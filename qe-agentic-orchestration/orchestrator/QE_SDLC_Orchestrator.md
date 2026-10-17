# Master Orchestrator Specification: QE_SDLC_Orchestrator

> **Component**: `QE_SDLC_Orchestrator`  
> **Role**: Autonomous Pipeline Governor & Gatekeeper  
> **Target Cycle**: 2-Week Agile Scrum Sprint  
> **Parent**: [`qe-agentic-orchestration/README.md`](../README.md)

---

## 1. Orchestration Architecture & Workflow

`QE_SDLC_Orchestrator` is the central control loop that automates the end-to-end Quality Engineering sprint lifecycle. It sequentially invokes the specialized agents, validates Quality and Release Gates, controls handoffs, logs execution traces, and halts execution if any gate fails.

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        ORCHESTRATOR CONTROL LOOP & STATE MACHINE                       │
│                                                                                        │
│   START PIPELINE                                                                       │
│         │                                                                              │
│         ▼                                                                              │
│   [Init Directories & Load State] ──▶ ${STATE_DIR}/sprint_state.json                   │
│         │                                                                              │
│         ▼                                                                              │
│   PHASE 1: Run QE_Requirements_Analysis_Agent                                          │
│         │                                                                              │
│         ├──▶ Evaluate: QG_REQ_READY_FOR_DEV                                            │
│         │        ├── FAILED ──▶ [HALT PIPELINE] ──▶ Notify Product Owner               │
│         │        └── PASSED ──▶ Write Handoff 1                                        │
│         ▼                                                                              │
│   PHASE 2: Run QE_Test_Design_Agent                                                    │
│         │                                                                              │
│         ├──▶ Evaluate: QG_DESIGN_REVIEW_PASS                                           │
│         │        ├── FAILED ──▶ [HALT PIPELINE] ──▶ Re-prompt agent for missing criteria│
│         │        └── PASSED ──▶ Write Handoff 2                                        │
│         ▼                                                                              │
│   PHASE 3: Run QE_Test_Automation_Agent                                                │
│         │    (Generates Playwright TS POMs & Specs into ${TESTS_DIR})                  │
│         │                                                                              │
│         ├──▶ Evaluate: QG_CODE_COMPILE_LINT_PASS                                       │
│         │        ├── FAILED ──▶ [Self-Healing Loop: Max 3 Reprompts]                   │
│         │        │                 └── Exhausted? ──▶ [HALT PIPELINE]                  │
│         │        └── PASSED ──▶ Git commit to ${TESTS_DIR} & Write Handoff 3           │
│         ▼                                                                              │
│   PHASE 4: Run QE_Test_Execution_Agent                                                 │
│         │    (Executes Playwright tests against staging environment)                   │
│         │                                                                              │
│         ├──▶ Evaluate: QG_TEST_EXECUTION_PASS                                          │
│         │        ├── FAILED ──▶ [HALT PIPELINE] ──▶ Open Jira Blocker with Traces      │
│         │        └── PASSED ──▶ Write Execution Summary to ${REPORTS_DIR}              │
│         ▼                                                                              │
│   PHASE 5: Run QE_Release_Gate_Agent                                                    │
│         │                                                                              │
│         ├──▶ Evaluate: RG_PRODUCTION_RELEASE_READY                                     │
│         │        ├── NO_GO  ──▶ [BLOCK RELEASE] ──▶ Publish NO_GO Audit Report        │
│         │        └── GO     ──▶ [APPROVE RELEASE] ──▶ Publish RELEASE_SIGN_OFF.md      │
│         ▼                                                                              │
│   SUCCESSFUL SPRINT COMPLETION                                                         │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Gate Governing & Circuit Breaker Logic

The orchestrator enforces the **Circuit Breaker Pattern**:
- If an upstream gate fails, downstream agents are **never invoked**. This prevents compiling tests against ambiguous requirements or deploying code with failing tests.
- Each gate evaluation records a structured entry in `sprint_state.json` and `traces.jsonl`.

### Gate Enforcement Policy Table
| Phase | Evaluated Gate | Halting Threshold | Remediation / Circuit Breaker Action |
|---|---|---|---|
| **Phase 1** | `QG_REQ_READY_FOR_DEV` | `Testability < 0.90` OR `Ambiguity > 0.10` | Halts pipeline; blocks user story from sprint backlog |
| **Phase 2** | `QG_DESIGN_REVIEW_PASS` | `Coverage < 100%` OR `Unmapped > 0` | Prompts agent to cover orphan requirements |
| **Phase 3** | `QG_CODE_COMPILE_LINT_PASS` | `tsc != 0` OR `eslint != 0` | Triggers self-healing repair cycle (up to 3 iterations) |
| **Phase 4** | `QG_TEST_EXECUTION_PASS` | `P0 Pass < 100%` OR `Flakiness > 2%` | Halts CI build; opens Jira defect ticket with traces |
| **Phase 5** | `RG_PRODUCTION_RELEASE_READY` | Upstream gate failed OR Open Blockers > 0 | Rejects release; generates `NO_GO` audit sign-off |

---

## 3. Self-Healing Automation Repair Loop

During **Phase 3 (`QE_Test_Automation_Agent`)**, compilation and linting failures trigger an automated self-healing loop:

```
┌────────────────────────────────────────────────────────────────────────┐
│                      SELF-HEALING REPAIR LOOP                          │
│                                                                        │
│   Generated Code in ${TESTS_DIR}                                       │
│         │                                                              │
│         ▼                                                              │
│   Run: npx tsc --noEmit && npx eslint                                  │
│         │                                                              │
│         ├── Exited Code 0 ──▶ QG_CODE_COMPILE_LINT_PASS = PASSED       │
│         │                                                              │
│         └── Errors Detected (Attempt <= 3)                             │
│                 │                                                      │
│                 ▼                                                      │
│             Feed compiler error stdout + original code                 │
│             back into QE_Test_Automation_Agent:                        │
│             "Fix these specific TypeScript / ESLint errors..."         │
│                 │                                                      │
│                 ▼                                                      │
│             Overwrite corrected files in ${TESTS_DIR}                  │
│                 │                                                      │
│                 └── Re-evaluate Gate (Attempt + 1)                     │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 4. CI/CD Pipeline Automation: GitHub Actions Specification

The orchestrator can be executed in GitHub Actions via this workflow specification:

```yaml
# .github/workflows/qe-agentic-pipeline.yml
name: QE Agentic Sprint Orchestrator

on:
  schedule:
    - cron: '0 2 * * 1' # Every Monday morning of the sprint
  workflow_dispatch:
    inputs:
      target_story_id:
        description: 'Target Story Key (e.g., US-2401)'
        required: true
        default: 'US-2401'

jobs:
  run-qe-orchestrator:
    runs-on: ubuntu-latest
    env:
      INPUT_DIR: ${{ github.workspace }}/specs
      TESTS_DIR: ${{ github.workspace }}/test-automation-repo
      STATE_DIR: ${{ github.workspace }}/state
      LOGS_DIR: ${{ github.workspace }}/logs
      REPORTS_DIR: ${{ github.workspace }}/reports
      TARGET_STORY_ID: ${{ github.event.inputs.target_story_id }}

    steps:
      - name: Checkout Agents Repository
        uses: actions/checkout@v4

      - name: Checkout Test Automation Repository
        uses: actions/checkout@v4
        with:
          repository: 'my-org/test-automation-repo'
          path: 'test-automation-repo'
          token: ${{ secrets.BOT_GITHUB_TOKEN }}

      - name: Initialize Environment & State Directories
        run: |
          mkdir -p ${INPUT_DIR} ${STATE_DIR}/handoffs ${LOGS_DIR} ${REPORTS_DIR}
          echo "Directories initialized."

      - name: Run Phase 1 - Requirements Analysis
        run: |
          echo "Executing QE_Requirements_Analysis_Agent..."
          # Evaluates QG_REQ_READY_FOR_DEV

      - name: Run Phase 2 - Test Design
        run: |
          echo "Executing QE_Test_Design_Agent..."
          # Evaluates QG_DESIGN_REVIEW_PASS

      - name: Run Phase 3 - Test Automation Code Generation
        run: |
          echo "Executing QE_Test_Automation_Agent..."
          # Writes Playwright tests directly into ${TESTS_DIR}
          # Evaluates QG_CODE_COMPILE_LINT_PASS

      - name: Run Phase 4 - Playwright Test Execution
        run: |
          cd ${TESTS_DIR}
          npm ci
          npx playwright install --with-deps
          npx playwright test --reporter=json:${REPORTS_DIR}/report.json
          # Evaluates QG_TEST_EXECUTION_PASS

      - name: Run Phase 5 - Production Release Gate Decision
        run: |
          echo "Executing QE_Release_Gate_Agent..."
          # Evaluates RG_PRODUCTION_RELEASE_READY
          # Publishes ${REPORTS_DIR}/RELEASE_SIGN_OFF.md

      - name: Publish Release Sign-Off Artifacts
        uses: actions/upload-artifact@v4
        with:
          name: release-gate-report
          path: ${{ env.REPORTS_DIR }}/RELEASE_SIGN_OFF.md
```
