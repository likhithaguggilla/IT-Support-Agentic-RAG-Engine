# IT Support Agentic RAG Engine

An intelligent, multi-agent IT support system built on **n8n** that autonomously resolves Tier-1 employee requests and triages complex issues to human agents, reducing operational load while maintaining enterprise-grade safety controls.

---

## Overview

This system uses **Semantic Reasoning + Retrieval-Augmented Generation (RAG)** to understand employee intent, retrieve accurate policy information, and take real actions, all while keeping a human in the loop for sensitive operations.

**Primary Users:**
- **Internal Employees** — Get instant answers to IT and policy questions without waiting hours for ticket responses.
- **IT Service Desk Managers** — Deflect high-volume Tier-1 tickets to focus human agents on complex infrastructure issues.

---

## Architecture

![Architecture Diagram](https://github.com/likhithaguggilla/enterprise-IT-support-autonomy-AI-engine/blob/main/assets/system-architecture.png)


The architecture follows a **Modular Multi-Workflow** design:
- `02_Support_Agent` — Main router and conversational interface
- `03_Tool_Create_Ticket` — Webhook-based ticket creation middleware
- `04_Ticket_Approver` — Human approval gate → writes to database
- `05_Agent_Evaluation` — Automated LLM-as-a-Judge QA pipeline

---

## Tech Stack

| Component | Technology | Rationale |
|---|---|---|
| Orchestration | [n8n](https://n8n.io) (Self-Hosted) | Native LangChain support, granular data control |
| LLM | Llama 3.3 70B via [Groq](https://groq.com) | Ultra-low latency; open-source, enterprise-safe |
| Vector Database | [Pinecone](https://pinecone.io) | Serverless, scalable semantic search |
| Embedding Model | `sentence-transformers/all-MiniLM-L6-v2` (HuggingFace) | Efficient 384-dim semantic embeddings |
| Document Storage | Google Drive | Cloud-native; bypasses Docker isolation limits |
| Ticket Database | Google Sheets (mock Jira) | Lightweight proof-of-concept persistence |
| Safety Layer | Gmail + Webhooks | Async human-in-the-loop approval flow |

---

## How It Works

#### Phase 1 — Knowledge Ingestion (ETL Pipeline)
PDF documents are ingested via a 4-step pipeline:
1. **Extract** — Pull PDFs from Google Drive
2. **Chunk** — Split into 500-char segments (50-char overlap) using Recursive Text Splitter
3. **Embed** — Convert chunks to 384-dim vectors via HuggingFace
4. **Index** — Store in Pinecone `support-agent` index for fast retrieval

#### Phase 2 — Agentic RAG
A **Tools Agent (ReAct Framework)** decides *when* to query the vector database vs. when to respond conversationally. The agent retrieves the top-4 most relevant document chunks and synthesizes a grounded response.

#### Phase 3 — Action Layer with HITL
When a user requests an action (e.g., creating a support ticket), the agent:
1. Recognizes actionable intent and triggers the ticket sub-workflow
2. Sends a **secure approval email** to a supervisor with a webhook link
3. Upon supervisor click, the ticket is written to Google Sheets and the user receives confirmation

This **"Fire and Forget"** pattern keeps the UX responsive (< 2s response time) while maintaining human oversight.

#### Phase 4 — Automated Evaluation
A dedicated QA workflow (`05_Agent_Evaluation`) runs structured test cases and uses an **LLM-as-a-Judge** to score responses on accuracy and tone (1–5 scale).

---
## Knowledge Base Documents

The system is tested against three real-world enterprise-style documents:

| Type | Document | Purpose |
|---|---|---|
| Policy (Text-heavy) | [Springfield College Information Security Policy](https://springfield.edu/sites/default/files/information-security-policy-v202504015_final.pdf) | Data classification reasoning |
| Handbook (Large structured doc) | [Mississippi State Employee Handbook](https://www.eab.ms.gov/sites/eab/files/EAB-pdfs/FY%202025%20Employee%20Handbook%20with%20cover.pdf) | Needle-in-haystack retrieval |
| Technical Guide | [University of Surrey IT Welcome Booklet](https://my.surrey.ac.uk/sites/default/files/2024-01/it-welcome-booklet.pdf) | Step-by-step instructional answers |

---


## Test Results

| Test | Query Type | Result |
|---|---|---|
| Fact Retrieval | "What is 'Yes-Hybrid' telework status?" |  PASS |
| Policy Constraint | "Can I use my state email for personal social media?" | PASS |
| Complex Logic | "Minimum leave I must retain to donate?" | PASS |
| Table Math | "Leave hours for 5 years of service?" | Hallucination — LLM interpolated tabular data |
| End-to-End Ticket Flow | "My VPN keeps rejecting my password" | PASS |



---

## Safety & Guardrails

- **Scope Enforcement** — Agent refuses questions outside IT Support and Company Policy (no salary, legal, or personal queries)
- **"I Don't Know" Protocol** — If RAG confidence is below threshold, the agent escalates rather than hallucinating an answer
- **Prompt Injection Defense** — System prompt uses rigid delimiter strategy; action nodes have secondary logic checks independent of LLM output
- **HITL Gate** — No write actions (ticket creation, password resets) execute without explicit human approval
- **Recency Weighting** — Newer documents are prioritized when conflicting policy versions exist in the vector store

---

## Getting Started

### Prerequisites
- Docker (for self-hosted n8n)
- Pinecone account
- Groq API key
- HuggingFace account (with Inference token)
- Google Cloud project (for Sheets + Drive + Gmail OAuth)

### Running n8n

```bash
docker start n8n
```

n8n UI available at: `http://localhost:5678`

### Configuration
Set the following credentials inside n8n:
- **Groq** — LLM inference (model: `llama-3.3-70b-versatile`)
- **Pinecone** — Vector store (index: `support-agent`)
- **HuggingFace** — Fine-Grained Inference Token (embedding model)
- **Google OAuth** — Drive, Sheets, and Gmail access

---

## Success Metrics (KPIs)

| Metric | Description |
|---|---|
| **Deflection Rate** | % of issues fully resolved by AI without human intervention |
| **Retrieval Accuracy** | % of correct document chunks surfaced for known queries |
| **Hallucination Rate** | % of incorrect answers generated (Target: < 1%) |

---
