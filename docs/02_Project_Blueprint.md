# Pulse — Project Blueprint

---

## PART I — BUSINESS & PRODUCT LOCK

### §1. Executive Vision — Confirmed, One Addition

Document 01's vision and six guiding principles (Unified Intelligence, Personalization, Transparency, Scientific Grounding, Modularity, Continuous Learning) stand. Document 10 surfaced one gap: none of the six principles explicitly address *partial or missing signal availability* — and we now know that's not an edge case, it's the majority case (HRV present 33% of days, SpO2 17%). Adding a seventh principle:

**Confidence-Awareness:** Every output states not just *what* was predicted but *how much evidence supported it*. A recovery score computed from 3 signals is presented differently than one computed from 7. This is not a caveat bolted onto the UI — it's an architectural property that has to be threaded through the data model, features, models, and LLM layer (see §11, §14, §17, §18).

### §2. Opportunity Assessment — Reframed Honestly

This isn't a market opportunity. It's an interview-signal opportunity, and it should be evaluated as one.

| nference requirement | WHIP capability demonstrating it | Evidence artifact |
|---|---|---|
| Python | ETL, feature engineering, ML pipeline | `/src` modules, notebooks |
| SQL | Star-schema warehouse, window-function analytics | `/sql` schema + queries |
| Power BI | 4-5 connected dashboards | `.pbix` + published screenshots |
| ML (preferred) | Recovery modeling, forecasting, anomaly detection, clustering | Model cards with validation results |
| LLM/RAG (preferred) | Explanation layer over structured ML output | Prompt contracts + example transcripts |
| High communication | This entire document series + README | `/docs` |

### §3. Industry & Competitor Analysis

| Platform | Known for | Gap WHIP targets |
|---|---|---|
| WHOOP | Recovery %, strain | Proprietary scoring, opaque methodology |
| Garmin | Training readiness | Fitness-athlete framing, not general wellness |
| Oura | Sleep, readiness | Single-device silo |
| Fitbit | Broad wellness metrics | Descriptive, not explanatory (confirmed directly in Document 10 — Fitbit's own composite scores like `stress_score` are themselves only 25% populated) |
| Apple Health | Aggregation | Aggregates, doesn't reason |

None of them show *why* a score changed with transparent, auditable evidence, and none of them communicate *confidence* when signal coverage is thin. That combination — not "another recovery score" — is WHIP's actual differentiation, and it's a differentiation Document 10's findings made *necessary*, not just nice-to-have.

### §4. Business Thesis

WHIP's thesis: an intelligence layer is more valuable — and more demonstrable as a DS skillset — when it's honest about uncertainty than when it's falsely complete. A model that says "moderate confidence, based on 4 of 7 signals" is more defensible in an interview than one that silently imputes and presents a clean number. This thesis is now directly evidence-backed by Document 10, not just a design preference.

### §5. Project Charter

**In scope (v1):**
- Full ETL + canonical warehouse for LifeSnaps (71 participants)
- EDA, feature engineering, statistical baselining
- 3 ML tasks (trimmed from original 6 — see §9)
- Explainability layer (SHAP/coefficient-based)
- LLM explanation assistant (single-turn + basic RAG over participant history)
- 4 Power BI dashboards
- Full documentation set

**Explicitly out of scope (v1) — and why:**
| Excluded | Reason |
|---|---|
| Live device integration / real-time ingestion | No live users; batch analytics on a static research dataset is the honest framing and still demonstrates the full pipeline |
| Multi-tenant auth/accounts | Not a product decision this role tests |
| Mobile app | Out of skill scope for the JD; would dilute focus |
| Recommendation engine as a trained model | Folded into LLM layer, see §9 — a fabricated "recommender" with no real interaction/feedback signal to learn from would be weaker, not stronger |
| Diagnosis or clinical claims | Explicit ethical/legal boundary — the platform explains patterns, never diagnoses |
| Apple Health / raw `fitbit.bson` integration | Deferred to v2 per the phased data strategy already agreed |

**Success criteria:** every component traceable to a JD requirement (§2), every claim traceable to real computed output (matching the standard already set by your Olist and A/B testing projects), repo readable end-to-end by a stranger in under 15 minutes via README.

### §6. Business Requirements (BRD) — Testable Statements

1. The system shall ingest LifeSnaps daily and hourly source files and produce a queryable warehouse without manual data manipulation.
2. The system shall compute personalized (not population-normed) baselines for each participant.
3. The system shall never present a numeric health output without an accompanying confidence indicator.
4. The system shall explain every ML prediction with the specific evidence (features/signals) that drove it.
5. The system shall communicate insights via natural language without the LLM independently generating physiological claims.
6. The system shall expose all analytical layers (warehouse, features, models) through Power BI.

### §7. Product Requirements (PRD)

**Demo persona decision (evidence-based, not arbitrary):** I profiled all 71 participants on longitudinal length × signal coverage. Participant `621e32e6...` has 97 days of data with 93.8% HRV coverage, 85.6% SpO2 coverage, and 92.8% sleep coverage simultaneously — the best *balance* of duration and multi-signal completeness (a 64-day participant scores marginally higher on coverage alone but doesn't give enough history for meaningful 28-day rolling baselines). **Recommendation: this participant becomes the flagship narrative thread** across README screenshots, dashboard defaults, and written case-study sections — while the underlying system stays fully participant-agnostic (every dashboard and model supports selecting any of the 71). This gives interviewers one coherent story to follow instead of an abstract "works on N=71" claim.

**Product surfaces:** GitHub repo (primary artifact) → Jupyter notebooks (narrated analysis, matching your Olist/A-B testing standard) → SQL warehouse (DuckDB file, inspectable) → Power BI dashboards (`.pbix` + exported screenshots, since Power BI Desktop isn't available in this sandbox — see §19) → LLM assistant (runnable notebook/script demo, not a hosted app).

### §8. Functional Requirements (FRD) — Representative Sample

- FR1: Given a participant and date, the system shall return a 28-day rolling HRV baseline *only if* ≥5 valid readings exist in that window; otherwise it shall return `confidence: insufficient` rather than a value.
- FR2: The recovery score shall be computed from whichever of {HRV, RHR, sleep efficiency, prior-day strain} are available for that day, with the confidence tier downgraded as fewer are present — never silently imputed to look complete.
- FR3: Every LLM-generated explanation shall cite the specific structured fields it was given; the system shall log the exact JSON payload passed to the LLM for auditability.
- FR4: Anomaly flags shall be participant-relative (deviation from that person's own distribution), never population-relative.

### §9. Machine Learning Requirements Specification (MLRS) — Trimmed and Justified

The original wishlist (recovery prediction, forecasting, anomaly detection, clustering, recommendation engine) gets one real cut, based on evidence, not on preserving the original list for its own sake:

**Cut: "Recommendation Engine" as a standalone trained model.** A recommender needs either collaborative signal (irrelevant — this is single-user personalization, not a multi-user marketplace) or interaction/reward data to learn from (doesn't exist in a static research dataset). Building one anyway would be a fake ML component — exactly what a senior interviewer flags fastest. **Folded into the LLM layer instead** (§18): recommendations become a direct, explained consequence of the stats/ML findings, communicated in natural language, not a separately "trained" system pretending to have learned preferences it never observed.

**Retained, with model family recommendations (full rationale in §16):**

| Task | Grain | Approach |
|---|---|---|
| Personalized recovery scoring | Daily | Statistical composite (not black-box ML) + confidence tiering |
| Trend forecasting | Daily, per-signal | Classical time-series (not deep learning — see §16) |
| Anomaly detection | Daily | Robust statistical (MAD-based), not unsupervised ML |
| Day-archetype clustering | Daily, within-participant | Unsupervised clustering on daily feature vectors |

---

## PART II — TECHNICAL ARCHITECTURE LOCK

### §10. Enterprise System Architecture

```
LifeSnaps CSV/BSON  ──►  Ingestion & Validation (Python)
                              │
                              ▼
                    Canonical Health Data Model  (§11)
                              │
                              ▼
                     SQL Warehouse — DuckDB  (§13)
                              │
                ┌─────────────┼─────────────┐
                ▼             ▼             ▼
        Feature Engine   Statistics    Power BI (§19)
           (§14)            (§15)
                │             │
                └──────┬──────┘
                       ▼
                ML Layer (§16) ──► Explainability (§17)
                       │
                       ▼
              LLM Communication Layer (§18)
```

**Key stack decision — the warehouse engine:**

| Option | Verdict |
|---|---|
| PostgreSQL | Rejected — requires a running server process; overkill infra for a single-node batch-analytics portfolio project, adds deployment friction with no analytical benefit here |
| SQLite | Rejected — row-oriented, weak window-function/analytical performance at scale |
| **DuckDB** | **Selected** — embedded (zero infra), columnar (built for exactly this kind of analytical query), full SQL window-function support, directly queryable from Python/pandas, and connects to Power BI via ODBC driver — resolves the open "DuckDB + Power BI connectivity" question directly. This is also a more sophisticated, current choice than defaulting to Postgres, and shows judgment about matching tooling to problem shape rather than reaching for the most familiar option. |

### §11. Canonical Health Data Model

Source-agnostic by design — LifeSnaps today, 2016 Fitbit or Apple Health later, without redesigning anything downstream.

| Entity | Grain | Key fields |
|---|---|---|
| `dim_participant` | 1 row/participant | participant_id, source_system, age_bucket, gender, bmi_bucket, enrollment_date |
| `dim_date` | 1 row/calendar date | date_key, day_of_week, is_weekend, study_week |
| `fact_daily_physiology` | 1 row/participant/day | resting_hr, hrv_rmssd **+ hrv_confidence_tier**, spo2 **+ spo2_confidence_tier**, respiratory_rate, skin_temp_variation, sleep_* fields, steps, calories, active_minutes_*, estimated_cardio_fitness (never "vo2max_measured" — naming enforces the §6.3 caveat from Document 10 at the schema level) |
| `fact_hourly_activity` | 1 row/participant/hour | bpm, steps, calories, hr_zone_minutes |
| `fact_mood_context` | 1 row/participant/day | mood_label, context_label, response_count |
| `fact_survey_response` | 1 row/response | survey_type, submit_date, normalized_score |

Every physiological metric that Document 10 flagged as high-missingness gets a paired `*_confidence_tier` column (`HIGH` / `MEDIUM` / `LOW` / `ABSENT`) computed at ingestion — confidence-awareness enforced structurally, not left to downstream code to remember.

### §12. Data Engineering Strategy

**Pipeline layers:** raw (untouched source files, checksummed) → staging (typed, parsed, one row = one source record) → canonical (mapped into §11's model) → warehouse (DuckDB, aggregated + indexed).

**Specific fixes required, each traced to a Document 10 finding:**
| Issue found | Handling |
|---|---|
| BMI binning collision (specific values + overlapping labels) | Explicit documented rule: collapse everything to consistent bucket ranges at staging; log every row that had to be coerced, don't silently overwrite |
| `activityType`/`badgeType` stringified lists | Parsed via `ast.literal_eval` at staging into proper multi-label indicator columns |
| Hourly `temperature` is relative-to-baseline, not absolute | Field renamed `temperature_variation_from_baseline` in canonical model so no downstream consumer misinterprets it |
| `filteredDemographicVO2Max` | Renamed `estimated_cardio_fitness`, documented inline as formula-based, not measured |
| Wear-time gaps (partial-day hourly coverage) | A `records_expected_vs_observed` completeness field per participant-day, feeding the confidence tiers directly |

**Orchestration: a plain Python script sequence (or a lightweight Makefile), not Airflow.** For a single-batch, single-developer, non-recurring pipeline, an orchestrator like Airflow adds operational surface area (scheduler, metadata DB, DAG server) with zero benefit — a senior reviewer would read Airflow-for-a-static-CSV as over-engineering, not sophistication. Idempotency instead comes from checksummed raw inputs and a pipeline that can be re-run end-to-end deterministically.

### §13. SQL Warehouse Strategy

Star schema, fact tables at §11's grain, dimensions `dim_participant` and `dim_date`. This directly supports the kind of query that actually demonstrates SQL skill for this JD: e.g., a 28-day trailing window function partitioned by participant, ordered by date, computing rolling median/MAD HRV — the exact query the personalized-baseline feature (§14) depends on, and a legitimate "here's a non-trivial window function" interview artifact rather than a `SELECT * WHERE`.

### §14. Feature Engineering Strategy

**Confidence tiers** (applied wherever a signal has >20% missingness per Document 10):
- `HIGH`: ≥80% of trailing window observed
- `MEDIUM`: 40–79% observed
- `LOW`: 15–39% observed
- `ABSENT`: <15% observed → feature excluded from that day's model input entirely, not imputed

**Personal baseline methodology:** rolling 28-day window, **median and MAD (median absolute deviation) rather than mean/SD** — robust to the outliers and irregular sampling Document 10 confirmed are real, with a minimum-observation floor (≥5 valid readings) below which the baseline itself is marked `insufficient` rather than computed from too few points.

### §15. Statistical Analysis Strategy

- Baseline construction: as above (§14), justified specifically because in-the-wild wearable data is known to have irregular, non-Gaussian missingness patterns — mean/SD would be distorted by exactly the sparse-sampling issue Document 10 quantified.
- Planned hypothesis tests: paired comparisons (e.g., weekday vs. weekend recovery) using Wilcoxon signed-rank rather than paired t-test by default, since daily physiological deltas are not reliably normal at N≈100/participant — falling back to the parametric test only where normality checks (Shapiro-Wilk) pass.
- Because many signals get tested against many outcomes, **Benjamini-Hochberg FDR correction** applies to any multi-comparison analysis — flagging this now so it's not an afterthought added post-hoc when a reviewer asks "did you correct for multiple comparisons?"

### §16. Machine Learning Strategy

| Task | Target | Candidates considered | Recommended | Why |
|---|---|---|---|---|
| Recovery scoring | Composite daily score | Black-box gradient boosting vs. transparent weighted composite | **Transparent statistical composite**, ML reserved for the *forecasting* task | A recovery "score" is a definitional/business construct (per Document 01's own philosophy: "Statistical models establish personalized baselines... predictions without interpretation are incomplete") — forcing it through an opaque model to check an "ML" box would directly contradict the project's own stated AI philosophy |
| Trend forecasting | Next-day HRV/RHR/sleep | LSTM/deep learning vs. classical (exponential smoothing, ARIMA, or gradient-boosted regression on lag features) | **Classical / GBM on lag features** | ~100 daily observations per participant is far too little for deep learning to learn anything an LSTM wouldn't just overfit; a defensible, correctly-scaled model choice matters more to a senior reviewer than using a fashionable architecture on data that can't support it |
| Anomaly detection | Daily physiological deviation | Isolation Forest / Autoencoder vs. MAD-based statistical thresholding | **Statistical (MAD threshold on personal baseline)** | Same data-scale argument — unsupervised ML anomaly detection needs more data density per participant than this dataset provides to be reliable; a statistically-grounded method is the more rigorous choice here, not the "simpler" one |
| Day-archetype clustering | Daily feature vector | K-Means vs. hierarchical vs. GMM | **K-Means with silhouette-validated k, within-participant** | Straightforward, interpretable, appropriate for the data volume |

**Validation strategy — the detail most likely to get checked:** all model validation uses **GroupKFold split by `participant_id`**, never a random row split. A random split would let the model see the same participant's other days in training, so it could partly "recognize the person" rather than generalize — a classic wearable-ML leakage bug. Using GroupKFold and stating why is a stronger signal than the model's actual metric.

**No deep learning anywhere in v1**, stated explicitly and defended above — this is a considered scope decision given N, not a capability gap.

### §17. Explainable AI Strategy

Tree/GBM-based forecasts explained via SHAP; the transparent statistical composite (§16) is explained by construction (its formula *is* its explanation — weighted contribution per signal, already interpretable). Every explanation surfaces the confidence tier (§14) alongside the evidence, so "why" answers also honestly answer "how sure."

### §18. LLM Communication Strategy

**Strict input contract:** the LLM receives *only* a structured JSON payload — computed feature values, model output, confidence tier, top SHAP factors — never raw physiological tables, and never asked to "figure out" a health conclusion itself. This is enforced at the prompt-construction layer, not just requested via instruction.

**RAG use:** retrieval scoped to the participant's *own* historical structured outputs (e.g., "how does this compare to last month" retrieves that participant's prior 30 days of already-computed daily records), never raw cross-participant data.

**Where recommendations now live** (per the §9 cut): the LLM turns a stats/ML finding plus its confidence tier into a plain-language, explained suggestion — grounded, sourced, never independently generated. This is a stronger, more honest "recommendation engine" than a fabricated trained model would have been.

**Guardrail:** explicit system-level instruction plus a lightweight output check that the assistant never uses diagnostic or prescriptive medical language — it explains patterns, it does not diagnose or prescribe.

### §19. Power BI Strategy

| Dashboard | Feeds from |
|---|---|
| Executive Overview | `fact_daily_physiology` + `dim_participant` aggregates |
| Daily/Recovery | Recovery composite + confidence tier |
| Sleep | Sleep architecture fields |
| ML Prediction | Forecast outputs + SHAP top factors |

**Import mode, not DirectQuery** — the warehouse is static/batch (a fixed research dataset, not a live feed), so Import gives better performance with no real downside. **Connectivity: DuckDB's ODBC driver** into Power BI Desktop, resolving the open technical question directly. One practical note: Power BI Desktop itself isn't installable in this sandbox, so dashboards get built by you locally against the DuckDB file this pipeline produces, or via exported `.csv`/screenshots for repo documentation — flagging this now rather than at the end of the roadmap.

### §20. Implementation Roadmap

| Phase | Deliverable | Depends on |
|---|---|---|
| 0 | Discovery (done) | — |
| 1 | Lock (this document) | Phase 0 |
| 2 | ETL + canonical model + DuckDB warehouse | Phase 1 |
| 3 | EDA + statistical baselining | Phase 2 |
| 4 | Feature engineering (confidence-tiered) | Phase 3 |
| 5 | ML v1 (3 tasks, GroupKFold-validated) | Phase 4 |
| 6 | Explainability (SHAP + composite transparency) | Phase 5 |
| 7 | LLM communication layer | Phase 6 |
| 8 | Power BI dashboards | Phase 3 (can start once warehouse exists) |
| 9 | README, architecture diagrams, final polish | All prior |

---

## Carried-Forward Risks (explicit, not hidden)

1. HRV/SpO2 sparsity limits daily-guaranteed recovery scoring — mitigated by confidence tiers, not solved.
2. 71 participants is enough for personalized (within-person) modeling, thin for any between-person generalization claims — the project should never claim population-level generalizability.
3. Power BI build step happens outside this sandbox — needs to be scheduled as your own follow-up, not something I can execute here.

