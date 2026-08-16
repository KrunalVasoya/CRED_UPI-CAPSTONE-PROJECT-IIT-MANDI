# CRED — The UPI Growth Story

Himshikhar 2026 Capstone, IIT Mandi — AI-Powered Coding and Analytics Programme

## The Question

Is UPI's growth in India spreading out across more apps, or is it turning into a two-player race between PhonePe and Google Pay? NPCI publishes the monthly numbers but almost nobody actually sits down and works with them properly. That's what we did — 24 months of real data, checked with evidence instead of guessing.

## The Data

NPCI's monthly UPI transaction statistics, app-wise, from **March 2024 to February 2026** (24 months) — total volume (in millions of transactions) and value (in crore rupees) for every app operating on UPI in that window.

This wasn't a clean, model-ready file. We found and fixed two real problems in it ourselves before doing any analysis — see Limitations below.

## Our Approach

1. **SQL** — loaded the data into a proper two-table setup (`apps` + `monthly_app_stats`), and wrote queries for market share over time, month-on-month growth, rankings, and a concentration index (HHI), using window functions and a CTE.
2. **Cleaning** — the raw data had two real issues that would've broken everything downstream if we hadn't caught them (see Limitations). Fixed both, with the reasoning documented in the notebook.
3. **Exploration** — looked at the growth trend, checked for seasonality (festive months), and tracked the concentration curve across the full 24 months.
4. **Modeling** — trained a linear regression to project total volume forward, tested two versions (plain monthly, and 3-month averaged), and picked the one that actually performed better instead of just going with the first one.
5. **Dashboard** — built to answer the core question at a glance: is it broadening or concentrating.
6. **Memo** — one page, direct answer, evidence, and what it means for a challenger like CRED specifically.

## What We Found

- **The market is broadening, not concentrating.** PhonePe + Google Pay's combined share fell from 85.9% to 80.4% over the 24 months. Our concentration index (HHI) fell about 11.5% over the same period, and it kept falling almost every month, not just once.
- The share the top two lost didn't go to one big challenger — it spread out. The "everyone else" bucket more than doubled its combined share, from about 5.0% to 11.6%. Navi and super.money picked up most of it.
- Volume itself keeps growing — roughly 1,124 million transactions more every quarter, based on the better-performing of our two models.
- October shows a real, repeat spike every year (festive season, most likely) — but November doesn't, so the usual "festive months = Oct and Nov" assumption isn't fully right here.
- A real regulatory risk sits just past our forecast window: NPCI's proposed 30% market-share cap on individual apps is set to take effect by end of 2026, which could speed up this broadening trend if it's actually enforced.

Full numbers and reasoning are in [`memo/CRED_UPI_Growth_Story_Memo.pdf`](memo/).

## Limitations

- **July–Aug 2025 volumes were showing almost double what they should.** Traced it to a copy-paste style error in the source file — a column called "On-us" (which is 0 for every app in every other month) had a stray figure copied in from two months earlier, for every app, in just those two months. Fixed it once we found the actual cause, not just patched the number.
- **36 app names turned out to be the same app under a different spelling** (a trailing "#", plus one legal-entity name for Paytm) — merged all of them after confirming each pair never appears in the same month, so it was safe to combine.
- **Our forecast is decent, not perfect.** A real dip in Feb 2026 volume (partly explained by that month having fewer days) is something a straight growth line can't fully predict — so the forecast should be read as a solid estimate, not a promise.
- The 3-month-averaged regression, which performed better, is based on only 8 data points — a real result, but from a small sample.

## Folder Map

| Path | Contents |
|---|---|
| `data/` | Cleaned source data (`CRED_UPI_Data_Final.xlsx`) — Master Data, Monthly Totals, 3-Month Averages |
| `sql/` | Schema + all queries (`cred_sql_queries_only.ipynb`) |
| `notebooks/` | Modeling notebooks (`cred_two_regressions.ipynb`) — both regressions, comparison, forecast |
| `dashboard/` | Tableau Public link (below) + dashboard-source CSVs |
| `memo/` | One-page decision memo (PDF) |
| `ai_appendix/` | Prompt log + judgment note |

## Dashboard

Published Tableau Public link: **[add your link here once published]**

## AI Workflow Appendix

See [`ai_appendix/`](ai_appendix/) — includes the moment an early AI theory about the data ("every number is roughly doubled") turned out to be imprecise, and how we traced it to the actual, specific root cause instead of accepting the first explanation.

## Video Walkthrough

**[add your video link here]** — 3-minute walkthrough of this repo and the dashboard, in our own voice.
