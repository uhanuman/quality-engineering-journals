# 🧠 AI Interview Cheat Sheet: From Prompt to Agentic Orchestration
 
---

## 1️⃣ What Happens When You Run a Prompt

**Flow of execution:**

---

## 1️⃣ Tokenization

- **What it does**: Splits text into tokens (subwords/characters).  
- **Claude (Anthropic)**:  
  - Uses **SentencePiece** tokenizer (variant of BPE).  
  - Library: [sentencepiece](https://github.com/google/sentencepiece).  
- **Copilot (OpenAI GPT-based)**:  
  - Uses **Byte Pair Encoding (BPE)** tokenizer.  
  - Library: [tiktoken](https://github.com/openai/tiktoken).  

---

## 2️⃣ Vector Embedding

- **What it does**: Converts token IDs into dense vectors.  
- **Claude**:  
  - Embedding layer built into the model (not separately exposed).  
  - Can integrate with **FAISS** or **Weaviate** for vector search.  
- **Copilot (OpenAI)**:  
  - Exposes embeddings via **OpenAI Embeddings API** (`text-embedding-ada-002`).  
  - Libraries: `openai` Python SDK, `langchain` for orchestration.  

---

## 3️⃣ LLM (Large Language Model)

- **Claude**:  
  - Models: **Claude 3.5 Sonnet, Claude 3 Opus**.  
  - Framework: Anthropic API.  
- **Copilot (OpenAI GPT)**:  
  - Models: **GPT-4, GPT-3.5**.  
  - Framework: OpenAI API (`chat/completions`).  

---

## 4️⃣ Agent Layer

- **What it does**: Interprets LLM outputs into actions (tool use, API calls).  
- **Claude**:  
  - Often integrated with **LangChain** or **LlamaIndex** for agent workflows.  
  - Supports “tool use” via structured JSON outputs.  
- **Copilot (OpenAI)**:  
  - Uses **LangChain Agents** or **Semantic Kernel (Microsoft)** for orchestration.  
  - Copilot Studio integrates with **plugins/tools**.  

---

## 5️⃣ Agentic AI

- **Claude**:  
  - Positioned as a **constitutional AI agent** (guided by safety rules).  
  - Often used with **LangChain** for autonomous workflows.  
- **Copilot (OpenAI GPT)**:  
  - Integrated into **Microsoft Copilot ecosystem** (Office, GitHub, Windows).  
  - Uses **Semantic Kernel** for agentic orchestration.  

---

## 6️⃣ Agentic Orchestration

- **Claude**:  
  - Orchestration via **LangChain Multi-Agent** or **CrewAI**.  
  - External orchestrators handle task routing.  
- **Copilot (OpenAI GPT)**:  
  - Orchestration via **Semantic Kernel** (Microsoft’s agent framework).  
  - Supports **plugin orchestration** in GitHub Copilot and M365 Copilot.  

---

## 📊 Diagram with Tool Mapping

# 🧠 AI Pipeline with Tool/Library Examples (Claude vs Copilot)

---

## 1️⃣ Tokenization

- **What it does**: Splits text into tokens (subwords/characters).  
- **Claude (Anthropic)**:  
  - Uses **SentencePiece** tokenizer (variant of BPE).  
  - Library: [sentencepiece](https://github.com/google/sentencepiece).  
- **Copilot (OpenAI GPT-based)**:  
  - Uses **Byte Pair Encoding (BPE)** tokenizer.  
  - Library: [tiktoken](https://github.com/openai/tiktoken).  

---

## 2️⃣ Vector Embedding

- **What it does**: Converts token IDs into dense vectors.  
- **Claude**:  
  - Embedding layer built into the model (not separately exposed).  
  - Can integrate with **FAISS** or **Weaviate** for vector search.  
- **Copilot (OpenAI)**:  
  - Exposes embeddings via **OpenAI Embeddings API** (`text-embedding-ada-002`).  
  - Libraries: `openai` Python SDK, `langchain` for orchestration.  

---

## 3️⃣ LLM (Large Language Model)

- **Claude**:  
  - Models: **Claude 3.5 Sonnet, Claude 3 Opus**.  
  - Framework: Anthropic API.  
- **Copilot (OpenAI GPT)**:  
  - Models: **GPT-4, GPT-3.5**.  
  - Framework: OpenAI API (`chat/completions`).  

---

## 4️⃣ Agent Layer

- **What it does**: Interprets LLM outputs into actions (tool use, API calls).  
- **Claude**:  
  - Often integrated with **LangChain** or **LlamaIndex** for agent workflows.  
  - Supports “tool use” via structured JSON outputs.  
- **Copilot (OpenAI)**:  
  - Uses **LangChain Agents** or **Semantic Kernel (Microsoft)** for orchestration.  
  - Copilot Studio integrates with **plugins/tools**.  

---

## 5️⃣ Agentic AI

- **Claude**:  
  - Positioned as a **constitutional AI agent** (guided by safety rules).  
  - Often used with **LangChain** for autonomous workflows.  
- **Copilot (OpenAI GPT)**:  
  - Integrated into **Microsoft Copilot ecosystem** (Office, GitHub, Windows).  
  - Uses **Semantic Kernel** for agentic orchestration.  

---

## 6️⃣ Agentic Orchestration

- **Claude**:  
  - Orchestration via **LangChain Multi-Agent** or **CrewAI**.  
  - External orchestrators handle task routing.  
- **Copilot (OpenAI GPT)**:  
  - Orchestration via **Semantic Kernel** (Microsoft’s agent framework).  
  - Supports **plugin orchestration** in GitHub Copilot and M365 Copilot.  

---
 

## 📊 Diagram: Prompt → Tokenization → Embedding → LLM → Agent → Orchestration

User Prompt
↓
[Tokenizer] → Tokens → IDs
↓
[Embedding Layer] → Vectors
↓
[LLM] → Predictions / Reasoning
↓
[Agent] → Actions (API, DB, Tools)
↓
[Orchestrator] → Multi-Agent Coordination


![alt text](/ai-images/Copilot_AI_CheatSheet.png)
---

## ✅ Interview Tips

- **Phrase it clearly**:  
  > “When a user runs a prompt, the text is first tokenized into smaller units, converted into embeddings, and processed by the LLM. The LLM generates outputs, but agents interpret those outputs into actions. Agentic AI refers to autonomous agents, and agentic orchestration is the coordination of multiple agents to achieve complex workflows.”

- **Use analogies**:  
  - Tokenization = chopping sentences into Lego blocks.  
  - Embeddings = placing Lego blocks in a 3D space where similar blocks are close.  
  - LLM = the brain predicting the next Lego block.  
  - Agent = the worker who uses those blocks to build something.  
  - Orchestration = the project manager ensuring all workers build the right parts together.

---
