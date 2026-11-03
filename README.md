# PW Ad Exchange Studio

Enter what you are selling. The tool builds a full-funnel **audience mix** as **one SQL query** for your warehouse (Trino / Athena), shows exactly who is **included and excluded**, builds a **journey** (awareness, then push and in-app, then WhatsApp for clickers and IVR or AI calling for the rest), and recommends **where to place it** across PW assets.

It is a static app: one `index.html`, no build step, no API key, no server. Your data and saved plans stay in the browser. The SQL runs in your own database, never in this app.

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
3. Go to vercel.com, click **Add New, Project**, and import the repository.
4. Set **Framework Preset** to **Other**. Leave **Build Command** and **Output Directory** empty. Click **Deploy**.
5. Every later `git push` to `main` redeploys automatically.

Command-line alternative: `npx vercel` in this folder, then `npx vercel --prod`.

Run locally: open `index.html`, or run `npm start`.

## How to use

1. Type the batch names, keywords or batch IDs you are selling. Optionally pick a **cohort**, which switches on cohort explorers and idle sign-ups. Pick the price band and start date.
2. Click **Build audience mix**. Copy the **Final audience query** exactly as shown, with no quotation marks around it, and run it in your SQL editor.
3. Read the audience mix (each segment has its own inclusion and exclusion), the three-track journey, and the placement table.
4. Optional queries: **counts by segment** (paste the result back to see real numbers), **cross-category insight** (which other batches behave like your buyers), and **save for journey lists** (creates a table that each journey step reads from).

## The audience mix

Each user is counted once, in the first segment that fits. Segment 1 is closest to buying.

| # | Segment | Funnel stage | Built from |
|---|---------|--------------|------------|
| 1 | Payment failed | Failed payment | `transaction_failed` on the product |
| 2 | Reached payment | Intent | `order_summary_details`, `pw_order_summary_page`, `buy_now_clicked` |
| 3 | Lead-form raisers | Intent | `form_visible` where `form_type = 'lead_gen_form'` |
| 4 | Repeat viewers | Interest | `batch_description_page`, `pw_batch_desc_page`, 2+ views or 2+ days |
| 5 | Single-visit viewers | Interest | one view, `batch_banner_click`, `batch_click` |
| 6 | Free to paid | Free learner | `free_batch_enroll_success` on the product family, not paid |
| 7 | Paid in related batches | Cross-sell | `transaction_success_paid` on batches learned as related |
| 8 | Cross-category browsers | Cross-sell | activity on those related batches, no touch on the product |
| 9 | Cohort explorers | Explore | `library_visit`, `test_series_page` (needs a cohort) |
| 10 | Idle sign-ups | Sign-up | `signup_success_<COHORT>`, no tracked activity (needs a cohort) |

Always excluded: anyone who paid for the product in the last 365 days, and users with no phone and no email. Each user is ranked inside a segment by intent depth, estimated time spent, return days, recent trend and cross-category affinity, then labelled Hot, Warm or Cool.

**Cross-category learning.** The query finds the other batches that buyers of your product also engage with far more than average (default: 2x lift, 20+ buyers). It uses them for segments 7 and 8. This is learned from your data, not guessed.

**Time spent** is estimated from the gap between a user's actions (capped at 30 minutes, 30 seconds if the user left). It is not a measured duration.

## Outside the database

YouTube viewers (by video type and by 30, 60 or 90 days), website cohort and blog visitors, and the prioritised dynamic audience are not in the warehouse. The tool lists how to build them and how to upload the SQL result as a custom audience.

## Data map: what to verify once

Table names come from the **Table Names** tab of the master doc (`pw_bq.silver_dbt_<event>`). Open **Data map** in the app to change any table or switch an event off. Please verify these, because they could not be confirmed from the docs:

- Every behaviour and purchase table carries the `batch_id` parameter. Only `form_visible` was confirmed by the sample query.
- `library_visit` and `test_series_page` carry a `cohort_name` parameter.
- `transaction_success_paid` and `free_batch_enroll_success` are the right purchase and free-enrolment tables for batches.
- `all_notification_clicked`, `popup_click` and `banner_click` exist (used only for the click and no-click split).
- `form_visible` is not in the Table Names tab but exists in your warehouse (your sample query uses it).

The Data map has a table-check query: run it, paste the result, and events whose table is missing are switched off.

## Journey and placements

Three tracks, chosen from the segments in the mix:

- **Fast track** (segments 1 to 3): Day 0 WhatsApp, push and in-app modal. Day 1 counsellor or AI call. Day 3 last reminder.
- **Standard track** (segments 4 to 8): Day 0 awareness. Day 3 push and in-app. Day 5 split: WhatsApp for those who clicked, IVR or AI calling for those who did not. Day 7 close. Day 9 last call.
- **Nurture track** (segments 9 and 10): Day 0 awareness, Day 2 engage, Day 4 split, Day 7 masterclass or scholarship test, Day 10 first offer.

Placements come from the **Activities** tab (in-app fixed, dynamic and personalisation, website, paid, teacher promotion, social, calls, programs, offline) and are ranked by the segments in the mix. Each row says the surface, exactly what to do there, which segments it serves, and when.

## Files

- `index.html`: the whole app
- `vercel.json`: static hosting and security headers
- `package.json`: optional, for `npm start`
