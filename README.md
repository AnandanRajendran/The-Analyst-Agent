# The-Analyst-Agent
An automated AI News Analyst built with n8n and Google Gemini. Features include article de-duplication, 'hype filtering' via LLMs, and structured data extraction to Google Sheets.

**Selected Task: Option 2 - The "Analyst" Agent (Advanced Automation)**

I chose this task to demonstrate engineering depth in **process automation, data pipeline architecture, and LLM integration**. It focuses on the practical challenge of filtering noise ("hype") from real signal in data streams, a critical problem in modern AI applications.

# Project Overview

This is an autonomous AI agent built with **n8n** that monitors news streams for "AI Startups." It performs real-time de-duplication, analyzes content using **Google Gemini**, filters out marketing "fluff," and saves structured insights into a database.

**Key Features**

* **Smart De-Duplication:** Filters out repetitive stories across different publishers to ensure a clean dataset.

* **"Hype" Filtration:** Uses an LLM to semantically analyze articles and discard generic marketing content, keeping only substantial events (Funding, Product Launches, Partnerships).

* **Structured Extraction:** Converts unstructured news text into a strict JSON schema (Company, Sentiment Score, Category).

* **Rate-Limit Handling:** Implements a robust batching loop (1 item every 4 seconds) to respect API constraints without crashing.

| Component | Choice | Justification |
| :------: | :---------: | :----- |
| Workflow Engine | n8n | Chosen for its visual logic flow which allows for rapid handling of complex branching and looping (e.g., the "Hype Filter" logic) significantly faster than writing boilerplate Python scripts. |
| LLM | Gemini 2.0 Flash | Selected for its speed and efficiency. Flash is optimized for high-volume text processing tasks like this. It provides the necessary reasoning capability for sentiment analysis at a fraction of the latency (and cost) of GPT-4. |
| Data Source | NewsAPI | Provides a reliable, structured stream of global tech news with robust query parameters for filtering. |
| Scripting | JavaScript (Native) | While Python is a strong tool for data, I utilized n8n's native JavaScript runtime for JSON parsing. This minimizes execution latency by avoiding the overhead of spinning up isolated Python sandboxes for every single article processed. |

# How to Set Up and Run Locally

**Prerequisites**

* **Node.js** (v16 or higher).

* **n8n** installed locally.

* A **Google Cloud Project** (for Sheets API & Gemini).

* A **NewsAPI** Key.

**Installation Steps**

**1. Clone the repository**

```
git clone https://github.com/AnandanRajendran/The-Analyst-Agent
cd The-Analyst-Agent
```

**2. Start n8n** You can run n8n using npx (Node.js required)
```
npx n8n
```
*Once running, open ```http://localhost:5678``` in your browser.*

**3. Import the workflow**

1. In the n8n interface, click **Workflows** (top right menu) > **Import from File**.

2. Select the ```workflow.json``` file included in this repository.

**4. Configure Credentials** For security, API keys are not included in the code. You must enter your own:

* NewsAPI: Open the HTTP Request node. In "Query Parameters", enter your API Key.

* Google Gemini: Open the Google Gemini node. Create a new credential using your Google AI Studio Key.

* Google Sheets: Open the Google Sheets node. Create a credential using "Sign in with Google" (OAuth2) or a Service Account.

**5. Database Setup**

1. Create a new Google Sheet

2. In the first row, add these exact headers: ```company_name```, ```category```, ```sentiment_score```, ```summary```, ```is_fluff```.

3. Update the Google Sheets node in n8n to point to your specific file.

**Execution**

Click the **Execute Workflow** button at the bottom of the canvas. The agent will begin fetching, filtering, and saving data in real-time.

# Logic Map

**This diagram represents the decision flow implemented in the agent:**

1. **Ingest:** Fetch raw JSON from NewsAPI.

2. **Normalize:** Split lists into individual items.

3. **Deduplicate:** Check Title against incoming batch; drop duplicates.

4. **Loop:** Process 1 item at a time (Rate Limit Protection).

5. **Analyze (LLM):** Gemini extracts entities and determines is_fluff boolean.

6. **Gatekeeper:**

  * IF ```is_fluff == true``` → Discard.

  * IF ```is_fluff == false``` → Save to Database.

7. **Throttle:** Wait 4 seconds.

8. **Repeat:** Continue loop.

***Submitted By Anandan Rajendran***
