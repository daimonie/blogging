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
```

### File 2: `Dockerfile`
*Uses the official Astral `uv` image for ultra-fast builds.*

```dockerfile
# We use the official image from Astral which includes python 3.11 and uv
FROM ghcr.io/astral-sh/uv:python3.11-bookworm-slim

WORKDIR /app

# Copy dependencies
COPY requirements.txt .

# Install dependencies into the system environment (no venv needed in container)
RUN uv pip install --system --no-cache -r requirements.txt

# Copy application logic
COPY main.py .

# Define the environment variable for data persistence
ENV DATA_DIR="/opt/data"

# Expose the web port
EXPOSE 8000

# Start the application
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### File 3: `docker-compose.yml`
*Orchestration and Volume Management.*

```yaml
services:
  llm-arena:
    build: .
    container_name: llm_arena
    ports:
      - "8000:8000"
    volumes:
      # Maps your local ./data folder to the container's /opt/data
      # This guarantees your logs and PDFs survive container restarts
      - ./data:/opt/data
    environment:
      # API Keys - Replace these or use an .env file
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - GEMINI_API_KEY=${GEMINI_API_KEY}
      - MISTRAL_API_KEY=${MISTRAL_API_KEY}
    restart: unless-stopped
```

### File 4: `main.py`
*The Application Logic.*

```python
import os
import json
import asyncio
import logging
import shutil
from datetime import datetime
from pathlib import Path
from typing import List, Dict

# Web Framework
from fastapi import FastAPI, UploadFile, File, Form, Request
from fastapi.responses import HTMLResponse
from fastapi.templating import Jinja2Templates

# Data & AI Libraries
import polars as pl
import plotly.express as px
from pypdf import PdfReader
from litellm import completion

# ==============================================================================
# 1. CONFIGURATION LAYER
# ==============================================================================

app = FastAPI(title="LLM Arena")
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# Persistence Paths
DATA_DIR = Path(os.getenv("DATA_DIR", "./data"))
INTERACTIONS_DIR = DATA_DIR / "interactions"
UPLOADS_DIR = DATA_DIR / "uploads"
RATINGS_FILE = DATA_DIR / "ratings.jsonl"

# Initialize Directories
for p in [INTERACTIONS_DIR, UPLOADS_DIR]:
    p.mkdir(parents=True, exist_ok=True)
if not RATINGS_FILE.exists():
    RATINGS_FILE.touch()

# AI Models (LiteLLM Identifiers)
MODELS = {
    "OpenAI": "gpt-4o",
    "Gemini": "gemini/gemini-1.5-pro",
    "Mistral": "mistral/mistral-large-latest"
}

# ==============================================================================
# 2. STORAGE LAYER (Historization)
# ==============================================================================

def save_uploaded_file(file: UploadFile) -> tuple[str, str]:
    """
    1. Saves raw binary stream to /uploads (Historization).
    2. Extracts text for the AI prompt.
    """
    if not file or not file.filename:
        return None, ""

    # Unique Filename Strategy
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    clean_name = "".join([c for c in file.filename if c.isalnum() or c in ('.','_')]).strip()
    saved_filename = f"{timestamp}_{clean_name}"
    saved_path = UPLOADS_DIR / saved_filename

    # Save Binary
    try:
        with open(saved_path, "wb") as buffer:
            shutil.copyfileobj(file.file, buffer)
    finally:
        file.file.seek(0) # Reset cursor

    # Extract Text
    text_content = ""
    try:
        if saved_filename.lower().endswith(".pdf"):
            reader = PdfReader(saved_path)
            text_content = "\n".join([page.extract_text() for page in reader.pages if page.extract_text()])
        else:
            with open(saved_path, "r", encoding="utf-8", errors="ignore") as f:
                text_content = f.read()
    except Exception as e:
        logger.error(f"Extraction failed: {e}")
        text_content = f"[System: Error extracting text from file]"

    return saved_filename, text_content

def save_interaction(prompt: str, uploaded_filename: str, responses: List[Dict]) -> str:
    """Logs the Request/Response cycle to a JSON file."""
    timestamp = datetime.now()
    slug = "".join([c for c in prompt[:20] if c.isalnum()]).strip() or "prompt"
    filename = f"{timestamp.strftime('%Y%m%d_%H%M%S')}_{slug}.json"
    
    record = {
        "timestamp": timestamp.isoformat(),
        "prompt": prompt,
        "uploaded_file": uploaded_filename, 
        "responses": responses
    }
    
    with open(INTERACTIONS_DIR / filename, "w", encoding="utf-8") as f:
        json.dump(record, f, indent=2)
    
    return filename

def append_rating(model: str, rating: int):
    """Appends a rating event to the JSONL ledger."""
    row = {"timestamp": datetime.now().isoformat(), "model": model, "rating": rating}
    with open(RATINGS_FILE, "a", encoding="utf-8") as f:
        f.write(json.dumps(row) + "\n")

# ==============================================================================
# 3. ANALYTICS LAYER (Polars + Plotly)
# ==============================================================================

def generate_chart_html() -> str:
    """Reads JSONL via Polars, returns Plotly HTML div."""
    if RATINGS_FILE.stat().st_size == 0:
        return "<div class='text-gray-400 text-center p-4'>No ratings yet.</div>"
    
    try:
        df = pl.read_ndjson(RATINGS_FILE).with_columns(pl.col("timestamp").str.to_datetime())
        
        fig = px.scatter(
            df.to_pandas(), 
            x="timestamp", y="rating", color="model",
            title="Model Performance Over Time",
            template="plotly_white", height=300
        )
        fig.update_traces(marker=dict(size=10, opacity=0.8))
        fig.update_layout(margin=dict(l=20,r=20,t=40,b=20), yaxis=dict(range=[0.5, 5.5], dtick=1))
        
        return fig.to_html(full_html=False, include_plotlyjs='cdn')
    except Exception as e:
        return f"<div class='text-red-500'>Error loading chart: {e}</div>"

# ==============================================================================
# 4. SERVICE LAYER (AI Communication)
# ==============================================================================

async def query_model(alias: str, model_name: str, full_prompt: str):
    start = datetime.now()
    try:
        # LiteLLM handles the specific API logic (OpenAI vs Vertex vs Mistral)
        resp = await asyncio.to_thread(
            completion, model=model_name, messages=[{"role": "user", "content": full_prompt}]
        )
        text = resp.choices[0].message.content
    except Exception as e:
        text = f"API Error: {str(e)}"
    
    return {
        "model": alias, "text": text, "duration": (datetime.now() - start).total_seconds()
    }

# ==============================================================================
# 5. PRESENTATION LAYER (Templates & Routes)
# ==============================================================================

HTML_BASE = """
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>LLM Arena</title>
    <script src="https://unpkg.com/htmx.org@1.9.10"></script>
    <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-gray-50 text-gray-900 p-6">
    <div class="max-w-7xl mx-auto grid grid-cols-1 lg:grid-cols-4 gap-6">
        <div class="lg:col-span-1 space-y-6">
            <h1 class="text-3xl font-extrabold text-indigo-900">LLM <span class="text-indigo-600">Arena</span></h1>
            
            <form hx-post="/run" hx-encoding="multipart/form-data" hx-target="#feed" hx-swap="afterbegin" class="bg-white p-4 rounded-xl shadow border">
                <input type="file" name="file" class="block w-full text-sm mb-4 file:mr-4 file:py-2 file:px-4 file:rounded-full file:bg-indigo-50 file:text-indigo-700 file:border-0"/>
                <textarea name="prompt" rows="6" class="w-full p-2 border rounded mb-4" placeholder="Enter prompt..." required></textarea>
                <button class="w-full bg-indigo-600 text-white py-2 rounded font-bold hover:bg-indigo-700 transition">
                    Run Comparison <img class="htmx-indicator inline w-4 invert" src="https://cdnjs.cloudflare.com/ajax/libs/galleriffic/2.0.1/css/loader.gif">
                </button>
            </form>

            <div class="bg-white p-2 rounded-xl shadow border" hx-get="/chart" hx-trigger="load, rate from:body"></div>
        </div>
        <div id="feed" class="lg:col-span-3 space-y-8 mt-4"></div>
    </div>
    <script>document.body.addEventListener('rate', console.log);</script>
</body>
</html>
"""

HTML_RESULT = """
<div class="bg-white rounded-xl shadow-lg border border-gray-100 overflow-hidden mb-8 fade-in">
    <div class="bg-gray-50 p-4 border-b">
        <div class="font-bold">{{ prompt }}</div>
        {% if filename %}<div class="text-xs text-indigo-600 mt-1">📎 {{ filename }}</div>{% endif %}
    </div>
    <div class="grid grid-cols-1 md:grid-cols-3 divide-y md:divide-y-0 md:divide-x">
        {% for r in responses %}
        <div class="p-4 flex flex-col">
            <div class="flex justify-between font-bold text-sm mb-2">
                <span>{{ r.model }}</span> <span class="text-gray-400">{{ r.duration|round(2) }}s</span>
            </div>
            <div class="prose prose-sm flex-grow overflow-y-auto max-h-[400px] mb-4">{{ r.text }}</div>
            <div class="pt-2 border-t flex justify-center gap-1" id="rate-{{ loop.index }}-{{ log }}">
                {% for i in range(1, 6) %}
                <button hx-post="/rate?model={{ r.model }}&val={{ i }}" hx-swap="outerHTML" hx-target="#rate-{{ loop.index }}-{{ log }}" 
                        class="text-xl text-gray-300 hover:text-yellow-400 transition">★</button>
                {% endfor %}
            </div>
        </div>
        {% endfor %}
    </div>
</div>
"""

templates = Jinja2Templates(directory=".")
templates.get_template = lambda name: type('T',(),{'render':lambda **k: from_str(name,**k)})()

def from_str(name, **kwargs):
    from jinja2 import Environment, BaseLoader
    tpl = HTML_BASE if name == "base" else (HTML_RESULT if name == "res" else "<span class='text-green-500 font-bold'>✓</span>")
    return Environment(loader=BaseLoader()).from_string(tpl).render(**kwargs)

# ==============================================================================
# 6. ROUTING LAYER
# ==============================================================================

@app.get("/", response_class=HTMLResponse)
async def index(req: Request): return templates.TemplateResponse("base", {"request": req})

@app.get("/chart", response_class=HTMLResponse)
async def chart(): return generate_chart_html()

@app.post("/run", response_class=HTMLResponse)
async def run(req: Request, prompt: str = Form(...), file: UploadFile = File(None)):
    # 1. Handle File
    fname, content = save_uploaded_file(file)
    final_prompt = f"Context:\n{content}\n\nUser: {prompt}" if content else prompt

    # 2. Parallel Execution
    tasks = [query_model(k, v, final_prompt) for k, v in MODELS.items()]
    results = await asyncio.gather(*tasks)

    # 3. Log
    log = save_interaction(prompt, fname, results)

    return templates.TemplateResponse("res", {"request": req, "prompt": prompt, "filename": fname, "responses": results, "log": log})

@app.post("/rate")
async def rate(model: str, val: int):
    append_rating(model, val)
    resp = HTMLResponse(from_str("rated"))
    resp.headers["HX-Trigger"] = "rate"
    return resp

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```
