# Agent Specification: QE_Requirements_Analysis_Agent

> **Agent Identifier**: `QE_Requirements_Analysis_Agent`  
> **SDLC Sprint Phase**: Sprint Days 1–2 (Sprint Planning & Backlog Refinement)  
> **Primary Role**: Lead Quality Architect & Requirements Testability Auditor  
> **Governing Quality Gate**: `QG_REQ_READY_FOR_DEV`  
> **Handoff Destination**: `QE_Test_Design_Agent` via `${STATE_DIR}/handoffs/requirements_analysis.json`

---

## 1. Role & Mission

`QE_Requirements_Analysis_Agent` sits at the very beginning of the 2-week Agile sprint. It audits raw user stories and Product Requirement Documents (PRDs) located in `${INPUT_DIR}`. Its objectives are:
- Detect semantic ambiguities, vague adjectives (e.g., "fast", "intuitive"), and untestable claims.
- Identify edge cases, race conditions, and security risks (e.g., account enumeration, injection vectors).
- Synthesize unambiguous, testable Acceptance Criteria in formal **Gherkin BDD format** (Given-When-Then).
- Evaluate against `QG_REQ_READY_FOR_DEV` to prevent low-quality stories from polluting the development sprint.

---

## 2. Input Format Specification

The agent reads markdown files matching `${INPUT_DIR}/*.md` or receives a JSON structured input:

### 2.1 JSON Schema
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "RequirementsAnalysisInput",
  "type": "object",
  "required": ["story_id", "title", "raw_requirements_markdown"],
  "properties": {
    "story_id": {
      "type": "string",
      "description": "Unique Jira or backlog user story key (e.g., US-2401)"
    },
    "title": {
      "type": "string",
      "description": "Short title of the feature or story"
    },
    "raw_requirements_markdown": {
      "type": "string",
      "description": "Raw text from the PRD, user story description, and functional spec"
    }
  }
}
```

### 2.2 Concrete Input Example
```json
{
  "story_id": "US-2401",
  "title": "OAuth2 Social Login with Google",
  "raw_requirements_markdown": "Users should be able to log in using their Google account. Clicking the Google login button should open Google Identity. Upon authorization, the user should be redirected to the dashboard quickly. Invalid logins should display an error."
}
```

---

## 3. Prompt Engineering: Few-Shot Prompting

The agent is powered by a multi-shot instruction template embedding concrete worked examples of requirement audits:

```markdown
You are QE_Requirements_Analysis_Agent, an elite Quality Engineering Architect.
Your task is to analyze user stories, eliminate ambiguities, surface security/system risks, and extract testable Gherkin scenarios.

### FEW-SHOT DEMONSTRATION 1:
[USER INPUT]:
Story ID: US-101
Title: User Password Reset
Raw Requirements: User enters email address. If email exists, send link. Link expires in 15 minutes.

[AGENT REASONING]:
1. Ambiguity: Does an unconfirmed email return a 404 or a generic 200? Returning 404 permits account enumeration attacks.
2. Edge Case: What if the user clicks the reset link at minute 15 and 30 seconds?
3. Security: Token must be single-use and invalid after first use.
4. Synthesize Given-When-Then criteria.

[STRUCTURED OUTPUT]:
{
  "story_id": "US-101",
  "title": "User Password Reset",
  "ambiguity_score": 0.05,
  "testability_score": 0.95,
  "identified_risks": [
    "Account enumeration attack if non-existent email returns distinct error message",
    "Race conditions on multiple rapid reset submissions"
  ],
  "acceptance_criteria_gherkin": [
    "Scenario: Reset requested for registered account\n  Given a registered user with email \"valid@corp.com\"\n  When user requests password reset for \"valid@corp.com\"\n  Then a generic confirmation notification is displayed\n  And a cryptographically secure token valid for 15 minutes is generated",
    "Scenario: Reset requested for unregistered email\n  Given no registered user with email \"unregistered@corp.com\"\n  When user requests password reset for \"unregistered@corp.com\"\n  Then the identical generic confirmation notification is displayed\n  And zero email dispatches occur"
  ]
}

### FEW-SHOT DEMONSTRATION 2:
[USER INPUT]:
Story ID: US-102
Title: Profile Avatar Upload
Raw Requirements: Users can upload avatar images up to 5MB. Must support JPG, PNG, and WebP.

[AGENT REASONING]:
1. Ambiguity: Is file validation performed via magic bytes or file extensions? (Extensions can be spoofed).
2. Boundary Condition: Exactly 5,242,880 bytes (5MB) must pass; 5,242,881 bytes must fail.
3. Security: SVG files must be rejected to prevent Stored XSS injection.

[STRUCTURED OUTPUT]:
{
  "story_id": "US-102",
  "title": "Profile Avatar Upload",
  "ambiguity_score": 0.03,
  "testability_score": 0.97,
  "identified_risks": [
    "SVG XSS payload injection if file type is not validated by magic bytes",
    "DoS vulnerability through uncompressed zip bomb disguised as PNG"
  ],
  "acceptance_criteria_gherkin": [
    "Scenario: Valid JPEG avatar upload under 5MB\n  Given an authenticated user on Profile Settings\n  When user uploads an image file \"profile.jpg\" of size 4.2MB\n  Then upload completes within 1000ms\n  And the avatar preview updates immediately",
    "Scenario: Upload exceeds maximum file size\n  Given an authenticated user on Profile Settings\n  When user attempts to upload \"giant.png\" of size 5.1MB\n  Then upload is blocked prior to network transmission\n  And client validation message \"File size cannot exceed 5MB\" is displayed"
  ]
}

### EXECUTION PROMPT:
Analyze the following story payload:
Story ID: {story_id}
Title: {title}
Raw Requirements:
{raw_requirements_markdown}
```

---

## 4. Output Format Specification

The agent writes its output to `${STATE_DIR}/handoffs/requirements_analysis.json`:

### 4.1 JSON Schema
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "RequirementsAnalysisOutput",
  "type": "object",
  "required": [
    "story_id",
    "title",
    "ambiguity_score",
    "testability_score",
    "identified_risks",
    "acceptance_criteria_gherkin",
    "quality_gate_evaluation"
  ],
  "properties": {
    "story_id": { "type": "string" },
    "title": { "type": "string" },
    "ambiguity_score": { "type": "number", "minimum": 0, "maximum": 1 },
    "testability_score": { "type": "number", "minimum": 0, "maximum": 1 },
    "identified_risks": { "type": "array", "items": { "type": "string" } },
    "acceptance_criteria_gherkin": { "type": "array", "items": { "type": "string" } },
    "quality_gate_evaluation": {
      "type": "object",
      "required": ["gate_id", "verdict", "score", "eval_metrics"],
      "properties": {
        "gate_id": { "type": "string", "enum": ["QG_REQ_READY_FOR_DEV"] },
        "verdict": { "type": "string", "enum": ["PASSED", "FAILED"] },
        "score": { "type": "number" },
        "eval_metrics": { "type": "object" }
      }
    }
  }
}
```

### 4.2 Concrete Output Example
```json
{
  "story_id": "US-2401",
  "title": "OAuth2 Social Login with Google",
  "ambiguity_score": 0.04,
  "testability_score": 0.96,
  "identified_risks": [
    "Race condition on concurrent token refreshes from duplicate browser tabs",
    "Third-party Google OAuth timeout leading to indefinite login spinner"
  ],
  "acceptance_criteria_gherkin": [
    "Scenario: Successful Google OAuth2 login\n  Given user navigates to \"/login\"\n  When user clicks \"Continue with Google\"\n  And authorizes via Google Identity Services\n  Then session cookie is set with HttpOnly and Secure flags\n  And user is redirected to \"/dashboard\" within 2000ms",
    "Scenario: User cancels OAuth consent window\n  Given user opens Google OAuth modal from \"/login\"\n  When user closes the consent popup without authorizing\n  Then error notification \"Authentication cancelled by user\" is displayed\n  And login page remains active and interactive"
  ],
  "quality_gate_evaluation": {
    "gate_id": "QG_REQ_READY_FOR_DEV",
    "verdict": "PASSED",
    "score": 0.96,
    "eval_metrics": {
      "EVAL_REQ_TESTABILITY": 0.96,
      "EVAL_AMBIGUITY_SCORE": 0.04,
      "GHERKIN_SCENARIOS_COUNT": 2
    }
  }
}
```

---

## 5. Evaluations & Quality Gate

### 5.1 Automated Evals Executed
1. **`EVAL_REQ_TESTABILITY`**: Evaluates whether every acceptance criterion contains an observable stimulus and a measurable assertion.  
   *Target Threshold*: **`>= 0.90`**
2. **`EVAL_AMBIGUITY_SCORE`**: Evaluates the density of undefined terms or vague adjectives.  
   *Target Threshold*: **`<= 0.10`**

### 5.2 Quality Gate: `QG_REQ_READY_FOR_DEV`
- **Pass Condition**: `EVAL_REQ_TESTABILITY >= 0.90` **AND** `EVAL_AMBIGUITY_SCORE <= 0.10` **AND** `GHERKIN_SCENARIOS_COUNT >= 1`.
- **Fail Action**: Halts sprint pipeline, notifies Product Owner via Jira ticket comment with line-item ambiguity reasons, and blocks story from moving to "In Progress".

---

## 6. Handoff Protocol & Context Persistence

1. **Write Output Artifact**: Write the JSON payload to `${STATE_DIR}/handoffs/requirements_analysis.json`.
2. **Update Persistent State**: Update `${STATE_DIR}/sprint_state.json`:
   - `active_agent`: `"QE_Test_Design_Agent"`
   - `agent_contexts.QE_Requirements_Analysis_Agent.status`: `"COMPLETED"`
   - `agent_contexts.QE_Requirements_Analysis_Agent.output_artifact`: `"${STATE_DIR}/handoffs/requirements_analysis.json"`
   - `gate_history`: Append `QG_REQ_READY_FOR_DEV` result.
3. **Team Context Continuation**: Any team member or CI runner reading `sprint_state.json` can immediately resume at Phase 2 without re-running Phase 1.

---

## 7. Execution Logging & Tracing

Every invocation appends an event to `${LOGS_DIR}/traces.jsonl`:
```json
{
  "trace_id": "tr-req-20261017-w42-01",
  "timestamp": "2026-10-17T22:42:10Z",
  "agent_name": "QE_Requirements_Analysis_Agent",
  "provider": "Anthropic",
  "model": "claude-3-5-sonnet-20241022",
  "operation": "audit_requirements_and_synthesize_gherkin",
  "token_usage": {
    "input_tokens": 1840,
    "output_tokens": 620,
    "cached_tokens": 1200,
    "total_tokens": 2460
  },
  "cost_usd": 0.0114,
  "latency_ms": 1180,
  "eval_results": {
    "EVAL_REQ_TESTABILITY": 0.96,
    "EVAL_AMBIGUITY_SCORE": 0.04
  },
  "gate_evaluated": "QG_REQ_READY_FOR_DEV",
  "gate_verdict": "PASSED"
}
```
