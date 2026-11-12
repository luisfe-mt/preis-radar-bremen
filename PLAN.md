# Project plan: Preis-Radar Bremen

_Last updated: 2026-11-12_

## Goal

Build a **forecast-and-explain engine for fuel prices at Bremen-area petrol stations**. It should:

1. forecast short-term price moves better than simple baselines,
2. explain each forecast in a way a non-expert can follow (SHAP), and
3. show which stations lead price changes and which follow them.

The project is a **non-commercial portfolio piece** (see [DECISIONS.md](DECISIONS.md)). It should show a complete, honest data/ML workflow: data engineering, time-series validation, model explanation, deployment and plain-language communication.

## Final output: what a visitor sees

A public Streamlit website (link: _(fill in: Cloud Run URL once deployed)_) with:

- **Map / list of Bremen-area stations** with current prices from the Tankerkönig API (CC BY 4.0, attributed).
- **Forecast view**: predicted price direction and level for the next _(fill in: forecast horizon, e.g. hours ahead)_ per station and fuel type _(fill in: E5 / E10 / Diesel, whichever are in scope)_.
- **"Why?" panel**: SHAP explanation of the drivers of a forecast, in plain words.
- **Model vs. baselines**: MAE in cents and direction hit rate on a held-out, later time period, compared with the "price stays the same" and "same hour yesterday" baselines.
- **Leaders and followers**: which stations tend to move first and which react, with the method and its limits explained.
- **Weekly memo (DE/EN)**: a short plain-language summary written by a small, cheap Claude model _(fill in: model chosen)_, built only from numbers the pipeline computed.
- **Data and licence notes**: attribution to tankerkoenig.de, the licences, and a link to the "What didn't work / limitations" section.

## Phases

### Phase 0: Project setup and governance
- [x] Create public repo with MIT licence
- [x] Write PLAN.md, DECISIONS.md, README.md
- [x] Add `.env.example` (placeholders only); confirm `.gitignore` excludes `.env`, `data/` and Python caches
- [ ] Set up Python environment and dependency file _(fill in: tool, e.g. uv / pip + requirements.txt)_
- [ ] Add basic project layout (`src/`, `notebooks/`, `tests/`) and linting

### Phase 1: Inspect the history archive (read-only)
- [ ] Locate the unpacked archive via `DATA_DIR` (outside the repo)
- [ ] Document the actual folder structure, file formats, columns and time coverage _(fill in after inspection; nothing is assumed yet)_
- [ ] Document how stations are identified and how station metadata (location, brand) is stored
- [ ] Note data-quality issues found (gaps, duplicates, outliers, time zones)

### Phase 2: Build the Bremen subset
- [ ] Define "Bremen area" _(fill in: e.g. radius around a point, postcode list, or city boundary, plus reasoning)_
- [ ] Define the time window _(fill in: start and end dates, plus reasoning)_
- [ ] Write a reproducible extraction script that reads from `DATA_DIR` and writes one small modelling file _(fill in: format and location; it must be gitignored)_
- [ ] Record row counts, station count and date range of the subset _(fill in from the actual output)_

### Phase 3: Exploratory analysis and data quality
- [ ] Price patterns by hour of day, weekday, station and brand
- [ ] Turn event-based price changes into a regular time grid _(fill in: grid size)_ and document how
- [ ] Decide how to handle missing data and stations that open or close

### Phase 4: Baselines and evaluation setup
- [ ] Time-based train / validation / test split (never random); fix the cut-off dates _(fill in)_
- [ ] Baseline 1: "price stays the same"
- [ ] Baseline 2: "same hour yesterday"
- [ ] Metrics: MAE in cents and direction hit rate, both on the test period
- [ ] Results table for the baselines _(fill in with real numbers only)_

### Phase 5: XGBoost model
- [ ] Feature engineering (calendar, lags, competitor prices, etc.; only information available at forecast time)
- [ ] Check for leakage (no future data in features)
- [ ] Tune with time-series cross-validation
- [ ] Compare with baselines on the same test period _(fill in: results, including if the model does NOT win)_

### Phase 6: Explanations (SHAP)
- [ ] Global feature importance
- [ ] Explanations for single forecasts
- [ ] Turn SHAP output into plain-language text for the website

### Phase 7: Leader/follower analysis
- [ ] Choose a method _(fill in: e.g. lagged correlation, timing of first move after a market-wide change, Granger-style tests)_
- [ ] Rank stations by how often they lead or follow
- [ ] Write down what the analysis can and cannot claim (correlation is not proof of strategy)

### Phase 8: Live data pipeline
- [ ] Fetch current prices from the Tankerkönig API (key from `.env` / secret manager, never in code)
- [ ] Respect API usage rules and rate limits _(fill in: limits from the Tankerkönig terms)_
- [ ] Store live snapshots in Cloud Storage
- [ ] Daily Cloud Scheduler job to refresh data and forecasts

### Phase 9: Weekly memo (DE/EN)
- [ ] Prompt template that only uses computed numbers (no free-form facts)
- [ ] Check that every number in the memo matches the pipeline output
- [ ] Cost cap per run _(fill in: budget)_
- [ ] Archive past memos

### Phase 10: Website and deployment
- [ ] Streamlit app with the views listed above
- [ ] Dockerfile
- [ ] Deploy to Cloud Run in `us-central1`, max 1 instance, min 0 instances (no always-on)
- [ ] Cloud Storage bucket for data; Cloud Scheduler for the daily job
- [ ] Budget alert in Google Cloud _(fill in: amount)_
- [ ] Secrets in Secret Manager or Cloud Run environment settings, never in the image

### Phase 11: Write-up
- [ ] Fill in results, charts and the "What didn't work / limitations" section in README
- [ ] Short project write-up for applications _(fill in: where, e.g. blog / LinkedIn)_

## Definition of done

The project is done when **all** of these are true:

- [ ] The extraction script rebuilds the Bremen subset from the archive in `DATA_DIR` with one command.
- [ ] The model and both baselines are evaluated on the same later test period, and the README reports MAE (cents) and direction hit rate for all three, whether or not the model wins.
- [ ] SHAP explanations are shown on the website for individual forecasts.
- [ ] The leader/follower analysis is published with its method and limits.
- [ ] The website runs on Cloud Run (us-central1, max 1 instance, min 0) and updates daily via Cloud Scheduler.
- [ ] A weekly DE/EN memo is generated, and its numbers are checked against the pipeline output.
- [ ] The repo contains no secrets and no raw or derived archive data; licences and attribution are shown on the site and in the README.
- [ ] The README "What didn't work / limitations" section is filled in honestly.
- [ ] A new reader can clone the repo and run it locally by following the README.
