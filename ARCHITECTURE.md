# Technical Architecture Design Document

## AI Executive Decision Intelligence Platform

This document describes the high-level architecture, module design, data workflows, and core engineering decisions behind the AI Executive Decision Intelligence Platform. It is designed to demonstrate engineering rigor for system architects, engineering leads, and technical hiring panels.

---

## 1. High-Level System Architecture

The platform uses a layered model-view-controller (MVC) inspired design tailored for Streamlit micro-frontends.

```
                  [ CSV Ingestion (app.py) ]
                              │
                              ▼
               [ Dataset Validation (data_processing.py) ]
                              │
                              ▼
            [ Automatic Detection / Regex Matcher ]
                              │
                              ▼
               [ Data Cleaning / Type Casting ]
                              │
                              ▼
             [ KPI Calculations (kpis.py) ]
                              │
               ┌──────────────┴──────────────┐
               ▼                             ▼
       [ Visualizations ]            [ Health Score ]
     (charts.py / Plotly)           (0-100 index math)
               │                             │
               └──────────────┬──────────────┘
                              ▼
                  [ AI Context Builder ]
                   (grounding template)
                              │
                              ▼
                  [ OpenAI / Rule Engine ]
                   (report.py / copilot.py)
                              │
                              ▼
               [ Strategic Recommendations ]
                              │
               ┌──────────────┴──────────────┐
               ▼                             ▼
       [ Executive Brief ]          [ Board PPTX Deck ]
          (pdf_generator.py)           (ppt_generator.py)
               │                             │
               └──────────────┬──────────────┘
                              ▼
                     [ Download Package ]
```

---

## 2. Folder Structure & Code Layout

The project follows a clean modular layout where frontend routing is kept separate from mathematical operations and document rendering.

```
ai_business_insights_platform/
│
├── app.py                      # Main Streamlit Frontend entry point and Layout Router
├── requirements.txt            # Package dependencies manifest
├── sample_sales_data.csv       # Default Mock dataset used for local execution demos
├── .env                        # Local environmental variables configuration (ignored by Git)
├── .gitignore                  # Git exclude criteria
│
└── src/                        # Platform Core Modules
    ├── __init__.py
    ├── data_processing.py      # Column matcher, CSV loaders, and data type sanitizers
    ├── kpis.py                 # Core financial indices & Business Health Score math
    ├── charts.py               # Plotly Express template renderers
    ├── report.py               # Strategic Report compiling routines (OpenAI / Heuristics)
    ├── copilot.py              # Chat Q&A Consultant engine (OpenAI / Heuristics)
    ├── pdf_generator.py        # PDF briefing document builder (ReportLab API)
    └── ppt_generator.py        # Widescreen PowerPoint slide compiler (python-pptx API)
```

### Module Responsibilities:

| File / Module | Core Responsibility |
| :--- | :--- |
| **`app.py`** | Handles Streamlit state caching, tab containers, sidebar file configurations, and user click actions. |
| **`src/data_processing.py`** | Executes regular-expression mapping to align raw headers with standard columns, parses date formats, and filters null rows. |
| **`src/kpis.py`** | Runs regional summaries, volatility indicators ($CV$), concentration risk ($HHI$), and calculates the Business Health Score. |
| **`src/charts.py`** | Renders donut distribution slices, horizontal bars, spline trendlines, and grouped metric charts. |
| **`src/report.py`** | Manages prompt templates, queries OpenAI, and runs local heuristic rules to assemble structured strategic reports. |
| **`src/copilot.py`** | Operates conversation history arrays and runs user question validation against domain keywords. |
| **`src/pdf_generator.py`** | Controls flowable tables, paragraphs, spacers, page headers, footers, and page numbers via ReportLab. |
| **`src/ppt_generator.py`** | Instantiates widescreen 16:9 layouts and compiles slides, metric cards, timelines, and next-quarter plans. |

---

## 3. System Data Flow

The lifecycle of an uploaded dataset flows through several stages:

1. **User Upload:** User uploads a raw CSV file via the Streamlit file-uploader widget.
2. **Validation:** `load_data` parses the file. If malformed or corrupted, it catches exceptions and flags a clean warning.
3. **Cleaning:** Casts headers, standardizes datetimes, cleans non-numeric currency characters, and strips null values in mapped columns.
4. **KPI Calculations:** Computes sums, regional margins, and category distributions deterministically in Python.
5. **Charts Generation:** Generates responsive Plotly charts with auto-truncated labels and Top-K groupings.
6. **Business Health Score Calculation:** Merges growth, volatility, and geographic HHI scores to compile a single 0-100 corporate health value.
7. **AI Prompt Construction:** Injects the pre-calculated deterministic KPIs into structured, markdown prompts.
8. **OpenAI Synthesis:** GPT-4o parses the grounded prompt context and produces formal consulting reports.
9. **Executive Report & Presentation Export:** The raw Markdown outputs and charts are fed into the PDF and PowerPoint generators to build final downloadable files.

---

## 4. Architectural Design Decisions

### Why Streamlit?
- **Rapid Prototyping:** Simplifies state management and client/server messaging, rendering widgets in Python.
- **Micro-Frontend Pattern:** Integrates data cleaning, charting, and LLM orchestration inside a single lightweight framework.

### Why Pandas?
- **Data Wrangling Speed:** Exceptional vectorization performance for datasets up to 100k rows.
- **Robust Cleaning Mechanics:** Simplifies datetime conversions and handles null values safely.

### Why Plotly?
- **Client-Side Rendering:** Uses WebGL to render charts, saving server CPU cycles.
- **Interactivity:** Provides out-of-the-box tooltips, zooming, and category filters.

### Why Context-Grounded Prompt Engineering?
- **Zero Hallucinations:** Generative AI is prone to calculation errors. By pre-aggregating numbers in Pandas and feeding them as constants to the LLM, the model's role is restricted to interpretation and wording.

---

## 5. Enterprise Scalability Strategy

* **Large Datasets (> 1M rows):** For massive corporate transaction ledgers, the data engine can be migrated to **Polars** (lazy execution) or calculations pushed down directly to a cloud database (Snowflake/Redshift) using SQLAlchemy.
* **Cache Architecture:** Heavy data aggregation routines use Streamlit's cache decorators, saving memory states across dashboard switches.
* **Database Integrations:** Replace static file uploaders with SQLAlchemy connection pools, allowing direct pipelines to enterprise data warehouses.

---

## 6. Security Architecture

* **Secure Secret Injection:** API keys are never stored in source code. They are stored in `.env` files and loaded server-side using environmental variables (`os.getenv`).
* **Safe Fallback Execution:** If the API key is missing, the platform automatically switches to a local heuristics-based rule engine, keeping services functional.
* **Strict Input Validation:** Mapped numeric columns are parsed and cast to numeric floats. Any injection attempts inside CSV fields are neutralized.
