# Configuration: Global Path Variables & Multi-Repo Resolution

> **Module**: Path Architecture & Resolution Specification  
> **Parent**: [`qe-agentic-orchestration/README.md`](../README.md)

---

## 1. Purpose

To eliminate hardcoded paths across agents, this specification establishes **universal path placeholders**. Every agent and the orchestrator consume these variables, allowing seamless execution across local developer workstations, Docker test runners, and multi-repo CI/CD environments.

---

## 2. Global Path Placeholders

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                              PATH RESOLUTION HIERARCHY                                 │
│                                                                                        │
│   ENVIRONMENT VARIABLE             DEFAULT RESOLUTION (RELATIVE TO THIS REPO)          │
│   ────────────────────             ──────────────────────────────────────────          │
│   ${INPUT_DIR}               ──▶   ./inputs/specs  OR  ../app-repo/specs               │
│   ${TESTS_DIR}               ──▶   ../test-automation-repo                             │
│   ${STATE_DIR}               ──▶   ./state                                             │
│   ${LOGS_DIR}                ──▶   ./logs                                              │
│   ${REPORTS_DIR}             ──▶   ./reports                                           │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

### Detailed Path Table

| Variable Placeholder | Target Entity | Expected File Types | Access Mode |
|---|---|---|---|
| **`${INPUT_DIR}`** | Product specifications, PRDs, user stories, OpenAPI/Swagger docs | `.md`, `.json`, `.yaml` | **Read-Only** |
| **`${TESTS_DIR}`** | External Playwright + TypeScript test automation repository | `.ts`, `.spec.ts`, `playwright.config.ts` | **Read & Write** |
| **`${STATE_DIR}`** | Persistent sprint state and inter-agent handoff contracts | `sprint_state.json`, `handoffs/*.json` | **Read & Write** |
| **`${LOGS_DIR}`** | Provider and model telemetry, execution traces, token costs | `traces.jsonl` | **Append-Only** |
| **`${REPORTS_DIR}`**| Quality Gate audit summaries, JUnit results, release sign-offs | `RELEASE_SIGN_OFF.md`, `results.xml` | **Write-Only** |

---

## 3. Sub-Directory Convention inside `${TESTS_DIR}`

When **`QE_Test_Automation_Agent`** generates code in the test automation repository, it targets these exact directories:

- **`${TESTS_DIR}/pages/`**: Page Object Model classes (e.g., `LoginPage.ts`, `DashboardPage.ts`).
- **`${TESTS_DIR}/tests/`**: Playwright test specification files (e.g., `oauth-login.spec.ts`).
- **`${TESTS_DIR}/fixtures/`**: Custom test fixtures (e.g., `baseTest.ts`).
- **`${TESTS_DIR}/playwright.config.ts`**: Playwright test runner configuration.

---

## 4. Multi-Repo Setup Scenarios

### Scenario A: Local Development (Side-by-Side Sibling Repos)
```bash
/workspace
  ├── app-repo/specs/                     # ${INPUT_DIR}
  ├── qe-agentic-orchestration/           # THIS REPO
  └── test-automation-repo/               # ${TESTS_DIR} (Playwright + TS)
```
Environment exports:
```bash
export INPUT_DIR="../app-repo/specs"
export TESTS_DIR="../test-automation-repo"
export STATE_DIR="./state"
export LOGS_DIR="./logs"
export REPORTS_DIR="./reports"
```

### Scenario B: Unified CI/CD Runner (GitHub Actions / GitLab CI)
```yaml
env:
  INPUT_DIR: ${{ github.workspace }}/specs
  TESTS_DIR: ${{ github.workspace }}/e2e-tests
  STATE_DIR: ${{ github.workspace }}/qe-state
  LOGS_DIR: ${{ github.workspace }}/qe-logs
  REPORTS_DIR: ${{ github.workspace }}/qe-reports
```

---

## 5. Path Validation Checklist for Agents

Before executing an operation, every agent verifies:
1. `[ -d "${INPUT_DIR}" ]` — Input directory exists and contains target story markdown.
2. `[ -d "${TESTS_DIR}" ]` — Test automation repository exists with write permissions.
3. `[ -d "${STATE_DIR}/handoffs" ]` — State directory is initialized for cross-agent JSON handoffs.
4. `[ -d "${LOGS_DIR}" ]` — Logging directory is writable for trace logging.
