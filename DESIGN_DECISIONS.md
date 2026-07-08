# Design Decisions & Architectural Trade-offs

This document outlines the core technical design decisions, trade-offs, and architecture choices made during the development of the AI Executive Decision Intelligence Platform.

---

## 1. Why Streamlit instead of React?
When starting this project, I chose **Streamlit** over building a custom React/Node.js web application. 
* **Development Velocity:** As a single engineer, building a custom React frontend with API endpoints for data visualization, CSV ingestion, and LLM streaming would have taken weeks. Streamlit allowed me to build the entire dashboard frontend in Python in a fraction of the time.
* **Data Integration:** Since the calculations, file processing, and document compilers are written in Python, keeping the presentation layer in Python eliminated the need for complex serialization over network protocols.
* **Trade-off:** Streamlit runs on a single main thread and re-executes the entire script on every user interaction. To keep the app responsive, I had to be careful with caching data structures and pre-aggregating charts.

---

## 2. Why Pandas for Data Processing?
I selected **Pandas** as the primary data processing engine:
* **Rich API:** Pandas is the industry standard for tabular operations in Python, offering straightforward grouping, date parsing, and descriptive statistics.
* **Memory Performance:** For the target datasets (up to 100k rows, like the Olist e-commerce dataset), Pandas executes calculations in memory in sub-second times.
* **Trade-off:** For large scale production pipelines (millions of rows), Pandas would consume significant RAM and block the UI thread. In a production system, these calculations should be handled by a lazy execution engine like Polars or pushed down to a SQL database.

---

## 3. Why KPIs are calculated before AI analysis
Generative AI models are notoriously poor at math. If we upload a raw transaction CSV and ask a Large Language Model to calculate sales percentages or margins, it will output incorrect values.
* **Decision:** We pre-compute every single KPI (margins, growth rates, HHI concentration) in python using deterministic math.
* **Role of AI:** The LLM is supplied with these constant calculated figures in the prompt context. Its role is strictly constrained to interpreting these metrics and drafting recommendations, eliminating math hallucinations entirely.

---

## 4. Why the Rule-Based Fallback exists
Enterprise systems must be resilient to API timeouts, usage limits, and missing configuration variables.
* **Resilience:** If the user does not have an OpenAI API key or if the connection fails, the platform automatically switches to a deterministic heuristics engine.
* **Continuity:** The user receives structured briefs, SWOT matrices, and board presentations even in offline environments.

---

## 5. Why the Business Health Score was added
Traditional dashboards give users dozens of numbers to look at, which can lead to cognitive overload. 
* **Synthesized Metric:** I designed a composite **Business Health Score (0-100)** to summarize sales consistency, operational growth, and concentration risks in a single figure.
* **Heuristic Design:** It uses heuristic weights to estimate stability, giving analysts an immediate health indicator before they dig into regional charts.

---

## 6. How Hallucinations are Reduced
In addition to pre-calculating all numbers, we applied the following constraints:
* **System Grounding:** The prompt instructs the model to act as a Strategy Consultant and answers *only* using the pre-computed KPI context.
* **Safety Keyword Lexicon:** All user questions in the chatbot are checked against a business keyword mapping. If a question is unrelated to business data, it is rejected before querying the API.

---

## 7. Trade-offs due to Time and Scope
* **File Uploads vs. Databases:** The app reads uploaded CSV files in-memory. In a production-grade system, we would connect directly to database tables (like Snowflake or PostgreSQL) to retrieve records dynamically.
* **Heuristic vs. Predictive models:** The health score and trend analysis are based on descriptive history. Incorporating predictive models (like Prophet or ARIMA) would improve forward-looking recommendations but was out of scope for the initial version.
