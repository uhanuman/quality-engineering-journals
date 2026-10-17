# State & Context Persistence Specification

> **Module**: Cross-Agent Memory & Handoff Schema  
> **Parent**: [`qe-agentic-orchestration/README.md`](../README.md)

---

## 1. Why State Persistence Matters

In a distributed engineering team, different engineers or CI runners execute different stages of the sprint:
- QA Architect runs **Phase 1** during refinement on Monday.
- SDET runs **Phase 3** on Thursday in their branch.
- CI Runner executes **Phase 4** on Friday night.
- Release Manager reviews **Phase 5** on the second Wednesday.

By committing state and handoffs to the repository in `${STATE_DIR}/`, any team member can pull the repository and **instantly inspect, resume, or replay** any phase of the agent pipeline with full context.

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        STATE & HANDOFF PERSISTENCE TOPOLOGY                            │
│                                                                                        │
│   ${STATE_DIR}/sprint_state.json (Master Single Source of Truth)                       │
│   ├── Active Agent Pointer (where is the pipeline currently?)                          │
│   ├── Cumulative Gate Evaluation History                                               │
│   └── Agent Context & Execution Checkpoints                                            │
│                                                                                        │
│   ${STATE_DIR}/handoffs/ (Structured Payloads Passed Between Agents)                   │
│   ├── requirements_analysis.json ──▶ Phase 1 output consumed by Phase 2               │
│   ├── test_scenarios.json        ──▶ Phase 2 output consumed by Phase 3               │
│   └── test_automation.json       ──▶ Phase 3 receipt consumed by Phase 4              │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Master State Schema: `${STATE_DIR}/sprint_state.json`

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "sprint_id": "Sprint-2026-W42",
  "sprint_name": "OAuth2 Social Login & Security Hardening",
  "timestamp": "2026-10-17T22:30:00Z",
  "active_agent": "QE_Test_Automation_Agent",
  "sprint_goal": "Implement and verify OAuth2 Social Login with automated Playwright suites",
  "paths": {
    "specs_dir": "${INPUT_DIR}",
    "test_automation_dir": "${TESTS_DIR}",
    "state_dir": "${STATE_DIR}",
    "logs_dir": "${LOGS_DIR}",
    "reports_dir": "${REPORTS_DIR}"
  },
  "gate_history": [
    {
      "gate_id": "QG_REQ_READY_FOR_DEV",
      "verdict": "PASSED",
      "score": 0.96,
      "timestamp": "2026-10-18T10:14:22Z",
      "evaluator": "QE_Requirements_Analysis_Agent"
    },
    {
      "gate_id": "QG_DESIGN_REVIEW_PASS",
      "verdict": "PASSED",
      "score": 1.0,
      "timestamp": "2026-10-20T16:45:10Z",
      "evaluator": "QE_Test_Design_Agent"
    }
  ],
  "agent_contexts": {
    "QE_Requirements_Analysis_Agent": {
      "status": "COMPLETED",
      "started_at": "2026-10-18T10:12:00Z",
      "completed_at": "2026-10-18T10:14:22Z",
      "output_artifact": "${STATE_DIR}/handoffs/requirements_analysis.json",
      "last_trace_id": "tr-req-20261017-w42-01"
    },
    "QE_Test_Design_Agent": {
      "status": "COMPLETED",
      "started_at": "2026-10-20T16:42:00Z",
      "completed_at": "2026-10-20T16:45:10Z",
      "output_artifact": "${STATE_DIR}/handoffs/test_scenarios.json",
      "last_trace_id": "tr-dsn-20261017-w42-02"
    },
    "QE_Test_Automation_Agent": {
      "status": "IN_PROGRESS",
      "started_at": "2026-10-22T09:00:00Z",
      "completed_at": null,
      "output_artifact": null,
      "last_trace_id": "tr-auto-20261017-w42-03"
    },
    "QE_Test_Execution_Agent": {
      "status": "NOT_STARTED",
      "output_artifact": null
    },
    "QE_Release_Gate_Agent": {
      "status": "NOT_STARTED",
      "output_artifact": null
    }
  }
}
```

---

## 3. Team Member Context Continuation Workflow

When a team member begins work in the sprint:
```bash
# 1. Pull latest branch with state updates
git pull origin sprint-24

# 2. Inspect active sprint status
cat state/sprint_state.json | jq '.active_agent, .gate_history'

# 3. Resume pipeline from current active agent
# The orchestrator automatically skips completed phases and starts at active_agent!
```
