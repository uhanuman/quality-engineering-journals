# Building with AI: Prompts → Skills → Agents → Agentic Pipelines

> **Journal Series**: Quality Engineering — AI Systems  
> **Edition**: Volume III — AI Artifacts & Workflows (User/Builder POV)  
> **Published**: October 2026  
> **Author**: Quality Engineering Team  
> **Companions**: [Vol I — AI Fundamentals](./01-ai-fundamentals.md) · [Vol II — Prompt & Context Engineering](./02-prompt-and-context-engineering.md)

---

## The Builder's Progression

Every AI practitioner travels the same journey — starting with a single prompt and eventually composing full autonomous pipelines. Each level unlocks more power, but also more responsibility: more failure modes, more testing surface, more operational overhead.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    THE BUILDER'S PROGRESSION                            │
│                                                                         │
│   Level 1           Level 2           Level 3           Level 4        │
│                                                                         │
│  ┌─────────┐       ┌─────────┐       ┌─────────┐       ┌──────────┐   │
│  │ PROMPT  │──▶    │  SKILL  │──▶    │  AGENT  │──▶    │ AGENTIC  │   │
│  │         │       │         │       │         │       │ PIPELINE │   │
│  │ Single  │       │ Reusable│       │ Reason  │       │ Multi-   │   │
│  │ LLM     │       │ Prompt  │       │ + Act   │       │ Agent    │   │
│  │ call    │       │ Pattern │       │ + Tools │       │ Workflow │   │
│  └─────────┘       └─────────┘       └─────────┘       └──────────┘   │
│                                                                         │
│  Complexity: ──────────────────────────────────────────────────────▶   │
│  Power:      ──────────────────────────────────────────────────────▶   │
│  Test Surface:─────────────────────────────────────────────────────▶   │
│  Failure Modes:────────────────────────────────────────────────────▶   │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Table of Contents

**Level 1 — Prompt**
1. [What Is a Prompt (as an Artifact)?](#1-what-is-a-prompt-as-an-artifact)
2. [Prompt Anatomy & Templates](#2-prompt-anatomy--templates)
3. [Prompt as a Versioned Artifact](#3-prompt-as-a-versioned-artifact)
4. [QE Concerns at the Prompt Level](#4-qe-concerns-at-the-prompt-level)

**Level 2 — Skill**
5. [What Is a Skill?](#5-what-is-a-skill)
6. [Anatomy of a Skill](#6-anatomy-of-a-skill)
7. [Skill Patterns & Types](#7-skill-patterns--types)
8. [Building & Publishing a Skill](#8-building--publishing-a-skill)
9. [QE Concerns at the Skill Level](#9-qe-concerns-at-the-skill-level)

**Level 3 — Agent**
10. [What Is an Agent (Builder POV)?](#10-what-is-an-agent-builder-pov)
11. [Agent Anatomy](#11-agent-anatomy)
12. [Agent Patterns](#12-agent-patterns)
13. [Building an Agent](#13-building-an-agent)
14. [QE Concerns at the Agent Level](#14-qe-concerns-at-the-agent-level)

**Level 4 — Agentic Pipeline**
15. [What Is an Agentic Pipeline?](#15-what-is-an-agentic-pipeline)
16. [Agentic Pipeline Anatomy](#16-agentic-pipeline-anatomy)
17. [Pipeline Topology Patterns](#17-pipeline-topology-patterns)
18. [Building an Agentic Pipeline](#18-building-an-agentic-pipeline)
19. [QE Concerns at the Pipeline Level](#19-qe-concerns-at-the-pipeline-level)

**Cross-Cutting**
20. [Artifact Comparison Matrix](#20-artifact-comparison-matrix)
21. [When to Use What](#21-when-to-use-what)
22. [End-to-End Example: Bug Triage System](#22-end-to-end-example-bug-triage-system)
23. [Libraries & Frameworks by Level](#23-libraries--frameworks-by-level)

---

# Level 1 — Prompt

## 1. What Is a Prompt (as an Artifact)?

In the context of **building with AI**, a **prompt** is not just something you type into ChatGPT — it is the **smallest deployable AI artifact**: a versioned, tested, parameterized instruction set that deterministically shapes an LLM's behavior for a specific task.

> **Mindset shift**: Stop thinking of a prompt as "the thing I said to the AI." Start thinking of it as a **function** — it takes inputs, calls the LLM, and returns an output. Like any function, it must be named, versioned, tested, and documented.

```
PROMPT AS A FUNCTION:

prompt_classify_ticket(
    input: {ticket_text: str, priority_hint: str},
    model: "gpt-4o",
    temperature: 0.0
) -> {category: str, severity: str, confidence: float}
```

### What Makes a Prompt an Artifact vs. a One-Off Query?

| One-Off Query | Prompt Artifact |
|---|---|
| Typed once, forgotten | Stored, versioned, reusable |
| No parameterization | Template variables `{{variable}}` |
| No testing | Eval suite of 50+ examples |
| No owner | Named, owned, documented |
| No monitoring | Usage tracked, quality measured |
| Any model | Pinned to model + version |

---

## 2. Prompt Anatomy & Templates

A production prompt artifact has these components:

```
┌──────────────────────────────────────────────────────────────────────┐
│                       PROMPT ARTIFACT ANATOMY                        │
│                                                                      │
│  METADATA                                                            │
│  ├── name:          "ticket_classifier_v2"                           │
│  ├── version:       "2.1.0"                                          │
│  ├── model:         "gpt-4o"                                         │
│  ├── temperature:   0.0                                              │
│  ├── max_tokens:    256                                              │
│  └── owner:         "qa-platform-team"                               │
│                                                                      │
│  SYSTEM PROMPT  (static, cacheable)                                  │
│  └── Role, rules, format instructions                                │
│                                                                      │
│  FEW-SHOT EXAMPLES  (static, cacheable)                              │
│  └── 3–5 input → output demonstrations                               │
│                                                                      │
│  USER TEMPLATE  (parameterized)                                      │
│  └── "Classify the following ticket:\n{{ticket_text}}"               │
│                                                                      │
│  OUTPUT SCHEMA  (validated)                                          │
│  └── Pydantic model / JSON Schema                                    │
│                                                                      │
│  EVAL SUITE                                                          │
│  └── 50+ labeled examples with expected outputs                      │
└──────────────────────────────────────────────────────────────────────┘
```

### Prompt Template Example (LangChain)

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI
from pydantic import BaseModel

# Output schema — acts as contract
class TicketClassification(BaseModel):
    category: str       # Bug | Feature | Question | Billing
    severity: str       # Low | Medium | High | Critical
    confidence: float   # 0.0 – 1.0
    reasoning: str      # One sentence explanation

# Prompt template — parameterized, reusable
prompt = ChatPromptTemplate.from_messages([
    ("system", """You are a QA triage specialist.
Classify support tickets into: Bug, Feature, Question, or Billing.
Severity: Low, Medium, High, or Critical.
Respond only in the specified JSON format."""),
    ("human", "Ticket: {ticket_text}\nPriority hint: {priority_hint}"),
])

# Bind model + structured output
llm = ChatOpenAI(model="gpt-4o", temperature=0.0)
chain = prompt | llm.with_structured_output(TicketClassification)

# Invoke with named inputs
result = chain.invoke({
    "ticket_text": "Login fails on Safari 17 after the latest deploy",
    "priority_hint": "Affecting 20% of users"
})
# result.category → "Bug"
# result.severity → "High"
```

---

## 3. Prompt as a Versioned Artifact

### Semantic Versioning for Prompts

Treat prompt changes like code changes:

| Change Type | Version Bump | Example |
|---|---|---|
| New output field added | **Minor** (1.0 → 1.1) | Added `reasoning` field |
| Output schema breaking change | **Major** (1.x → 2.0) | Renamed `category` → `ticket_type` |
| Wording tweak, same behavior | **Patch** (1.1.0 → 1.1.1) | Fixed typo in system prompt |
| Model change (same family) | **Minor** (1.1 → 1.2) | gpt-4o → gpt-4o-2024-11-20 |
| Model family change | **Major** | OpenAI → Claude |

### Prompt Versioning with LangSmith Hub

```python
from langchain import hub

# Pull a versioned prompt from LangSmith Hub
prompt = hub.pull("qa-team/ticket-classifier:v2.1.0")

# Push a new version after testing
hub.push("qa-team/ticket-classifier", updated_prompt, tags=["v2.2.0"])
```

### Prompt Version Registry Pattern

```yaml
# prompts/registry.yaml
prompts:
  ticket_classifier:
    current: "2.1.0"
    model: "gpt-4o"
    eval_score: 0.94
    versions:
      "2.1.0":
        file: "ticket_classifier/v2.1.0/system.txt"
        eval_accuracy: 0.94
        deployed: "2026-09-15"
      "2.0.0":
        file: "ticket_classifier/v2.0.0/system.txt"
        eval_accuracy: 0.89
        deprecated: "2026-09-15"
```

---

## 4. QE Concerns at the Prompt Level

| Concern | Test Approach | Tool |
|---|---|---|
| **Output format compliance** | Run 50+ examples through parser; fail if schema breaks | Pydantic, Guardrails |
| **Instruction following** | LLM-as-judge: does output obey all stated rules? | DeepEval `GEval`, LangSmith |
| **Edge case coverage** | Test on: empty input, max-length input, special characters, non-English | Promptfoo |
| **Hallucination rate** | % of outputs with unsupported claims | RAGAS `faithfulness` |
| **Regression on prompt change** | Run full eval suite before/after any version bump | Promptfoo CI, LangSmith |
| **Model pinning** | Output should not change on model API update | Pin exact model version |
| **Latency budget** | p50/p95/p99 response times within SLA | LangSmith traces |
| **Cost per call** | Token usage × price, tracked per version | Langfuse, Helicone |

---

# Level 2 — Skill

## 5. What Is a Skill?

A **Skill** is a **named, reusable, self-contained AI capability** that combines one or more prompts with supporting logic (tools, parsers, validation, retry) into a composable unit that can be called by other systems, agents, or users.

> **Analogy**: If a Prompt is a function, a Skill is a **library function** — documented, tested, dependency-managed, and importable by anyone who needs it.

```
SKILL = Prompt(s) + Input Schema + Output Schema + Tools + Validation + Tests
```

### Skill vs. Prompt

| Dimension | Prompt | Skill |
|---|---|---|
| **Scope** | Single LLM call | Multi-step capability (may call LLM multiple times) |
| **Tools** | None | Can invoke external tools / APIs |
| **Retry Logic** | None | Built-in retry, fallback, error handling |
| **Input Validation** | None | Typed, validated inputs |
| **Output Guarantee** | Best-effort | Validated, contract-enforced output |
| **Discoverability** | Must know the prompt exists | Registered in a skill catalog |
| **Composability** | Not composable | Designed to be called by agents and pipelines |

---

## 6. Anatomy of a Skill

```
┌──────────────────────────────────────────────────────────────────────┐
│                         SKILL ANATOMY                                │
│                                                                      │
│  SKILL DEFINITION                                                    │
│  ├── name:          "summarize_test_results"                         │
│  ├── description:   "Summarizes a QA test run into a human-readable  │
│  │                   report with severity breakdown."                │
│  ├── version:       "1.3.0"                                          │
│  └── tags:          ["qa", "reporting", "summarization"]             │
│                                                                      │
│  INPUT CONTRACT                                                      │
│  ├── test_results:  List[TestResult]  (typed, validated)             │
│  ├── project_name:  str                                              │
│  └── format:        "markdown" | "json" | "html"                     │
│                                                                      │
│  INTERNAL STEPS                                                      │
│  ├── Step 1: Parse & validate inputs                                 │
│  ├── Step 2: Retrieve context (RAG if needed)                        │
│  ├── Step 3: Run LLM prompt(s)                                       │
│  ├── Step 4: Validate output schema                                  │
│  └── Step 5: Retry on failure (max 3)                                │
│                                                                      │
│  OUTPUT CONTRACT                                                     │
│  ├── summary:       str                                              │
│  ├── severity_breakdown: Dict[str, int]                              │
│  ├── top_issues:    List[str]                                         │
│  └── confidence:    float                                            │
│                                                                      │
│  OBSERVABILITY                                                        │
│  └── Emits traces, token counts, latency to observability platform   │
└──────────────────────────────────────────────────────────────────────┘
```

### Skill Definition in Code (LangChain Tool / Structured Tool)

```python
from langchain.tools import StructuredTool
from pydantic import BaseModel, Field
from typing import Literal

class SummarizeTestResultsInput(BaseModel):
    test_results: list[dict] = Field(..., description="List of test result objects")
    project_name: str = Field(..., description="Name of the project under test")
    output_format: Literal["markdown", "json"] = Field(default="markdown")

class SummarizeTestResultsOutput(BaseModel):
    summary: str
    total_tests: int
    pass_count: int
    fail_count: int
    top_failures: list[str]

def summarize_test_results_fn(
    test_results: list[dict],
    project_name: str,
    output_format: str = "markdown"
) -> SummarizeTestResultsOutput:
    """Core skill logic — calls LLM, validates, returns structured output."""
    # ... prompt construction + LLM call + validation
    pass

# Register as a discoverable Skill / Tool
summarize_skill = StructuredTool(
    name="summarize_test_results",
    description="Summarize a QA test run into a structured report with severity breakdown.",
    args_schema=SummarizeTestResultsInput,
    func=summarize_test_results_fn,
)
```

---

## 7. Skill Patterns & Types

### By Capability Type

| Skill Type | Description | Example |
|---|---|---|
| **Transform** | Convert data from one form to another | `json_to_natural_language`, `code_to_docs` |
| **Classify** | Categorize an input into defined classes | `ticket_classifier`, `sentiment_analyzer` |
| **Extract** | Pull structured data from unstructured text | `extract_requirements`, `parse_stack_trace` |
| **Generate** | Create new content from a spec | `write_test_cases`, `generate_pr_description` |
| **Evaluate** | Score or judge quality of an artifact | `review_code`, `grade_test_coverage` |
| **Retrieve** | Fetch and surface relevant information | `search_knowledge_base`, `find_similar_bugs` |
| **Decide** | Make a boolean or multi-class decision | `should_escalate`, `is_regression` |
| **Summarize** | Condense large content | `summarize_test_run`, `tldr_pr` |

### By Interaction Pattern

```
SINGLE-CALL SKILL              MULTI-STEP SKILL
━━━━━━━━━━━━━━━━━━━            ━━━━━━━━━━━━━━━━━━━━━━━━━━━
Input ──▶ Prompt ──▶ Output    Input ──▶ Step 1 ──▶ Step 2
                                   ──▶ Step 3 ──▶ Validate
                                   ──▶ Output

RETRIEVAL-AUGMENTED SKILL      TOOL-CALLING SKILL
━━━━━━━━━━━━━━━━━━━━━━━━━━━    ━━━━━━━━━━━━━━━━━━━━━━━━━
Input ──▶ Retrieve ──▶ Prompt  Input ──▶ Prompt
              │                    ──▶ Tool call(s)
          Vector DB               ──▶ Incorporate results
                                   ──▶ Final output
```

---

## 8. Building & Publishing a Skill

### Step-by-Step: Building a Skill

```
1. DEFINE THE CONTRACT
   ├── Write the input schema (what does this skill accept?)
   ├── Write the output schema (what does it guarantee to return?)
   └── Write the natural language description (for agent tool selection)

2. BUILD THE CORE
   ├── Write the prompt(s)
   ├── Add input validation
   ├── Add output parser + validation
   └── Add retry logic (max 3, with exponential backoff)

3. TEST
   ├── Unit tests: 20+ examples covering happy path + edge cases
   ├── Schema tests: every output passes the output schema
   ├── Regression tests: run before every version change
   └── Adversarial tests: malformed input, injection attempts

4. INSTRUMENT
   ├── Emit traces (LangSmith / Langfuse)
   ├── Log token usage and latency
   └── Emit custom metrics (skill accuracy, confidence distribution)

5. REGISTER
   ├── Add to skill catalog / registry
   ├── Tag with capability type and domain
   └── Document examples and known limitations

6. DEPLOY & MONITOR
   ├── Pin model version
   ├── Set latency + quality alerts
   └── Schedule periodic eval runs to detect drift
```

### Skill Catalog Pattern

```python
# skills/catalog.py — Central skill registry

from skills.classify import ticket_classifier_skill
from skills.summarize import test_run_summarizer_skill
from skills.extract import requirement_extractor_skill
from skills.evaluate import code_reviewer_skill

SKILL_CATALOG = {
    "ticket_classifier":    ticket_classifier_skill,
    "test_run_summarizer":  test_run_summarizer_skill,
    "requirement_extractor": requirement_extractor_skill,
    "code_reviewer":        code_reviewer_skill,
}

def get_skill(name: str):
    if name not in SKILL_CATALOG:
        raise ValueError(f"Skill '{name}' not found in catalog.")
    return SKILL_CATALOG[name]
```

---

## 9. QE Concerns at the Skill Level

| Concern | Description | Test Approach |
|---|---|---|
| **Contract stability** | Output schema must not break callers on update | Schema diff tests on every PR |
| **Retry correctness** | Retries must not cause duplicate side effects | Idempotency tests |
| **Tool call accuracy** | Skill calls the right tools with the right args | Tool call assertion tests |
| **Latency SLA** | Skill completes within agreed time budget | p95 latency monitoring |
| **Graceful degradation** | Skill returns meaningful error, not exception | Error type + message tests |
| **Input boundary testing** | Empty, null, oversized, special-character inputs | Boundary value analysis |
| **Skill isolation** | Skill does not depend on hidden global state | Stateless design validation |
| **Observability completeness** | All calls emit traces + metrics | Trace coverage tests |

---

# Level 3 — Agent

## 10. What Is an Agent (Builder POV)?

An **Agent** is an AI system that can **autonomously decide what actions to take**, **execute those actions** (via skills and tools), **observe the results**, and **iterate** until a goal is achieved — without a human specifying every step.

> **Key distinction from a Skill**: A skill always runs the same steps for a given input. An **agent** decides at runtime which steps to run, in what order, and how many times — based on what it observes.

```
SKILL (deterministic execution):
Input ──▶ Step A ──▶ Step B ──▶ Step C ──▶ Output
          (always the same path)

AGENT (dynamic, goal-directed):
Goal ──▶ Think: "What do I need?"
       ──▶ Act:  call skill_A
       ──▶ Observe: result A
       ──▶ Think: "Result A says I need more info..."
       ──▶ Act:  call skill_B with new args
       ──▶ Observe: result B
       ──▶ Think: "I now have enough to answer."
       ──▶ Output: final answer
```

---

## 11. Agent Anatomy

```
┌──────────────────────────────────────────────────────────────────────┐
│                          AGENT ANATOMY                               │
│                                                                      │
│  IDENTITY                                                            │
│  ├── name:        "qa_triage_agent"                                  │
│  ├── role:        "Senior QA Triage Engineer"                        │
│  ├── goal:        "Classify, prioritize, and route incoming bugs"    │
│  └── model:       "claude-3-5-sonnet-20241022"                       │
│                                                                      │
│  BRAIN (LLM)                                                         │
│  └── Drives all reasoning, planning, and tool selection decisions    │
│                                                                      │
│  MEMORY                                                              │
│  ├── Short-term: current conversation / task context                 │
│  ├── Working:    intermediate results, tool outputs                  │
│  └── Long-term:  past decisions, user preferences (vector DB)        │
│                                                                      │
│  SKILLS / TOOLS  (the agent's hands)                                 │
│  ├── ticket_classifier (Skill)                                       │
│  ├── search_knowledge_base (Skill)                                   │
│  ├── get_similar_bugs (Skill)                                        │
│  ├── assign_ticket (Tool → Jira API)                                 │
│  └── notify_team (Tool → Slack API)                                  │
│                                                                      │
│  CONTROL LOOP                                                        │
│  └── ReAct: Think → Act → Observe → Repeat until goal met           │
│                                                                      │
│  GUARDRAILS                                                          │
│  ├── Max iterations: 10                                              │
│  ├── Allowed tools: whitelist only                                   │
│  ├── Budget: $0.50 max per task                                      │
│  └── Human escalation trigger conditions                             │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 12. Agent Patterns

### 12.1 ReAct Agent (Most Common)

The standard pattern — interleaved reasoning and acting:

```
┌─────────────────────────────────────────────────────────────────────┐
│                         REACT AGENT LOOP                            │
│                                                                     │
│  ┌─────────┐    ┌──────────┐    ┌───────────┐    ┌──────────────┐  │
│  │  Task   │───▶│  THINK   │───▶│   ACT     │───▶│   OBSERVE    │  │
│  │  / Goal │    │          │    │ (tool call)│    │ (tool result)│  │
│  └─────────┘    └──────────┘    └───────────┘    └──────────────┘  │
│                      ▲                                    │         │
│                      └────────────────────────────────────┘         │
│                              (loop until done or max_iter)          │
└─────────────────────────────────────────────────────────────────────┘
```

### 12.2 Plan-and-Execute Agent

Separates **planning** from **execution**:

```
Goal
 │
 ▼
PLANNER LLM ──▶ Generates full step-by-step plan upfront
                [Step 1, Step 2, Step 3, Step 4]
                      │
                      ▼
              EXECUTOR: Runs each step sequentially
                      │
                      ▼ (if a step fails or conditions change)
              REPLANNER: Updates remaining steps
                      │
                      ▼
              Final Output
```

**Best for**: Long, predictable tasks where upfront planning is possible.  
**Framework**: LangChain `plan-and-execute` chain, LangGraph with planner node.

### 12.3 Reflection / Critique Agent

Adds a **self-reflection loop** — agent critiques its own output and iterates:

```
Task ──▶ Generate Draft ──▶ Critique Draft ──▶ Revised Draft
              ▲                    │
              └────────────────────┘
              (repeat until quality threshold met or max rounds)
```

**Best for**: Writing, code generation, analysis — tasks where quality can be self-assessed.

### 12.4 Tool-Calling Agent (OpenAI / Anthropic Native)

Uses model-native **parallel tool calling** for efficiency:

```python
from langchain_openai import ChatOpenAI
from langchain_core.tools import tool

@tool
def classify_ticket(text: str) -> dict:
    """Classify a support ticket. Returns category and severity."""
    ...

@tool
def search_similar_bugs(description: str) -> list[dict]:
    """Search for similar past bugs in the knowledge base."""
    ...

llm = ChatOpenAI(model="gpt-4o").bind_tools(
    [classify_ticket, search_similar_bugs],
    tool_choice="auto"   # model decides when to call tools
)

# Model can now call BOTH tools in parallel in a single turn
response = llm.invoke("Triage this new bug: login fails on Safari")
```

### 12.5 Human-in-the-Loop Agent

Adds **checkpoints** where the agent pauses and waits for human approval before proceeding:

```
Agent runs ──▶ Reaches CHECKPOINT (high-stakes action)
                    │
                    ▼
              "I'm about to [create 500 Jira tickets / delete 100 records].
               Approve? (yes/no/modify)"
                    │
              Human approves ──▶ Agent continues
              Human rejects  ──▶ Agent replans
              Human modifies ──▶ Agent adjusts and continues
```

**Framework**: LangGraph `interrupt()`, `Command(resume=...)`.

---

## 13. Building an Agent

### Step-by-Step: Building a QA Triage Agent

```python
from langchain_anthropic import ChatAnthropic
from langchain_core.tools import tool
from langgraph.prebuilt import create_react_agent
from langgraph.checkpoint.memory import MemorySaver

# 1. DEFINE TOOLS (Skills exposed as tools)
@tool
def classify_ticket(ticket_text: str) -> dict:
    """Classify a support ticket into category, severity, and confidence."""
    # Call classify_ticket_skill internally
    return {"category": "Bug", "severity": "High", "confidence": 0.94}

@tool
def search_similar_bugs(description: str, limit: int = 5) -> list[dict]:
    """Search the bug knowledge base for similar past issues."""
    # Call vector DB retrieval
    return [{"id": "BUG-1234", "title": "Login fails on Safari 16"}]

@tool
def assign_to_team(ticket_id: str, team: str, reason: str) -> dict:
    """Assign a ticket to a specific engineering team in Jira."""
    # Call Jira API
    return {"status": "assigned", "ticket_id": ticket_id}

# 2. DEFINE AGENT
model = ChatAnthropic(model="claude-3-5-sonnet-20241022", temperature=0)

system_prompt = """You are a Senior QA Triage Engineer.
Your goal is to:
1. Classify the incoming ticket (category + severity)
2. Search for similar past bugs to inform priority
3. Assign the ticket to the correct team

Always provide reasoning for your decisions.
Escalate to human if severity is Critical and confidence < 0.85."""

# 3. WIRE UP WITH LANGGRAPH (with memory + checkpointing)
checkpointer = MemorySaver()  # Replace with SqliteSaver / PostgresSaver in prod

agent = create_react_agent(
    model=model,
    tools=[classify_ticket, search_similar_bugs, assign_to_team],
    state_modifier=system_prompt,
    checkpointer=checkpointer,
)

# 4. INVOKE
config = {"configurable": {"thread_id": "ticket-5678"}}
result = agent.invoke(
    {"messages": [("user", "New bug: Dashboard shows blank after login on Chrome 119")]},
    config=config
)
```

### Agent Configuration Checklist

```
AGENT CONFIGURATION CHECKLIST:
☐ Model selected and version pinned
☐ System prompt reviewed and tested
☐ Tool list is a whitelist (not open-ended)
☐ Max iterations set (default: 10)
☐ Token budget cap per run
☐ Memory type chosen (in-context / vector / persistent)
☐ Checkpointing enabled (for resumability)
☐ Human-in-the-loop triggers defined
☐ Error handling: what happens on tool failure?
☐ Observability: traces emitted to LangSmith / Langfuse
```

---

## 14. QE Concerns at the Agent Level

| Concern | Description | Test Approach |
|---|---|---|
| **Goal completion rate** | Does the agent actually achieve the stated goal? | E2E test suite with goal assertions |
| **Tool selection accuracy** | Does the agent call the right tool for the situation? | Trace analysis + tool call assertions |
| **Infinite loop prevention** | Does the agent terminate within `max_iter`? | Adversarial tests with no-solution inputs |
| **Tool argument correctness** | Are tool arguments well-formed and semantically correct? | Argument schema validation per call |
| **Memory correctness** | Does the agent correctly recall prior context? | Multi-turn memory recall tests |
| **Checkpoint / resume** | Can the agent resume correctly from a mid-run checkpoint? | Kill + resume integration tests |
| **Human escalation triggers** | Does the agent escalate when confidence is low? | Boundary condition tests |
| **Cost per task** | Total token spend per task is within budget | Cost tracking + alerting |
| **Concurrency safety** | Multiple simultaneous agent runs don't interfere | Parallel run tests with shared state |
| **Graceful tool failure** | Agent recovers and replans when a tool throws an error | Fault injection tests |

---

# Level 4 — Agentic Pipeline

## 15. What Is an Agentic Pipeline?

An **Agentic Pipeline** is a structured, orchestrated workflow composed of **multiple agents and/or skills**, each specialized for a sub-task, working together to achieve a complex goal that no single agent could accomplish efficiently alone.

> **The key difference from a single agent**: An agentic pipeline has **explicit structure** — defined topology, handoff protocols, shared state contracts, and specialized roles. It is to a single agent what a microservices architecture is to a monolith.

```
SINGLE AGENT:                  AGENTIC PIPELINE:
                               ┌──────────────────────────────────────┐
Goal ──▶ Agent                 │  Input                               │
             │                 │    │                                 │
         (does everything)     │    ▼                                 │
             │                 │  Intake Agent (classify + route)     │
             ▼                 │    │           │           │         │
         Output                │    ▼           ▼           ▼         │
                               │  Research   Drafting   Validation    │
                               │  Agent      Agent      Agent         │
                               │    │           │           │         │
                               │    └─────────▶ Merge ◀────┘         │
                               │                │                     │
                               │                ▼                     │
                               │           Final Output               │
                               └──────────────────────────────────────┘
```

---

## 16. Agentic Pipeline Anatomy

```
┌──────────────────────────────────────────────────────────────────────┐
│                    AGENTIC PIPELINE ANATOMY                          │
│                                                                      │
│  PIPELINE DEFINITION                                                 │
│  ├── name:         "bug_triage_and_routing_pipeline"                 │
│  ├── version:      "3.0.0"                                           │
│  ├── trigger:      "new_ticket_created" event                        │
│  └── sla:          "95% completion in < 30 seconds"                  │
│                                                                      │
│  SHARED STATE  (passed between agents)                               │
│  ├── ticket: TicketObject                                            │
│  ├── classification: ClassificationResult | None                     │
│  ├── similar_bugs: List[Bug] | None                                  │
│  ├── routing_decision: RoutingDecision | None                        │
│  └── pipeline_status: "running" | "awaiting_human" | "done"         │
│                                                                      │
│  AGENTS / NODES                                                      │
│  ├── intake_agent      → classifies ticket                           │
│  ├── research_agent    → finds similar bugs, checks release notes    │
│  ├── prioritization_agent → scores priority with business context    │
│  ├── routing_agent     → decides team assignment                     │
│  └── notification_agent → sends alerts, creates Jira ticket         │
│                                                                      │
│  EDGES / TRANSITIONS                                                 │
│  ├── intake ──▶ research (always)                                    │
│  ├── research ──▶ prioritization (always)                            │
│  ├── prioritization ──▶ routing (if confidence ≥ 0.85)              │
│  ├── prioritization ──▶ human_review (if confidence < 0.85)         │
│  └── routing ──▶ notification (always)                               │
│                                                                      │
│  CHECKPOINTING                                                       │
│  └── State persisted to DB after each node — resumable on failure   │
│                                                                      │
│  OBSERVABILITY                                                       │
│  └── Full trace tree: every agent, every tool call, every token     │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 17. Pipeline Topology Patterns

### 17.1 Sequential Pipeline

Each agent runs after the previous one completes:

```
Input ──▶ Agent A ──▶ Agent B ──▶ Agent C ──▶ Output
```

**Use when**: Tasks are strictly dependent on the output of the previous step.  
**Risk**: Single failure blocks the entire pipeline.

---

### 17.2 Parallel Pipeline

Independent agents run concurrently, results merged:

```
          ┌──▶ Agent A (research) ──▶┐
Input ────┼──▶ Agent B (classify) ──▶┼──▶ Merge ──▶ Output
          └──▶ Agent C (check SLA) ──▶┘
```

**Use when**: Sub-tasks are independent and latency matters.  
**Gain**: Latency = max(A, B, C) not sum(A + B + C).  
**Framework**: LangGraph parallel branches, `asyncio.gather()`.

---

### 17.3 Supervisor / Hierarchical Pipeline

A **supervisor agent** routes tasks to specialized sub-agents:

```
                  ┌─────────────────┐
   User Task ────▶│   SUPERVISOR    │
                  │ (routes + merges)│
                  └─────────────────┘
                    │       │       │
                    ▼       ▼       ▼
               Research  Writing  Validation
               Agent     Agent    Agent
                    │       │       │
                    └───────┴───────┘
                            │
                    Supervisor merges
                            │
                        Final Output
```

**Use when**: Dynamic routing — supervisor decides which specialist to call based on task content.  
**Framework**: LangGraph supervisor pattern, AutoGen `GroupChat` with `GroupChatManager`.

---

### 17.4 Peer-to-Peer / Swarm

Agents communicate directly with each other, no central coordinator:

```
Agent A ◀──────────▶ Agent B
   ▲                     │
   │                     ▼
Agent D ◀──────── Agent C
```

**Use when**: Collaborative tasks where the optimal communication pattern is emergent.  
**Framework**: OpenAI Swarm, AutoGen peer-to-peer mode.

---

### 17.5 Map-Reduce Pipeline

Fan out a task across many sub-agents, aggregate results:

```
                 ┌──▶ Worker Agent (chunk 1) ──▶┐
                 │                              │
Input (large) ───┼──▶ Worker Agent (chunk 2) ──▶┼──▶ Aggregator ──▶ Output
                 │                              │
                 └──▶ Worker Agent (chunk N) ──▶┘
```

**Use when**: Large documents, large test suites, or any task that can be parallelized by splitting input.  
**Example**: "Run security review across all 200 endpoints" → split into batches of 10, run in parallel.

---

### 17.6 Feedback Loop Pipeline (Critic-Actor)

An **actor** generates output, a **critic** evaluates it, actor revises until quality threshold:

```
Task ──▶ Actor Agent ──▶ Draft
                            │
                            ▼
                    Critic Agent ──▶ Score < threshold? ──▶ Back to Actor
                            │
                      Score ≥ threshold ──▶ Final Output
```

**Use when**: Quality of output is the primary constraint (reports, code, test plans).

---

## 18. Building an Agentic Pipeline

### Full LangGraph Pipeline Example

```python
from typing import TypedDict, Annotated
from langgraph.graph import StateGraph, START, END
from langgraph.checkpoint.sqlite import SqliteSaver

# 1. DEFINE SHARED STATE
class PipelineState(TypedDict):
    ticket_text: str
    classification: dict | None
    similar_bugs: list[dict] | None
    priority_score: float | None
    routing_decision: dict | None
    human_review_needed: bool
    final_status: str | None

# 2. DEFINE AGENT NODES
def intake_agent(state: PipelineState) -> PipelineState:
    """Classifies the incoming ticket."""
    result = classify_ticket_skill(state["ticket_text"])
    return {"classification": result}

def research_agent(state: PipelineState) -> PipelineState:
    """Finds similar past bugs for context."""
    bugs = search_similar_bugs_skill(state["ticket_text"])
    return {"similar_bugs": bugs}

def prioritization_agent(state: PipelineState) -> PipelineState:
    """Scores business priority considering classification + similar bugs."""
    score = compute_priority_score(
        state["classification"],
        state["similar_bugs"]
    )
    needs_review = score > 0.9 and state["classification"]["confidence"] < 0.85
    return {"priority_score": score, "human_review_needed": needs_review}

def routing_agent(state: PipelineState) -> PipelineState:
    """Determines which team to assign the ticket to."""
    decision = route_ticket(state["classification"], state["priority_score"])
    return {"routing_decision": decision}

def notification_agent(state: PipelineState) -> PipelineState:
    """Creates Jira ticket and notifies team via Slack."""
    create_jira_ticket(state)
    notify_slack(state)
    return {"final_status": "completed"}

# 3. DEFINE CONDITIONAL ROUTING
def should_route_to_human(state: PipelineState) -> str:
    return "human_review" if state["human_review_needed"] else "routing_agent"

# 4. BUILD THE GRAPH
graph = StateGraph(PipelineState)

# Add nodes
graph.add_node("intake_agent", intake_agent)
graph.add_node("research_agent", research_agent)
graph.add_node("prioritization_agent", prioritization_agent)
graph.add_node("routing_agent", routing_agent)
graph.add_node("notification_agent", notification_agent)
graph.add_node("human_review", lambda s: s)  # Interrupt point

# Add edges
graph.add_edge(START, "intake_agent")
graph.add_edge("intake_agent", "research_agent")
graph.add_edge("research_agent", "prioritization_agent")

# Conditional edge: human review or direct routing
graph.add_conditional_edges(
    "prioritization_agent",
    should_route_to_human,
    {"human_review": "human_review", "routing_agent": "routing_agent"}
)

graph.add_edge("routing_agent", "notification_agent")
graph.add_edge("notification_agent", END)

# 5. COMPILE WITH PERSISTENCE
with SqliteSaver.from_conn_string("./pipeline_state.db") as checkpointer:
    pipeline = graph.compile(
        checkpointer=checkpointer,
        interrupt_before=["human_review"],   # Pause for human approval
    )

# 6. INVOKE
config = {"configurable": {"thread_id": "ticket-9876"}}

result = pipeline.invoke(
    {"ticket_text": "Dashboard blank after login on Chrome 119, affects 30% of users"},
    config=config
)

# If interrupted for human review:
if result.get("human_review_needed"):
    # Human reviews state, approves continuation
    pipeline.invoke(None, config=config)  # Resume from checkpoint
```

---

## 19. QE Concerns at the Pipeline Level

| Concern | Description | Test Approach |
|---|---|---|
| **Pipeline completion rate** | % of pipelines that reach END without error | E2E test suite |
| **State integrity** | Shared state is never corrupted between agents | State schema validation at each node exit |
| **Handoff correctness** | Output of Agent A is valid input to Agent B | Contract tests between adjacent agents |
| **Conditional routing accuracy** | Branching decisions are correct (e.g., human escalation triggers correctly) | Boundary condition tests for each branch |
| **Resume correctness** | Pipeline resumes from correct checkpoint after failure | Kill-and-resume integration tests |
| **Parallel branch isolation** | Concurrent agents don't write to shared state incorrectly | Race condition / concurrency tests |
| **End-to-end latency** | Total pipeline time within SLA | Load testing + trace-level timing |
| **Cost per pipeline run** | Total token + API spend per invocation | Cost tracking per run |
| **Failure propagation** | One agent failure should not silently corrupt downstream results | Fault injection at each node |
| **Audit trail completeness** | Full trace of every decision, tool call, and state change | Trace coverage audit |
| **Idempotency** | Re-running the same pipeline input produces the same result | Idempotency tests |
| **Version compatibility** | When one agent updates, adjacent agents still work | Contract-driven agent versioning |

### Pipeline Testing Pyramid

```
                    ▲
                   / \
                  /   \
                 / E2E  \          ← Full pipeline happy + sad paths
                /─────────\
               / Integration\       ← Adjacent agent pair tests (handoffs)
              /─────────────\
             / Agent Tests   \      ← Individual agent goal completion
            /─────────────────\
           /   Skill Tests      \   ← Individual skill contract + accuracy
          /─────────────────────\
         /    Prompt Tests        \ ← Format, instruction following, regression
        └───────────────────────────┘
```

---

# Cross-Cutting Concerns

## 20. Artifact Comparison Matrix

| Dimension | Prompt | Skill | Agent | Agentic Pipeline |
|---|---|---|---|---|
| **Definition** | Parameterized LLM instruction | Composable AI capability | Goal-directed autonomous loop | Multi-agent orchestrated workflow |
| **Execution** | Single LLM call | Deterministic steps | Dynamic, iterative loop | Structured multi-agent graph |
| **Tools** | None | Optional | Yes — core capability | Yes — per agent |
| **Memory** | None | None (stateless) | In-context + optional persistent | Shared state + persistent checkpoints |
| **Autonomy** | None | None | High | Highest |
| **Predictability** | High | High | Medium | Low–Medium |
| **Test Surface** | Small | Medium | Large | Largest |
| **Failure Modes** | Format, hallucination | + Tool errors, retry failure | + Loops, wrong tool selection | + Handoff failures, state corruption |
| **Build Time** | Hours | Days | Days–Weeks | Weeks–Months |
| **Observability Need** | Basic | Medium | High | Critical |
| **Best Framework** | LangChain, DSPy | LangChain Tools, Semantic Kernel | LangGraph, AutoGen, CrewAI | LangGraph, Temporal, Prefect |

---

## 21. When to Use What

```
DECISION GUIDE:
═══════════════════════════════════════════════════════════════════════

Is the task fully described by one LLM call with a fixed template?
    YES ──▶ BUILD A PROMPT
    NO  ──▶ Continue...

Does the task require multiple steps but the steps are ALWAYS the same?
    YES ──▶ BUILD A SKILL
    NO  ──▶ Continue...

Does the task require DECIDING at runtime what steps to take?
    YES ──▶ BUILD AN AGENT
    NO  ──▶ (you probably just need a skill)

Does the task require MULTIPLE SPECIALIZED AGENTS working together?
    YES ──▶ BUILD AN AGENTIC PIPELINE
    NO  ──▶ (one agent is sufficient)

═══════════════════════════════════════════════════════════════════════

RULE OF THUMB:
  Start with the simplest thing that works.
  A prompt that works beats an agent that almost works.
  An agent that works beats a pipeline that almost works.
```

---

## 22. End-to-End Example: Bug Triage System

This shows how all four levels compose into one complete system:

```
┌─────────────────────────────────────────────────────────────────────────┐
│               BUG TRIAGE SYSTEM — FULL ARTIFACT MAP                    │
│                                                                         │
│  PROMPTS (Level 1):                                                     │
│  ├── classify_ticket_prompt        → category + severity               │
│  ├── find_similar_bugs_prompt      → similarity search query            │
│  ├── score_business_priority_prompt → priority 0.0–1.0 + reasoning      │
│  └── generate_routing_prompt       → team + justification               │
│                                                                         │
│  SKILLS (Level 2):                                                      │
│  ├── ticket_classifier_skill       (uses classify_ticket_prompt)        │
│  ├── similar_bug_searcher_skill    (uses find_similar_bugs_prompt       │
│  │                                  + vector DB retrieval)              │
│  ├── priority_scorer_skill         (uses score_business_priority_prompt)│
│  └── ticket_router_skill           (uses generate_routing_prompt)       │
│                                                                         │
│  AGENTS (Level 3):                                                      │
│  ├── intake_agent     (uses: ticket_classifier_skill)                   │
│  ├── research_agent   (uses: similar_bug_searcher_skill, web_search)    │
│  ├── priority_agent   (uses: priority_scorer_skill, jira_context_tool)  │
│  └── routing_agent    (uses: ticket_router_skill, org_chart_tool)       │
│                                                                         │
│  AGENTIC PIPELINE (Level 4):                                            │
│  intake_agent ──▶ research_agent ──▶ priority_agent                    │
│                                             │                           │
│                                   confidence < 0.85?                    │
│                                    │              │                     │
│                              human_review    routing_agent              │
│                                    │              │                     │
│                                    └──────────────┘                     │
│                                             │                           │
│                                    notification_agent                   │
│                                      (Jira + Slack)                    │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 23. Libraries & Frameworks by Level

### Level 1 — Prompt

| Library | Purpose |
|---|---|
| **LangChain** `PromptTemplate` / `ChatPromptTemplate` | Parameterized prompt templates |
| **LangChain Hub** / **LangSmith** | Prompt versioning, sharing, A/B testing |
| **DSPy** | Auto-optimizing prompt programs |
| **Promptfoo** | Prompt regression testing + red-teaming |
| **Langfuse** | Open-source prompt versioning + cost tracking |
| **Instructor** | Structured output via Pydantic schemas |
| **Outlines** | Constrained decoding for guaranteed output format |
| **Guidance** | Template-based constrained generation |

### Level 2 — Skill

| Library | Purpose |
|---|---|
| **LangChain** `StructuredTool` / `@tool` decorator | Skill definition + registration |
| **Semantic Kernel** Skills | Enterprise skill packaging + discovery |
| **LlamaIndex** Query Engines | Document-centric skill patterns |
| **Haystack** Components | Pipeline-native skill building blocks |
| **Guardrails AI** | Input/output validation for skills |
| **Pydantic** | Input + output schema definition and validation |

### Level 3 — Agent

| Library | Purpose |
|---|---|
| **LangGraph** `create_react_agent` | Production-grade ReAct agents with state |
| **LangChain** `AgentExecutor` | Classic agent execution loop |
| **AutoGen** | Role-based multi-agent conversation |
| **CrewAI** | Role + task based agent crews |
| **Semantic Kernel** Agents | Microsoft enterprise agent framework |
| **Google ADK** | Gemini-native agent development |
| **Phidata** | Batteries-included agent framework |

### Level 4 — Agentic Pipeline

| Library | Purpose |
|---|---|
| **LangGraph** | State machine agent graph — primary framework |
| **LangGraph Platform** | Hosted, scalable LangGraph deployments |
| **AutoGen / Magentic-One** | Multi-agent conversation orchestration |
| **CrewAI Flows** | Structured multi-agent workflows |
| **Temporal** | Durable, fault-tolerant long-running pipelines |
| **Prefect** | Observability-first ML/AI pipeline orchestration |
| **Dagster** | Data-asset-centric pipeline orchestration |
| **Apache Airflow** | DAG-based scheduling for AI workflows |
| **LangSmith** | Tracing + evaluation across full pipeline |
| **Langfuse** | Open-source full-pipeline observability |

---

> **Previous**: [Volume II — Prompt & Context Engineering](./02-prompt-and-context-engineering.md)  
> **Next in Series**: Volume IV — RAG Architecture Deep Dive  
> **Related**: [Agent Testing Patterns](#) | [LLM Observability](#) | [AI Security & Red-Teaming](#)

---

*Quality Engineering Journals — Building confidence in AI systems through rigorous measurement.*

---
Antigravity > Claude Sonnet 4.6