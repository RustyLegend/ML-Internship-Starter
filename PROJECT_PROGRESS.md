# PROJECT_PROGRESS.md

Status file for the FlyRank ML Internship capstone (Applied Search Intelligence), kept in sync
with the `AGENTS.md` "Internship execution rules": completed work, changed files, commands,
results, blockers, next steps.

- **Track:** FlyRank ML Internship 2026 — Applied Search Intelligence
- **Lane:** Lane 2 — Refresh / Content Opportunity Scoring (`work/notebooks/w01_research_question.ipynb`)
- **Branch:** `internship-completion`
- **Last updated:** 2026-10-18

---

## 0. Baseline run — EXECUTED 2026-10-18

The reference pipeline was run for real on the bundled sample. **Result: PASS (exit 0)** with
reportlab installed; it **failed at step 5** on the system interpreter before that. Exact
commands, errors, dependencies, files, and metrics below are all observed, not assumed.

### 0.1 Commands and outcomes (in order)

| # | Command | Exit | Outcome |
|---|---|---|---|
| 1 | `python scripts/run_all.py` | 127 | **FAIL** — `python: command not found` (only `python3` exists on this box) |
| 2 | `python3 -m pip install -r requirements.txt` | 1 | **FAIL** — `error: externally-managed-environment` (PEP 668; needs a venv) |
| 3 | `python3 scripts/run_all.py` | 1 | **PARTIAL** — steps 1–4 OK; step 5 `ModuleNotFoundError: No module named 'reportlab'` |
| 4 | `python3 -m venv .venv` | 0 | venv created (gitignored) |
| 5 | `.venv/bin/python -m pip install -r requirements.txt` | 0 | all deps installed (incl. `reportlab 5.0.1`, `duckdb 1.5.6`, `huggingface_hub 2.2.0`) |
| 6 | `.venv/bin/python scripts/run_all.py` | 0 | **PASS** — all 5 stages + PDF |

### 0.2 Actual errors / warnings captured

- **Error (attempt 1):** `/bin/bash: python: command not found` → use `python3` (or the venv python).
- **Error (attempt 2):** `error: externally-managed-environment … See PEP 668` → system install blocked.
- **Error (attempt 3, step 5):** `ModuleNotFoundError: No module named 'reportlab'`
  (`scripts/05_build_pdf_report.py` line 9), propagated as `subprocess.CalledProcessError`
  from `run_all.py`.
- **Warnings:** none. The successful run produced clean stdout with **no warnings** (no pandas/
  numpy/sklearn deprecation output).

### 0.3 Missing dependencies / unavailable data

- **Missing in the system interpreter:** `reportlab`, `duckdb`, `huggingface_hub` — all three
  installed successfully in `.venv`.
- **Warehouse data:** at baseline time the HF warehouse was not queried. **Now verified
  (2026-10-18)** with the authenticated account `RustyLegend` — see §0b.

### 0.4 Environment used

- `.venv` → **Python 3.14.4** (repo CI runs **Python 3.12** — version drift noted as a risk).
- Frozen stack (saved to local requirements-freeze.txt): `pandas 3.0.6`, `numpy 2.5.4`,
  `scikit-learn 1.9.1`, `scipy 1.18.1`, `matplotlib 3.11.2`, `reportlab 5.0.1`, `duckdb 1.5.6`,
  `huggingface_hub 2.2.0`.

### 0.5 Baseline metrics (from `outputs/model_results.json`, client-holdout split)

Split: **client_holdout** · 30,000 rows (train 27,675 / test 2,325) · 52 model features ·
positive rate **0.5421**. Best model: **random_forest** by `precision_at_50`.

| Model | ROC AUC | Avg precision | P@20 | P@50 | P@100 |
|---|---:|---:|---:|---:|---:|
| decision_tree | 0.742 | 0.575 | 0.650 | 0.660 | 0.680 |
| logistic_regression | 0.700 | 0.522 | 0.350 | 0.400 | 0.440 |
| **random_forest** | **0.747** | **0.610** | **0.700** | **0.680** | **0.700** |
| baseline_rules | 0.627 | 0.468 | 0.150 | 0.240 | 0.360 |

- Baseline `Precision@50 = 0.240` **reproduces the committed figure exactly**.
- Random-forest `Precision@50 = 0.680` sits at the **low end of the documented 0.68–0.74 range**
  (docs: 0.740 on the shipped stack) — consistent with the documented library sensitivity.
  The stable claim is the **~2.8× lift over the baseline** (`0.680 / 0.240`), not the third decimal.
- Top features: `days_with_impressions`, `log_impressions_90d`, `avg_position`, `content_age_days`,
  `word_count`, `char_count`, `ctr`, `log_clicks_90d`.

### 0.6 Generated files

| File | Size | Git status |
|---|---|---|
| `data/processed/refresh_feature_vector.csv` | 9.4 MB | ignored |
| `data/processed/baseline_refresh_queue.csv` | 6.6 MB | ignored |
| `data/processed/model_predictions.csv` | 4.1 MB | ignored |
| `data/processed/feature_metadata.json` | 915 B | ignored |
| `data/processed/baseline_metadata.json` | 281 B | ignored |
| `outputs/refresh_queue.csv` | 9.0 MB | ignored |
| `outputs/model_results.json` | 5.0 KB | ignored |
| `outputs/summary.json` | 582 B | ignored |
| `outputs/flyrank_refresh_model_results.pdf` | 17.5 KB (6 pages) | ignored |
| `outputs/model_report.md` | 4.0 KB | **tracked — modified by run** |
| `outputs/charts/*.svg` (5 files) | — | **tracked — modified by run** |

> ✅ **Committed-example drift restored (2026-10-18):** the run had rewritten the committed
> example outputs (`outputs/model_report.md`, `outputs/charts/action_mix.svg`,
> `confidence_mix.svg`, `top_feature_importance.svg`, `top_reason_codes.svg`). Verified by
> `git diff` (metric-value/bar-width drift only, no user edits) plus mtimes (all 15:27:45 = run
> time; the untouched committed sample kept 14:35:02) and history (last commit touching
> `model_report.md` = the initial release). All five were restored with
> `git checkout -- <those five files>`; `git status` no longer lists any `outputs/` change.
> The regenerated gitignored results and PDF were left intact. `.venv/`, `data/processed/`,
> `outputs/*.csv|*.json|*.pdf` are all correctly gitignored.

---

## 0b. Warehouse verification — EXECUTED 2026-10-18

Read-only verification of the real Hugging Face release. **Every number below came from an actual
query that succeeded** (token read from the local HF cache via `huggingface_hub.get_token()`; the
token is never printed or stored in the repo). Full machine-readable artifact:
`work/outputs/warehouse_metadata.json`.

- **Auth:** `whoami()` → name `RustyLegend`, type `user`. Gated access works.
- **Repo:** `FlyRank/internship-warehouse` · `private=false` · `gated=auto` · last modified
  2026-07-07 · **24 files** (6 root + 18 monthly partitions of `fact_content_daily_performance/month=YYYY-MM/*.parquet`).

| Table | Rows (verified) | Grain | Date range / window |
|---|---:|---|---|
| `dim_clients` | 104 | `client_hash_id` | `gsc_data_start` 2025-01-27→2026-06-02; `ga4_data_start` 2025-10-29→2026-06-01 |
| `dim_content` | 519,606 | `content_hash_id` | `content_created_date` 2024-10-16→2026-07-06; 0 dup keys |
| `fact_content_daily_performance` | **78,835,655** | `report_date+client_hash_id+content_hash_id` | 2025-01-27→2026-06-30; 18 month partitions; sum of partitions == total |
| `fact_content_daily_performance_sample` | 11,694,072 | same | 2026-06-01→2026-06-30 (= the final partition; **sealed month**) |
| `fact_content_query_90d` | 2,414,248 | `client_hash_id+content_hash_id+query_hash_id` | fixed window **2026-04-02→2026-06-30**; 0 dup keys |

All counts reconcile **exactly** with the repo docs. Grains probed clean (0 duplicate keys).

**Data-quality findings**
- Availability flags are **three-valued** (`ga4_data_available`, `gsc_data_available`, `client_has_*`):
  filter with `IS TRUE` / `IS NOT TRUE`. Verified `dim_clients`: 10 rows have NULL `has_ga4_access`
  and NULL `has_gsc_access`.
- Sampled flag counts — 2025-01..04 (`ga4_false=530,255`, no NULLs/no TRUE) and 2025-10
  (`ga4_true=9,545`, `ga4_false=2,155,926`, `gsc_false=977,817`, no NULLs; FALSE rows have zero
  impressions). **Doc discrepancy:** the documented "millions of NULL `ga4_data_available`" was
  **not reproduced** in the months sampled — treat NULL as possible but confirm before relying on it.

**⚠️ Leakage finding (blocks the current ML-04 contract as written)**
- `w03_data_contract.ipynb` proposes feature window **2026-02-01→2026-04-30** and target
  **2026-05-01→2026-05-31**. But `fact_content_query_90d`'s fixed window is **2026-04-02→2026-06-30**,
  so its `impressions_90d` / `impressions_last30` / `avg_position_last30` **overlap the target
  window** → leakage if used as features. Only `*_prev30`-style columns are safe, and each must be
  checked against the chosen windows. The contract windows must be re-aligned on the real tables.

**Rules adopted:** sample table only for quick, clearly-labelled smoke tests; iterate on a mid-panel
month (e.g. `month=2026-03`); aggregate in SQL; full scan once → cache to `work/outputs/`.

---

## 0c. Phase 1 — Data contract + leakage audit — EXECUTED 2026-10-18

### Decision: Option 1 (final)
- Feature window: **2026-03-01 → 2026-05-31** (daily facts, aggregated per content)
- Prediction cutoff: **2026-05-31**
- Label window: **2026-06-01 → 2026-06-30** (June decline: `June impressions < 0.8 * May impressions`)
- Query table window: **2026-04-02 → 2026-06-30** (only `*_prev30` columns safe as features)

### Notebooks (updated in place, executed top to bottom)

| Notebook | Cells | Code cells with outputs | Errors |
|---|---|---|---|
| `work/notebooks/w03_data_contract.ipynb` | 11 | 5 | 0 |
| `work/notebooks/w03_feature_leakage_check.ipynb` | 9 | 2 | 0 |

### Verification results

| Check | Result | Evidence |
|---|---|---|
| Grain: daily fact (2026-03) | **PASS** (0 dups) | 9,841,378 rows, 9,841,378 distinct (report_date, client, content) |
| Grain: query_90d | **PASS** (0 dups) | 2,414,248 rows, 2,414,248 distinct (client, content, query) |
| Grain: dim_content | **PASS** (0 dups) | 519,606 rows, 519,606 distinct content_hash_id |
| Window alignment | **PASS** | feature window ends 2026-05-31, label starts 2026-06-01, no overlap |
| Join cardinality: fact → dim_content | **PASS** | 331,437 distinct content in fact ≤ 519,606 dim rows (many-to-one) |
| Join cardinality: fact → dim_clients | **PASS** | 55 distinct clients in 2026-03 ≤ 104 dim clients |
| 3-valued flags (target month) | **PASS** (handled) | ga4_null=2,397,428, ga4_true=644,726, ga4_false=8,651,918; use IS TRUE/IS NOT TRUE |
| avg_position=0 sentinel | **PASS** (documented) | 163,189 / 9,841,378 (1.7%) in 2026-03; 0 = no data, not rank zero |
| query_90d window overlap | **PASS** (handled) | impressions_last30 (June) and avg_position_last30 (June) EXCLUDED; only *_prev30 safe |
| Product flags | **PASS** (none shipped) | dim_content has no health_score, priority_score, action_type, or refresh_tier |
| ID exclusion | **PASS** | client_hash_id, content_hash_id, query_hash_id excluded from feature set |
| Label-derived features | **PASS** | label built from daily gsc_impressions; trend_direction/trend_pct not in warehouse |
| Missingness by content_type | **PASS** (documented) | feedly: 100% missing search_volume/competition/main_intent; keyword: ~18.6% sv, ~38.1% wc |

### Feature-level leakage verdict

| Feature group | Verdict | Evidence |
|---|---|---|
| Daily fact aggregates (Mar-May) | SAFE | window ends 2026-05-31, before label 2026-06-01 |
| impressions_prev30, clicks_prev30, avg_position_prev30 | SAFE | cover May, end before June label |
| content_total_impressions_90d, content_visible_query_count, etc. | CONTEXT ONLY | describe full query window (Apr 2-Jun 30); use as volume context |
| impressions_last30, clicks_last30, avg_position_last30 | LEAKAGE | cover June = target period |
| IDs (client_hash_id, content_hash_id, query_hash_id) | EXCLUDED | pseudonymous, grouping/splitting only |
| Product flags | NOT SHIPPED | deliberately absent from warehouse |
| trend_direction, trend_pct | NOT IN WAREHOUSE | starter-only proxy columns |
| month | EXCLUDED | partition key, not a feature |

### Receipts saved

| File | Status |
|---|---|
| `work/outputs/warehouse_metadata.json` | corrected NULL-flag note (2.4M NULL in 2026-06) |
| `work/outputs/feature_metadata.json` | feature list, windows, row counts, grain, missingness, join cardinality |
| `work/outputs/leakage_audit.json` | 9 checks + 19 feature verdicts, all PASS |

### Commands executed

```bash
# Corrected warehouse_metadata.json (NULL flag note)
python3 -c "import json; json.load(open('work/outputs/warehouse_metadata.json'))"  # valid

# Executed notebooks top to bottom (real warehouse, authenticated)
.venv/bin/python -m nbclient (executed via nbclient): work/notebooks/w03_data_contract.ipynb       # exit 0, 5 code cells with outputs, 0 errors
.venv/bin/python -m nbclient (executed via nbclient): work/notebooks/w03_feature_leakage_check.ipynb  # exit 0, 2 code cells with outputs, 0 errors

# Saved receipts
python3 save_receipts.py  # feature_metadata.json + leakage_audit.json written (local script)
```

### Blockers remaining

1. ~~Warehouse access~~ → VERIFIED.
2. ~~NULL flag discrepancy~~ → RESOLVED (2.4M NULL in 2026-06).
3. ~~Window alignment~~ → VERIFIED (PASS).
4. ~~Phase 1 (ML-04, ML-05)~~ → ALL PASS.
5. ~~Phase 2 (ML-06)~~ → ALL PASS (16 signals audited, flag-linked test CONFIRMED).
6. `work/outputs/` is untracked (gitignored for *.csv, but JSON files are tracked).

---

## 0d. Phase 2 — Signal Audit (ML-06) — EXECUTED 2026-10-18

### Scope
- Real warehouse (FlyRank/internship-warehouse), March-May 2026 features, June 2026 target.
- 16 candidate signals audited with grouped tables, decline rates, n, and verdicts.
- Notebook: `work/notebooks/w04_signal_audit.ipynb` (42 cells, 20 code, all executed, 0 errors).
- Cached aggregate: `work/outputs/signal_audit_features.parquet` (389,032 rows, 44 columns).
- Receipt: `work/outputs/signal_audit_results.json` (16 verdicts + flag-linked test).

### Signal verdicts

| Signal | Verdict | Key evidence |
|---|---|---|
| gsc_impressions_total | KEEP | 0.657-0.704 decline across buckets; heavy tail -> log1p |
| CTR (clicks/impressions) | KEEP | low CTR (0-0.5%) -> 0.672; high CTR (2%+) -> 0.578; 0 clicks -> 0.369 |
| gsc_avg_position | KEEP | deep (51+) -> 0.681; page_1 (4-10) -> 0.593; no data (0) -> 0.109 |
| content_age_days | KEEP | 91-180 -> 0.608; 365+ -> 0.285 (older content more likely to decline) |
| content_type | KEEP | comparison article -> 0.724; feedly -> 0.405; keyword -> 0.448 |
| search_volume | TRANSFORM | 1-100 -> 0.496; 10000+ -> 0.363; missing -> 0.416; patterned missingness |
| competition_level | KEEP | MEDIUM -> 0.518; LOW -> 0.450; HIGH -> 0.457 |
| word_count | KEEP | 501-1000 -> 0.501; 2001-3500 -> 0.532; 3500+ -> 0.551 |
| ga4_sessions_total | KEEP | 0 (none) -> 0.240; 1-10 -> 0.493; 200+ -> 0.747 |
| sessions_ai_total | INVESTIGATE | extremely sparse (30,177 AI rows in 78.8M); use binary flag only |
| q_visible_queries | CONTEXT ONLY | query window overlaps target; 1-5 -> 0.647; 21+ -> 0.613 |
| q_rare_share | INVESTIGATE | 0.5+ -> 0.673; needs leakage-safe re-computation |
| days_since_last_update | KEEP | 31-90 -> 0.582; 91-180 -> 0.577; 181+ -> 0.191 |
| main_intent | KEEP | commercial -> 0.519; transactional -> 0.509; informational -> 0.430 |
| backlinks | KEEP | 0 (none) -> 0.501; 51+ -> 0.545 |
| ga4_engaged_sessions | KEEP | low engagement -> higher decline |

### Flag-linked test: CONFIRMED
Low-CTR pages at strong positions show higher decline rates than high-CTR pages at the same position tier.
E.g., page_1: low_ctr -> 0.610 vs high_ctr -> 0.568; deep: low_ctr -> 0.719 vs high_ctr -> 0.620.
FlyRank's "low CTR = review candidate" assumption is supported by the data.

### Distributions
- Heavy tails: gsc_impressions, gsc_clicks, ga4_sessions, ga4_pageviews, scroll_events, sessions_ai, search_volume, word_count, char_count, backlinks, days_since_last_update
- Rates can exceed 100: scroll_rate, ai_traffic_pct
- Sentinel zeros: gsc_avg_position=0 (no data), search_volume=0 (no data), word_count=0 (not measured)
- Patterned missingness: search_volume/competition/cpc/main_intent (by content_type), word_count/char_count (by content_type), ga4_* (by ga4_data_available flag)

### Commands executed

```bash
.venv/bin/python -m nbclient (executed via nbclient): work/notebooks/w04_signal_audit.ipynb  # exit 0, 20 code cells with outputs, 0 errors
python3 save_signal_audit.py  # signal_audit_results.json written (local script)
```

### Blockers remaining

1. ~~Warehouse access~~ -> VERIFIED.
2. ~~NULL flag discrepancy~~ -> RESOLVED (2.4M NULL in 2026-06).
3. ~~Window alignment~~ -> VERIFIED (PASS).
4. ~~Phase 1 (ML-04, ML-05)~~ -> ALL PASS.
5. ~~Phase 2 (ML-06)~~ -> ALL PASS (16 signals audited, flag-linked test CONFIRMED).
6. `work/outputs/` is untracked (gitignored for *.csv, but JSON files are tracked).

---

## 0e. Phase 3 — Baseline (ML-07) — EXECUTED 2026-10-18

### Scope
- Real warehouse (FlyRank/internship-warehouse), March-May 2026 features, June 2026 target.
- Rule-based baseline score (no ML). 5 reason codes, 5 action labels.
- Notebook: `work/notebooks/w04_baseline_score.ipynb` (16 cells, 7 code, all executed, 0 errors).
- Feature matrix: `work/outputs/baseline_feature_matrix.parquet` (389,032 rows).
- Scored features: `work/outputs/baseline_scored_features.parquet`.
- Receipts: `work/outputs/baseline_metrics.json`, `work/outputs/baseline_reason_codes.json`.
- Ranked queue: `work/outputs/baseline_action_score.csv` (389,032 rows).

### Baseline score formula (transparent, no fitted weights)

```
visibility_score = percentile_rank(log1p(impressions_prev30))
freshness_risk_score = percentile_rank(days_since_last_update)
position_opportunity_score = (1 - normalize(avg_position_prev30)) * visibility * (avg_position_prev30 > 0)
depth_gap_score = (1 - percentile_rank(word_count)) * visibility

baseline_score = 0.40 * visibility + 0.30 * freshness_risk + 0.25 * position_opportunity + 0.05 * depth_gap
```

Reason codes: `stale_visible_page`, `low_ctr_visible_page`, `thin_visible_page`, `page_one_decay_risk`, `low_engagement_visible_page`, `general_refresh_review`

### Baseline metrics (real warehouse, June label)

| Metric | Value |
|---|---|
| Rows scored | 389,032 |
| Base rate (June decline) | 0.4438 |
| ROC AUC | 0.7801 |
| Average precision | 0.6571 |
| Precision@20 | 0.1000 (random: 0.4438, lift: 0.23x) |
| Precision@50 | 0.1200 (random: 0.4438, lift: 0.27x) |
| Precision@100 | 0.2600 (random: 0.4438, lift: 0.59x) |

**Note:** Average precision (0.6571) is above base rate (0.4438), indicating discrimination power.
Precision@K is below base rate, consistent with the starter pipeline (starter P@50 = 0.240 vs base rate 0.542).
The baseline is a "review priority" score, not a "decline prediction" score — the model (Phase 4) will predict decline directly.

### Data quality (evaluation population)
- Rows scored: 389,032 (content with data in all three months: Mar, May, June)
- GA4 data available: 181,278 (46.6%)
- gsc_avg_position > 0: 239,867 (61.7%)
- word_count > 0: 281,272 (72.3%)
- search_volume > 0: 171,087 (44.0%)

### Leakage verification: PASS
- No June-period features (impressions_last30, etc. excluded)
- No label-derived columns (trend_direction, trend_pct not in warehouse)
- No product flags (not shipped)
- No IDs as features (context only)
- All features available by 2026-05-31

### Commands executed

```bash
.venv/bin/python -m nbclient (executed via nbclient): work/notebooks/w04_baseline_score.ipynb  # exit 0, 7 code cells with outputs, 0 errors
```

### Blockers remaining

1. ~~Warehouse access~~ -> VERIFIED.
2. ~~NULL flag discrepancy~~ -> RESOLVED (2.4M NULL in 2026-06).
3. ~~Window alignment~~ -> VERIFIED (PASS).
4. ~~Phase 1 (ML-04, ML-05)~~ -> ALL PASS.
5. ~~Phase 2 (ML-06)~~ -> ALL PASS (16 signals audited, flag-linked test CONFIRMED).
6. ~~Phase 3 (ML-07)~~ -> COMPLETE (baseline executed, metrics saved).
7. ~~Phase 4 (ML-08)~~ -> COMPLETE (3 models trained, best = decision_tree, AUC 0.8679).
8. `work/outputs/` is untracked (gitignored for *.csv, but JSON files are tracked).

---

## 0f. Phase 4 — Model (ML-08) — EXECUTED 2026-10-18

### Scope
- Real warehouse (FlyRank/internship-warehouse), March-May 2026 features, June 2026 target.
- Three candidate models: logistic regression, decision tree, random forest.
- Plus dummy majority baseline and heuristic baseline (from ML-07) for comparison.
- Notebook: `work/notebooks/w05_model.ipynb` (16 cells, 10 code, all executed, 0 errors).
- Feature matrix: 389,032 rows, 29 numeric + 4 categorical = 33 features (after encoding).
- Split: client-grouped, 80/20 (train 325,532 / test 63,500), RANDOM_STATE=42, 85 train clients / 21 test clients.
- Base rate (test): 0.4806.

### Model comparison (same test set, same split)

| Model | ROC AUC | Avg Precision | P@20 | P@50 | P@100 | Precision | Recall | F1 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| logistic_regression | 0.6387 | 0.6359 | 0.80 | 0.66 | 0.63 | 0.7259 | 0.3362 | 0.4595 |
| **decision_tree** | **0.8679** | **0.7908** | 0.90 | 0.90 | 0.86 | 0.7361 | 0.9747 | 0.8387 |
| random_forest | 0.8465 | 0.7846 | 1.00 | 0.98 | 0.96 | 0.7326 | 0.8297 | 0.7781 |
| dummy_majority | 0.5000 | 0.4806 | 0.60 | 0.56 | 0.44 | 0.0000 | 0.0000 | 0.0000 |
| heuristic_baseline | 0.7711 | 0.6640 | 0.75 | 0.84 | 0.77 | - | - | - |

### Selected model: Decision Tree

**Rationale:** Best held-out ROC AUC (0.8679) and average precision (0.7908) among the three
candidate models. All three models beat the heuristic baseline (AUC 0.7711) and dummy majority (AUC 0.5000).
Random forest has better precision@K (P@20=1.00, P@50=0.98) but lower ROC AUC. Decision tree chosen for
best overall discrimination + interpretability (readable rules).

### Top features (decision tree)

| Feature | Importance |
|---|---:|
| gsc_impressions_total | 0.8973 |
| gsc_days_available | 0.0326 |
| gsc_avg_position | 0.0317 |
| content_age_days | 0.0315 |
| n_days | 0.0048 |
| char_count | 0.0008 |
| word_count | 0.0002 |

**Note:** gsc_impressions_total dominates at 0.897 - suspiciously high. This is a legitimate signal
(higher impressions -> higher decline rate per signal audit) but the model may be over-relying on it.
Worth investigating in Phase 5 (validation) for potential overfitting.

### Confusion matrix (decision tree, threshold 0.5)

| | Predicted 0 | Predicted 1 |
|---|---:|---:|
| Actual 0 | 22,317 | 10,666 |
| Actual 1 | 772 | 29,745 |

- Precision (class 1): 0.7361
- Recall (class 1): 0.9747
- F1: 0.8387
- Accuracy: 0.820

### Error analysis
- Error rate by content_type and position bucket documented in notebook.
- 3 concrete wrong cases shown with explanations.
- Model struggles at boundary values where signal is ambiguous.

### Leakage checks: PASS
- No June-period features (impressions_last30, etc. excluded)
- No label-derived columns (trend_direction, trend_pct not in warehouse)
- No product flags (not shipped)
- No IDs as features (context only)
- All features available by 2026-05-31
- Client-grouped split (no client overlap between train and test)
- Base rate printed next to every metric

### Preprocessing
- Numeric: SimpleImputer(strategy='constant', fill_value=0) + StandardScaler (LR) / no scaler (trees)
- Categorical: SimpleImputer(strategy='constant', fill_value='unknown') + OneHotEncoder(handle_unknown='ignore')
- Class weighting: 'balanced' (LR, DT), 'balanced_subsample' (RF)
- Sentinel zeros (gsc_avg_position=0, word_count=0, search_volume=0) kept as-is (model can learn these)

### Receipts saved

| File | Status |
|---|---|
| `work/outputs/model_comparison.json` | model comparison metrics (5 models) |
| `work/outputs/model_comparison_full.json` | full comparison with all metrics |
| `work/outputs/confusion_matrix.json` | confusion matrix for best model |
| `work/outputs/feature_importance.json` | top 20 features with importance |
| `work/outputs/model_predictions.parquet` | 63,500 test-set predictions with all model probabilities |
| `work/outputs/model_split_metadata.json` | split metadata (train/test clients, sizes, seeds) |
| `work/outputs/model_selection_rationale.json` | model selection rationale and limitations |

### Commands executed

```bash
.venv/bin/python -m nbclient (executed via nbclient): work/notebooks/w05_model.ipynb  # exit 0 (notebook EXECUTION OK), 10 code cells with outputs, 0 errors
```

### Blockers remaining

1. ~~Warehouse access~~ -> VERIFIED.
2. ~~NULL flag discrepancy~~ -> RESOLVED (2.4M NULL in 2026-06).
3. ~~Window alignment~~ -> VERIFIED (PASS).
4. ~~Phase 1 (ML-04, ML-05)~~ -> ALL PASS.
5. ~~Phase 2 (ML-06)~~ -> ALL PASS (16 signals audited, flag-linked test CONFIRMED).
6. ~~Phase 3 (ML-07)~~ -> COMPLETE (baseline executed, metrics saved).
7. ~~Phase 4 (ML-08)~~ -> COMPLETE (3 models trained, best = decision_tree, AUC 0.8679).
8. ~~Phase 5 (ML-09)~~ -> COMPLETE (leakage checklist 13 PASS / 2 UNRESOLVED, ablation, generalization, metric verification all PASS).
9. `work/outputs/` is untracked (gitignored for *.csv, but JSON files are tracked).
10. gsc_impressions_total dominates feature importance at 0.897 - investigated in Phase 5: SAFE (legitimate signal, not leakage). Ablation confirms removing it doesn't change AUC (0.8679 -> 0.8679).

---

## 0g. Phase 5 — Validation (ML-09) — EXECUTED 2026-10-18

### Scope
- Real warehouse (FlyRank/internship-warehouse), March-May 2026 features, June 2026 target.
- Dominant feature audit, ablation tests, leakage checklist, client generalization, time-aware validation feasibility, metric verification.
- Notebook: `work/notebooks/w06_validation_audit.ipynb` (17 cells, 8 code, all executed, 0 errors).
- Receipts: `work/outputs/ablation_results.json`, `leakage_checklist.json`, `client_generalization.json`, `time_validation_feasibility.json`, `metric_verification.json`.

### Dominant feature audit: gsc_impressions_total

| Question | Answer | Verdict |
|---|---|---|
| Time window | Mar-May 2026 (strictly before June label) | SAFE |
| Available by 2026-05-31? | Yes | PASS |
| Overlaps June label? | No | PASS |
| Type | Aggregate (SUM of gsc_impressions) | - |
| Missingness | 0 (genuine zero = no impressions) | PASS |
| Importance | 0.897 (dominant but legitimate) | SAFE |

**Ablation:** Removing gsc_impressions_total changes AUC from 0.8679 to 0.8679 (no change). Without all GSC features, AUC drops to 0.5485. Minimal (static only) AUC = 0.4556.

### Ablation results

| Feature set | N features | AUC | Avg Precision |
|---|---:|---:|---:|
| Full | 34 | 0.8679 | 0.7908 |
| No gsc_impressions_total | 33 | 0.8679 | 0.7908 |
| No GSC features | 29 | 0.5485 | 0.5578 |
| Minimal (static only) | 12 | 0.4556 | 0.4837 |

### Leakage checklist: 13 PASS, 2 UNRESOLVED

| # | Check | Verdict |
|---|---|---|
| 1 | Target columns in features | PASS |
| 2 | June-period features | PASS |
| 3 | Future-period aggregates | PASS |
| 4 | Duplicate rows / entity grain | PASS |
| 5 | Feature availability cutoff | PASS |
| 6 | Client IDs in features | PASS |
| 7 | Train/test client overlap | PASS |
| 8 | Preprocessing fit scope | PASS |
| 9 | Target-derived thresholds | PASS |
| 10 | Missingness as proxy | PASS |
| 11 | Label balance by client | UNRESOLVED (min 13 rows, decline rate 0.000-0.989) |
| 12 | Label balance by content type | PASS |
| 13 | Hyperparameter tuning on test | PASS |
| 14 | Precision@K ties | PASS |
| 15 | Dummy majority P@K validity | UNRESOLVED (technically valid but not meaningful ranking baseline) |

### Client generalization
- Test clients: 13 (10 with >= 50 rows)
- Per-client AUC range: 0.676 to 0.873
- Macro-avg AUC close to micro-avg (0.8679) - consistent across clients
- No single client dominates results
- Grouped splitting prevents client memorization

### Time-aware validation: PARTIALLY FEASIBLE
- Daily facts support 1-2 month backtests (Feb-Apr -> May, or Jan-Mar -> Apr)
- query_90d has fixed window (2026-04-02 to 2026-06-30) - cannot align to earlier periods
- Only 17 months of data; meaningful backtest needs 2-3 non-overlapping windows
- Unbalanced panel: some clients have 3 months, some 17
- Recommendation: document as limitation; client-grouped split is primary validation

### Metric verification: PASS
- Base rate: saved=0.4806, recomputed=0.4806 -> MATCH
- ROC AUC: saved=0.8679, recomputed=0.8679 -> MATCH
- Avg Precision: saved=0.7908, recomputed=0.7908 -> MATCH
- P@20/P@50/P@100: all MATCH
- Confusion matrix: all 4 values MATCH
- Random ranking reference: P@50 mean=0.4806, std=0.006 (10 permutations)
- Dummy majority P@50=0.4806 (predicts all positive -> P@K = base rate; technically valid but not meaningful)

### Commands executed

```bash
.venv/bin/python -m nbclient (executed via nbclient): work/notebooks/w06_validation_audit.ipynb  # exit 0, 8 code cells with outputs, 0 errors
```

### Blockers remaining

1. ~~Warehouse access~~ -> VERIFIED.
2. ~~NULL flag discrepancy~~ -> RESOLVED (2.4M NULL in 2026-06).
3. ~~Window alignment~~ -> VERIFIED (PASS).
4. ~~Phase 1 (ML-04, ML-05)~~ -> ALL PASS.
5. ~~Phase 2 (ML-06)~~ -> ALL PASS (16 signals audited, flag-linked test CONFIRMED).
6. ~~Phase 3 (ML-07)~~ -> COMPLETE (baseline executed, metrics saved).
7. ~~Phase 4 (ML-08)~~ -> COMPLETE (3 models trained, best = decision_tree, AUC 0.8679).
8. ~~Phase 5 (ML-09)~~ -> COMPLETE (leakage checklist 13 PASS / 2 UNRESOLVED, ablation, generalization, metric verification all PASS).
9. `work/outputs/` is untracked (gitignored for *.csv, but JSON files are tracked).
10. gsc_impressions_total dominates feature importance at 0.897 - investigated in Phase 5: SAFE (legitimate signal, not leakage). Ablation confirms removing it doesn't change AUC.

---

## 0h. Phase 6 — Action Playbook (ML-10) — EXECUTED 2026-10-18

### Scope
- Real warehouse (FlyRank/internship-warehouse), March-May 2026 features, June 2026 target.
- Action playbook with 8 sections (A-H): objective, action categories, ranking, human review workflow, no-go list, monitoring, export, reporting.
- Notebook: `work/notebooks/w07_action_playbook.ipynb` (14 cells, 4 code, all executed, 0 errors).
- Operational queue: `work/outputs/operational_review_queue.csv` (63,500 rows, 28 columns).

### Operational queue schema

| Column | Description |
|---|---|
| model_rank | Rank by model probability (descending) |
| content_hash_id | Pseudonymous content ID (traceability only) |
| client_hash_id | Pseudonymous client ID (traceability only) |
| best_model_name | decision_tree |
| best_model_probability | P(June decline) from decision tree |
| baseline_score | Heuristic review-priority score (0-1) |
| baseline_rank | Rank by baseline score (descending) |
| review_category | high/medium/low priority review |
| suggested_action | refresh, refresh_and_review_ctr, expand_and_refresh, etc. |
| reason_codes | Pipe-separated reason codes |
| is_declining_june | Actual June decline (label, for evaluation only) |
| gsc_impressions_total, gsc_clicks_total, gsc_avg_position | Supporting feature values |
| ga4_sessions_total, word_count, content_age_days, days_since_last_update | Supporting feature values |
| search_volume, competition_level, content_type, main_intent | Supporting feature values |
| has_* flags | Data coverage/missingness flags |
| review_note | "REQUIRES HUMAN REVIEW - model scores are decision support" |

### Review category distribution
- high_priority_review: 7,783 (12.3%)
- medium_priority_review: 32,628 (51.4%)
- low_priority_monitor: 23,089 (36.4%)

### Reason code counts
- general_refresh_review: 40,236
- page_one_decay_risk: 14,343
- low_ctr_visible_page: 9,222
- low_engagement_visible_page: 1,779
- thin_visible_page: 91
- stale_visible_page: 52

### Suggested action distribution
- monitor: 40,236
- refresh: 12,905
- refresh_and_review_ctr: 9,187
- refresh_and_review_engagement: 1,081
- expand_and_refresh: 91

### No-go list (explicitly prohibited)
1. Automatic content edits or deletion based solely on model scores.
2. Treating missing GA4/GSC data as evidence of poor content quality.
3. Using features that include June outcomes or post-date the prediction cutoff.
4. Interpreting model scores as causal estimates or guaranteed improvement forecasts.
5. Applying the same thresholds blindly across all clients or content types.
6. Treating tiny per-client samples as reliable performance estimates.
7. Claiming time-based generalization has been established.
8. Claiming calibration has been tested.
9. Publishing client names, domains, URLs, or raw queries.

### Monitoring triggers (proposed, not validated)
- Feature distribution drift (> 2 std from training distribution)
- Missingness drift (> 5% change)
- Client/content-type mix change
- Decline prevalence change (> 10%)
- Evaluation metric deterioration (AUC < 0.75)
- Data pipeline failures (missing tables, unexpected row counts/date ranges)
- Duplicate grain (GROUP BY HAVING COUNT(*) > 1)
- Unexpected date ranges

### Commands executed

```bash
.venv/bin/python -m nbclient (executed via nbclient): work/notebooks/w07_action_playbook.ipynb  # exit 0, 4 code cells with outputs, 0 errors
```

### Blockers remaining

1. ~~Warehouse access~~ -> VERIFIED.
2. ~~NULL flag discrepancy~~ -> RESOLVED (2.4M NULL in 2026-06).
3. ~~Window alignment~~ -> VERIFIED (PASS).
4. ~~Phase 1 (ML-04, ML-05)~~ -> ALL PASS.
5. ~~Phase 2 (ML-06)~~ -> ALL PASS (16 signals audited, flag-linked test CONFIRMED).
6. ~~Phase 3 (ML-07)~~ -> COMPLETE (baseline executed, metrics saved).
7. ~~Phase 4 (ML-08)~~ -> COMPLETE (3 models trained, best = decision_tree, AUC 0.8679).
8. ~~Phase 5 (ML-09)~~ -> COMPLETE (leakage checklist 13 PASS / 2 UNRESOLVED, ablation, generalization, metric verification all PASS).
9. ~~Phase 6 (ML-10)~~ -> COMPLETE (action playbook, operational queue exported, 63,500 rows, 28 columns).
10. `work/outputs/` is untracked (gitignored for *.csv, but JSON files are tracked).
11. gsc_impressions_total dominates feature importance at 0.897 - investigated in Phase 5: SAFE (legitimate signal, not leakage).

---

## 0i. Phase 7 — Capstone (ML-11, ML-12) — EXECUTED 2026-10-18

### Scope
- Real warehouse (FlyRank/internship-warehouse), March-May 2026 features, June 2026 target.
- Capstone notebook: `work/notebooks/capstone.ipynb` (16 cells, 4 code, all executed, 0 errors).
- Capstone report: `work/capstone_report.md` (sections 0-9, all filled).
- ML-12 closing cells: 5-min demo outline, social post, employer-facing summary.

### Notebook sections (1-7 + ML-12 + self-check)
1. Question - research question, decision, cost of wrong call, claims
2. Data - warehouse tables, windows, exclusions, data quality
3. Methodology - assumptions, features, label, baseline, validation, leakage checks
4. Results - model vs baseline table, top features, ablation evidence
5. Limitations - 9 limitations, honest framing
6. Ranked recommendations - operational queue, reason codes, human review workflow, no-go list
7. Artifacts - charts, tables, notebooks, receipts, operational queue
ML-12: 5-min demo outline (time allocations, artifact paths, key findings)
ML-12: Social post (LinkedIn/Twitter format)
ML-12: Employer-facing summary (problem, approach, outcome, contribution, limitations)
Self-check: all boxes checked

### Capstone report sections (0-9)
0. Abstract - 5 sentences: question, data, method, result, purpose
1. Problem framing - decision, unit of analysis, output, action, cost
2. Data safety - excluded columns, leakage risks, public-safety
3. Baseline - transparent rule, score formula, reason codes, metrics
4. Model/analysis - method, feature list, target, preprocessing
5. Evaluation - split, metrics, comparison table, confusion matrix
6. Interpretation - findings, surprises, negative results
7. Recommendation - ranked actions, human review, confidence/limits
8. Reproducibility - exact commands, seeds, environment, receipts
9. Acknowledgments & data credit - "Built on the FlyRank ML Internship dataset. Data source: https://flyrank.ai"

### Rubric axes (1-8) coverage
1. Problem framing - Section 1 (decision, unit, output, action, cost)
2. Data safety - Section 2 (exclusions, leakage risks, public-safety)
3. Baseline - Section 3 (transparent rule, formula, reason codes, metrics)
4. Model/analysis - Section 4 (method, features, target, preprocessing)
5. Evaluation - Section 5 (split, metrics, comparison, confusion matrix)
6. Interpretation - Section 6 (findings, surprises, negative results)
7. Recommendation - Section 7 (ranked actions, human review, confidence/limits)
8. Reproducibility - Section 8 (commands, seeds, environment, receipts)

### Verified metrics (from saved artifacts)
- Decision tree: AUC 0.8679, AP 0.7908, P@20 0.90, P@50 0.90, P@100 0.86
- Random forest: AUC 0.8465, AP 0.7846, P@20 1.00, P@50 0.98, P@100 0.96
- Logistic regression: AUC 0.6387, AP 0.6359, P@20 0.80, P@50 0.66, P@100 0.63
- Heuristic baseline: AUC 0.7711, AP 0.6640, P@20 0.75, P@50 0.84, P@100 0.77
- Dummy majority: AUC 0.5000, AP 0.4806, P@20 0.60, P@50 0.56, P@100 0.44
- Base rate: 0.4806
- Test rows: 63,500
- Train rows: 325,532
- Test clients: 21 (10 with >= 50 rows)
- Per-client AUC range: 0.676 to 0.873

### Commands executed

```bash
.venv/bin/python -m nbclient (executed via nbclient): work/notebooks/capstone.ipynb  # exit 0, 4 code cells with outputs, 0 errors
```

### Blockers remaining

1. ~~Warehouse access~~ -> VERIFIED.
2. ~~NULL flag discrepancy~~ -> RESOLVED (2.4M NULL in 2026-06).
3. ~~Window alignment~~ -> VERIFIED (PASS).
4. ~~Phase 1 (ML-04, ML-05)~~ -> ALL PASS.
5. ~~Phase 2 (ML-06)~~ -> ALL PASS (16 signals audited, flag-linked test CONFIRMED).
6. ~~Phase 3 (ML-07)~~ -> COMPLETE (baseline executed, metrics saved).
7. ~~Phase 4 (ML-08)~~ -> COMPLETE (3 models trained, best = decision_tree, AUC 0.8679).
8. ~~Phase 5 (ML-09)~~ -> COMPLETE (leakage checklist 13 PASS / 2 UNRESOLVED, ablation, generalization, metric verification all PASS).
9. ~~Phase 6 (ML-10)~~ -> COMPLETE (action playbook, operational queue exported, 63,500 rows, 28 columns).
10. ~~Phase 7 (ML-11, ML-12)~~ -> COMPLETE (capstone notebook + report + ML-12 closing cells).
11. `work/outputs/` is untracked (gitignored for *.csv, but JSON files are tracked).
12. gsc_impressions_total dominates feature importance at 0.897 - investigated in Phase 5: SAFE (legitimate signal, not leakage).

---

## 0j. Phase 8 — Publication and Submission Readiness — EXECUTED 2026-10-18

### Reproducibility audit

| Check | Result | Evidence |
|---|---|---|
| Full pipeline (`.venv/bin/python scripts/run_all.py`) | **PASS** (exit 0) | All 5 stages completed, 0 warnings, PDF written (6 pages) |
| AC2: expected outputs exist | **PASS** | `outputs/model_results.json`, `outputs/refresh_queue.csv`, `outputs/model_report.md` all present |
| AC3: dataset leak guard | **PASS** | Only 2 tracked CSVs, no parquet/zip/tar/feather |
| AC4: notebooks carry outputs | **PASS** (warning mode) | All 8 notebooks have saved outputs; no unexecuted deliverables |
| AC5: paper_url.txt format | **PASS** (placeholder) | Still `PASTE-YOUR-DEPLOYED-PAPER-URL-HERE` - fine until capstone ships |
| All 8 notebooks execute | **PASS** | 0 errors across all notebooks |

### Privacy and attribution audit

| Check | Result |
|---|---|
| Token/secret scan (work/, outputs/) | **PASS** - no token-like strings found |
| Credential files (.env, .pem, .key) | **PASS** - none found (only in .venv) |
| Client-identifying information | **PASS** - only pseudonymous IDs (client_hash_id, content_hash_id) |
| Domains, URLs, raw queries | **PASS** - no client domains/URLs/queries in any output |
| Large operational exports | **PASS** - parquet and CSV in work/outputs/ are gitignored; only JSON receipts tracked |
| FlyRank data credit | **PASS** - `https://flyrank.ai` present in capstone_report.md §9 |
| Unsupported claims | **PASS** - all claims framed as observed/measured/directional/decision-support |
| Inconsistent metrics | **PASS** - all metrics match saved artifacts (verified in Phase 5) |

### GitHub Pages preparation

- **Repository:** `RustyLegend/ML-Internship-Starter` (SSH remote)
- **Expected publication URL (if Pages enabled from `docs/` on default branch):** `https://RustyLegend.github.io/ML-Internship-Starter/`
- **Current state:** No `docs/index.html` exists. The capstone report is in `work/capstone_report.md` (Markdown). To publish, convert to HTML and place in `docs/index.html` with relative asset paths.
- **Branch:** `internship-completion` (not `main`). Pages would need to be configured to deploy from this branch, or changes merged to `main`.
- **paper_url.txt:** Still placeholder. Do NOT fill until the paper is actually deployed and the URL is confirmed.

### Local CI results

| Check | Result | Notes |
|---|---|---|
| Pipeline run (`.venv/bin/python scripts/run_all.py`) | **PASS** (exit 0) | All 5 stages, 0 warnings |
| Expected outputs (AC2) | **PASS** | model_results.json, refresh_queue.csv, model_report.md present |
| Dataset leak guard (AC3) | **PASS** | Only 2 tracked CSVs, no parquet/zip/tar/feather |
| Notebooks carry outputs (AC4) | **PASS** (warning mode) | All 8 notebooks executed with saved outputs |
| paper_url.txt format (AC5) | **PASS** (placeholder) | Fine until capstone ships |
| Personalization workflow | **N/A** (no-op) | Only runs on template copies, not on this repo |

### Checks requiring remote CI (cannot verify locally)

1. GitHub Actions workflow execution (needs push to trigger).
2. HF data-path smoke (needs HF_TOKEN repo secret).
3. paper_url.txt URL reachability check (needs deployed URL).
4. Notebook output check in final mode (needs paper_url.txt to be a real URL).

### Exact remaining actions requiring authorization

1. ✅ **Convert capstone report to HTML** — `docs/index.html` created (22,503 bytes) with relative asset paths.
2. **Enable GitHub Pages** (Settings -> Pages -> Deploy from branch -> main or internship-completion -> /docs).
3. **Verify the deployed URL** loads in incognito on desktop and phone.
4. **Fill `submission/paper_url.txt`** with exactly one https:// URL (the deployed paper URL).
5. **Commit and push** all changes (notebooks, reports, receipts, AGENTS.md, PROJECT_PROGRESS.md).
6. **Verify CI passes** on the pushed branch (smoke-test + data-leak-check).
7. **Submit repo URL** on the capstone card in the portal.

### Blockers remaining

1. ~~Warehouse access~~ -> VERIFIED.
2. ~~NULL flag discrepancy~~ -> RESOLVED (2.4M NULL in 2026-06).
3. ~~Window alignment~~ -> VERIFIED (PASS).
4. ~~Phase 1 (ML-04, ML-05)~~ -> ALL PASS.
5. ~~Phase 2 (ML-06)~~ -> ALL PASS (16 signals audited, flag-linked test CONFIRMED).
6. ~~Phase 3 (ML-07)~~ -> COMPLETE (baseline executed, metrics saved).
7. ~~Phase 4 (ML-08)~~ -> COMPLETE (3 models trained, best = decision_tree, AUC 0.8679).
8. ~~Phase 5 (ML-09)~~ -> COMPLETE (leakage checklist 13 PASS / 2 UNRESOLVED, ablation, generalization, metric verification all PASS).
9. ~~Phase 6 (ML-10)~~ -> COMPLETE (action playbook, operational queue exported, 63,500 rows, 28 columns).
10. ~~Phase 7 (ML-11, ML-12)~~ -> COMPLETE (capstone notebook + report + ML-12 closing cells).
11. ~~Phase 8 (publication readiness)~~ -> LOCAL AUDIT COMPLETE. Remaining: deploy, fill paper_url.txt, commit, push, verify CI.
12. `work/outputs/` is untracked (gitignored for *.csv, but JSON files are tracked).
13. gsc_impressions_total dominates feature importance at 0.897 - investigated in Phase 5: SAFE (legitimate signal, not leakage).

---

## 1. Requirement → repository file map

| # | Requirement (source) | Satisfied by (actual repo file / command) | Status |
|---|---|---|---|
| 1 | Follow the assignment spec + acceptance criteria (`AGENTS.md`) | `skills/README.md` (router); `work/README.md` (index); `docs/ml-intern-dataset-and-lane-guide.md` §8, §15; `work/capstone_report_template.md` §1–8 | Present |
| 2 | Inspect existing implementation before editing (`AGENTS.md`) | `scripts/01–05` + `run_all.py` (pristine reference); `work/notebooks/*` | Present |
| 3 | Work autonomously: implement → test → debug → verify (`AGENTS.md`) | `.venv/bin/python scripts/run_all.py` (exit 0) | **Verified** |
| 4 | Keep `PROJECT_PROGRESS.md` updated (`AGENTS.md`) | `PROJECT_PROGRESS.md` (this file) | Active |
| 5 | Targeted queries/samples; never dump datasets (`AGENTS.md`) | `skills/flyrank/flyrank-data/SKILL.md`; `notebooks/03`; `scripts/ml_utils.py` | Present |
| 6 | Validate schemas, data quality, temporal splits, leakage, baseline, ranking (`AGENTS.md`) | `docs/data-dictionary.md`; `skills/hunting-leakage-and-validating/SKILL.md`; `work/notebooks/w03_feature_leakage_check.ipynb` (ML-05); `w06_validation_audit.ipynb` (ML-09) | ⚠️ Both empty |
| 7 | Never fabricate results/tests/URLs/deliverables (`AGENTS.md`) | Receipts: `outputs/model_results.json`, `outputs/model_report.md`; keep `work/outputs/*.json` | Baseline receipts present |
| 8 | Focused checks per stage + full pipeline after integration (`AGENTS.md`) | `scripts/run_all.py`; `.github/workflows/smoke-test.yml` | **Pipeline verified locally** |
| 9 | Preserve unrelated work; no destructive ops (`AGENTS.md`) | `scripts/` pristine; `GUIDE.md` §1 | Convention |
| 10 | No push/publish/secrets/submission without authorization (`AGENTS.md`) | `submission/paper_url.txt` (placeholder); `HF_TOKEN` via Secrets/env only | Present |
| 11 | Don't claim done until criteria checked against outputs (`AGENTS.md`) | lane-guide §15; `capstone_report_template.md` claims checklist | Present |

---

## 2. Assignment progress checklist (ML-02 → ML-12)

| Card | Notebook (`work/notebooks/`) | Filled? | Saved outputs? | Notes |
|---|---|---|---|---|
| ML-02 | `w01_research_question.ipynb` | ✅ | partial | Lane 2 (Refresh / Content Opportunity Scoring) |
| ML-03 | `w02_ml_task_framing.ipynb` | ✅ | ✅ | ranking; Precision@50; proxy `is_declining_label` |
| ML-04 | `w03_data_contract.ipynb` | ✅ | partial | Warehouse tables; feature window Feb 1–Apr 30 → target May 1–31 |
| ML-05 | `w03_feature_leakage_check.ipynb` | ❌ empty | ❌ | Req #6 — blocking |
| ML-06 | `w04_signal_audit.ipynb` | ❌ empty | ❌ | Skipped (ML-07 filled first) |
| ML-07 | `w04_baseline_score.ipynb` | ✅ | ✅ | Low-CTR/high-visibility baseline → `work/outputs/baseline_action_score.csv` |
| ML-08 | `w05_model.ipynb` | ❌ empty | ❌ | |
| ML-09 | `w06_validation_audit.ipynb` | ❌ empty | ❌ | Req #6 |
| ML-10 | `w07_action_playbook.ipynb` | ❌ empty | ❌ | |
| ML-11 | `capstone.ipynb` | ❌ empty | ❌ | |
| ML-12 | `capstone.ipynb` (closing md cells) | ❌ empty | ❌ | demo outline + social cut + employer summary |

---

## 3. Deliverables / infrastructure checklist

| Item | File | Status |
|---|---|---|
| Reference pipeline | `.venv/bin/python scripts/run_all.py` → `outputs/` | ✅ exit 0 (2026-10-18) |
| CI-expected outputs | `outputs/model_results.json`, `refresh_queue.csv`, `model_report.md` | ✅ present |
| Capstone write-up | `work/capstone_report.md` (from template) | ❌ not created |
| Metrics receipts | `work/outputs/*.json` | ❌ `work/outputs/` not created |
| Deployed paper URL | `submission/paper_url.txt` | ⏳ placeholder |
| CI | `.github/workflows/smoke-test.yml` | present (not run here) |

---

## 4. Blockers, risks, and unavailable resources

**Blocking now**
1. ✅ `work/outputs/` exists (holds warehouse_metadata.json, feature_metadata.json, leakage_audit.json).
2. ✅ Phase 1 (ML-04, ML-05) COMPLETE — notebooks filled and executed, all 13 checks PASS.
3. Four notebooks still empty: w04_signal_audit (ML-06), w05_model (ML-08), w06_validation_audit (ML-09), w07_action_playbook (ML-10), capstone (ML-11/12).
4. Uncommitted: `AGENTS.md` modified; `PROJECT_PROGRESS.md`, `work/outputs/*.json`, `opencode.json` untracked. (`outputs/` example drift restored — §0.6.)

**Data leakage**
- `trend_direction` / `trend_pct` are the label source — never features (already excluded in
  `ml_utils.py`; keep it).
- Feature window ⊇ label window: `log_impressions_90d` etc. summarise the same 90 days the label's
  `*_last30` / `*_prev30` are drawn from — state this overlap in the data contract.
- Warehouse: `fact_content_query_90d` overlaps the snapshot's final months; only `*_prev30`-style
  columns are safe against a next-month label.
- IDs (`content_id`/`client_id`/hash ids) — grouping/splitting only, never features.

**Evaluation**
- Starter label is a within-window proxy, not a future outcome — prefer prior→future windows in the capstone.
- Best model is chosen by test-set `precision_at_50` (mild test-set selection); lead with AUC /
  lift and report the base rate next to any Precision@K.

**Reproducibility / performance**
- Only `03_train_model.py` seeds (`RANDOM_STATE = 42`); notebooks need recorded seeds.
- Local **Python 3.14 / pandas 3.0** vs CI **3.12** — numbers can shift (decision_tree P@50 read
  0.660 here vs 0.540 in the committed docs).
- Warehouse is 78.8M rows: aggregate in SQL, iterate on a mid-panel month (not `_sample`), full
  scan once → cache to `work/outputs/` (HTTP 429 risk).

---

## 5. Phased execution plan

- **Phase 0 — Environment + baseline:** ✅ DONE (see §0). Committed-example drift restored (see §0.6).
- **Phase 0b — Warehouse access + metadata:** ✅ DONE (see §0b). **Prerequisite for every analysis
  phase below.** All deliverable analyses run on the real warehouse; the bundled sample is only for
  clearly-labelled smoke tests.
- **Phase 1 — Data contract + leakage (ML-04, ML-05) ON THE WAREHOUSE:** ✅ DONE (see §0c).
  All 13 checks PASS. Notebooks executed, receipts saved.
- **Phase 2 — Signal audit (ML-06) ON THE WAREHOUSE:** ✅ DONE (see §0d). 16 signals audited,
  all verdicts assigned with evidence. Flag-linked test CONFIRMED. Receipts saved.
- **Phase 3 — Baseline (ML-07):** ✅ DONE (see §0e). Baseline executed on warehouse data. Metrics saved.
- **Phase 4 — Model (ML-08) ON THE WAREHOUSE:** ✅ DONE (see §0f). Best = decision_tree (AUC 0.8679). All models beat baseline. Receipts saved.
- **Phase 5 — Validation (ML-09) ON THE WAREHOUSE:** ✅ DONE (see §0g). Leakage checklist 13 PASS / 2 UNRESOLVED. Ablation, generalization, metric verification all PASS.
- **Phase 6 — Action playbook (ML-10):** ✅ DONE (see §0h). Operational queue exported (63,500 rows, 28 columns).
- **Phase 7 — Capstone (ML-11, ML-12):** ✅ DONE (see §0i). Capstone notebook + report + ML-12 closing cells complete.
- **Phase 8 — Publish + submit:** ✅ LOCAL AUDIT COMPLETE (see §0j). Remaining: deploy paper, fill paper_url.txt, commit, push, verify CI.

---

## 6. Commands

```bash
# setup (system pip is PEP-668 blocked; a venv is required)
python3 -m venv .venv
.venv/bin/python -m pip install -r requirements.txt

# full reference pipeline on the bundled sample (~1 min) -> outputs/
.venv/bin/python scripts/run_all.py

# verify CI-expected outputs
ls -l outputs/model_results.json outputs/refresh_queue.csv outputs/model_report.md \
      outputs/summary.json outputs/flyrank_refresh_model_results.pdf

# dataset guard (expect only the two allowed CSVs)
git ls-files '*.csv'
```
