# Quality & Release Gates Master Specification

> **Module**: Enterprise Gate Catalog  
> **Parent**: [`qe-agentic-orchestration/README.md`](../README.md)

---

## 1. Overview

Quality Gates (`QG_*`) represent in-sprint milestone checkpoints that must be validated before an artifact can progress to the next phase. Release Gates (`RG_*`) represent pre-deployment governance checkpoints required to authorize production release.

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        QUALITY & RELEASE GATES HIERARCHY                               │
│                                                                                        │
│   PHASE 1                   PHASE 2             PHASE 3               PHASE 4          │
│   ┌────────────────────┐   ┌─────────────────┐ ┌───────────────────┐ ┌───────────────┐ │
│   │QG_REQ_READY_FOR_DEV│──▶│QG_DESIGN_REVIEW │─▶QG_CODE_COMPILE_   │─▶QG_TEST_EXECU- │ │
│   │                    │   │   _PASS         │ │   LINT_PASS       │ │  TION_PASS    │ │
│   └────────────────────┘   └─────────────────┘ └───────────────────┘ └───────────────┘ │
│                                                                              │         │
│   FINAL SPRINT GATE (Day 10)                                                 ▼         │
│   ┌────────────────────────────────────────────────────────────────────────────────┐   │
│   │                         RG_PRODUCTION_RELEASE_READY                            │   │
│   └────────────────────────────────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. In-Sprint Quality Gates (`QG_*`) Catalog

### 2.1 `QG_REQ_READY_FOR_DEV`
- **Sprint Phase**: Days 1–2 (Sprint Planning & Backlog Refinement)
- **Governing Agent**: `QE_Requirements_Analysis_Agent`
- **Objective**: Ensure requirements are unambiguous, testable, and have complete Gherkin acceptance criteria before developers write code.
- **Evaluation Metrics & Formulas**:
  - `EVAL_REQ_TESTABILITY`: $\frac{\text{Testable Criteria}}{\text{Total Criteria}} \ge 0.90$
  - `EVAL_AMBIGUITY_SCORE`: $\frac{\text{Ambiguous Phrases}}{\text{Total Sentences}} \le 0.10$
  - `GHERKIN_SCENARIO_COUNT`: $\ge 1$
- **Pass Threshold**: `EVAL_REQ_TESTABILITY >= 0.90` AND `EVAL_AMBIGUITY_SCORE <= 0.10`.
- **Fail Action**: Blocks story from entering the sprint; alerts Product Owner with specific ambiguity reasons.

---

### 2.2 `QG_DESIGN_REVIEW_PASS`
- **Sprint Phase**: Days 3–5 (Test Design & Strategy)
- **Governing Agent**: `QE_Test_Design_Agent`
- **Objective**: Ensure 100% of acceptance criteria are decomposed into prioritized P0/P1/P2 test scenarios with explicit Page Object Model mappings.
- **Evaluation Metrics & Formulas**:
  - `EVAL_SCENARIO_COVERAGE`: $\frac{\text{Mapped Criteria}}{\text{Total Acceptance Criteria}} == 1.0 \ (100\%)$
  - `UNMAPPED_CRITERIA_COUNT`: $== 0$
  - `P0_SCENARIO_COUNT`: $\ge 1$
- **Pass Threshold**: 100% coverage of P0 and P1 criteria with zero orphans.
- **Fail Action**: Re-prompts the test design agent with the list of missing criteria; halts automation until resolved.

---

### 2.3 `QG_CODE_COMPILE_LINT_PASS`
- **Sprint Phase**: Days 5–8 (Test Automation Development)
- **Governing Agent**: `QE_Test_Automation_Agent`
- **Objective**: Ensure generated Playwright + TypeScript code strictly compiles, adheres to ESLint rules, and utilizes web-first locators without XPath.
- **Evaluation Metrics & Formulas**:
  - `EVAL_CODE_SYNTAX_VALIDITY`: `npx tsc --noEmit` exit code $== 0$
  - `EVAL_CODE_LINT_PASS`: `npx eslint` error count $== 0$
  - `EVAL_POM_COMPLIANCE`: $\frac{\text{Locators encapsulated in Page Objects}}{\text{Total Locators}} \ge 0.95$
- **Pass Threshold**: Clean compile, 0 lint errors, and $\ge 95\%$ POM encapsulation.
- **Fail Action**: Triggers automated self-healing loop (up to 3 iterations) feeding compiler errors back into model.

---

### 2.4 `QG_TEST_EXECUTION_PASS`
- **Sprint Phase**: Days 8–9 (Continuous Testing & Failure Triage)
- **Governing Agent**: `QE_Test_Execution_Agent`
- **Objective**: Ensure automated Playwright tests pass against target staging/preview environments with zero blockers and minimal flakiness.
- **Evaluation Metrics & Formulas**:
  - `EVAL_P0_PASS_RATE`: $\frac{\text{Passed P0 Tests}}{\text{Total P0 Tests}} == 1.0 \ (100\%)$
  - `EVAL_P1_PASS_RATE`: $\frac{\text{Passed P1 Tests}}{\text{Total P1 Tests}} \ge 0.98 \ (98\%)$
  - `EVAL_FLAKINESS_INDEX`: $\frac{\text{Tests Passing Only on Retry}}{\text{Total Tests Executed}} \le 0.02 \ (2\%)$
- **Pass Threshold**: 100% P0 pass, $\ge 98\%$ P1 pass, $\le 2\%$ flakiness index.
- **Fail Action**: Halts CI pipeline; creates Jira defect ticket with traces, screenshots, and failure categorization.

---

## 3. Pre-Deployment Release Gates (`RG_*`) Catalog

### 3.1 `RG_REGRESSION_FREE_PASS`
- **Sprint Phase**: Day 10 (Sprint Review)
- **Governing Agent**: `QE_Release_Gate_Agent`
- **Objective**: Verify that existing regression suites across unaffected domains remain 100% passing.
- **Pass Threshold**: Zero regression test failures across baseline suites.

---

### 3.2 `RG_PRODUCTION_RELEASE_READY`
- **Sprint Phase**: Day 10 (Final Deployment Authorization)
- **Governing Agent**: `QE_Release_Gate_Agent`
- **Objective**: Aggregate all in-sprint and regression data to issue the definitive **GO / NO-GO** release sign-off.
- **Evaluation Metrics & Formulas**:
  - `ALL_UPSTREAM_GATES_PASSED`: `QG_REQ_READY_FOR_DEV` AND `QG_DESIGN_REVIEW_PASS` AND `QG_CODE_COMPILE_LINT_PASS` AND `QG_TEST_EXECUTION_PASS` $== \text{PASSED}$
  - `OPEN_BLOCKER_DEFECTS`: $== 0$
  - `EVAL_RELEASE_CONFIDENCE`: Weighted composite score $\ge 0.95 \ (95\%)$
- **Pass Threshold**: All upstream gates passed, 0 open blockers, confidence score $\ge 0.95$.
- **Fail Action**: Blocks production deployment pipeline; publishes `NO_GO` audit report to `${REPORTS_DIR}/RELEASE_SIGN_OFF.md`.
