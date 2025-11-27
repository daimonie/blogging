# LLM Arena: Architecture & Prototyping Guide

This document provides a technical overview and the complete source code for **LLM Arena**, a rapid-prototyping tool for comparing Large Language Models (LLMs) in a "Single Shot" fashion.

## 1. System Architecture

The application follows a **Modular Monolith** pattern designed for containerization. It adheres to **SRP (Single Responsibility Principle)** and **DRY (Don't Repeat Yourself)** by leveraging smart abstraction layers.

### 1.1 The "Smart" AI Layer (LiteLLM)
Instead of writing three separate API clients (OpenAI, Google, Mistral), this application uses **LiteLLM**. 
*   **Abstraction:** It standardizes the request/response format (OpenAI format) across all providers.
*   **Concurrency:** We use Python's `asyncio.gather()` to dispatch requests to all three models simultaneously. This drastically reduces wait time compared to sequential processing.
*   **Document Handling:** The app acts as a **RAG-lite** system. It does not send the raw binary PDF to the API (which varies wildly by provider). Instead, it:
    1.  Streams the raw binary to the local disk (Historization).
    2.  Extracts text using `pypdf`.
    3.  Injects the text into the context window of the prompt.
    4.  Sends the text-enriched prompt to the APIs.

### 1.2 The Storage Layer (File-System & Polars)
We avoid heavy database servers (PostgreSQL/MySQL) for this prototype in favor of **Data-as-Files**:
*   **Raw Uploads:** Stored in `/opt/data/uploads`. (Source of Truth).
*   **Interaction Logs:** Stored in `/opt/data/interactions` as individual JSON files. This ensures high write throughput without lock contention.
*   **Ratings:** Stored in `/opt/data/ratings.jsonl`.
*   **Analytics:** We use **Polars** to lazy-load the JSONL file. Polars is significantly faster than Pandas for this type of file-based querying and allows us to generate real-time scatter plots instantly.

### 1.3 The Presentation Layer (HTMX + Tailwind)
We bypass the complexity of a React/Vue build pipeline.
*   **Server-Side Rendering (SSR):** Python generates HTML fragments using Jinja2.
*   **HTMX:** Handles the interactivity. When you click "Run", HTMX sends the form data and swaps the response into the DOM without a full page reload.
*   **Visualization:** Plotly generates the chart as a static HTML `<div>` which is injected dynamically.

---

## 2. Implementation Guide

To run this prototype, create a directory (e.g., `llm_arena`) and add the four files below.

### File 1: `requirements.txt`
*Dependencies include `uv` compatible versions.*

```text
fastapi
uvicorn
python-multipart
jinja2
litellm
polars
plotly
pypdf
