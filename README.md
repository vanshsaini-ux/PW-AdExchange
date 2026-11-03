# PW Ad Exchange Studio

Click **Build segment**. The tool scans the full PW funnel, picks the best audience to campaign to right now, and gives you **one SQL query** for it. Each user is matched to the batch they are most interested in, so the result is ready for campaigning. Add product details only if you want the audience for one specific product.

It also shows who is included and excluded in each segment, builds the journey (awareness, then push and in-app, then WhatsApp for clickers and IVR or AI calling for the rest), and recommends where to place it on PW assets.

It is a static app: one `index.html`, no build step, no API key, no server. Saved plans stay in your browser. The SQL runs in your own database through Metabase.

## Deploy on Vercel with GitHub

1. Create an empty repository on GitHub, for example `pw-ad-exchange-studio`.
2. In this folder run:
   ```bash
   git init
   git add .
   git commit -m "PW Ad Exchange Studio"
   git branch -M main
   git remote add origin https://github.com/<your-org>/pw-ad-exchange-studio.git
   git push -u origin main
   ```
3. On vercel.com choose **Add New, Project** and import the repository.
4. Set **Framework Preset** to **Other**. Leave Build Command and Output Directory empty. Click **Deploy**.
5. Every later `git push` to `main` redeploys automatically.

## Using it with Metabase

- Open **New, SQL query**, pick your Trino database, paste the query, and run it.
- Copy the query exactly as shown. Do not add quotation marks around it.
- Queries contain no semicolons, no `--` comments and no table creation, so Metabase can run them as they are.
- Download the result as CSV for your campaign tool. Filter by `segment`, `tier` or `batchname` first if you want one slice.
- If Metabase shows an error, open **Data map** (top right) and run the check query there. It names the table that is missing or shows which table has no `batch_id`. Untick that event and build again.

## Auto mode and product mode

- **Auto mode (default, nothing to fill in):** finds every user with a buying signal on any batch, ranks them, and returns the one batch to pitch to each user (`batchname`).
- **Product mode:** type batch names, keywords or batch IDs in the optional box. The audience is built for that product, with cross-category learning from past buyers.
- **Customise (optional):** choose a cohort, switch segments on or off, set limits, add filters or your own SQL condition.

## The audience mix

Each user is counted once, in the first segment that fits. Segment 1 is closest to buying.

| # | Segment | Funnel stage | Built from |
|---|---------|--------------|------------|
| 1 | Payment failed | Failed payment | `transaction_failed` |
| 2 | Reached payment | Intent | `order_summary_details`, `pw_order_summary_page`, `buy_now_clicked` |
| 3 | Lead-form raisers | Intent | `form_visible` where `form_type = 'lead_gen_form'` |
| 4 | Repeat viewers | Interest | `batch_description_page`, `pw_batch_desc_page`, 2+ views or 2+ days, never paid |
| 5 | Single-visit viewers | Interest | one view, `batch_banner_click`, `batch_click`, never paid |
| 6 | Free to paid | Free learner | `free_batch_enroll_success`, never paid |
| 7 | Paid learners, new interest | Cross-sell | paid before (`transaction_success_paid`) and now interested in another batch |
| 8 | Cross-category browsers | Cross-sell | product mode only: active on batches whose buyers behave like buyers of the product |
| 9 | Cohort explorers | Explore | `library_visit`, `test_series_page` (needs a cohort) |
| 10 | Idle sign-ups | Sign-up | `signup_success_<COHORT>` with no activity (needs a cohort) |

Always excluded: anyone who paid for the batch they are interested in (last 365 days), and users with no phone and no email.

Ranking inside a segment uses intent depth, estimated time spent, return days, recent trend and cross-category affinity, then labels users Hot, Warm or Cool. Time spent is estimated from the gap between a user's actions (capped at 30 minutes; 30 seconds if the user left). It is not a measured duration.

## Which cohorts to target

Every user in the audience has a **cohort** column: the cohort of the batch we recommend to them, so one audience can cover several PW cohorts (for example IIT JEE, NEET UG, UPSC, School).

For the ranked view, open **Cohorts to target** in the SQL tabs, run that query in Metabase, and paste the result into the box at the top of the audience card. The tool ranks the cohorts and marks them **Target first**, **Then** or **Later**, using the number of ready-to-buy users first and audience size second. Filter the segment result by `cohort` to pull one cohort at a time.

The cohort is read from the batch name (JEE, NEET, UPSC, SSC, Banking, Railway, GATE, CUET, School and so on). If your batches table has an exam or cohort column, enter it in **Data map** under "Batches: cohort or exam column" and the cohort will be exact.

## Outside the database

YouTube viewers (by video type and by 30, 60 or 90 days), website cohort and blog visitors, and the prioritised dynamic audience are not in the warehouse. The tool lists how to build them and how to upload the SQL result as a custom audience.

## Data map: verify once

Table names come from the Table Names tab of the master doc (`pw_bq.silver_dbt_<event>`). Open **Data map** to change a table or switch an event off. These could not be confirmed from the docs:

- Every behaviour and purchase table carries a `batch_id` parameter (only `form_visible` was confirmed by the sample query).
- `library_visit` and `test_series_page` carry `cohort_name`.
- `transaction_success_paid` and `free_batch_enroll_success` are the right purchase and free-enrolment tables.
- `all_notification_clicked`, `popup_click` and `banner_click` exist (used only for the click and no-click split).
- `form_visible` is not in the Table Names tab but exists in your warehouse.

## Journey and placements

- **Fast track** (segments 1 to 3): Day 0 WhatsApp, push and in-app modal. Day 1 counsellor call. Day 3 last reminder.
- **Standard track** (segments 4 to 8): Day 0 awareness. Day 3 push and in-app. Day 5 split: WhatsApp for those who clicked, IVR or AI calling for those who did not. Day 7 close. Day 9 last call.
- **Nurture track** (segments 9 and 10): Day 0 awareness, Day 2 engage, Day 4 split, Day 7 masterclass or scholarship test, Day 10 first offer.

Each journey list is a complete query: it rebuilds the audience, keeps only the users for that step, and removes anyone who bought since the start date. No saved table is needed.

Placements come from the Activities tab and are ranked by the segments in the mix. Each row gives the surface, exactly what to do there, which segments it serves, and when. In auto mode, messages use `[batch name]`: fill it from the `batchname` column.

## Files

- `index.html`: the whole app
- `vercel.json`: static hosting and security headers
- `package.json`: optional, for `npm start`
