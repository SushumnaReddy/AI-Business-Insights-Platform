# System Design & Strategy Interview Notes

## AI Executive Decision Intelligence Platform

This document compiles potential interview questions, suggested structures for answers (STAR method/Direct architecture answers), tradeoffs, and technical deep-dives. It is structured for management consulting (Accenture) and system design (Goldman Sachs) interviews.

---

## 1. System Design Questions & Answers

### Q1: "How do you guarantee that the AI doesn't hallucinate numbers and stats when presenting findings to a CEO?"
* **Suggested Answer:**
  "We enforce strict separation of concerns between **calculation** and **interpretation**. 
  
  LLMs are notoriously unreliable at math operations (such as summing columns, calculating standard deviations, or determining percentage margins). To prevent hallucinations, the platform calculates every metric deterministically in Python using Pandas before the LLM is queried. 
  
  The raw dataset is never sent to the LLM. Instead, we compile a grounded context prompt containing the exact pre-calculated metrics (e.g. *Total revenue: $24.95M, Top product segment margin: 18.2%*). The LLM is instructed to act as a translator and strategist, referencing only these pre-computed metrics. 
  
  Furthermore, a safety filter validates user queries against a business keyword lexicon, rejecting prompts that try to divert the AI from the dataset context."

### Q2: "Streamlit runs on a single main thread and re-runs the entire script on user interaction. How did you optimize this platform for speed and responsiveness?"
* **Suggested Answer:**
  "We addressed Streamlit's runtime performance model through three architectural patterns:
  1. **Data Caching:** Used `@st.cache_data` decorators on the raw file loader. This bypasses slow disk/network reads during subsequent script re-runs.
  2. **Pre-Aggregation:** We aggregate dataframes (e.g., grouping by region or month) before passing them to the visual engines. Rerendering pre-aggregated datasets of 10 rows takes milliseconds, compared to processing millions of raw rows in real-time.
  3. **Plotly WebGL Serialization:** Rendered Plotly charts using standard templates that utilize client-side canvas rendering rather than server-side image generation, minimizing CPU usage on the host server."

### Q3: "What happens if a user uploads a dataset that does not contain a profit column, like parts of the Olist e-commerce dataset? How does your system adapt?"
* **Suggested Answer:**
  "We designed a graceful fallback mechanism into both the database cleaning layer and the mathematical engines:
  * **Dynamic Column Detector:** Regular expressions try to match headers. If a profit column is not detected, it is set to `None`.
  * **Score Weight Redistribution:** The Business Health Score normally allocates 30% of its weight to profit margins. If profit is absent, this 30% weight is redistributed proportionally among the remaining metrics (monthly growth, consistency, and geographic concentration), allowing the scoring engine to remain reliable.
  * **UI & Report Placeholders:** Visualizations that require profit show a clean information card instead of a crash traceback. The PPTX and PDF exporters adapt their layout grids, removing profit tables and inserting revenue-focused alternatives."

---

## 2. Technical & Strategic Trade-offs

| Design Choice | Pros | Cons | Mitigation / Trade-off |
| :--- | :--- | :--- | :--- |
| **In-Memory Processing (Pandas)** | Sub-second calculations, zero database configuration. | Limited by server RAM (crash risk above 2GB uploads). | Trade-off: Ideal for typical spreadsheet analytics. For large enterprise scales, we would migrate to database pushdowns (SQL). |
| **OpenAI GPT-4o API** | High strategic logic capabilities, formal consulting tone. | Cost per token, network latency, dependency on external API keys. | Trade-off: Added a local rule-based heuristics fallback engine that works offline if the API key is missing. |
| **Streamlit Micro-Frontend** | Fast development, Python-native state management. | Limited UI layout control compared to React/Vue. | Trade-off: Injected custom CSS rules to constrain widths, style active tabs, and control card spacing. |

---

## 3. Goldman Sachs System Design Deep-Dive

### "How would you scale this platform to support 100,000 active concurrent users uploading datasets?"
1. **Decouple the Monolith:** 
   Extract the processing logic from the Streamlit frontend. Build a stateless REST API (using FastAPI or Go) to handle data cleaning, KPI calculations, and PDF/PPTX compilation.
2. **Asynchronous Task Queue:** 
   Move file processing and PDF/PPTX generation to worker nodes using **Celery** or **AWS SQS/Lambda**. Upon upload, the file is saved to an S3 bucket, and a message is pushed to a queue. The frontend polls the job status, preventing web server timeouts.
3. **Database Pushdown:** 
   Instead of uploading CSV files to memory, upload files directly to **Snowflake** or **Amazon Redshift**. Run data cleaning and aggregation operations inside the database using SQL queries, returning only the results to the web application.
4. **State Storage:** 
   Move user session states and chat histories from memory to a shared Redis cluster. This allows the API containers to scale horizontally behind a Load Balancer.
