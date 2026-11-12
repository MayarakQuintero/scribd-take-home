# Scribd Take-Home: AI-Native Self-Serve Analytics

A small data product that lets technical and non-technical employees answer their own questions about the business: audited and cleaned data, eight governed metrics, a dashboard, and a natural-language interface (Databricks Genie) that is grounded in those metrics and tested for hallucinations.

Built on Databricks Free Edition (Unity Catalog, Delta tables, SQL views, AI/BI dashboard, Genie).

## Headline findings (August 2026)

| Metric | Value | Range or 95% CI |
|---|---|---|
| MAU | 401 | — |
| Active paid subscribers | 285 | up to 314 |
| MRR | $3,956.01 | up to $4,441.65 |
| Churn, June–August 2026 (3-month rate) | 13.2% | 9.8%–17.7% |

- **Raw data overstates MRR by 134%.** Raw MRR is $9,245.77; clean MRR is $3,956.01. Placeholder prices of $999.99 explain 90% of the gap.
- **Churn depends on billing cycle, not business.** Monthly-billed customers churned 27.2% over June–August vs. 4.8% for lifetime deals (Fisher p < 0.001). This is an association; how customers get a lifetime deal is unknown. Differences across businesses are not reliable (chi-square p = 0.66).
- **Older content is under-discovered.** 66% of items older than a year were engaged with in August vs. 97% of newer items (Fisher p < 0.001); older items are 67 of the 70 items nobody engaged with. New content is 7.7% of the catalog but gets 29.2% of engagement.
- **Grounding works, but it's iterative.** Genie went from 10 to 14 fully correct answers out of 17 test questions after targeted instructions, with no wrong answers. The gain is concentrated in the questions the fix targeted; held-out questions stayed flat (see [Genie accuracy](#genie-accuracy)).

## How it's built

```
raw (exercise_*)  →  clean (clean_*) + quarantine (quarantine_*)  →  metrics (metrics_* views)  →  dashboard + Genie
     never changed        every raw row lands in exactly one            one view per metric,           read only the
                          of the two; fixes keep the original           built only from clean           metric views
                          value in a *_raw column                       tables
```

1. **Audit** ([notebooks/01_audit.ipynb](notebooks/01_audit.ipynb)): profiled all six tables and logged 25 issues with rows affected, treatment and impact ([docs/issue_log.md](docs/issue_log.md)).
2. **Clean layer** ([02_clean_layer](notebooks/02_clean_layer.ipynb)): test accounts, test content and duplicates quarantined with a reason; invalid values fixed or flagged (`dq_flags`), never silently dropped. 26 automated checks, all passing, including raw = clean + quarantine for every table.
3. **Metrics layer** ([03_metrics_layer](notebooks/03_metrics_layer.ipynb)): one view per metric, by business and month, with every column documented in a table or column comment. Definitions: [metrics/metric_definitions.md](metrics/metric_definitions.md).
4. **Analysis** ([04_ds_analysis](notebooks/04_ds_analysis.ipynb)): significance tests, confidence intervals and power analysis.
5. **Impact** ([05_impact_table](notebooks/05_impact_table.ipynb)): every headline metric computed on raw vs. clean data, plus the MRR bridge from raw to clean.
6. **Golden questions** ([06_golden_questions](notebooks/06_golden_questions.ipynb)): the answer key for testing Genie, with the SQL proving each answer.
7. **Genie accuracy test** ([07_accuracy_test](notebooks/07_accuracy_test.ipynb)): three scored runs, plus the exact instructions and example queries used.

## Data trust: what cleaning changed

| Metric | Raw | Clean | Difference | Impact | Main driver |
|---|---|---|---|---|---|
| MRR | $9,245.77 | $3,956.01 | +133.7% | High | Placeholder prices (#18) |
| Active paid subscribers | 308 | 285 | +8.1% | Medium | Test accounts and test statuses (#14, #16) |
| MAU | 422 | 401 | +5.2% | Medium | Test accounts (#14) |
| Churn (Jun–Aug) | 13.9% | 13.2% | +4.6% | Low | Plan changes vs. test accounts, partly offsetting |
| Catalog reach | 78.2% | 75.5% | +3.6% | Low | Test content (#4) |
| Metadata completeness | 93.9% | 96.5% | −2.7% | Low | Test content counted as invalid items (#4) |

Impact levels: High = 20%+ difference, Medium = 5–20%, Low = under 5% (a judgment call). Issue numbers refer to [docs/issue_log.md](docs/issue_log.md).

Where the data can't settle a question, numbers are reported as ranges rather than guesses: subscriptions with an unreliable status (a test status on real accounts, trials that never ended) are left out of the headline numbers and added back in the upper bound.

## Metrics

Eight governed metrics, each with a numerator, denominator, exclusions and caveats in [metrics/metric_definitions.md](metrics/metric_definitions.md):

| # | Metric | View |
|---|---|---|
| 1 | Content engagement share by content age | `metrics_content_age` |
| 2 | Catalog reach (with Wilson 95% CIs) | `metrics_catalog_reach` |
| 3 | Metadata completeness | `metrics_metadata_completeness` |
| 4 | MAU | `metrics_mau` |
| 5 | Active paid subscribers (with upper bound) | `metrics_active_subscribers` |
| 6 | MRR (with upper bound) | `metrics_mrr` |
| 7 | Churn, trailing 3 months (with Wilson 95% CIs) | `metrics_churn` |
| 8 | Data-health summary per table | `metrics_data_health`, `metrics_data_health_reasons` |

Supporting views: `metrics_headline` (August 2026 summary), `metrics_impact` and `metrics_mrr_bridge` (raw vs. clean).

The definitions file also lists metrics deliberately **not** built and why (for example, content completion rate: 99.7% of completions have no open event, so the rate would exceed 100%).

## Dashboard

Three pages, all reading the metric views, so the dashboard and Genie always agree:

- **Overview:** headline tiles with ranges, trends, and churn by non-overlapping 3-month windows.
- **Content:** engagement share vs. catalog share, catalog reach by category (with error bars) and by content age over time.
- **Data trust:** raw vs. clean impact table, MRR waterfall, row reconciliation and quarantine reasons.

Screenshots and an export are in [dashboard/](dashboard/). The live dashboard runs in the Databricks workspace.

## Genie accuracy

Genie sees only the 12 metric views (no raw, clean or eval tables), so it can't invent its own version of a metric by aggregating rows. It was tested on 17 questions: 12 golden questions (including 5 traps, such as asking for monthly churn, data past the cutoff, or a metric that isn't defined) and 5 held-out questions that test the same rules in different words. Each question was asked in a new chat, in Chat mode, and scored Correct, Partial or Wrong against [eval/golden_questions.md](eval/golden_questions.md).

| | Baseline (views only) | Run 2 (instructions) | Run 3 (targeted fix) |
|---|---|---|---|
| All 17 | 10 C · 6 P · 1 W | 10 C · 6 P · 1 W | **14 C · 3 P · 0 W** |
| Traps (8) | 5 · 2 · 1 | 6 · 2 · 0 | 7 · 1 · 0 |
| Normal (9) | 5 · 4 · 0 | 4 · 4 · 1 | 7 · 2 · 0 |
| Golden (12), correct | 8 | 7 | 11 |
| Held-out (5), correct | 2 | 3 | 3 |

- **Baseline:** the view and column comments alone got most numbers right; failures were judgment errors (accepting a false premise, comparing overlapping windows, presenting an unofficial ratio as a metric).
- **Run 2:** instructions fixed the traps, but trimming them to avoid Genie's length warning dropped context, and normal questions regressed.
- **Run 3:** a targeted fix for each failure recovered the normal questions without breaking the traps.
- **Caveat:** the gain is concentrated in the golden questions the fix was written for, and the held-out questions stayed flat, so part of the improvement is fitting the test set. With 17 questions and some run-to-run variation, the pattern matters more than the totals. The honest claim is "the fixes work on these failure types," not "Genie is 82% accurate."

## Key assumptions and caveats

- **Reporting month is August 2026.** Users, subscriptions and content stop in mid-September; later months would show false drops.
- **Subscription status is a current snapshot.** Point-in-time numbers as of August 31, 2026 are reliable; historical subscription trends are approximate.
- **MRR uses listed prices**, not collected revenue (no payment data). Lifetime deals are included at their monthly price, treated as locked for life.
- **Small numbers.** About 290 paying customers means wide intervals: detecting a 3-point drop in churn would need about 1,800 customers per group, about 13 times today's base.
- **Associations, not causes.** None of the comparisons are experiments.

## What I'd do next

- Automate the clean-layer checks and alert when % quarantined or % flagged jumps between loads.
- Keep a status history for subscriptions, so historical subscriber and MRR numbers stop being approximate.
- Ask engineering to log search queries, result counts and clicks, so discovery can be measured directly.
- Re-run the Genie eval after every instruction change, with new held-out questions and Genie Benchmarks for the number questions.

## Repository contents

```
├── README.md
├── docs/
│   └── issue_log.md                25 data-quality issues: check, rows affected, treatment, impact
├── metrics/
│   └── metric_definitions.md       single source of truth for every metric
├── eval/
│   └── golden_questions.md         Genie answer key: 12 golden + 5 held-out questions
├── notebooks/                      01_audit … 07_accuracy_test (with cell outputs)
└── dashboard/                      screenshots and export
```

The raw dataset is not included; it lives in the Databricks workspace (`workspace.scribd_exercise`).
