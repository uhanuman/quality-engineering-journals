# Execution Tracing & Audit Logging Specification

> **Module**: Observability, Telemetry & Cost Accounting  
> **Parent**: [`qe-agentic-orchestration/README.md`](../README.md)

---

## 1. Overview

Every invocation of an agent, tool call, evaluation, and gate check records an append-only JSONL record in `${LOGS_DIR}/traces.jsonl`. This guarantees that:
1. Every AI decision is fully auditable with exact prompt and response metadata.
2. Token consumption and API expenditures are tracked per agent and per sprint.
3. Model drift and evaluation accuracy are continuously monitored.
4. Compliance teams have an unbroken chain of custody for production release sign-offs.

---

## 2. Structured Trace Event Schema

Each line in `${LOGS_DIR}/traces.jsonl` conforms to this JSON schema:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "TraceEvent",
  "type": "object",
  "required": [
    "trace_id",
    "span_id",
    "timestamp",
    "agent_name",
    "provider",
    "model",
    "operation",
    "token_usage",
    "cost_usd",
    "latency_ms",
    "gate_verdict"
  ],
  "properties": {
    "trace_id": { "type": "string" },
    "span_id": { "type": "string" },
    "timestamp": { "type": "string", "format": "date-time" },
    "agent_name": { "type": "string" },
    "provider": { "type": "string", "enum": ["Anthropic", "OpenAI", "Google", "Ollama", "Local"] },
    "model": { "type": "string" },
    "operation": { "type": "string" },
    "token_usage": {
      "type": "object",
      "required": ["input_tokens", "output_tokens", "cached_tokens", "total_tokens"],
      "properties": {
        "input_tokens": { "type": "number" },
        "output_tokens": { "type": "number" },
        "cached_tokens": { "type": "number" },
        "total_tokens": { "type": "number" }
      }
    },
    "cost_usd": { "type": "number" },
    "latency_ms": { "type": "number" },
    "eval_results": { "type": "object" },
    "gate_evaluated": { "type": "string" },
    "gate_verdict": { "type": "string", "enum": ["PASSED", "FAILED", "BLOCKED"] },
    "error_details": { "type": "string" }
  }
}
```

---

## 3. Real Trace Log Example (JSONL)

```json
{"trace_id":"tr-req-20261017-w42-01","span_id":"sp-req-01","timestamp":"2026-10-18T10:14:22Z","agent_name":"QE_Requirements_Analysis_Agent","provider":"Anthropic","model":"claude-3-5-sonnet-20241022","operation":"audit_requirements_and_synthesize_gherkin","token_usage":{"input_tokens":1840,"output_tokens":620,"cached_tokens":1200,"total_tokens":2460},"cost_usd":0.0114,"latency_ms":1180,"eval_results":{"EVAL_REQ_TESTABILITY":0.96,"EVAL_AMBIGUITY_SCORE":0.04},"gate_evaluated":"QG_REQ_READY_FOR_DEV","gate_verdict":"PASSED"}
{"trace_id":"tr-dsn-20261017-w42-02","span_id":"sp-dsn-01","timestamp":"2026-10-20T16:45:10Z","agent_name":"QE_Test_Design_Agent","provider":"Anthropic","model":"claude-3-5-sonnet-20241022","operation":"decompose_scenarios_and_test_strategy","token_usage":{"input_tokens":2150,"output_tokens":780,"cached_tokens":1400,"total_tokens":2930},"cost_usd":0.0142,"latency_ms":1340,"eval_results":{"EVAL_SCENARIO_COVERAGE":1.0,"UNMAPPED_CRITERIA_COUNT":0},"gate_evaluated":"QG_DESIGN_REVIEW_PASS","gate_verdict":"PASSED"}
{"trace_id":"tr-auto-20261017-w42-03","span_id":"sp-auto-01","timestamp":"2026-10-22T09:12:30Z","agent_name":"QE_Test_Automation_Agent","provider":"Anthropic","model":"claude-3-5-sonnet-20241022","operation":"generate_playwright_pom_and_specs","token_usage":{"input_tokens":3100,"output_tokens":1150,"cached_tokens":2200,"total_tokens":4250},"cost_usd":0.0212,"latency_ms":1680,"eval_results":{"EVAL_CODE_SYNTAX_VALIDITY":1.0,"EVAL_CODE_LINT_PASS":1.0,"EVAL_POM_COMPLIANCE":0.98},"gate_evaluated":"QG_CODE_COMPILE_LINT_PASS","gate_verdict":"PASSED"}
{"trace_id":"tr-exec-20261017-w42-04","span_id":"sp-exec-01","timestamp":"2026-10-23T14:30:00Z","agent_name":"QE_Test_Execution_Agent","provider":"Google","model":"gemini-2.0-flash","operation":"execute_and_triage_playwright_suite","token_usage":{"input_tokens":1420,"output_tokens":480,"cached_tokens":800,"total_tokens":1900},"cost_usd":0.0003,"latency_ms":68400,"eval_results":{"EVAL_P0_PASS_RATE":1.0,"EVAL_P1_PASS_RATE":1.0,"EVAL_FLAKINESS_INDEX":0.0},"gate_evaluated":"QG_TEST_EXECUTION_PASS","gate_verdict":"PASSED"}
{"trace_id":"tr-rel-20261017-w42-05","span_id":"sp-rel-01","timestamp":"2026-10-24T11:00:00Z","agent_name":"QE_Release_Gate_Agent","provider":"Anthropic","model":"claude-3-5-sonnet-20241022","operation":"audit_sprint_and_render_release_decision","token_usage":{"input_tokens":2800,"output_tokens":940,"cached_tokens":1900,"total_tokens":3740},"cost_usd":0.0182,"latency_ms":1520,"eval_results":{"EVAL_UPSTREAM_GATES_INTEGRITY":1.0,"EVAL_RELEASE_CONFIDENCE":0.99},"gate_evaluated":"RG_PRODUCTION_RELEASE_READY","gate_verdict":"PASSED"}
```

---

## 4. Cost Accounting & Token Calculation Formula

$$Cost_{Total} = \frac{Tokens_{Input} - Tokens_{Cached}}{1,000,000} \times Rate_{Input} + \frac{Tokens_{Cached}}{1,000,000} \times Rate_{Cached} + \frac{Tokens_{Output}}{1,000,000} \times Rate_{Output}$$

| Provider / Model | Input Rate (/1M) | Cached Input Rate (/1M) | Output Rate (/1M) |
|---|---|---|---|
| **Claude 3.5 Sonnet** | \$3.00 | \$0.30 | \$15.00 |
| **GPT-4o** | \$2.50 | \$1.25 | \$10.00 |
| **Gemini 2.0 Flash** | \$0.10 | \$0.025 | \$0.40 |
| **Ollama (Local Qwen 2.5)** | **\$0.00** | **\$0.00** | **\$0.00** |
