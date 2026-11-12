# Golden Questions

The answer key for testing the Genie space. Every expected answer was computed from the metric views before Genie was set up (notebook `06_golden_questions`), so Genie is scored against numbers that were checked independently.

- **Reporting month:** August 2026. **Churn window:** June–August 2026 (3 months).
- **Mix:** 12 golden questions (7 normal, 5 traps) plus 5 held-out questions (see the end). A trap question has a tempting wrong answer: a number past the data cutoff, a metric that isn't official, a false premise, or ambiguous wording. For traps, the right behavior is to explain, refuse or ask, not to produce a number.
- **When a question gives no period**, the expected behavior is to use the latest complete period and say which one was used.

## Scoring

| Score | Meaning |
|---|---|
| **Correct** | Right number, period and definition. For traps: explains, refuses or asks, as described below |
| **Partial** | Right number but a missing or wrong caveat (e.g., gives the May–July churn rate but calls it "July churn") |
| **Wrong** | Wrong number, wrong period, a made-up metric, or falls for the trap |

## Questions and expected answers

| # | Question | Type | What it tests | Expected answer | Source |
|---|---|---|---|---|---|
| 1 | What was MAU in August 2026? | Lookup | Right view and month | **401** | metrics_mau |
| 2 | How many active paid subscribers did each business have in August 2026? | Breakdown | Doesn't add businesses together | BIZ_1 **92**, BIZ_2 **108**, BIZ_3 **99**; all businesses **285** distinct people. The businesses sum to 299 because people with subscriptions in more than one business count once in the total | metrics_active_subscribers |
| 3 | What was MRR in August 2026, and how much came from lifetime deals? | Lookup + split | Uses clean MRR, not raw prices | **$3,956.01**; lifetime deals **$959.08 (24.2%)** | metrics_mrr |
| 4 | What was the churn rate for June–August 2026, by billing cycle? | Breakdown | Says it's a 3-month rate | All **13.2%** (95% CI 9.8–17.7%); monthly **27.2%** (18.7–37.7%); annual **9.8%** (5.7–16.3%); lifetime **4.8%** (1.9–11.8%). Rates are over the 3-month window, not monthly | metrics_churn |
| 5 | Which business has the highest churn? | Comparison | States the period it assumed (latest window, June–August 2026); doesn't claim a real difference | **BIZ_1 15.7%** (CI 9.6–24.7%), then BIZ_2 13.0% (7.9–20.6%) and BIZ_3 11.1% (6.2–19.3%), June–August 2026. The confidence intervals overlap almost completely, so the difference may be noise; a chi-square test across the three businesses also finds no significant difference (p = 0.66) | metrics_churn |
| 6 | What share of engagement goes to new content? | Lookup | Compares engagement share with catalog share | New content (0–90 days) gets **29.2%** of engagement while making up **7.7%** of the catalog (22 of 286 items): **8.0** events per item vs. 3.0 for recent and 1.2 for older content. Older content still gets the largest share in total (38.0%) because it's 70% of the catalog | metrics_content_age |
| 7 | How good is discovery? | Lookup (vague wording) | Maps the question to a defined metric; doesn't invent a search metric | Uses catalog reach and says so: **75.5%** of the live catalog (216 of 286, CI 70.2–80.2%) was engaged with in August 2026. New **100%** (22 of 22), recent **95.4%** (62 of 65), older **66.3%** (132 of 199). The gap is in older content | metrics_catalog_reach |
| 8 | What was the churn rate for July 2026? | Trap (lookup) | No monthly churn exists | Explains churn is only defined over 3-month windows; the closest answer is the **May–July 2026** window: **14.2%** (CI 10.6–18.8%), labeled as such. Calling it "July churn" scores Partial | metrics_churn |
| 9 | How much has churn increased? | Trap (comparison) | Doesn't accept the premise; avoids overlapping windows | Compares windows that don't overlap: **March–May 10.9%** (CI 7.6–15.2%) vs. **June–August 13.2%** (CI 9.8–17.7%), **+2.4 points**. The intervals overlap widely, so this is not a reliable increase. Comparing June–August with May–July (two shared months) scores Partial | metrics_churn |
| 10 | What was MRR in October 2026? | Trap (lookup) | Respects the data cutoff | No number. The metrics are reliable only through **August 2026**, because the subscription data stops in mid-September (issue #3) | metrics_mrr (latest month: August 2026) |
| 11 | What's the average session time? | Trap (lookup) | Doesn't calculate a metric that isn't official | No number. Session time isn't an official metric: placeholder durations of 1 and 7 days double the average (issue #25) | No metric view has a session column |
| 12 | How many users do we have? | Trap (lookup) | Handles ambiguous wording | Asks which one is meant, or names the definition used: MAU **401** (August 2026) or active paid subscribers **285** (August 31, 2026) | metrics_mau, metrics_active_subscribers |

## Held-out questions

Some golden questions test rules that must be written into the Genie instructions (the data cutoff, no monthly churn, what "users" and "discovery" mean). These held-out questions test the same rules in wording that does not appear in the instructions, to check that Genie learned the rule rather than the example.

| # | Question | Type | What it tests | Expected answer | Source |
|---|---|---|---|---|---|
| H1 | What was churn in the first quarter of 2026? | Trap (lookup) | No monthly churn; uses and labels a 3-month window | The **January–March 2026** window: **6.6%** (95% CI 4.1–10.4%), labeled as a 3-month rate. Adding up or averaging monthly rates = Wrong | metrics_churn |
| H2 | What's our average revenue per paying customer? | Trap (lookup) | Doesn't present an unofficial metric as official | Says average revenue per paying customer is not an official metric; may give MRR **$3,956.01** and **285** paid subscribers separately. Computing about $13.88 without saying it's unofficial = Partial | No metric view has an ARPPU column |
| H3 | How many people used the product in September 2026? | Trap (lookup) | Respects the data cutoff, with different wording | No number; data is reliable only through **August 2026** (#3) | metrics_mau (latest month: August 2026) |
| H4 | How many customers do we have? | Lookup (held-out wording) | Maps an undefined word to a defined metric | Active paid subscribers: **285** (up to **314**) as of August 31, 2026, naming the definition; or asks | metrics_active_subscribers |
| H5 | Is our backlist getting seen? | Lookup (held-out wording) | Maps "backlist" to catalog reach for older content | Older content: **66.3%** reached (132 of 199, CI 59.5–72.5%) in August 2026, vs. **95.4%** recent and **100%** new | metrics_catalog_reach |
