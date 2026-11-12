# Metric Definitions

Single source of truth for every metric in the dashboard, the metric views and the Genie space. If a number doesn't follow these definitions, it's wrong. Issue numbers (#) refer to `issue_log.md`.

## Rules that apply to every metric

- **Layers:** raw (`exercise_*`, never changed) → clean (`clean_*`, with `quarantine_*` for removed rows) → metrics (`metrics_*` views, one per metric).
- **Source:** each metric view is built from clean tables only, following this file. Raw and quarantine tables are never used for metrics.
- **Consumers:** the dashboard and the Genie space read the `metrics_*` views. Clean tables are used only for drill-downs that a metric view doesn't cover.
- **Reporting month:** August 2026 is the latest complete month in every table. September 2026 is incomplete and later months are almost empty (#3). Headline numbers are for August 2026 or as of August 31, 2026.
- **Trend start:** trend charts start in January 2023; earlier months have too little activity to be meaningful.
- **Time:** months are calendar months based on the event or subscription date (UTC).
- **Always excluded:** test accounts and all their activity (#14), duplicate rows (#11), invalid event types (#21), and records quarantined for impossible dates (#6, #15).
- **Test content (#4):** the 8 test content items are removed from the catalog and from all content metrics. Regular users' events on them are kept and flagged `on_test_content`: they count for MAU (the user activity is real) but not for content metrics.
- **Business attribution:** events have no business of their own. They're assigned to a business through their content. Events without content (search, page views, subscription events) can't be assigned to a business.
- **Segments with bad values:** missing or invalid age band, acquisition channel and device are shown as an explicit `Unknown` group, never dropped (#12, #13, #22).
- **Subscription status is a current snapshot, not history.** Point-in-time subscription metrics as of August 31, 2026 are reliable; historical subscription trends are approximate.

## Shared definitions

- **Regular user:** a user in `clean_users` (test accounts and duplicate rows removed).
- **Qualifying event (for MAU):** `content_open`, `content_complete`, `like`, `share`, `comment_post`, `download`, `search_query`.
  Not qualifying: `page_view` (passive), `subscription_start` and `subscription_cancel` (account events, not product use; also unreliable, #24).
- **Content event (for content metrics):** a qualifying event with a content item: `content_open`, `content_complete`, `like`, `share`, `comment_post`, `download`. Events flagged `on_test_content` are excluded (#4).
- **Live catalog on a date:** clean content items published on or before that date. Assumption: items stay available after publishing (there are no removal dates).
- **Content age at an event:** days between the content's publish date and the event date.
  - New: 0–90 days
  - Recent: 91–365 days
  - Older: 366+ days
- **Active subscription on a date D:** start date on or before D; no end date, or an end date after D; current status is not `paused`, `trialing` or `unknown` (#16, #19). Free plans are subscriptions at $0.
- **Current subscription:** when a user has more than one active subscription to the same business, only the most recent active one counts (#20).
- **Paid subscription:** plan tier other than `Free`.

---

## 1. Content engagement share by content age

- **Business question:** Is engagement driven by new content or by older content? (A pointer for where to investigate: content quality vs. discovery.)
- **Grain:** monthly. Headline: August 2026.
- **Definition:** for each age band, content events in that band ÷ all content events in the month.
- **Companion columns (always shown together):**
  - **Catalog share:** live catalog items in the band at month end ÷ all live catalog items at month end.
  - **Engagement per item:** content events in the band ÷ live catalog items in the band at month end.
- **How to read it:** compare engagement share with catalog share. If new content gets less engagement per item than older content, that's a lead to investigate (quality or discovery), not a conclusion.
- **Excluded:** events on test content (#4); events dated before their content was published, including the flagged 1–30 day cases, since their age would be negative (#6).
- **Dimensions:** business, category, content type.
- **Caveats:** only a handful of items are published each month, so the "New" band holds few items and is volatile. Content age is based on publish date, assuming items stay live.
- **Example question:** "In August 2026, what share of engagement went to content published in the last 90 days, and how does that compare with its share of the catalog?"

## 2. Catalog reach

- **Business question:** How much of our catalog actually gets seen? (A visibility measure.)
- **Grain:** monthly. Headline: August 2026.
- **Definition:** live catalog items with at least one content event from a regular user in the month ÷ live catalog items at month end.
- **Dimensions:** business, category, content type, content age at month end (New 0–90 days, Recent 91–365, Older 366+, same bands as metric 1). Runtime within content type was considered but left out: about 15 items per combination is too few for reliable comparisons.
- **Uncertainty:** each reach comes with a 95% Wilson confidence interval; overlapping intervals mean a difference may be chance.
- **Excluded:** test content (#4); events flagged `on_test_content`.
- **Caveats:** assumes published items stay available. Engagement is very evenly spread across items in this dataset, so differences between segments may be small.
- **Example question:** "What percentage of BIZ_1's catalog had at least one engaged user in August 2026?"

## 3. Metadata completeness rate

- **Business question:** How much of our catalog has complete, trustworthy metadata? (A data-health metric for content understanding.)
- **Grain:** snapshot of the live catalog as of August 31, 2026; can be recomputed monthly.
- **Definition:** live catalog items where all required metadata is valid ÷ live catalog items.
- **Required metadata:**
  - valid content type (not `TEST_TYPE`; test items are already excluded from the catalog)
  - category present
  - valid runtime (not a placeholder, #7)
  - creator linked, for the `creator_content` business line only (#1)
- **Companion:** the pass rate for each requirement separately, so the cause of incompleteness is visible.
- **Dimensions:** business, content type.
- **Caveats:** measures whether metadata is present and valid, not whether it is correct (e.g., a wrong category would still pass).
- **Example question:** "What share of the catalog has complete metadata, and which field fails most often?"

## 4. MAU (Monthly Active Users)

- **Business question:** How many real people used the product this month?
- **Grain:** monthly. Headline: August 2026.
- **Definition:** distinct regular users with at least one qualifying event in the calendar month.
- **Included:** regular users' events on test content (`on_test_content`, #4), since the user activity is real; events flagged for short date gaps (`event_before_publish_short`, `activity_before_signup`); events with an invalid or long session duration (#25), since duration doesn't affect activity.
- **Excluded:** `page_view`, subscription events and `glitch_in_the_matrix`; test accounts; quarantined events (#6, #15, #21).
- **MAU by business:** distinct users with at least one content event on that business's content. `search_query` can't be assigned to a business, and neither can events on test content (that content is not in the clean catalog), so business MAUs don't add up to total MAU. A user active in two businesses counts in both.
- **Dimensions:** business, device, acquisition channel, age band, country. By device, a user can appear under more than one device, so device counts don't add up either.
- **Caveats:** subscription status plays no role in MAU; free, paying and canceled users all count if they used the product.
- **Example question:** "What was MAU in August 2026, and how did it change from July?"

## 5. Active subscribers

- **Business question:** How many customers are paying for each business right now?
- **Grain:** point in time, as of month end. Headline: as of August 31, 2026.
- **Definition:** distinct regular users with at least one active **paid** subscription to the business on the date.
- **Total across businesses ("All businesses"):** distinct paying people (someone paying for two businesses counts once), consistent with MAU. Business counts therefore do not add up to the total.
- **Companion:** active Free-plan users, reported separately.
- **Excluded:** test accounts; subscriptions with an unknown status (#16); paused and trialing subscriptions, including stale trials (#19).
- **Reported as a range:** up to +15 if the paid test-status subscriptions are active (#16); up to +25 if paid stale trials converted (#19). Both are upper bounds, since some of these users may already be counted through another active subscription.
- **Dimensions:** business, plan tier, billing cycle.
- **Caveats:** status is a current snapshot, so historical counts are approximate. Some statuses may be out of date (#19, #24).
- **Example question:** "How many active paid subscribers did BIZ_3 have at the end of August 2026?"

## 6. MRR (Monthly Recurring Revenue)

- **Business question:** How much recurring revenue do we have right now?
- **Grain:** point in time, as of month end. Headline: as of August 31, 2026.
- **Definition:** sum of `monthly_price_usd` of each user's current active subscription, per business, on the date.
- **Prices:** placeholder prices (-9.99, 999.99) are replaced with the standard price for the plan and billing cycle (#18). `monthly_price_usd` is already a monthly amount for every billing cycle, including annual.
- **Lifetime deals:** included at their listed monthly price, treated as a price locked for life (#17). `billing_cycle` is available as a dimension so MRR can be shown with and without them.
- **Excluded:** test accounts; superseded overlapping subscriptions (#20); unknown status (#16); paused and trialing subscriptions (#19). Free plans contribute $0.
- **Reported as a range:** up to about +$166/month for the test-status subscriptions (#16) and up to +$354/month for paid stale trials (#19).
- **Dimensions:** business, plan tier, billing cycle.
- **Caveats:** listed price, not collected revenue (there is no payment data). Status is a current snapshot, so historical MRR is approximate.
- **Example question:** "What was MRR at the end of August 2026, and how much of it comes from lifetime deals?"

## 7. Subscription churn rate (trailing 3 months)

- **Business question:** What share of paying customers are we losing?
- **Grain:** trailing 3 months, updated monthly. Headline: the 3 months ending August 31, 2026 (June–August 2026).
- **Definition:** for each business, users with an active paid subscription on the first day of the window who have none on the last day because their subscription ended during the window ÷ users with an active paid subscription on the first day of the window.
- **Not counted as churn:** a user who switched plan or billing cycle and still has an active paid subscription at the end of the window; new subscribers who joined during the window (not in the starting group).
- **Source:** the subscriptions table only (end dates). Cancellation events are not used (#24).
- **Excluded:** test accounts; unknown status (#16); paused and trialing subscriptions (#19).
- **Dimensions:** business, billing cycle, plan tier.
- **Caveats:** small numbers per business, hence the 3-month window. Churn may be understated if some statuses were never updated (#19, #24).
- **Example question:** "What was the churn rate for monthly-billed subscriptions in the 3 months ending August 2026?"

## 8. Data-health summary per table

- **Business question (for engineering):** How much of each table is usable, how much was removed or flagged, and why?
- **Grain:** one row per table, as of the latest data load; recomputed every time the clean layer is rebuilt.
- **Definition (one row per table):**
  - **Raw rows:** rows in the `exercise_*` table.
  - **Clean rows:** rows in the `clean_*` table.
  - **Quarantined rows:** rows in the `quarantine_*` table; **% quarantined** = quarantined ÷ raw.
  - **Flagged rows:** clean rows with a non-empty `dq_flag`; **% flagged** = flagged ÷ clean.
  - **Top reasons:** the most common quarantine reasons and flags, with counts.
  - **Reconciliation check:** raw rows = clean rows + quarantined rows (pass/fail).
- **Tables covered:** businesses, creators, content items, users, subscriptions, engagement events.
- **How to use it:** a reconciliation failure means rows were lost or duplicated during cleaning. A rise in % quarantined or % flagged between loads signals a new data problem upstream (for example, test data leaking into production again). Thresholds for alerts can be set per table.
- **Covers:** test records in production (as quarantine reasons `test_account`, `test_content`) and invalid values (as flags such as `invalid_segment_value`, `invalid_duration`, `price_imputed_from_grid`), so these don't need separate metrics.
- **Caveats:** measures the issues the current checks look for. New kinds of problems need new checks.
- **Example question:** "What share of engagement events were quarantined or flagged, and for what reasons?"

---

## Metrics deliberately not built

| Metric | Why not |
|---|---|
| Content completion rate or open-to-complete funnel | 99.7% of completions have no open event; the rate would be about 480% (#23) |
| Trial conversion rate | Trials never lead to a new subscription record, and trial statuses are stale (#19) |
| Revenue collected | No payment, invoice or renewal data; MRR uses listed prices |
| Search success or content gaps | Search events have no query text and no content |
| Historical status-based metrics (e.g., paused subscribers by month) | Status has no history; only the current value is stored |
| Content popularity rankings | Engagement is too evenly spread across items (9–31 events each); rankings would mostly be noise |
| Average session time | Placeholder durations double it (#25); use the median instead |

---

## Appendix: calculable metrics not built (time limits)

These can be calculated with the clean data, subject to the caveats noted. They weren't built to keep the scope focused; they are the natural next additions.

**Subscriptions and revenue**

| Metric | What it would show | Caveat |
|---|---|---|
| ARPPU (MRR ÷ active paid subscribers) | Average revenue per paying customer, by business and plan | Listed prices, not collected revenue |
| Plan and billing-cycle mix | Share of paid subscriptions by plan tier and billing cycle | Current snapshot only |
| Free vs. paid share | How many active subscriptions are Free | Current snapshot only |
| Engaged subscribers | Active paid subscribers with at least one qualifying event in the month | About 1 in 3 paid subscriptions shows no use of its own business |
| Win-backs | Customers who canceled and later subscribed again to the same business | Small numbers (11 user–business pairs) |

**Engagement**

| Metric | What it would show | Caveat |
|---|---|---|
| Events per active user | Depth of engagement among MAU | Sparse activity (median 7 events per user overall) |
| Median session duration | Typical time spent per session | Use the median; placeholders double the average (#25) |
| Content opens and completes (separate counts) | Volume of each action | Not a funnel or rate; opens are likely under-logged (#23) |
| Engagement by device, channel or age band | Which segments are most active | Show `Unknown` groups explicitly (#12, #13, #22) |
| User activity retention | Share of month-M active users active again in M+1 | Sparse activity, so values will look low |

**Growth and content supply**

| Metric | What it would show | Caveat |
|---|---|---|
| New signups per month | User acquisition, by channel | Excludes test accounts; channel has `Unknown` groups |
| New and active creators per quarter (BIZ_3) | Creator growth and how many creators publish | BIZ_3 only; small numbers |
| Content per active creator (BIZ_3) | Creator productivity (the driver of BIZ_3's content growth) | BIZ_3 only; quarterly |
| Creator engagement share vs. content share (BIZ_3) | Whether a few creators drive engagement or simply publish more | Engagement per item is very even, so differences may be small |

**Data health**

| Metric | What it would show | Caveat |
|---|---|---|
| Data-health trend over time | % quarantined and % flagged per table across loads, with alert thresholds | Requires keeping a history of each load's summary (metric 8 shows the latest load only) |
| Freshness check per table | Latest date in each table vs. the others, to catch tables extracted at different times (#3) | — |
