# AI Fundamentals: The Complete Pipeline

> **Journal Series**: Quality Engineering — AI Systems  
> **Edition**: Volume I — AI Fundamentals  
> **Published**: October 2026  
> **Author**: Quality Engineering Team

---

## Table of Contents

1. [Overview](#overview)
2. [The Full AI Pipeline](#the-full-ai-pipeline)
3. [Stage 1 — User Prompt](#stage-1--user-prompt)
4. [Stage 2 — Tokenization](#stage-2--tokenization)
5. [Stage 3 — Embeddings](#stage-3--embeddings)
6. [Stage 4 — Vector Databases & RAG](#stage-4--vector-databases--rag)
7. [Stage 5 — Large Language Models & Attention](#stage-5--large-language-models--attention)
8. [Stage 6 — Agents (ReAct, Memory, Control Loops)](#stage-6--agents-react-memory-control-loops)
9. [Stage 7 — Tools, MCP & Connectors](#stage-7--tools-mcp--connectors)
10. [Stage 8 — Orchestration (Workflows, State, Handoff)](#stage-8--orchestration-workflows-state-handoff)
11. [Stage 9 — Output & Output Parsers](#stage-9--output--output-parsers)
12. [Output Quality: Evaluation & Observability](#output-quality-evaluation--observability)
13. [Key Libraries & Frameworks Quick Reference](#key-libraries--frameworks-quick-reference)
14. [Summary](#summary)

---

## Overview

Modern AI systems are not monolithic black boxes — they are carefully engineered **pipelines** of composable stages, each with its own abstractions, failure modes, and quality levers. This journal documents every stage of a production AI system: from the raw text a user types to the verified, evaluated response that comes back.

Understanding this pipeline end-to-end is the foundation of **AI Quality Engineering** — knowing where things go wrong, what to measure, and how to intervene.

```
User Prompt
    │
    ▼
Tokenization  ──────────────────────── (SentencePiece · WordPiece · TikToken · BPE)
    │
    ▼
Embeddings  ─────────────────────────── (sentence-transformers · OpenAI Ada · Cohere)
    │
    ▼
Vector DB / RAG ─────────────────────── (Pinecone · Weaviate · ChromaDB · FAISS)
    │
    ▼
LLM + Attention ─────────────────────── (GPT-4o · Gemini · Claude · Llama · Mistral)
    │
    ▼
Agents (ReAct · Memory · Control) ───── (LangChain · LangGraph · AutoGen · CrewAI)
    │
    ▼
Tools / MCP / Connectors ────────────── (LangChain Tools · OpenAI Functions · MCP)
    │
    ▼
Orchestration ───────────────────────── (LangGraph · Prefect · Temporal · Airflow)
    │
    ▼
Output ──────────────────────────────── (Structured · Streaming · Multimodal)
    │
    ▼
Output Parsers ──────────────────────── (Pydantic · Guardrails · Instructor · LCEL)
    │
    ▼
Evaluation & Observability ──────────── (RAGAS · DeepEval · LangSmith · Arize · Helicone)
```

---

## The Full AI Pipeline

The pipeline below is not linear in practice — it involves **feedback loops**, **conditional branches**, and **multi-agent coordination** — but understanding it stage by stage provides the mental model needed to debug, test, and improve any AI application.

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                          PRODUCTION AI SYSTEM PIPELINE                          │
│                                                                                 │
│  ┌──────────┐   ┌────────────┐   ┌──────────────┐   ┌──────────────────────┐  │
│  │  USER    │──▶│ TOKENIZER  │──▶│  EMBEDDINGS  │──▶│  VECTOR DB + RAG     │  │
│  │  PROMPT  │   │            │   │              │   │                      │  │
│  └──────────┘   └────────────┘   └──────────────┘   └──────────────────────┘  │
│                                                                  │              │
│                                                                  ▼              │
│  ┌──────────────┐   ┌─────────────────────────────────────────────────────┐   │
│  │  OUTPUT      │◀──│         LLM CORE (Transformer + Attention)          │   │
│  │  PARSERS     │   │   GPT · Gemini · Claude · LLaMA · Mistral · Mixtral │   │
│  └──────────────┘   └─────────────────────────────────────────────────────┘   │
│         │                              │                                        │
│         ▼                              ▼                                        │
│  ┌──────────────┐   ┌─────────────────────────────────────────────────────┐   │
│  │  EVALUATION  │   │  AGENTS (ReAct · Memory · Control Loops)            │   │
│  │  & OBSERV.   │   │  Tools · MCP · Connectors · Orchestration           │   │
│  └──────────────┘   └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## Stage 1 — User Prompt

### What It Is

The **user prompt** is the entry point to the entire AI pipeline. It is raw natural language text — a question, instruction, conversation turn, or structured request — submitted by a human (or another system) to the AI.

### Prompt Anatomy

A well-structured prompt in production typically contains several layers:

| Component | Role | Example |
|---|---|---|
| **System Prompt** | Sets the AI's persona, constraints, and output format | `"You are a precise QA engineer. Answer only with verified facts."` |
| **Context / Memory** | Prior conversation, retrieved documents, session state | Prior turns, RAG chunks |
| **User Message** | The actual request for this turn | `"Explain attention mechanisms in transformers."` |
| **Instructions / Format** | Output schema, length, style constraints | `"Respond in JSON with keys: summary, steps, confidence"` |
| **Few-shot Examples** | Examples of desired input/output pairs | 2–5 worked examples |

### Prompt Engineering Techniques

```
Zero-Shot    ──▶  Direct instruction, no examples
Few-Shot     ──▶  2–5 input/output demonstration examples
Chain-of-Thought (CoT) ──▶  "Think step by step..."
Tree-of-Thought (ToT)  ──▶  Explore multiple reasoning branches
Self-Consistency       ──▶  Sample multiple responses, vote on best
ReAct Prompting        ──▶  Reason + Act interleaved with tool calls
```

### Relevant Libraries & Tools

| Library / Tool | Purpose |
|---|---|
| **LangChain** (`PromptTemplate`, `ChatPromptTemplate`) | Templated, composable prompt construction |
| **LlamaIndex** (`PromptHelper`) | Context-aware prompt management |
| **DSPy** | Programmatic, auto-optimizing prompt compilation |
| **Guidance** (Microsoft) | Constrained generation via structured prompts |
| **PromptFlow** (Azure) | Visual prompt pipeline design and evaluation |

---

## Stage 2 — Tokenization

### What It Is

Before text reaches the model, it must be **tokenized** — converted from raw Unicode characters into a sequence of integer IDs that the model was trained on. Tokens are the atomic unit of all language model computation.

> **Key insight**: Models do not see words or characters — they see tokens. A token is typically 3–4 characters of English text on average, but this varies wildly by language, casing, and vocabulary.

### How Tokenization Works

```
Input string:   "Quality Engineering"
                       │
                       ▼
            ┌──────────────────────┐
            │  Tokenizer Algorithm │  (BPE · WordPiece · SentencePiece · Unigram)
            └──────────────────────┘
                       │
                       ▼
Token strings:  ["Quality", " Eng", "ineering"]
Token IDs:      [  18173,   22195,    4467    ]
```

### Tokenization Algorithms

#### Byte-Pair Encoding (BPE)
- Starts from individual characters and iteratively merges the most frequent adjacent pairs
- Used by: GPT-2, GPT-3, GPT-4, LLaMA, Mistral, Falcon
- Vocabulary size: typically 32K–100K tokens

#### WordPiece
- Similar to BPE but uses a likelihood-based merge criterion instead of raw frequency
- Used by: BERT, DistilBERT, RoBERTa, ALBERT
- Marks continuation sub-words with `##` prefix (e.g., `engineer` → `engineer`, `##ing`)

#### SentencePiece
- Language-agnostic, treats the text as a raw byte stream (no pre-tokenization step)
- Supports both BPE and Unigram Language Model algorithms
- Used by: T5, mT5, XLNet, Gemma, PaLM, ALBERT, most multilingual models
- Key advantage: works for any language including those without whitespace (Japanese, Chinese, Thai)

#### Unigram Language Model
- Starts from a large vocabulary and prunes tokens based on likelihood reduction
- More probabilistic / flexible than BPE
- Used by: XLNet, mBART, many multilingual/code models

### Tokenization Libraries

| Library | Creator | Used By / Notes |
|---|---|---|
| **TikToken** | OpenAI | GPT-3.5, GPT-4, GPT-4o, o1, o3 — extremely fast (Rust core) |
| **SentencePiece** | Google | T5, mT5, Gemma, PaLM, most Google models |
| **HuggingFace Tokenizers** | HuggingFace | Wraps BPE, WordPiece, SentencePiece, Unigram — universal interface |
| **WordPiece** (via `transformers`) | HuggingFace / Google | BERT family, DistilBERT, RoBERTa |
| **tokenizers** (Rust) | HuggingFace | High-performance tokenizer backend powering the `transformers` library |

### Quality Engineering Concern: Token Budget

```
Context Window = Max Tokens In + Max Tokens Out

Model          Context Window
─────────────────────────────
GPT-4o         128,000 tokens (~96,000 words)
Claude 3.5     200,000 tokens (~150,000 words)
Gemini 1.5 Pro 1,000,000 tokens (~750,000 words)
Gemini 2.5 Pro 1,048,576 tokens
LLaMA 3.1      128,000 tokens
```

**Testing concern**: Token overflow, truncation strategies, and cost-per-token must be monitored in production.

---

## Stage 3 — Embeddings

### What It Is

**Embeddings** are dense, high-dimensional numerical vectors that represent the semantic meaning of text. Unlike token IDs (which are arbitrary integers), embeddings place semantically similar text close together in vector space.

```
"The dog ran fast"        ──▶  [0.12, -0.45, 0.88, 0.03, ... ]  (768 or 1536 dims)
"The canine sprinted"     ──▶  [0.11, -0.47, 0.86, 0.04, ... ]  ← Very close!
"The stock market fell"   ──▶  [-0.72, 0.33, -0.11, 0.91, ...]  ← Far away
```

### Embedding Model Types

#### Encoder-Only Transformers (Bi-directional)
Best for embeddings because they see the full context in both directions:
- **BERT** (Google, 2018) — foundational model
- **RoBERTa** (Meta, 2019) — optimized BERT training
- **sentence-transformers / SBERT** — fine-tuned BERT for semantic similarity
- **E5** (Microsoft) — state-of-the-art multilingual embeddings
- **BGE** (BAAI) — top-ranked on MTEB leaderboard

#### API-Based Embedding Services
| Model | Provider | Dimensions | Notes |
|---|---|---|---|
| `text-embedding-3-small` | OpenAI | 1,536 | Cost-efficient, high quality |
| `text-embedding-3-large` | OpenAI | 3,072 | Highest accuracy |
| `embed-english-v3.0` | Cohere | 1,024 | Supports compression |
| `textembedding-gecko` | Google Vertex | 768 | Multilingual variant available |
| `amazon.titan-embed-text-v2` | AWS Bedrock | 1,024 | Natively integrated with AWS |

### Key Libraries

| Library | Purpose |
|---|---|
| **sentence-transformers** | High-level API for computing sentence / paragraph embeddings |
| **transformers** (HuggingFace) | Raw access to all embedding models |
| **LangChain Embeddings** | Unified interface wrapping OpenAI, Cohere, HuggingFace, etc. |
| **LlamaIndex Embeddings** | Similar abstraction, document-centric |
| **Instructor** (MTEB) | Instruction-tuned embeddings for task-specific retrieval |

### Similarity Metrics

```python
# Cosine Similarity — most common for semantic search
cos_sim = dot(A, B) / (|A| * |B|)

# Euclidean Distance — L2 distance in vector space
l2 = sqrt(sum((a_i - b_i)^2))

# Dot Product — used when embeddings are normalized
dot_product = sum(a_i * b_i)
```

---

## Stage 4 — Vector Databases & RAG

### Vector Databases

A **vector database** stores embedding vectors alongside their source documents and supports **approximate nearest neighbor (ANN)** search at massive scale.

```
Query: "How does attention work?"
    │
    ▼
Embed query ──▶ [0.12, 0.45, ...]
    │
    ▼
ANN Search ──▶ Find top-K most similar vectors in DB
    │
    ▼
Retrieved Chunks ──▶ ["Attention is...", "The transformer architecture...", ...]
```

### Vector Database Comparison

| Database | Type | Highlights | Best For |
|---|---|---|---|
| **Pinecone** | Managed Cloud | Serverless, auto-scaling, hybrid search | Production SaaS |
| **Weaviate** | Open-source / Cloud | GraphQL API, hybrid search, multi-modal | Flexible deployment |
| **ChromaDB** | Open-source | Lightweight, in-process, developer-first | Prototyping, local dev |
| **FAISS** | Library (Meta) | Extremely fast, CPU/GPU, in-memory | High-throughput research |
| **Qdrant** | Open-source / Cloud | Rust-based, payload filtering, sparse+dense | Performance-critical apps |
| **Milvus** | Open-source | Distributed, enterprise-grade, Kubernetes-native | Large-scale enterprise |
| **pgvector** | PostgreSQL extension | Stays in your existing DB | Simple/existing Postgres infra |
| **LanceDB** | Open-source | Columnar (Lance format), embedded | Edge / notebook use cases |
| **Redis VSS** | Redis extension | Ultra-low latency, in-memory | Real-time applications |

### RAG — Retrieval-Augmented Generation

**RAG** is the pattern of augmenting an LLM's prompt with relevant external knowledge retrieved at inference time, instead of relying solely on the model's parametric memory.

```
┌────────────────────────────────────────────────────────────────┐
│                      RAG PIPELINE                              │
│                                                                │
│  INDEXING (offline):                                           │
│  Documents ──▶ Chunk ──▶ Embed ──▶ Store in Vector DB         │
│                                                                │
│  RETRIEVAL (online):                                           │
│  Query ──▶ Embed ──▶ ANN Search ──▶ Top-K Chunks              │
│                                                                │
│  GENERATION (online):                                          │
│  [System Prompt] + [Retrieved Chunks] + [User Query]          │
│          │                                                     │
│          ▼                                                     │
│        LLM ──▶ Grounded Response                              │
└────────────────────────────────────────────────────────────────┘
```

### RAG Variants

| Variant | Description | When to Use |
|---|---|---|
| **Naive RAG** | Single retrieval → generation pass | Simple Q&A over documents |
| **Advanced RAG** | Re-ranking, query rewriting, HyDE | Production accuracy requirements |
| **Modular RAG** | Pluggable retriever, re-ranker, reader components | Customizable pipelines |
| **Self-RAG** | Model decides when to retrieve | Dynamic, adaptive retrieval |
| **Corrective RAG (CRAG)** | Evaluates retrieval quality, falls back if poor | High-reliability systems |
| **Agentic RAG** | Agent iteratively retrieves and reasons | Complex multi-hop questions |
| **Graph RAG** (Microsoft) | Knowledge graph + vector hybrid | Entity-rich domains |
| **HyDE** | Hypothetical Document Embeddings | Improves sparse query matching |

### Chunking Strategies

```
Fixed-Size Chunking     ──▶  Split every N tokens (fast, may cut context)
Recursive Chunking      ──▶  Split on paragraphs → sentences → words
Semantic Chunking       ──▶  Split on embedding similarity boundaries
Document-Based          ──▶  Respect natural document sections (headings, pages)
Sliding Window          ──▶  Overlapping chunks to preserve context at boundaries
```

### Relevant Libraries

| Library | Purpose |
|---|---|
| **LangChain** (`RecursiveCharacterTextSplitter`, `VectorstoreRetriever`) | End-to-end RAG pipeline construction |
| **LlamaIndex** | Document-centric RAG with rich indexing strategies |
| **Haystack** (deepset) | Production RAG with pipelines and evaluation built-in |
| **Unstructured.io** | Parse PDFs, Word, HTML, images into clean text chunks |
| **RAGAS** | Automated RAG evaluation (faithfulness, context recall, etc.) |

---
## Stage 5 — Large Language Models & Attention

### The Transformer Architecture

All modern LLMs are based on the **Transformer** architecture introduced in *"Attention is All You Need"* (Vaswani et al., 2017). The critical innovation is the **self-attention mechanism**.

```
Input Tokens (IDs)
        │
        ▼
┌──────────────────────┐
│  Token Embeddings    │  (lookup table: token ID → embedding vector)
│  + Positional Encoding│
└──────────────────────┘
        │
        ▼
┌──────────────────────────────────────────────────────┐
│              Transformer Blocks × N                  │
│                                                      │
│  ┌──────────────────────────────────────────────┐   │
│  │           Multi-Head Self-Attention           │   │
│  │  Q = X·Wq   K = X·Wk   V = X·Wv            │   │
│  │  Attention(Q,K,V) = softmax(QKᵀ/√d_k) · V   │   │
│  └──────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────┐   │
│  │         Feed-Forward Network (FFN)            │   │
│  │         Layer Norm + Residual Connections     │   │
│  └──────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────┘
        │
        ▼
┌──────────────────────┐
│  Output Head         │  (linear layer → softmax → probability over vocabulary)
└──────────────────────┘
        │
        ▼
   Next Token Prediction
```

### Attention Mechanism Deep Dive

**Scaled Dot-Product Attention** is the core operation:

$$\text{Attention}(Q, K, V) = \text{softmax}\!\left(\frac{QK^\top}{\sqrt{d_k}}\right)V$$

| Symbol | Meaning |
|---|---|
| **Q** (Query) | "What am I looking for?" — current token's representation |
| **K** (Key) | "What do I contain?" — each other token's identity |
| **V** (Value) | "What do I contribute?" — actual content to aggregate |
| **d_k** | Dimensionality of key vectors (scaling factor for stability) |

**Multi-Head Attention** runs `h` attention heads in parallel, each learning different relational patterns (syntax, coreference, long-range dependencies), then concatenates and projects their outputs.

### Attention Variants

| Variant | Key Idea | Used By |
|---|---|---|
| **Full Self-Attention** | Every token attends to every other token — O(n²) | BERT, GPT-2 |
| **Causal (Masked) Attention** | Tokens only attend to past tokens — prevents lookahead | GPT family, LLaMA, Claude |
| **Flash Attention** | Memory-efficient tiling — same result, less VRAM | Most modern models |
| **Multi-Query Attention (MQA)** | Shared K/V heads across Q heads — faster inference | PaLM, Falcon |
| **Grouped-Query Attention (GQA)** | Balance between MHA and MQA | LLaMA 2/3, Mistral, Gemma |
| **Sliding Window Attention** | Each token attends only to local window | Mistral, Longformer |
| **Sparse Attention** | Attend to a sparse subset of positions | BigBird, Longformer |

### Key Model Families

| Model | Creator | Parameters | Context | License |
|---|---|---|---|---|
| **GPT-4o** | OpenAI | ~200B (est) | 128K | Proprietary |
| **o1, o3, o4-mini** | OpenAI | Unknown | 128K | Proprietary |
| **Claude 3.5 / 4** | Anthropic | Unknown | 200K | Proprietary |
| **Gemini 1.5 / 2.0 / 2.5 Pro** | Google DeepMind | Unknown | 1M+ | Proprietary |
| **LLaMA 3.1 / 3.3** | Meta | 8B–405B | 128K | Meta License |
| **Mistral / Mixtral** | Mistral AI | 7B–141B | 32K–128K | Apache 2.0 |
| **Gemma 2 / 3** | Google | 2B–27B | 128K | Gemma License |
| **Qwen 2.5** | Alibaba | 0.5B–72B | 128K | Apache 2.0 |
| **Falcon 3** | TII UAE | 1B–10B | 32K | Apache 2.0 |
| **Command R+** | Cohere | 104B | 128K | CC-BY-NC |
| **Phi-4** | Microsoft | 14B | 16K | MIT |
| **DeepSeek V3 / R1** | DeepSeek | 671B (MoE) | 128K | MIT |

### Inference Parameters That Matter

| Parameter | Description | Impact |
|---|---|---|
| **Temperature** | Randomness of sampling (0 = deterministic, 1 = full distribution) | Creativity vs. consistency |
| **Top-P (nucleus)** | Sample from the smallest set of tokens comprising P% probability | Quality filter on low-prob tokens |
| **Top-K** | Sample only from the K most likely next tokens | Hard token count limit |
| **Max Tokens** | Maximum response length in tokens | Cost & latency control |
| **Stop Sequences** | Halt generation at specific strings | Structured output control |
| **Frequency Penalty** | Penalize tokens that have appeared frequently | Reduces repetition |
| **Presence Penalty** | Penalize tokens that have appeared at all | Increases topic diversity |
| **Seed** | Fixed random seed for deterministic outputs | Reproducibility in testing |

---

## Stage 6 — Agents (ReAct, Memory, Control Loops)

### What Is an AI Agent?

An **AI Agent** is an LLM that has been given the ability to:
1. **Reason** about a goal
2. **Take actions** (call tools, APIs, sub-agents)
3. **Observe** the results of those actions
4. **Iterate** until the goal is achieved

> The fundamental shift: from a single prompt → response to a **control loop** that reasons and acts over multiple steps.

### ReAct Pattern (Reason + Act)

**ReAct** (*Yao et al., 2023*) is the foundational agent prompting pattern that interleaves reasoning traces with actions:

```
Thought:  I need to find the current price of AAPL stock.
Action:   search_web("AAPL stock price today")
Observation: Apple Inc. (AAPL) is trading at $227.50

Thought:  I have the price. Now I need to calculate the P/E ratio.
Action:   calculator("227.50 / 6.43")
Observation: 35.38

Thought:  I have both values. I can now provide the answer.
Final Answer: AAPL P/E ratio is approximately 35.4 based on today's price of $227.50.
```

### Agent Memory Architecture

Agents maintain multiple types of memory:

```
┌─────────────────────────────────────────────────────────────┐
│                    AGENT MEMORY TYPES                       │
│                                                             │
│  ┌─────────────────┐    ┌─────────────────────────────┐    │
│  │  IN-CONTEXT     │    │  EXTERNAL / LONG-TERM       │    │
│  │  (Short-Term)   │    │                             │    │
│  │                 │    │  Episodic  ──▶ Vector DB    │    │
│  │  • Chat history │    │  Semantic  ──▶ Knowledge    │    │
│  │  • Tool results │    │                  Graph      │    │
│  │  • Scratchpad   │    │  Procedural──▶ Tool specs   │    │
│  │  • Retrieved    │    │  Working   ──▶ Shared state │    │
│  │    chunks       │    │                  store      │    │
│  └─────────────────┘    └─────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

| Memory Type | Storage | Example |
|---|---|---|
| **Conversational Buffer** | In-context window | Full chat history |
| **Conversation Summary** | LLM-compressed summary in context | Older turns summarized |
| **Buffer Window** | Last N turns only | Rolling window |
| **Entity Memory** | Tracked entities (people, places) | `{"Alice": "is the project manager"}` |
| **Vector Store Memory** | Semantic search over past conversations | Retrieve relevant past exchanges |
| **Knowledge Graph** | Graph DB (Neo4j, Neptune) | Entity relationships |

### Control Loops & Planning Strategies

```
┌──────────────────────────────────────────────────────────────────┐
│                   AGENT CONTROL PATTERNS                         │
│                                                                  │
│  REACT LOOP          PLAN-AND-EXECUTE        REFLECTION          │
│                                                                  │
│  Goal ──▶ Think      Goal ──▶ Plan           Act ──▶ Evaluate   │
│         ──▶ Act             ──▶ Execute              ──▶ Reflect  │
│         ──▶ Observe         ──▶ Observe              ──▶ Refine   │
│         ──▶ Repeat          ──▶ Replan (if needed)   ──▶ Re-Act   │
└──────────────────────────────────────────────────────────────────┘
```

| Planning Strategy | Description | Frameworks |
|---|---|---|
| **ReAct** | Interleaved reason + act loop | LangChain, LlamaIndex |
| **Plan-and-Execute** | Upfront plan, then execute each step | LangChain Plan-and-Execute |
| **Tree-of-Thoughts (ToT)** | Explore multiple reasoning branches | Research, custom |
| **Reflexion** | Self-reflection and self-correction loop | LangGraph |
| **LATS** | Language Agent Tree Search | Research |
| **HumanLoop** | Human-in-the-loop approval at key steps | LangGraph, Langsmith |

### Agent Frameworks

| Framework | Creator | Highlights |
|---|---|---|
| **LangChain Agents** | LangChain | Most widely used, rich ecosystem, `AgentExecutor` |
| **LangGraph** | LangChain | State machine graph for complex, stateful, multi-agent flows |
| **AutoGen** | Microsoft | Multi-agent conversation framework, role-based agents |
| **CrewAI** | CrewAI | Role-based crew of agents with tasks and delegation |
| **Semantic Kernel** | Microsoft | Enterprise-grade, .NET and Python, plugin-based |
| **Haystack Agents** | deepset | Pipeline-native agent with built-in tool support |
| **Phidata** | Phidata | Production-ready agent framework with built-in memory/tools |
| **Agency Swarm** | VRSEN | GPT-based multi-agent communication |
| **Google ADK** | Google | Agent Development Kit for Gemini-based agents |
| **Vertex AI Agents** | Google Cloud | Managed agent platform with Dialogflow integration |
| **Amazon Bedrock Agents** | AWS | Managed agent service with Action Groups |

---

## Stage 7 — Tools, MCP & Connectors

### What Are Tools?

**Tools** are the hands of an AI agent — they extend what the LLM can do beyond generating text by allowing it to interact with external systems.

```
Tool Call Flow:
LLM generates structured tool call ──▶ Tool executor invokes the function
──▶ Result returned as observation ──▶ LLM incorporates result in next reasoning step
```

### Tool Categories

| Category | Examples | Libraries |
|---|---|---|
| **Search / Web** | Tavily, SerpAPI, DuckDuckGo, Bing Search | `langchain-community` |
| **Code Execution** | Python REPL, Jupyter kernel, E2B sandbox | `langchain.tools.python`, E2B |
| **File System** | Read/write files, list directories | LangChain filesystem tools |
| **Database** | SQL queries, NoSQL, GraphQL | SQLDatabase toolkit, LlamaIndex |
| **APIs / REST** | Any HTTP endpoint | `requests`, OpenAPI tool builder |
| **Document Processing** | PDF, Word, CSV, HTML parsing | Unstructured.io, PyMuPDF |
| **Communication** | Email (Gmail), Slack, Teams | LangChain Gmail, Slack tools |
| **Browser / Scraping** | Playwright, Puppeteer, Selenium | `langchain-playwright` |
| **Memory / State** | Redis, SQLite, VectorStore ops | LangChain memory tools |
| **Math / Code** | Wolfram Alpha, calculator, sympy | Built-in LangChain tools |
| **Image / Vision** | DALL-E, Stable Diffusion, Google Vision | Multimodal tool integrations |

### OpenAI Function Calling vs. Tool Calling

```json
// Tool Definition (OpenAI format — also used by many frameworks)
{
  "type": "function",
  "function": {
    "name": "get_weather",
    "description": "Get current weather for a city",
    "parameters": {
      "type": "object",
      "properties": {
        "city": { "type": "string", "description": "City name" },
        "units": { "type": "string", "enum": ["celsius", "fahrenheit"] }
      },
      "required": ["city"]
    }
  }
}
```

### Model Context Protocol (MCP)

**MCP** (Model Context Protocol), introduced by Anthropic in 2024 and rapidly adopted across the ecosystem, is an **open standard** for connecting AI models to external data sources and tools via a unified protocol.

```
┌────────────────────────────────────────────────────────────┐
│                    MCP ARCHITECTURE                        │
│                                                            │
│  ┌─────────────┐      MCP Protocol      ┌───────────────┐ │
│  │  MCP Client │◀─────────────────────▶│  MCP Server   │ │
│  │  (Host App) │   JSON-RPC over        │               │ │
│  │  Claude /   │   stdio / SSE /        │  Exposes:     │ │
│  │  Cursor /   │   WebSocket            │  - Tools      │ │
│  │  Windsurf   │                        │  - Resources  │ │
│  └─────────────┘                        │  - Prompts    │ │
│                                         └───────────────┘ │
└────────────────────────────────────────────────────────────┘
```

| MCP Primitive | Description | Example |
|---|---|---|
| **Tools** | Functions the model can invoke | `execute_sql`, `search_files` |
| **Resources** | Data the model can read | File contents, DB records |
| **Prompts** | Reusable prompt templates | `code_review_prompt` |
| **Sampling** | Ask the host to call the LLM | For agent recursion |

**Notable MCP Servers:**
- `@modelcontextprotocol/server-filesystem` — File system access
- `@modelcontextprotocol/server-github` — GitHub API
- `@modelcontextprotocol/server-postgres` — PostgreSQL access
- `@modelcontextprotocol/server-brave-search` — Brave web search
- `mcp-server-sqlite` — SQLite database
- `mcp-atlassian` — Jira and Confluence

### LangChain Connectors (Integrations)

LangChain provides **300+** pre-built integrations via packages:

```
langchain-openai          ──▶  OpenAI, Azure OpenAI
langchain-anthropic       ──▶  Claude models
langchain-google-genai    ──▶  Gemini models
langchain-community       ──▶  Community-maintained integrations
langchain-aws             ──▶  Amazon Bedrock
langchain-mistralai       ──▶  Mistral models
```

---

## Stage 8 — Orchestration (Workflows, State, Handoff)

### What Is Orchestration?

**Orchestration** governs how multiple agents, tools, and LLM calls are coordinated into coherent, reliable workflows. It manages:
- **State** — what has happened and what is the current situation
- **Routing** — which agent or tool handles which subtask
- **Handoff** — transferring control and context between components
- **Error handling** — retries, fallbacks, human escalation
- **Parallelism** — running independent tasks concurrently

### Orchestration Patterns

```
┌──────────────────────────────────────────────────────────────────────┐
│                   ORCHESTRATION PATTERNS                             │
│                                                                      │
│  SEQUENTIAL          PARALLEL              CONDITIONAL ROUTING       │
│                                                                      │
│  A ──▶ B ──▶ C       A ──▶┐               Input ──▶ Router          │
│                       B ──▶┤──▶ Merge            ├──▶ Path A        │
│                       C ──▶┘               (condition)              │
│                                                  └──▶ Path B        │
│                                                                      │
│  SUPERVISOR / HIERARCHICAL      PEER-TO-PEER (SWARM)                │
│                                                                      │
│  Supervisor                     Agent A ◀──▶ Agent B                │
│    ├──▶ Sub-agent A                   ◀──▶ Agent C                  │
│    ├──▶ Sub-agent B                                                  │
│    └──▶ Sub-agent C                                                  │
└──────────────────────────────────────────────────────────────────────┘
```

### LangGraph (Primary Orchestration for LangChain Agents)

LangGraph models agent workflows as **directed (possibly cyclic) graphs** using a state machine approach:

```python
# Conceptual LangGraph structure
from langgraph.graph import StateGraph, END

graph = StateGraph(AgentState)
graph.add_node("researcher", research_agent)
graph.add_node("writer", writing_agent)
graph.add_node("reviewer", review_agent)

graph.add_edge("researcher", "writer")
graph.add_conditional_edges(
    "reviewer",
    should_revise,                   # Routing function
    {"revise": "writer", "done": END}
)

app = graph.compile(checkpointer=MemorySaver())
```

**Key LangGraph features:**
- **Persistent state** across turns (checkpointing)
- **Human-in-the-loop** interrupts at any node
- **Time-travel** — replay from any past checkpoint
- **Streaming** — token-level and node-level event streaming
- **LangGraph Platform** — hosted, scalable deployment

### Agent Handoff Mechanisms

| Handoff Type | Description | Use Case |
|---|---|---|
| **Direct Handoff** | Agent A explicitly transfers to Agent B | Supervisor → specialized worker |
| **Shared State** | Agents read/write a common state object | Collaborative document editing |
| **Message Passing** | Agents communicate via message queue | Asynchronous pipelines |
| **Function Handoff** | OpenAI Swarm-style handoff via function return | Lightweight multi-agent |
| **Human Escalation** | Agent pauses and waits for human approval | High-stakes decisions |

### Other Orchestration Frameworks

| Framework | Creator | Best For |
|---|---|---|
| **LangGraph** | LangChain | Stateful, cyclic agent workflows |
| **AutoGen / Magentic-One** | Microsoft | Multi-agent conversation orchestration |
| **CrewAI** | CrewAI | Role-based agent crews with task pipelines |
| **Prefect** | Prefect Technologies | Data/ML pipeline orchestration with observability |
| **Temporal** | Temporal Technologies | Durable, fault-tolerant long-running workflows |
| **Apache Airflow** | Apache | DAG-based batch workflow orchestration |
| **Dagster** | Dagster | Data-asset-centric pipeline orchestration |
| **Semantic Kernel Process** | Microsoft | Process-based agent workflows in .NET/Python |

---

## Stage 9 — Output & Output Parsers

### Output Formats

LLMs produce raw text, but production systems require **structured, validated, reliable outputs**:

| Format | Use Case | Risk |
|---|---|---|
| **Plain Text** | Conversational responses, summaries | Hard to parse programmatically |
| **Markdown** | Documentation, reports | Rendering-dependent |
| **JSON** | Structured data extraction, APIs | Model may hallucinate wrong schema |
| **XML** | Legacy system integration | Verbose |
| **YAML** | Config generation | Indentation-sensitive |
| **Code** | Code generation, SQL, shell scripts | Must be executed safely |
| **CSV / TSV** | Tabular data extraction | Delimiter escaping issues |

### Output Parsers

**Output parsers** transform raw LLM text into structured Python/TypeScript objects:

| Parser | Library | Description |
|---|---|---|
| `PydanticOutputParser` | LangChain | Parse JSON output into Pydantic models |
| `JsonOutputParser` | LangChain | Parse raw JSON strings |
| `StructuredOutputParser` | LangChain | Parse defined response schemas |
| `CommaSeparatedListOutputParser` | LangChain | Split CSV output into list |
| `DatetimeOutputParser` | LangChain | Parse datetime strings |
| **Instructor** | `instructor` library | Structured outputs via OpenAI function calling + Pydantic |
| **Outlines** | dottxt-ai | Constrained generation — guarantee valid JSON/regex at decoding |
| **Guardrails AI** | Guardrails | Validation + correction + retry loop for structured output |
| **LMQL** | ETH Zurich | Query language for constrained LLM output |

### Structured Output with Instructor (Example Pattern)

```python
import instructor
from openai import OpenAI
from pydantic import BaseModel

class QAResponse(BaseModel):
    answer: str
    confidence: float  # 0.0 to 1.0
    sources: list[str]
    needs_human_review: bool

client = instructor.from_openai(OpenAI())

response = client.chat.completions.create(
    model="gpt-4o",
    response_model=QAResponse,
    messages=[{"role": "user", "content": "Explain attention mechanisms."}]
)
# response is a validated QAResponse instance — not raw text
```

### Streaming

Production systems increasingly use **streaming** to improve perceived latency:

```
Non-Streaming: User waits 8s → full response appears at once
Streaming:     First token appears in ~200ms → tokens stream in progressively
```

| Streaming Type | Description |
|---|---|
| **Token Streaming** | Yield each token as generated |
| **LangChain Streaming** | `stream()` and `astream()` on any Runnable |
| **LangGraph Streaming** | Node-level events + token-level events |
| **Server-Sent Events (SSE)** | HTTP streaming protocol for web frontends |
| **WebSocket Streaming** | Bidirectional streaming (e.g., Live API) |

---

## Output Quality: Evaluation & Observability

### The Evaluation Framework

> **Core principle of AI Quality Engineering**: *You cannot improve what you cannot measure.*

```
┌──────────────────────────────────────────────────────────────────────────┐
│                    AI OUTPUT QUALITY DIMENSIONS                          │
│                                                                          │
│   Correctness         Faithfulness        Safety             Efficiency  │
│   (Is it right?)     (Is it grounded?)   (Is it safe?)     (Is it fast) │
│        │                    │                  │                  │      │
│        ▼                    ▼                  ▼                  ▼      │
│   Ground Truth        RAG Faithfulness    Toxicity          Latency/Cost │
│   Benchmark           Context Recall      PII Detection     Token usage  │
│   Similarity          Citation Accuracy   Jailbreak         Throughput   │
└──────────────────────────────────────────────────────────────────────────┘
```

### Output Relevancy

**Output Relevancy** measures whether the response actually addresses what was asked.

| Metric | Description | Range |
|---|---|---|
| **Answer Relevance** | Does the answer address the question? | 0–1 (RAGAS) |
| **Context Relevance** | Are retrieved chunks relevant to the query? | 0–1 (RAGAS) |
| **BERTScore** | Semantic similarity using BERT embeddings | F1 score 0–1 |
| **BLEU / ROUGE** | N-gram overlap against reference | 0–1 (lower quality) |
| **Semantic Similarity** | Cosine similarity of embeddings | -1 to 1 |

### Hallucination

**Hallucination** is when the model generates factually incorrect, fabricated, or unsupported content with apparent confidence.

```
Types of Hallucination:
├── Factual Hallucination    ──▶ Wrong facts ("Einstein won the Pulitzer Prize")
├── Faithfulness Hallucination ──▶ Response contradicts or extrapolates beyond context
├── Entity Hallucination     ──▶ Invents people, places, citations that don't exist
└── Logical Hallucination    ──▶ Internally inconsistent reasoning
```

| Tool / Library | Approach | Notes |
|---|---|---|
| **RAGAS** | `faithfulness` metric — NLI-based | Checks if claims are supported by context |
| **DeepEval** | `HallucinationMetric` | GEval-based hallucination scoring |
| **TruLens** | RAG triad: context relevance, groundedness, answer relevance | End-to-end RAG eval |
| **Langchain Evals** | LLM-as-judge hallucination check | Flexible custom criteria |
| **FactScore** | Fine-grained factual precision (splits claims) | Research-grade |
| **Vectara HHEM** | Hughes Hallucination Evaluation Model (open) | Specialized hallucination classifier |

### Safety

| Dimension | Description | Tools |
|---|---|---|
| **Toxicity** | Profanity, hate speech, harmful content | Perspective API, Llama Guard, ShieldLM |
| **PII Detection** | Names, emails, SSNs, phone numbers | Microsoft Presidio, spaCy NER, Amazon Comprehend |
| **Jailbreak Detection** | Attempts to bypass safety guardrails | Garak, custom probing |
| **Prompt Injection** | Malicious instructions embedded in user input | Rebuff, custom classifiers |
| **Bias Detection** | Demographic, gender, racial bias in outputs | BOLD, WinoBias, AI Fairness 360 |
| **Content Moderation** | Flagging NSFW, violent, illegal content | OpenAI Moderation API, AWS Rekognition |

### Ground Truth

**Ground Truth** is the validated, correct reference answer against which model outputs are compared.

```
Ground Truth Sources:
├── Human-annotated datasets    ──▶ Expert-labeled Q&A pairs
├── Golden datasets             ──▶ Curated, high-confidence examples
├── Synthetic ground truth      ──▶ LLM-generated + human-validated
├── Knowledge base              ──▶ Structured facts from authoritative sources
└── Logged production data      ──▶ User feedback signals (thumbs up/down)
```

| Benchmark Dataset | Domain | Metric |
|---|---|---|
| **MMLU** | Massive multitask language understanding | Accuracy |
| **HumanEval** | Code generation | pass@k |
| **GSM8K** | Grade school math | Accuracy |
| **TruthfulQA** | Factual truthfulness | % truthful |
| **BIG-Bench Hard** | Challenging reasoning tasks | Accuracy |
| **MT-Bench** | Multi-turn conversation quality | GPT-4 judge score |
| **RAGAS Testset** | RAG-specific: context precision, recall, faithfulness | RAGAS metrics |

### Evals (LLM-as-Judge)

The emerging standard for scalable, nuanced evaluation is using a **powerful LLM as an evaluator**:

```python
# Example: LLM-as-Judge using DeepEval
from deepeval.metrics import GEval
from deepeval.test_case import LLMTestCaseParams

correctness_metric = GEval(
    name="Correctness",
    criteria="Determine whether the actual output is factually correct "
             "based on the expected output.",
    evaluation_params=[
        LLMTestCaseParams.INPUT,
        LLMTestCaseParams.ACTUAL_OUTPUT,
        LLMTestCaseParams.EXPECTED_OUTPUT
    ],
    threshold=0.7
)
```

| Eval Framework | Highlights |
|---|---|
| **RAGAS** | RAG-specific metrics: faithfulness, answer relevance, context recall |
| **DeepEval** | 14+ metrics, CI/CD integration, Confident AI platform |
| **TruLens** | RAG triad evaluation, feedback functions, dashboards |
| **LangSmith** | LangChain-native evals, dataset management, annotation UI |
| **Promptfoo** | CLI-based LLM testing, regression testing, red-teaming |
| **Evals** (OpenAI) | OpenAI's own eval harness — open source |
| **EleutherAI lm-evaluation-harness** | Standard academic benchmark runner |
| **Phoenix** (Arize) | Open-source LLM tracing and evals |
| **Weights & Biases (W&B)** | `weave` for LLM tracing + evaluation |

### Drift Detection

**Drift** occurs when model behavior degrades silently over time — due to model updates, data distribution shifts, or changing user behavior.

```
Types of Drift:
├── Data Drift       ──▶ Input distribution changes (different query patterns)
├── Concept Drift    ──▶ Ground truth relationship changes (world facts change)
├── Model Drift      ──▶ Underlying model version changes (silent API update)
└── Output Drift     ──▶ Response quality degrades without obvious cause
```

| Tool | Purpose |
|---|---|
| **Arize AI** | Production ML monitoring, LLM-specific drift dashboards |
| **WhyLabs / whylogs** | Data quality and drift profiling |
| **Evidently AI** | Open-source drift detection and LLM monitoring |
| **Arthur AI** | Enterprise ML/LLM monitoring with drift alerts |
| **LangSmith** | Trace-level monitoring, dataset comparison over time |
| **Helicone** | Request-level LLM observability with cost, latency, quality tracking |

### Hallucination Testing (Proactive)

Proactive hallucination testing goes beyond monitoring — it **deliberately probes** the model with questions designed to elicit hallucinations:

```
Test Categories:
├── Fabricated Entity Probes      ──▶ "Tell me about [invented person/company]"
├── False Premise Questions       ──▶ "Why did Einstein win the Pulitzer Prize?"
├── Knowledge Cutoff Probes       ──▶ Questions about events after training cutoff
├── Citation Verification         ──▶ "Cite three papers on X" → verify they exist
├── Contradiction Probes          ──▶ Ask the same question different ways
└── Domain-Specific Fact Checks   ──▶ Medical, legal, financial fact verification
```

| Tool | Approach |
|---|---|
| **Garak** | LLM vulnerability scanner — probes for hallucinations, jailbreaks, bias |
| **Promptfoo** | Configurable red-teaming and hallucination test suites |
| **DeepEval `HallucinationMetric`** | Automated hallucination scoring per test case |
| **RAGAS `faithfulness`** | Checks each claim against retrieved context |
| **Custom pipelines** | Scrape facts from knowledge base, ask model, verify answer |

### Observability

**Observability** is the ability to understand the internal state of an AI system from its external outputs — essential for debugging and improving production AI.

```
Observability Stack:
┌─────────────────────────────────────────────────────────────────┐
│  TRACES         │  Every LLM call, tool call, agent step        │
│  METRICS        │  Latency, tokens, cost, success rate          │
│  LOGS           │  Full input/output pairs, errors              │
│  EVALUATIONS    │  Quality scores attached to traces            │
│  DATASETS       │  Curated examples for regression testing      │
└─────────────────────────────────────────────────────────────────┘
```

| Platform | Type | Highlights |
|---|---|---|
| **LangSmith** | Commercial (LangChain-native) | Deep LangChain integration, annotation UI, evals |
| **Arize Phoenix** | Open-source | OTEL-based tracing, embedding visualizations |
| **Helicone** | Open-source / Cloud | Lightweight proxy, cost tracking, prompt management |
| **Weights & Biases Weave** | Commercial | Rich trace UI, model comparisons, evals |
| **Langfuse** | Open-source / Cloud | Self-hostable, prompt management, cost tracking |
| **Braintrust** | Commercial | Evaluation-first observability platform |
| **OpenTelemetry (OTEL)** | Standard | Vendor-neutral tracing standard — foundation layer |
| **Datadog LLM Observability** | Commercial | Enterprise-grade, integrates with existing Datadog stack |
| **New Relic AI Monitoring** | Commercial | Full-stack AI observability |

---

## Key Libraries & Frameworks Quick Reference

### Core AI Development Stack

| Category | Library / Tool | Language | Maintainer |
|---|---|---|---|
| **LLM Orchestration** | LangChain | Python, JS/TS | LangChain Inc. |
| **Agent Graphs** | LangGraph | Python, JS/TS | LangChain Inc. |
| **Document RAG** | LlamaIndex | Python, TS | LlamaIndex Inc. |
| **Multi-Agent** | AutoGen | Python | Microsoft |
| **Multi-Agent** | CrewAI | Python | CrewAI |
| **Enterprise Agents** | Semantic Kernel | Python, .NET, Java | Microsoft |
| **Tokenization** | TikToken | Python | OpenAI |
| **Tokenization** | SentencePiece | Python, C++ | Google |
| **Tokenization** | HuggingFace Tokenizers | Python | HuggingFace |
| **Embeddings** | sentence-transformers | Python | HuggingFace |
| **Model Hub** | transformers | Python | HuggingFace |
| **Model Serving** | vLLM | Python | UC Berkeley |
| **Model Serving** | Ollama | Go | Ollama |
| **Structured Output** | Instructor | Python | Jason Liu |
| **Structured Output** | Outlines | Python | dottxt-ai |
| **Guardrails** | Guardrails AI | Python | Guardrails AI |
| **RAG Eval** | RAGAS | Python | ExplodingGradients |
| **LLM Testing** | DeepEval | Python | Confident AI |
| **LLM Testing** | Promptfoo | JS/TS | Promptfoo |
| **Observability** | LangSmith | Python, JS/TS | LangChain Inc. |
| **Observability** | Langfuse | Python, JS/TS | Langfuse |
| **Observability** | Phoenix (Arize) | Python | Arize AI |
| **Vector DB** | ChromaDB | Python | Chroma |
| **Vector DB** | Weaviate | Python | Weaviate |
| **Vector DB** | Qdrant | Python | Qdrant |
| **Vector DB (lib)** | FAISS | Python, C++ | Meta |
| **Red-Teaming** | Garak | Python | NVIDIA |
| **Fine-Tuning** | Unsloth | Python | Unsloth AI |
| **Fine-Tuning** | Axolotl | Python | OpenAccess AI |
| **Prompt Optimization** | DSPy | Python | Stanford NLP |
| **Workflow** | Prefect | Python | Prefect |
| **Workflow** | Temporal | Multi-lang | Temporal |

---

## Summary

The following table maps each pipeline stage to its key failure modes and quality gates — the core concern of AI Quality Engineering:

| Stage | Failure Mode | Quality Gate |
|---|---|---|
| **User Prompt** | Ambiguous/adversarial input, prompt injection | Input validation, PII scrubbing, intent classification |
| **Tokenization** | Context overflow, encoding errors | Token count checks, truncation strategy tests |
| **Embeddings** | Poor semantic capture, stale embeddings | Embedding quality benchmarks, retrieval recall@K |
| **Vector DB / RAG** | Retrieval miss, irrelevant chunks, staleness | Context recall, context precision (RAGAS) |
| **Context Engineering** | Context pollution, overflow, lost-in-middle, stale content | Signal-to-noise ratio, redundancy score, compaction loss, positional audit |
| **LLM** | Hallucination, off-topic, unsafe content | Faithfulness scores, safety classifiers |
| **Agents** | Infinite loops, tool misuse, planning failures | Max iteration limits, tool call logging, plan coherence |
| **Tools / MCP** | API failures, incorrect tool selection, injections | Tool call success rate, schema validation |
| **Orchestration** | State corruption, deadlocks, lost handoffs | State checkpointing, integration tests, idempotency |
| **Output Parser** | Schema mismatch, malformed JSON, truncation | Pydantic validation, retry-on-parse-failure |
| **Evaluation** | Metric gaming, distribution shift, annotation bias | Multi-metric dashboards, A/B regression testing |

---

> **Next in Series**: Volume II — Advanced RAG Architectures & Chunking Strategies  
> **Related Reading**: [AI Agent Testing Patterns](#) | [LLM Security & Red-Teaming](#) | [Prompt Engineering Playbook](#)

---

*Quality Engineering Journals — Building confidence in AI systems through rigorous measurement.*


---
Antigravity > Claude Sonnet 4.6