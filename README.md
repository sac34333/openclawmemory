# 🦞 OpenClaw + QMD: The Complete Local Memory Architecture

> **Personal AI assistant with a semantic brain — running entirely on your AWS EC2 instance.**

---

## 📖 Table of Contents

- [What is QMD?](#-what-is-qmd)
- [System Architecture](#-system-architecture)
- [Installation & Setup](#-installation--setup)
- [Operational Configuration](#-operational-configuration)
- [Workflows & Automation](#-workflows--automation)
- [Verification](#-verification-cost--privacy)
- [Benefits Summary](#-benefits-summary)

---

## What is QMD?

**QMD** is not just a search tool — it is a **Local Hybrid Intelligence engine**. It combines three distinct search technologies running entirely **offline** on your server.

### The Hybrid Search Triad

```
┌─────────────────────────────────────────────────┐
│  BM25 (keyword) + Vector (semantic) + LLM Rerank │
│  All running offline via node-llama-cpp          │
└─────────────────────────────────────────────────┘
```

| Technology | Description |
|:-----------|:------------|
| **BM25 (Keywords)** | Fast, exact matching (like `Ctrl+F` on steroids). Finds "budget" when you type "budget". |
| **Vector Search (Semantic)** | Understands meaning. Finds "tax plans" when you search "financial changes". |
| **LLM Re-ranking** | A local AI scores the results to ensure the top answer is actually relevant. |

### How Semantic Similarity Works (The "Magic")

The system converts your text into numbers (vectors) to "understand" them.

```
Text ───► Embedding Model ───► Vector (list of numbers)
                                    │
                                    ▼
                              Compare vectors
                              (cosine similarity)
                                    │
                                    ▼
                        Find "closest" meaning matches
```

> **💡 Analogy:**
> - **Keyword Search:** Looks for word matches ("apple" finds "apple").
> - **Semantic Search:** Looks for concept matches ("apple" finds "fruit", "pie", "cider").

---

## System Architecture

### The Memory Layer Diagram

This diagram illustrates how your daily logs and permanent files feed into the QMD brain before reaching the chat.

```
┌─────────────────────────────────────────────────────────┐
│                    QMD Layer                            │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐      │
│  │  Daily Logs │  │ MEMORY.md   │  │  Preferences│      │
│  │ (memory/*.md)│ │ (curated)   │  │  (JSON/User)│      │
│  └─────────────┘  └─────────────┘  └─────────────┘      │
│                        │                                │
│                        ▼                                │
│              QMD Hybrid Search                          │
│         (keyword + semantic + rerank)                   │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
                   OpenClaw Chat
```

### The Local Models (The "Brains")

These models live in `~/.cache/qmd/models`. They are the reason you don't need OpenAI API keys for memory.

| Model | Purpose | Size |
|:------|:--------|:-----|
| `embeddinggemma-300M` | Converts text → vectors | ~300 MB |
| `qwen3-reranker-0.6b` | Re-ranks results for relevance | ~640 MB |
| `qmd-query-expansion-1.7B` | Expands vague queries | ~1.1 GB |

> **Total Disk Impact:** ~2.1 GB

---

## Installation & Setup

### Prerequisites

```bash
# 1. Install Bun (JavaScript Runtime)
curl -fsSL https://bun.sh/install | bash
source ~/.bashrc

# 2. Install Document Tools (Pandoc)
sudo apt-get install -y pandoc poppler-utils
```

### Step 1: Install QMD

```bash
# Install directly from source
bun install -g github:tobi/qmd
```

### Step 2: Connect the Brain

We tell QMD to watch the OpenClaw workspace.

```bash
# Add the collection
qmd collection add ~/.openclaw/workspace --name brain --mask "**/*.md"
```

### Step 3: The "Models Download" (Embed)

This is the critical step where the models are downloaded.

```bash
# 1. Update the index
qmd update

# 2. Download models & Generate vectors
qmd embed --collection brain
```

> ** Success Indicator:** If you see a progress bar downloading ~300MB+, it is working.

> [!NOTE]
> **Progressive Model Downloads:**  
> In the first instance, it's highly possible that only the **embedding model (~300MB)** gets installed. The **reranker** and **query-expansion** models download automatically when you run a complex query for the first time.

### Step 4: Trigger Full Model Download (Optional)

To ensure all models are downloaded, run a complex query. A good way to do this is:

1. **Implement an OpenClaw skill** (e.g., Google Workspace integration)
2. **Ask the assistant to store all workflow steps into a skill**
3. **Query through QMD** to trigger the full model download

> [!TIP]
> **You don't need to use the terminal!**  
> OpenClaw integrates with your messaging channels, so you can run these queries directly from **Telegram**, **WhatsApp**, **Slack**, **Discord**, or any other connected channel.

**Option A: Via Terminal**
```bash
# Example: Run a complex semantic query
qmd query "How do I authenticate with Google Workspace?" --collection brain
```

**Option B: Via Your Messaging Channel**

Simply message your OpenClaw assistant on Telegram/WhatsApp/Slack:

```
"How do I authenticate with Google Workspace?"
```

OpenClaw will automatically use QMD to search your knowledge base and respond with relevant information. This also triggers the model downloads behind the scenes!

This forces QMD to use the reranker and query-expansion models, triggering their automatic download.

---

## Operational Configuration

### Directory Structure

```
~/.openclaw/workspace/
├── AGENTS.md           # Operational Guidelines (provided by OpenClaw but by default is empty)
├── TOOLS.md            # Environment Context (provided by OpenClaw)
├── MEMORY.md           # Long-Term Memory (provided by OpenClaw and has to be Curated)
├── USER.md             # User Preferences
├── memory/             # DAILY LOGS (Raw chat history)
│   └── 2026-02-01.md
├── knowledge/          # THE BRAIN (Indexed text - We create this)
│   └── artifacts/      # Converted text files
└── artifacts/          # THE DROPBOX (Raw binaries - We create this)
    └── invoice.pdf     # (Ignored by QMD)
```

### Agent Instructions (`AGENTS.md`)

Create this file to guide the AI on how to use QMD:

```markdown
# AGENTS.md - Operational Guidelines

##  QMD (Semantic Brain)
**Status:** Installed & Active
**Role:** Your primary method for retrieving information.

### **How to Search**
- **Command:** `qmd query "natural language question" --collection brain`
- **Examples:**
  - *"What is the server IP address?"*
  - *"How did we fix the disk space issue?"*

### **How to Learn (Indexing)**
- **Command:** `qmd update && qmd embed --collection brain`

##  Memory Standards
| Content Type | File Location |
| :--- | :--- |
| **User Preferences** | `USER.md` |
| **Daily Logs** | `memory/YYYY-MM-DD.md` |
| **Long-Term Facts** | `MEMORY.md` |
| **Technical Knowledge** | `knowledge/TOPIC.md` |

##  Document Ingestion (Artifacts)
**Workflow for Uploads:**
1. **Save Original:** `artifacts/filename.ext`
2. **Convert to Text:**
   - **PDF:** `pdftotext -layout artifacts/file.pdf knowledge/artifacts/file.md`
   - **Word:** `pandoc artifacts/file.docx -o knowledge/artifacts/file.md`
3. **Index:** Run `qmd update && qmd embed --collection brain`
```

---

## Workflows & Automation

### A. Ingesting Documents (PDF/Word)

| Step | Action |
|:-----|:-------|
| 1. **Upload** | User sends file |
| 2. **Convert** | Bot runs `pdftotext` to create a Markdown copy in `knowledge/` |
| 3. **Index** | Bot runs `qmd update` |

### B. Automated Maintenance (Cron Job)

To ensure memory is updated without you asking, we use a Cron job.

**Command:** `crontab -e`

**Schedule:** Every 5 minutes.

```bash
*/5 * * * * cd /home/ubuntu/.openclaw/workspace && /home/ubuntu/.bun/bin/qmd update && /home/ubuntu/.bun/bin/qmd embed --collection brain >> /tmp/qmd.log 2>&1
```

### C. Memory Maintenance Strategy

| Type | Description |
|:-----|:------------|
| **Real-time** | Bot logs decisions to `memory/YYYY-MM-DD.md` |
| **Periodic** | Bot reviews daily logs and updates `MEMORY.md` with permanent facts |

---

## Verification: Cost & Privacy

> **Claim:** No API tokens are used for memory.

**Proof:** The models are local.

Run this command to see the physical files on your disk:

```bash
ls -lh ~/.cache/qmd/models
```

**Expected Output:**

```
hf_ggml-org_embeddinggemma-300M-Q8_0.gguf      (~300MB)
hf_ggml-org_qwen3-reranker-0.6b-q8_0.gguf      (~640MB)
hf_tobil_qmd-query-expansion-1.7B-q4_k_m.gguf  (~1.1GB)
```

---

## Benefits Summary

| Feature | Benefit |
|:--------|:--------|
| **Local & Private** | No data leaves your server |
| **Semantic Search** | "Find memory about X" works even if keywords differ |
| **Shell Integration** | Bot calls `qmd query` natively via terminal |
| **Markdown Native** | Your memory files stay as clean, readable text |
| **Zero Cost** | No monthly API bill for embeddings or storage |

---

## Resources

- **OpenClaw:** [GitHub](https://github.com/openclaw/openclaw) · [Docs](https://docs.openclaw.ai) · [Getting Started](https://docs.openclaw.ai/start/getting-started)
- **QMD:** [GitHub](https://github.com/tobi/qmd)
