# Extreme Text Prediction: The Actual Energy Cost of AI

We need to stop anthropomorphizing AI. It isn’t “thinking”; it is **extreme text prediction**.

When you ask an AI to summarize a report, it isn’t reflecting on the content—it’s calculating the probability of the next word. As engineers, we know that calculation requires energy.

In 2025, the debate isn’t just about capability—it’s about physics. Whether you run a sovereign local server for privacy or scale cloud agents as a CTO, you are generating heat. To understand the trade-offs, we need to measure the cost of these calculations.

**Units of Measurement**
To make the physics readable, we will use three units:
1.  **Joules (J):** The raw physics.
2.  **Phone Battery:** A typical 2025 flagship (≈ 15 Wh). **1% ≈ 540 J**.
3.  **Google Search Equivalent (GSE):** A “complex Google search”—the industry benchmark—consumes ≈ 1,080 J (0.3 Wh). **1 GSE = 1,080 J**.

---

### 1. Training vs. Inference: Then and Now

To understand the energy bill, we must distinguish between building the engine and driving the car.

*   **Training (The Sunk Cost):** Building frontier models like GPT-4 or Llama-3 demands immense compute, measured in Megawatt-hours. For example, training GPT-3 consumed ~1,287 MWh ([Knowledge at Wharton][1]). This is a massive, one-time ecological debt.
*   **Inference (The Recurring Cost):** Once deployed, real-world usage—“prompting”—becomes the dominant driver. A 2025 benchmarking study confirms that inference now drives the majority of total AI energy use, water consumption, and CO₂ emissions ([arXiv][3]).

**The Reality:** Even though training is heavy, **Inference—used constantly at a global scale—is the real environmental burden.**

---

### 2. Cloud Benefits: The "Public Bus" Effect

Cloud providers (Google, OpenAI, Anthropic) utilize the "Public Bus" model. They run massive clusters of hardware (like Nvidia Blackwell B200s) optimized for **Continuous Batching**. They never run your request alone; they run it alongside 500 others, amortizing the energy cost.

**The 2025 Data:**
According to a 2025 Google Cloud report, the “median text prompt” in their infrastructure consumes about **0.24 Wh (864 Joules)** ([Google Cloud][4]).
*   **Energy:** **864 Joules**.
*   **Phone Battery:** **1.6%**.
*   **Google Search Equiv:** **0.8 GSE**.

**Verdict:** Rapid gains in hardware efficiency (up to 33x year-over-year) have made individual cloud prompts surprisingly green—roughly equivalent to running a microwave for one second.

---

### 3. Local Inefficiency: The "Single-Tenant" Tax

For those demanding data sovereignty (privacy), "Local AI" is the only option. But privacy comes with a power bill.

If you run a model like Llama-3-70B on your local rig (e.g., a watercooled **RTX 5090**), you are a **Single Tenant**. You are dedicating a 600W system entirely to one stream of text.

*   **The Overflow Trap:** If your model (42GB) barely fits your VRAM (32GB), it overflows into System RAM. The fast GPU waits for the slow RAM.
*   **Energy per Request:** **~40,500 Joules**.
*   **Phone Battery:** **75%** (One heavy task drains most of a phone).
*   **Google Search Equiv:** **37.5 GSE**.

**Verdict:** You are paying a **40x energy premium** compared to the cloud estimate (864 J) for the guarantee that your data never leaves your building.

---

### 4. Local Efficiency: The Batching Fix

Local AI doesn't *have* to be inefficient. The inefficiency comes from "Chatting" (Serial processing). The efficiency comes from "Processing" (Batching).

If you use that same RTX 5090 to summarize 1,000 documents at once, the GPU loads the model *once* and applies it to the entire queue simultaneously.

*   **Energy per Request:** **~200 Joules**.
*   **Google Search Equiv:** **0.18 GSE**.
*   **Verdict:** **Parity.** If you saturate the hardware, local becomes as green as the cloud.

---

### 5. The Golden Rule: Reading vs. Writing

The most important physics lesson for 2025: **Input is Cheap. Output is Expensive.**

*   **Input (Reading):** Your GPU processes thousands of words instantly (Parallel).
*   **Output (Writing):** Your GPU generates words one by one (Serial).

| Scenario | Ratio | Verdict |
| :--- | :--- | :--- |
| **The Summarizer** | 5k In / 200 Out | **Efficient.** Reading is cheap. |
| **The Generator** | 100 In / 4k Out | **Wasteful.** Writing is the battery killer. |
| **The "Reasoning"** | 50 In / 3k Out | **Hidden Cost.** "Thinking" models generate invisible tokens that burn real watts. |

---

### 6. The Rebound Effect: Machine Scale

If per-prompt efficiency is improving (0.24 Wh), why is global AI energy consumption projected to hit **29.3 TWh annually** ([Frontiers][5])?

**The Jevons Paradox:** As efficiency increases, consumption increases to match.
We are moving from **Chatbots** (Human speed) to **Agents** (Machine speed).
*   **Human Scale:** You prompt. You read. You think. (5 queries/hour).
*   **Machine Scale:** An agent wakes up, scrapes your SharePoint, summarizes 5,000 PDFs, and updates a CRM. (50,000 queries/hour).

Efficiency gains matter, but they are currently being outpaced by volume. We made "thought" cheap, so now we are wasting it on infinite loops.

---

### 7. Conclusions

**1. The Sovereignty Tax.**
Running local AI is less efficient than the cloud, but the trade-off is valid. Spending **37 Google Searches** worth of energy to keep proprietary IP or patient data secure is cheap insurance.

**2. Stop "Chatting" Locally.**
If you have a powerful local rig, don't use it for casual chat. Use it for **RAG** (Reading) and **Batch Processing**. Reading on a 5090 is nearly free; writing is expensive.

**3. Smarter Deployment.**
Sustainability isn't just about hardware; it's about usage. Stop building agents that summarize documents nobody reads. Stop training new models for marginal gains when existing ones suffice. The era of "free" compute is over—every token has a physical cost.

---

[1]: https://knowledge.wharton.upenn.edu/article/the-hidden-cost-of-ai-energy-consumption/ "The Hidden Cost of AI Energy Consumption"
[2]: https://arxiv.org/pdf/2505.09598 "How Hungry is AI? Benchmarking Energy, Water, and Carbon..."
[3]: https://arxiv.org/abs/2505.09598 "Benchmarking Energy, Water, and Carbon Footprint of LLM Inference"
[4]: https://cloud.google.com/blog/products/infrastructure/measuring-the-environmental-impact-of-ai-inference "Measuring the environmental impact of AI inference"
[5]: https://www.frontiersin.org/journals/communication/articles/10.3389/fcomm.2025.1572947/epub "Energy costs of communicating with AI"
[6]: https://www.researchgate.net/publication/368826870_Trends_in_AI_inference_energy_consumption "Trends in AI inference energy consumption"
[7]: https://vu.nl/en/news/2025/ai-rapidly-on-its-way-to-becoming-the-largest-energy-consumer "AI rapidly on its way to becoming the largest energy consumer"
