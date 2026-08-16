# Analysis Notebook Guide — CRED UPI Growth Story

*(Completed version — matches the structure of the reference `ANALYSIS_GUIDE.md`, filled in with what we actually did)*

## 1. Load

Pulled the monthly app-stats query results into pandas from `CRED_UPI_Data_Final.xlsx` (Master Data sheet). Confirmed all 24 months (March 2024 – February 2026) are present — no missing months, checked via row counts per month (ranged 62–92 apps/month, no month came back empty).

## 2. Clean

- [x] **Reconciled app-name inconsistencies across months.** Found 36 app names that were the same app under a different spelling — a trailing "#" (35 cases, e.g. `Phone Pe #` → `PhonePe`) plus one legal-entity name for Paytm (`Paytm (OCL)` → `Paytm`). Verified each merge was safe by confirming the variant and canonical name never appear in the same month before combining them. 141 raw labels → 105 real apps.
- [x] **Fixed a unit/duplication inconsistency.** July and August 2025 showed total volume roughly double every other month. Traced it to the `On-us` sub-category column — which is 0.00 for every app in every other month — containing a stray figure copied in from that same app's **May 2025** Customer Initiated volume, for every app, in just those two months. Fixed by recalculating Total as `Customer Initiated + B2C + B2B` for July/Aug 2025 only, dropping the erroneous On-us figure.
- [x] **No missing months or apps to handle** beyond the naming issue above — every app that appears at all has a value (possibly 0) for every month it's expected to report in.

## 3. Explore

- [x] **Overall volume and value trend, full window.** Total volume grew from ~13,461 Mn (Mar 2024) to ~19,963 Mn (Feb 2026), a steady upward trend with one real dip in the final month.
- [x] **Market share trend for top apps, over time.** PhonePe and Google Pay remain the top two throughout, but both are losing share slowly and consistently — see concentration curve below for the combined effect.
- [x] **Seasonality — do festive months spike?** October shows a real, repeat spike both years (+10.2% in 2024, +4.9% in 2025), but November does **not** sustain it (-6.7% and -0.9%) — the common "Oct-Nov both spike" assumption only holds for October. Bonus finding: March 2025 shows the single biggest jump in the whole series (+13.6%), likely India's fiscal year-end effect rather than a festive one.
- [x] **The concentration curve over time.** Falling. HHI declined from 0.383 to 0.339 (−11.5%) across the 24 months, and top-2 combined share fell from 85.9% to 80.4% — both moved in the same direction almost every month, not just once.
- [x] **One sentence per chart:**
  - Volume trend chart → *"UPI volume has grown consistently, with one real dip in the final month."*
  - Market share chart → *"PhonePe and Google Pay dominate throughout, but both are losing ground slowly."*
  - Seasonality chart → *"October spikes every year; November does not — the festive effect is real but narrower than assumed."*
  - Concentration chart → *"The market has been broadening steadily, not concentrating, for the entire window studied."*

## 4. Feature Engineering (for the projection model)

- [x] **Time feature and target decided:** month index (1–24) as the time feature; total transaction **volume** (Mn) as the target, since the strategy question is about transaction growth, not rupee value specifically. Also built a second version using 3-month-averaged volume against a period index (1–8), since it fit the trend better.
- [x] **Structural break noted before modelling:** the July/Aug 2025 anomaly (see Section 2) had to be fixed *before* any feature engineering — modelling on the uncorrected data produced a badly broken fit (test R² around −200) purely from that data error, not from the underlying trend.

## 5. Model

- [x] **Train/test split:** chronological, not random — trained on the earlier months, tested on the most recent ones (19/5 split for monthly, 6/2 for the 3-month-averaged version).
- [x] **Linear regression trained, next quarter projected.** Monthly version: slope ≈ 367 Mn/month. 3-month-averaged version: slope ≈ 1,124 Mn/period. Both project continued growth over the next 3 months (~21,771 / 22,138 / 22,505 Mn for the monthly version).
- [x] **Error reported in units a strategy reader understands:** Monthly — train MAE 373.0, test MAE 530.9 (i.e., "off by about 531 million transactions on average" for unseen months). 3-month-averaged — train MAE 176.3, test MAE 282.3.
- [x] **Overfitting/fit check:** train vs. test error compared for both versions — no signs of severe overfitting in either. The monthly version's test R² is negative (−0.628); the 3-month-averaged version's is positive (+0.576) — the averaged version tracks recent periods noticeably better.
- [x] **Trend coefficient in plain English:** *"UPI total transaction volume has grown by roughly 367 million transactions per month, on average, over the 24-month period studied — or about 1,124 million per 3-month period when smoothed."*

## 6. Honest Limitations (Standout)

- [x] **Specific events named that could break this projection:**
  - **NPCI's proposed 30% market-share cap** on individual UPI apps — delayed twice, currently set to take effect **December 31, 2026**. Mechanism: if enforced, PhonePe (~45-46% share) and Google Pay (~34-38% share) would have to stop onboarding new users, which would likely *accelerate* the broadening trend already found, rather than reverse it — but this sits beyond our 3-month forecast window, so any longer-range projection should flag it explicitly.
  - **The February 2026 dip** — a real, broad-based ~6.2% decline across nearly every major app proportionally, partly (not fully) explained by February having 3 fewer calendar days than January. This is exactly why the monthly model's test R² stays negative — a straight growth line cannot predict a real dip, and this one falls inside a test set of only 5 months.
  - **Small sample size on the better-performing model** — the 3-month-averaged version's positive R² is a genuine result, but rests on only 8 data points (6 train, 2 test). Worth stating plainly as "promising, not proven at scale."
