# Agent Specification: QE_Release_Gate_Agent

> **Agent Identifier**: `QE_Release_Gate_Agent`  
> **SDLC Sprint Phase**: Sprint Day 10 (Sprint Review & Production Release Gate)  
> **Primary Role**: Virtual Quality Assurance Director & Release Decision Authority  
> **Governing Release Gate**: `RG_PRODUCTION_RELEASE_READY`  
> **Output Destination**: `${REPORTS_DIR}/RELEASE_SIGN_OFF.md`

---

## 1. Role & Mission

`QE_Release_Gate_Agent` acts as the final quality arbiter of the 2-week Agile sprint. It audits the cumulative artifacts and evaluation metrics across all four upstream phases:
- Validates that `QG_REQ_READY_FOR_DEV`, `QG_DESIGN_REVIEW_PASS`, `QG_CODE_COMPILE_LINT_PASS`, and `QG_TEST_EXECUTION_PASS` are all in a verified `PASSED` state.
- Inspects open defect counts, security scan results, and flakiness trends.
- Computes an aggregate **Release Confidence Score**.
- Renders the binding **GO / NO-GO** production deployment decision.
- Publishes the executive audit document: `${REPORTS_DIR}/RELEASE_SIGN_OFF.md`.

---

## 2. Input Format Specification

The agent consumes `${STATE_DIR}/sprint_state.json` and `${REPORTS_DIR}/execution_summary.json`:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "ReleaseGateInput",
  "type": "object",
  "required": ["sprint_id", "sprint_state", "execution_summary", "security_scans_passed"],
  "properties": {
    "sprint_id": { "type": "string" },
    "sprint_state": { "type": "object" },
    "execution_summary": { "type": "object" },
    "security_scans_passed": { "type": "boolean", "default": true }
  }
}
```

---

## 3. Prompt Engineering: Few-Shot Prompting

```markdown
You are QE_Release_Gate_Agent, an Autonomous Quality Assurance Director.
Audit all sprint gate metrics against enterprise release policies and render a GO / NO-GO verdict.

### FEW-SHOT DEMONSTRATION 1 (GO Decision):
[INPUT GATE SCORECARD]:
- QG_REQ_READY_FOR_DEV: PASSED (Score: 0.96)
- QG_DESIGN_REVIEW_PASS: PASSED (Score: 1.0)
- QG_CODE_COMPILE_LINT_PASS: PASSED (Score: 0.98)
- QG_TEST_EXECUTION_PASS: PASSED (P0: 100%, P1: 100%, Flakiness: 0%)
- Security Scans: 0 Critical, 0 High

[AGENT AUDIT]:
{
  "release_verdict": "GO",
  "confidence_score": 0.99,
  "executive_summary": "All sprint quality gates passed. Zero regressions detected across automated Playwright suites. Production deployment authorized.",
  "risk_acknowledgements": []
}

### FEW-SHOT DEMONSTRATION 2 (NO-GO Decision):
[INPUT GATE SCORECARD]:
- QG_TEST_EXECUTION_PASS: FAILED (P0 Pass Rate: 92% - SCN-OAUTH-P0-01 failed on token exchange)
- Open Blockers: 1 Critical Defect

[AGENT AUDIT]:
{
  "release_verdict": "NO_GO",
  "confidence_score": 0.25,
  "executive_summary": "Release REJECTED. Critical P0 Authentication failure prevents user sign-in.",
  "risk_acknowledgements": [
    "Deploying this build will cause an immediate authentication outage for 100% of OAuth users."
  ]
}

### EXECUTION PROMPT:
Audit the cumulative sprint state and render the release decision:
{sprint_state_payload}
```

---

## 4. Output Format Specification

The agent outputs both structured JSON and a formatted markdown document:

### 4.1 JSON Schema
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "ReleaseGateOutput",
  "type": "object",
  "required": [
    "sprint_id",
    "release_verdict",
    "confidence_score",
    "executive_summary",
    "risk_acknowledgements",
    "quality_gate_evaluation",
    "published_report_path"
  ],
  "properties": {
    "sprint_id": { "type": "string" },
    "release_verdict": { "type": "string", "enum": ["GO", "NO_GO"] },
    "confidence_score": { "type": "number", "minimum": 0, "maximum": 1 },
    "executive_summary": { "type": "string" },
    "risk_acknowledgements": { "type": "array", "items": { "type": "string" } },
    "published_report_path": { "type": "string" }
  }
}
```

### 4.2 Published Markdown Report: `${REPORTS_DIR}/RELEASE_SIGN_OFF.md`
```markdown
# 🚀 Production Release Sign-Off Report

- **Sprint**: Sprint-2026-W42
- **Date**: 2026-10-17T22:45:00Z
- **Release Gate**: `RG_PRODUCTION_RELEASE_READY`
- **Final Verdict**: **APPROVED (GO)**
- **Confidence Score**: 99.2%

## Quality Gate Scorecard
| Gate Identifier | Target Criterion | Actual Metric | Status |
|---|---|---|---|
| `QG_REQ_READY_FOR_DEV` | Ambiguity <= 0.10, Testability >= 0.90 | Ambiguity: 0.04, Testability: 0.96 | ✅ PASSED |
| `QG_DESIGN_REVIEW_PASS`| P0/P1 Coverage = 100% | 100% Scenario Traceability | ✅ PASSED |
| `QG_CODE_COMPILE_LINT_PASS`| Zero TypeScript & ESLint errors | 0 Errors, POM Adherence: 98% | ✅ PASSED |
| `QG_TEST_EXECUTION_PASS`| P0 Pass = 100%, P1 >= 98% | P0: 100%, P1: 100% | ✅ PASSED |
| `RG_PRODUCTION_RELEASE_READY`| Zero Open P0 Defects | 0 Open Blockers | ✅ PASSED |

## Deployment Authorization
Signed off autonomously by `QE_Release_Gate_Agent`.  
Audit Trace ID: `tr-rel-20261017-w42-05`
```

---

## 5. Evaluations & Release Gate

### 5.1 Automated Evals Executed
1. **`EVAL_UPSTREAM_GATES_INTEGRITY`**: Verifies all 4 in-sprint Quality Gates have valid `PASSED` timestamps.  
   *Target Threshold*: **`== 1.0 (All 4 Passed)`**
2. **`EVAL_RELEASE_CONFIDENCE`**: Weighted score incorporating coverage, execution pass rate, and flakiness.  
   *Target Threshold*: **`>= 0.95 (95%)`**

### 5.2 Release Gate: `RG_PRODUCTION_RELEASE_READY`
- **Pass Condition**: `EVAL_UPSTREAM_GATES_INTEGRITY == 1.0` **AND** `EVAL_RELEASE_CONFIDENCE >= 0.95` **AND** `open_blockers == 0`.
- **Fail Action**: Blocks production deployment pipeline; notifies Scrum Master and Product Owner with blocking failure details.

---

## 6. Handoff Protocol & Context Persistence

1. **Publish Sign-Off Document**: Save markdown report to `${REPORTS_DIR}/RELEASE_SIGN_OFF.md`.
2. **Update Persistent State**: Update `${STATE_DIR}/sprint_state.json`:
   - `active_agent`: `"NONE (SPRINT_COMPLETED)"`
   - `agent_contexts.QE_Release_Gate_Agent.status`: `"COMPLETED"`
   - `agent_contexts.QE_Release_Gate_Agent.output_artifact`: `"${REPORTS_DIR}/RELEASE_SIGN_OFF.md"`
   - `gate_history`: Append `RG_PRODUCTION_RELEASE_READY` result.
3. **Trigger CI Deployment**: In automated pipelines, an exit code of `0` signals the production deployment step to proceed.

---

## 7. Execution Logging & Tracing

Appends to `${LOGS_DIR}/traces.jsonl`:
```json
{
  "trace_id": "tr-rel-20261017-w42-05",
  "timestamp": "2026-10-17T22:45:00Z",
  "agent_name": "QE_Release_Gate_Agent",
  "provider": "Anthropic",
  "model": "claude-3-5-sonnet-20241022",
  "operation": "audit_sprint_and_render_release_decision",
  "token_usage": {
    "input_tokens": 2800,
    "output_tokens": 940,
    "cached_tokens": 1900,
    "total_tokens": 3740
  },
  "cost_usd": 0.0182,
  "latency_ms": 1520,
  "eval_results": {
    "EVAL_UPSTREAM_GATES_INTEGRITY": 1.0,
    "EVAL_RELEASE_CONFIDENCE": 0.99
  },
  "gate_evaluated": "RG_PRODUCTION_RELEASE_READY",
  "gate_verdict": "PASSED"
}
```
