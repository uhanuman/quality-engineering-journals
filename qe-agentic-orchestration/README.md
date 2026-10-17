# Quality Engineering Agentic Orchestration Repository

> **Repository**: `qe-agentic-orchestration`  
> **Purpose**: Custom AI Agents for Quality Engineering in an AI-Assisted Agile SDLC (2-Week Sprint)  
> **Architecture**: Modular Markdown Agent Specifications, State Persistence, and Gate-Governed Orchestration  
> **Companions**: [QE Journal Vol V — SDLC Agents](../ai-fundamentals/05-sdlc-agents-and-orchestration.md)

---

## 1. Executive Summary

This repository houses the complete **Quality Engineering AI Agent Ecosystem** for an Agile Scrum team operating on two-week sprint cycles. 

Instead of treating AI as an ad-hoc conversational chatbot, this repository formalizes each Quality Engineering capability into a **dedicated, autonomous Agent document** with:
1. Strict **Input and Output formats** (typed schemas).
2. Embedded **Few-Shot Prompting** templates with concrete demonstrations.
3. Automated **Evals, Quality Gates (`QG_*`), and Release Gates (`RG_*`)** with numerical thresholds.
4. Defined **Handoff protocols** passing structured JSON state from one sprint phase to the next.
5. Persistent **Sprint Context & State** committed to the repo so team members can inspect, resume, or replay steps.
6. Comprehensive **Trace & Audit Logging** capturing providers, models, token consumption, latency, and gate verdicts.

---

## 2. Multi-Repository Architecture

The agents in this repository operate across a decoupled multi-repository landscape using configurable global path variables:

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                              MULTI-REPOSITORY LANDSCAPE                                │
│                                                                                        │
│   APP / SPECS REPOSITORY               THIS REPO: qe-agentic-orchestration             │
│   ┌─────────────────────────────┐      ┌───────────────────────────────────────────┐   │
│   │ ${INPUT_DIR}                │      │ /config                                   │   │
│   │   ├── PRD-Sprint-24.md      │◀─┐   │   └── environment-paths.md                │   │
│   │   └── user-stories.json     │  │   │ /agents                                   │   │
│   │ /src (App Source Code)      │  │   │   ├── QE_Requirements_Analysis_Agent.md   │   │
│   └─────────────────────────────┘  │   │   ├── QE_Test_Design_Agent.md             │   │
│                                    │   │   ├── QE_Test_Automation_Agent.md         │   │
│                                    ├───│   ├── QE_Test_Execution_Agent.md          │   │
│   TEST AUTOMATION REPOSITORY       │   │   └── QE_Release_Gate_Agent.md            │   │
│   (Playwright + TypeScript)        │   │ /orchestrator                             │   │
│   ┌─────────────────────────────┐  │   │   └── QE_SDLC_Orchestrator.md             │   │
│   │ ${TESTS_DIR}                │◀─┘   │ /gates                                    │   │
│   │   ├── /pages (Page Objects) │      │   └── Quality_And_Release_Gates.md        │   │
│   │   ├── /tests (Specs)        │      │ /state                                    │   │
│   │   └── playwright.config.ts  │      │   └── sprint_state_and_handoffs.md         │   │
│   └─────────────────────────────┘      │ /logs                                     │   │
│                                        │   └── execution_tracing_and_audit.md      │   │
│                                        └───────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Global Path Placeholders

No agent hardcodes filesystem paths. All references consume standardized path variables:

| Variable | Description | Default Fallback |
|---|---|---|
| `${INPUT_DIR}` | Directory containing requirements, PRDs, and user stories | `./inputs/specs` or `../app-repo/specs` |
| `${TESTS_DIR}` | Root of the standalone Playwright + TypeScript test repository | `../test-automation-repo` |
| `${STATE_DIR}` | Directory where cross-agent sprint state and handoffs reside | `./state` |
| `${LOGS_DIR}` | Directory where trace logs (`traces.jsonl`) are recorded | `./logs` |
| `${REPORTS_DIR}`| Directory where Quality Gate reports and sign-offs are published | `./reports` |

---

## 4. Two-Week Sprint Lifecycle Mapping

| Sprint Timeline | Responsible Agent | Phase Deliverable | Governing Gate |
|---|---|---|---|
| **Days 1–2** | [`QE_Requirements_Analysis_Agent`](./agents/QE_Requirements_Analysis_Agent.md) | Gherkin BDD Acceptance Criteria & Risk Audit | `QG_REQ_READY_FOR_DEV` |
| **Days 3–5** | [`QE_Test_Design_Agent`](./agents/QE_Test_Design_Agent.md) | P0/P1/P2 Test Scenario Catalog & Test Strategy | `QG_DESIGN_REVIEW_PASS` |
| **Days 5–8** | [`QE_Test_Automation_Agent`](./agents/QE_Test_Automation_Agent.md) | Playwright TypeScript Page Objects & Spec Files | `QG_CODE_COMPILE_LINT_PASS` |
| **Days 8–9** | [`QE_Test_Execution_Agent`](./agents/QE_Test_Execution_Agent.md) | CI Execution Results, Traces & Defect Triage | `QG_TEST_EXECUTION_PASS` |
| **Day 10** | [`QE_Release_Gate_Agent`](./agents/QE_Release_Gate_Agent.md) | Production Release Audit & Sign-Off Report | `RG_PRODUCTION_RELEASE_READY` |
| **Continuous** | [`QE_SDLC_Orchestrator`](./orchestrator/QE_SDLC_Orchestrator.md) | End-to-end automated pipeline runner & gatekeeper | *Enforces all gates* |

---

## 5. Repository Structure

```
qe-agentic-orchestration/
├── README.md                                   # This repository index
├── config/
│   └── environment-paths.md                    # Global path variables and resolution rules
├── agents/
│   ├── QE_Requirements_Analysis_Agent.md       # Phase 1: Requirements testability & Gherkin
│   ├── QE_Test_Design_Agent.md                 # Phase 2: Scenario design & test strategy
│   ├── QE_Test_Automation_Agent.md             # Phase 3: Playwright TypeScript code generation
│   ├── QE_Test_Execution_Agent.md              # Phase 4: Execution runner & defect triage
│   └── QE_Release_Gate_Agent.md                # Phase 5: Production Go/No-Go sign-off
├── orchestrator/
│   └── QE_SDLC_Orchestrator.md                 # Automated pipeline loop & gatekeeper
├── gates/
│   └── Quality_And_Release_Gates.md            # Detailed QG_* and RG_* threshold catalog
├── state/
│   └── sprint_state_and_handoffs.md            # State persistence schema & handoffs
└── logs/
    └── execution_tracing_and_audit.md          # Trace schema, token auditing & cost logs
```
