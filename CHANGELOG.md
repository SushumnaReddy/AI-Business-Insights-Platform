# Changelog

All notable changes to the AI Executive Decision Intelligence Platform will be documented in this file.

---

## [v1.5] - 2026-07-08
### Added
- Added `pdf_generator.py` for exporting reports as styled PDFs.
- Added `ppt_generator.py` for programmatically compiling widescreen 16:9 board presentation slide decks.
- Added a robust offline rules engine fallback when OpenAI API keys are absent.
- Added structured business query filtering to the AI Consultant interface.
### Changed
- Refactored `app.py` UI layouts with a premium dark-slate styling and responsive column margins.
- Standardized `requirements.txt` dependencies.
- Added strict column mapping regex heuristics to support e-commerce datasets (e.g. Olist states and payment values).

---

## [v1.4] - 2026-07-01
### Added
- Introduced the composite Business Health Score (0-100) aggregating volatility and concentration metrics.
- Added responsive locale-aware KPI formatting (e.g. `24.95M`, `150.4K`) to prevent card wrapping.

---

## [v1.3] - 2026-06-20
### Added
- Implemented the conversational "AI Consultant" Strategy Q&A tab.
- Integrated OpenAI session memory to persist dialogue contexts.

---

## [v1.2] - 2026-06-10
### Added
- Added the "AI Executive Brief" tab compiling automated briefings using GPT-4o.
- Built prompt engineering templates grounded in pre-calculated numerical variables.

---

## [v1.1] - 2026-05-28
### Added
- Integrated Plotly Express charts (regional donuts, horizontal bar charts, and monthly line trends).
- Added the "Data Preview" tab for inspectable statistics.

---

## [v1.0] - 2026-05-15
### Added
- Initial release featuring core CSV uploads, auto-mapping matching rules, and raw KPI card dashboard modules.
