# 🔬 Multi-Agent AI Deep Researcher
### Group 15 — AI Agent Project

> An AI-powered research assistant that performs **multi-hop**, **multi-source** deep investigations — decomposing any research topic into targeted sub-queries, retrieving live web and document sources, validating and analysing findings, detecting contradictions, generating strategic insights, and compiling a structured Markdown report — all orchestrated via **LangGraph**.  
> Supports **text input**, **PDF upload**, and **voice input** via microphone.

---

## 📌 Table of Contents

- [Overview](#-overview)
- [Architecture](#-architecture)
- [Agent Pipeline](#-agent-pipeline)
- [Features](#-features)
- [Tech Stack](#-tech-stack)
- [Project Structure](#-project-structure)
- [Running on Google Colab](#-running-on-google-colab)
- [API Keys Required](#-api-keys-required)
- [Installation](#-installation)
- [Usage](#-usage)
- [LangGraph State Schema](#-langgraph-state-schema)
- [Example Report Output](#-example-report-output)
- [Design Patterns Used](#-design-patterns-used)
- [Team](#-team)

---

## 🧭 Overview

The **Multi-Agent AI Deep Researcher** spins up **6 specialised AI agents** that collaborate in a stateful LangGraph pipeline to conduct thorough research on any topic. It showcases:

- **Multi-hop querying** — breaks the topic into 5 targeted sub-queries, each feeding the next retrieval step
- **Retrieval-Augmented Generation (RAG)** — grounds LLM responses in real web and PDF document sources
- **Reflection design pattern** — the Critical Analysis agent self-critiques and flags contradictions between sources
- **Agent collaboration** — each agent's output becomes the next agent's input via shared `ResearchState`
- **Voice-to-research** — speak your research topic directly using the built-in microphone transcriber
- **Long-context synthesis** — condenses thousands of characters of source material into concise, structured reports

---

## 🏗️ Architecture

```
         User Input
    ┌────────────────────────────┐
    │  Text  │  PDF Upload  │ 🎤 │
    └────────────────────────────┘
                   │
                   ▼
    ┌──────────────────────────┐
    │   1. Query Planner       │  Decomposes topic → 5 sub-queries
    └────────────┬─────────────┘
                 │
                 ▼
    ┌──────────────────────────┐
    │   2. Contextual          │  Tavily web search (5 results/query)
    │      Retriever           │  + ChromaDB PDF semantic search
    └────────────┬─────────────┘
                 │
                 ▼
    ┌──────────────────────────┐
    │   3. Source Validator    │  Relevance (0–10) + Credibility (0–10)
    │                          │  Keeps sources: relevance ≥ 5 & cred ≥ 4
    └────────────┬─────────────┘
                 │
                 ▼
    ┌──────────────────────────┐
    │   4. Critical Analysis   │  Summarises findings (4–6 paragraphs)
    │                          │  + Detects contradictions & disputes
    └────────────┬─────────────┘
                 │
                 ▼
    ┌──────────────────────────┐
    │   5. Insight Generator   │  3 Key trends · 2 Hypotheses
    │                          │  3 Recommendations · Future directions
    └────────────┬─────────────┘
                 │
                 ▼
    ┌──────────────────────────┐
    │   6. Report Builder      │  Structured Markdown report
    │                          │  + References + Agent Activity Log
    └────────────┬─────────────┘
                 │
                 ▼
         Gradio Web UI
    (Rendered report + .md download)
```

---

## 🤖 Agent Pipeline

| # | Agent | Role | Technology |
|---|---|---|---|
| 1 | **Query Planner** | Decomposes the research topic into exactly 5 targeted sub-queries for multi-hop coverage; parses JSON output with newline fallback | `ChatOpenAI` via OpenRouter |
| 2 | **Contextual Retriever** | Runs each sub-query through Tavily (5 results/query, 1500-char cap per source); loads & chunks PDFs then runs top-3 sub-queries against ChromaDB vector store | Tavily API · PyPDFLoader · `RecursiveCharacterTextSplitter` (chunk=800, overlap=100) · ChromaDB |
| 3 | **Source Validator** | Sends up to 20 sources to LLM in one call for relevance + credibility scoring; filters by relevance ≥ 5 AND credibility ≥ 4 | GPT via OpenRouter (JSON structured output) |
| 4 | **Critical Analysis** | Builds a ~6000-char condensed context from validated sources + PDF excerpts; generates 4–6 paragraph summary then runs a separate contradiction-detection pass | GPT via OpenRouter *(Reflection pattern)* |
| 5 | **Insight Generator** | Uses reasoning chains on the summary + contradictions to produce 3 trends, 2 hypotheses, 3 recommendations, and future directions | GPT via OpenRouter |
| 6 | **Report Builder** | Compiles all agent outputs into a 6-section Markdown report with timestamped header, numbered references (URLs), and full agent activity log | GPT via OpenRouter |

---

## ✨ Features

- 🔍 **Multi-hop web research** via Tavily API — live, current results
- 📄 **PDF semantic search** — upload research papers and get relevant excerpts via ChromaDB
- 🎤 **Voice input** — speak your research topic; transcribed via Google Speech Recognition (`SpeechRecognition` + `pydub`)
- 🔗 **Source validation** — every source scored and filtered before analysis
- ⚡ **Contradiction detection** — conflicting claims across sources are flagged automatically
- 💡 **Insight synthesis** — trends, hypotheses, and actionable recommendations generated
- 📥 **Downloadable report** — save the final report as a `.md` file
- 🖥️ **Gradio web UI** — clean interface with example topics and Soft theme
- 📟 **Live console logging** — all 6 agents print activity to the Colab console in real time
- 🧪 **Headless mode** — run the full pipeline without the UI (Cell 8)

---

## 🛠️ Tech Stack

| Layer | Technology | Notes |
|---|---|---|
| **LLM** | `openai/gpt-4o-mini` via [OpenRouter](https://openrouter.ai) | Swap to `gpt-4o` for higher quality |
| **Embeddings** | `openai/text-embedding-ada-002` via OpenRouter | Used for PDF chunk search |
| **Orchestration** | [LangGraph](https://github.com/langchain-ai/langgraph) | `StateGraph` with typed `ResearchState` |
| **LLM Framework** | [LangChain](https://langchain.com/) | `langchain`, `langchain-openai`, `langchain-community` |
| **Web Search** | [Tavily API](https://tavily.com) | `TavilySearchResults`, max 5 results/query |
| **PDF Loading** | `PyPDFLoader` | From `langchain-community` |
| **Text Splitting** | `RecursiveCharacterTextSplitter` | chunk=800, overlap=100 |
| **Vector Store** | [ChromaDB](https://www.trychroma.com/) | In-memory, `similarity_search` k=3 |
| **UI** | [Gradio](https://gradio.app/) | `gr.Blocks`, Soft theme, public share link |
| **Voice Input** | `SpeechRecognition` + `pydub` | Google Speech Recognition backend |
| **Runtime** | Google Colab | Python 3.10+ |

---

## 📁 Project Structure

```
Multi-Agent-AI-Deep-Researcher/
│
├── copy_of_multi_agent_ai_deep_researcher.py   # Full source (exported from Colab)
├── Multi_Agent_AI_Deep_Researcher.ipynb        # Original Colab notebook
├── README.md                                   # This file
│
└── (generated at runtime in Colab /tmp/)
    └── tmpXXXXXX.md                            # Downloaded research report
```

> The project is a **single self-contained file** — all agents, orchestration, voice input, and UI live together. No separate modules or config files needed.

---

## 🚀 Running on Google Colab

### Quick Start

1. Go to **[colab.research.google.com](https://colab.research.google.com)**
2. Click **File → Upload notebook** → select `Multi_Agent_AI_Deep_Researcher.ipynb`
3. Run cells **in order**:

| Cell | What It Does |
|---|---|
| **Cell 1** | Installs all pip packages (~2 min on first run) |
| **Cell 2** | Prompts for API keys securely via `getpass` (hidden input) |
| **Cell 3** | Imports libraries; initialises LLM, Embeddings, and Tavily tool |
| **Cell 4** | Defines the `ResearchState` TypedDict shared across all agents |
| **Cell 5** | Defines all 6 agent functions |
| **Cell 6** | Builds and compiles the LangGraph pipeline |
| **Cell 7** | Launches the Gradio UI with a public shareable link |
| **Cell 8** | *(Optional)* Headless pipeline run for scripted testing |

> 💡 **Tip:** Use **Runtime → Run all** after entering your API keys in Cell 2.

---

## 🔑 API Keys Required

| Service | Where to Get It | Free Tier |
|---|---|---|
| **OpenRouter** | [openrouter.ai](https://openrouter.ai) | ✅ Free credits on signup |
| **Tavily** | [tavily.com](https://tavily.com) | ✅ 1,000 searches/month free |

Keys are entered securely via `getpass` in Cell 2 — they are **never stored, printed, or logged**.

```python
OPENROUTER_API_KEY = getpass("OpenRouter API Key: ")   # hidden input
TAVILY_API_KEY     = getpass("Tavily API Key:     ")   # hidden input
```

---

## ⚙️ Installation

All dependencies are installed automatically in Cell 1 of the notebook:

```bash
# Core agent pipeline
pip install langchain langchain-openai langchain-community langgraph
pip install tavily-python chromadb pypdf langchain-text-splitters
pip install gradio openai tiktoken

# Voice input (Speech-to-Text)
pip install SpeechRecognition pydub
```

No `requirements.txt`, virtual environments, or local setup needed — Colab handles everything.

---

## 💻 Usage

### Via Gradio UI (Recommended)

After Cell 7 launches, open the public Gradio link and:

1. **Type** a research topic in the text box, **OR**
2. Click the **🎤 microphone** → speak your topic → click **🗣️ Transcribe Voice** to auto-fill the text box
3. **Upload PDFs** (optional) — adds semantic search on top of web results
4. Click **🚀 Run Deep Research**
5. Watch all 6 agents print live activity in the **Colab console** (~2–4 minutes)
6. Read the rendered report in the UI, or click **⬇️ Download Report (.md)**

### Example Topics (built into UI)

```
"Impact of large language models on software engineering productivity"
"Climate change adaptation strategies for coastal cities in 2025"
"Quantum computing applications in drug discovery"
"The future of autonomous vehicles and urban mobility"
```

### Headless / Scripted Mode (Cell 8)

```python
TEST_TOPIC = "Your research topic here"

initial_state = {
    "topic":             TEST_TOPIC,
    "pdf_paths":         [],    # e.g. ["/content/paper.pdf"]
    "sub_queries":       [],
    "raw_sources":       [],
    "pdf_excerpts":      [],
    "validated_sources": [],
    "contradictions":    [],
    "summary":           "",
    "insights":          "",
    "final_report":      "",
    "log":               [],
}

result = app.invoke(initial_state)
print(result["final_report"])
```

---

## 🗂️ LangGraph State Schema

The `ResearchState` TypedDict is the shared memory passed through every agent node:

```python
class ResearchState(TypedDict):
    # ── Inputs ──────────────────────────────────────────────
    topic:              str           # the research topic string
    pdf_paths:          List[str]     # file paths to uploaded PDFs

    # ── Agent Outputs (populated sequentially) ──────────────
    sub_queries:        List[str]     # Query Planner  → 5 sub-queries
    raw_sources:        List[dict]    # Retriever      → web results (title, url, content, score)
    pdf_excerpts:       List[str]     # Retriever      → PDF semantic search chunks
    validated_sources:  List[dict]    # Validator      → filtered source list
    contradictions:     List[str]     # Critical Analysis → conflict bullet points
    summary:            str           # Critical Analysis → 4–6 paragraph summary
    insights:           str           # Insight Generator → trends & hypotheses (Markdown)
    final_report:       str           # Report Builder    → full structured Markdown report

    # ── Accumulator (appended by every agent) ───────────────
    log: Annotated[List[str], operator.add]
```

---

## 📄 Example Report Output Structure

Every pipeline run produces a timestamped report with the following structure:

```markdown
# 🔬 Deep Research Report
**Topic:** ...  |  **Generated:** YYYY-MM-DD HH:MM  |  **Sources analysed:** N

---

## 1. Executive Summary
## 2. Research Scope & Methodology
## 3. Key Findings
## 4. Contradictions & Debates
## 5. Strategic Insights & Recommendations
## 6. Conclusion

---

## 📚 References
1. [Source Title](https://source-url.com)
...

---

## 🤖 Agent Activity Log
- [Query Planner] Generated 5 sub-queries for: ...
- [Retriever] 23 web sources + 6 PDF excerpts retrieved.
- [Validator] Kept 17/23 sources.
- [Critical Analysis] Summary done. 4 contradictions noted.
- [Insight Generator] Trends, hypotheses, and recommendations synthesised.
- [Report Builder] Final report compiled. 17 sources cited.
```

---

## 🧩 Design Patterns Used

| Pattern | Where Applied in Code |
|---|---|
| **Multi-hop querying** | `query_planner_agent` — 5 chained sub-queries each fed independently to the retriever |
| **RAG (Retrieval-Augmented Generation)** | `retriever_agent` — all LLM outputs grounded in real web + PDF evidence |
| **Reflection** | `critical_analysis_agent` — two separate LLM passes: *summarise* → then *critique for contradictions* |
| **Stateful agent collaboration** | `ResearchState` TypedDict flows through every LangGraph node; each agent reads all prior outputs |
| **Tool use** | `TavilySearchResults` tool invoked programmatically inside `retriever_agent` |
| **Long-context synthesis** | Sources capped at 1500 chars each; total context capped at ~6000 chars before LLM analysis |
| **Structured output parsing** | `try/except` JSON parsing with newline-split fallback in `query_planner_agent` and `source_validator_agent` |
| **Rate limiting** | `time.sleep(0.5)` between Tavily calls to respect API limits |
| **Voice-to-text input** | `transcribe_audio()` converts mic recording → WAV → Google Speech Recognition → fills topic box |

---

## 👥 Team

**Group 15**

| Member | Role / Contribution |
|---|---|
| Vijay Yadav | Architecture design · API integration planning · Query Planning agent |
| Iain Clark | Architecture design · API integration planning · Query Planning agent |
| Pushkar Raj | Architecture design · API integration planning · Query Planning agent |
| Sudarshan | Architecture design · API integration planning · Query Planning agent |
| Ajay | Architecture design · API integration planning · Query Planning agent |

---

## 📜 License

This project was created for academic and educational purposes as part of a Group 15 AI Agent course project.

---

*Built with ❤️ using LangGraph · LangChain · Tavily · ChromaDB · Gradio · OpenRouter · SpeechRecognition*
