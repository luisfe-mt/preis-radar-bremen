# Decision log

Newest entries at the bottom. Each entry records the context, the decision, the alternatives considered and the consequences. Decisions can be revised; when that happens, add a new entry that points back to the old one instead of editing it.

---

## 2026-11-12: D-001 Topic: fuel prices in the Bremen area

**Context.** I need one portfolio project for Data/ML and Pricing internship applications (summer 2027). It should use real, public data and involve forecasting, explanation and a pricing angle.

**Decision.** Forecast and explain fuel prices at Bremen-area stations using Tankerkönig / MTS-K data.

**Why.**
- German stations must report price changes to the Markttransparenzstelle für Kraftstoffe (MTS-K), so the data is detailed (station level, every change) and publicly available through Tankerkönig.
- Fuel pricing is a real pricing problem: stations react to competitors, time of day and input costs. That fits Pricing roles directly.
- A local focus (Bremen) keeps the data small enough to work with on a laptop and makes the results concrete.

**Alternatives considered.** _(fill in: other topics considered, if any)_

**Consequences.** The project depends on Tankerkönig's API and archive terms (see D-004, D-006).

---

## 2026-11-12: D-002 Model: XGBoost with SHAP

**Decision.** Use gradient-boosted trees (XGBoost) as the main model and SHAP to explain its predictions.

**Why.**
- Works well on tabular features (lags, calendar, competitor prices) without much tuning.
- Fast to train on a regional subset on a laptop.
- SHAP gives per-forecast explanations, which the website and the weekly memo need.
- Common in industry, so it is relevant for the roles I am applying to.

**Must hold.** The model has to beat two simple baselines, "price stays the same" and "same hour yesterday", on MAE (cents) and direction hit rate, using a time-based split (never random). If it does not beat them, that result is reported as it is.

**Alternatives considered.** Pure time-series models (e.g. ARIMA-type) or deep learning models. _(fill in: whether any are tried as extra comparisons)_

**Consequences.** Features must be built only from information available at forecast time, to avoid leakage.

---

## 2026-11-12: D-003 Cloud region: us-central1

**Decision.** Deploy on Google Cloud in `us-central1`: Cloud Run (max 1 instance, min 0, so no always-on instances), Cloud Scheduler for a daily job, Cloud Storage for data.

**Why.**
- Cost: this is an unpaid student project. Some Google Cloud free-tier allowances apply only in certain US regions. _(fill in: check the current free-tier terms for Cloud Run, Cloud Storage and Cloud Scheduler and note which ones apply in us-central1)_
- Scale-to-zero and a single instance keep costs bounded.

**Trade-offs.**
- Higher latency for visitors in Germany than a European region. Acceptable for a low-traffic portfolio site.
- Data is stored in the US. The data is public fuel-price data with no personal data, so this is acceptable here. This should be revisited if the project ever handles personal data.
- Cold starts after idle periods because there are no always-on instances.

---

## 2026-11-12: D-004 History archive is for non-commercial use only

**Context.** Tankerkönig provides two data sources with different licences:
- **Current prices (API):** CC BY 4.0, attribution to tankerkoenig.de required.
- **History archive:** CC BY-NC-SA 4.0 (Attribution, NonCommercial, ShareAlike).

**Decision.** The whole project is treated as a non-commercial portfolio piece. The history archive is used only for research, model training and evaluation in that non-commercial context.

**Why.** The NC term of the archive licence does not allow commercial use. Because the trained model is built from the archive, it is simpler and safer to keep the whole project non-commercial than to try to separate commercial and non-commercial parts.

**Consequences.**
- No paid features, ads or commercial reuse of this project while it is built on the archive.
- If this ever becomes a commercial product, the history data would need a different licence or source. _(fill in: open question, not legal advice; check with Tankerkönig before any commercial step)_

---

## 2026-11-12: D-005 Raw history data is never committed and lives outside the repo

**Decision.** The raw history archive (about 25 GB compressed, about 100 GB unpacked) stays on a local drive outside this repository and outside any notes vault. Code finds it through the `DATA_DIR` variable in a local `.env` file (see `.env.example`).

**Why.**
- Size: far too large for Git or GitHub.
- Licence: publishing raw archive files would redistribute data under CC BY-NC-SA terms without control over how they are used.
- Portability: anyone with their own copy of the archive can set `DATA_DIR` to their path; no machine-specific paths are hard-coded.

**Consequences.**
- `.env` and `data/` are in `.gitignore`.
- Scripts must fail with a clear message if `DATA_DIR` is not set.
- The folder structure of the archive is not assumed; it will be inspected and documented first (PLAN.md, Phase 1).

---

## 2026-11-12: D-006 Licence of derived datasets; live site uses the CC BY 4.0 API

**Decision.**
- Any dataset **derived from the history archive** that we publish (for example a cleaned Bremen subset or aggregated tables) must carry the **same licence, CC BY-NC-SA 4.0**, with attribution to tankerkoenig.de, and its use stays non-commercial.
- The **live website** shows current prices from the **Tankerkönig API (CC BY 4.0)**, with attribution to tankerkoenig.de.
- The project's **code** is MIT-licensed. The MIT licence does not cover the data.

**Why.** ShareAlike requires adapted material to be shared under the same licence. Keeping the two data sources and the code licence clearly separate avoids mixing up what is allowed for which part.

**Open question.** Whether model forecasts or model files trained on the archive count as "adapted material" is not clear to me. Until clarified, they are treated as covered by the archive's terms (non-commercial, attributed). _(fill in: outcome if clarified)_

**Consequences.** README and website show both licences and the attribution. Any published data file gets a licence note next to it.

---

## 2026-11-12: D-007 Train on a Bremen-only, time-limited subset

**Decision.** Do not model the full national archive. Filter it to Bremen-area stations and a limited time window, and save one small file for modelling.

**Why.**
- Fits on a laptop; fast iteration.
- Matches the project's local focus and keeps the leader/follower analysis readable.
- Smaller data is easier to check by hand for quality problems.

**Open parameters.**
- Definition of "Bremen area": _(fill in: radius / postcodes / boundary, and why)_
- Time window: _(fill in: dates, and why, e.g. avoiding unusual periods or making sure the test period is recent)_
- File format and location of the subset: _(fill in; it stays gitignored)_

**Trade-offs.** Results may not transfer to other regions or periods. This goes into the README limitations section.
