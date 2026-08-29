# Olist E-Commerce: Why Are Customers Unhappy?

Analysis of 99,441 Brazilian e-commerce orders (September 2016 – October 2018) from the Olist marketplace, to identify what actually drives customer dissatisfaction: delivery delay, freight cost, order value, product category, seller, or geography.

## Key Findings

- **Delivery delay is the single strongest driver of dissatisfaction.** Orders delivered on time or early average **4.29 stars**; orders delivered late average **2.27 stars** — a gap of exactly 2 stars (Welch's t-test, t = 100.64, p < 0.001). Correlation between delay and review score: **r = -0.267**.
- **The effect scales with how late the order is.** Very early deliveries (7+ days ahead of estimate) average 4.31 stars, on-time/slightly-early orders average 4.17, orders 1–7 days late drop to 2.72, and orders more than a week late collapse to **1.70 stars**.
- **Only 6.7% of delivered orders arrive late**, but that small slice accounts for a disproportionate share of the 1-star reviews (11,424 of 99,918 reviews are 1-star — more than double the 2-star count of 3,151).
- **Freight cost and order value barely matter.** Correlation of freight-as-%-of-order-value with review score is **r = -0.024**; correlation of total order value with review score is **r = -0.035**. Both are close to noise. This is a genuinely surprising negative finding worth stating plainly rather than downplaying.
- **Total delivery time (not just lateness vs. estimate) also matters**: raw delivery time in days has a correlation of **r = -0.334** with review score — actually a stronger relationship than delay-vs-estimate alone. Speed matters even when the estimate itself was hit.
- **Geography compounds the delivery problem.** Northeastern states have by far the worst on-time performance: **Alagoas (AL) is worst at 21.4% of orders delivered late**, followed by Maranhão (MA, 17.4%), Sergipe (SE, 15.2%), Piauí (PI, 13.9%), and Ceará (CE, 13.8%). Compare that to São Paulo (SP), the largest state by order volume (40,494 orders), at just **4.5% late**, or the best performers — Amazonas (AM) and Rondônia (RO) at **~2.8–2.9% late**.
- **Diapers & hygiene is the worst-rated category** at 3.26 stars average (n=39, minimum-count filter applied), followed by office furniture (3.49, n=1,677 — a large enough sample that this one is worth taking seriously), fashion/male clothing (3.64), and fixed telephony (3.68).
- **Books dominate the best-rated categories**: general-interest books (4.45), construction tools (4.44), flowers (4.42), imported books (4.40), and technical books (4.36) are the five highest-rated categories with at least 30 reviews.
- **A small number of sellers account for outsized harm.** Of 887 sellers with 20+ reviews, 64 average below 3.5 stars. The single worst-performing seller (20 reviews) averages 2.10 stars. Separately, looking at raw volume of 1-star reviews rather than averages, one seller alone has generated 323 individual 1-star reviews — worth flagging even though their average may be diluted by high order volume.

## Data

- **Source**: [Olist Brazilian E-Commerce dataset, Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)
- **Scope**: 99,441 total orders; 96,478 delivered (96,478/99,441 = 97.0%); the rest are shipped, canceled, unavailable, invoiced, processing, created, or approved and were excluded from delivery/review analysis since they lack a real completed-delivery experience to rate
- **Time span**: 2016-09-04 to 2018-10-17
- **Overall average review score**: 4.16 / 5
- **Review score distribution**: 1★ = 11,424 · 2★ = 3,151 · 3★ = 8,179 · 4★ = 19,142 · 5★ = 57,328 — note the U-shape: 1-star reviews outnumber 2-star by more than 3-to-1, meaning dissatisfied customers overwhelmingly go straight to the lowest score rather than a middling one
- **58.7% of all reviews have no written comment** — score-based analysis uses all reviews; any future text/NLP analysis only has the remaining 41.3% to work with

## Project Structure

```
olist-customer-satisfaction/
│
├── data/
│   ├── raw/            # original 9 CSVs from Kaggle, not committed to git
│   ├── interim/        # cleaned/translated/aggregated intermediate tables
│   └── processed/      # master_orders.csv, delivered_orders.csv
│
├── notebooks/
│   ├── 01_data_cleaning.ipynb
│   ├── 02_merge_master_table.ipynb
│   ├── 03_eda_delivery_and_reviews.ipynb
│   ├── 04_eda_sellers_and_categories.ipynb
│   ├── 05_eda_geography.ipynb
│   └── 06_summary_dashboard.ipynb
│
├── reports/
│   ├── figures/         # exported PNG/HTML charts
│   └── final_report.md
│
├── requirements.txt
└── README.md
```

## How to Run

1. Clone this repo
2. Download the dataset from [Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) and place the 9 CSVs in `data/raw/`
3. `pip install -r requirements.txt`
4. Run notebooks in order: `01` → `02` → `03` → `04` → `05` → `06`

## Tools

pandas, numpy, matplotlib, seaborn, plotly, scipy, scikit-learn, jupyter

## Methodology Notes

- **Minimum sample sizes were enforced** before ranking anything: categories need ≥30 reviews, sellers need ≥20 reviews, and states need ≥50 orders to appear in the "worst/best" rankings. Without this, a category with 2 orders and one bad review would misleadingly appear as the single worst category in the dataset.
- **`customer_id` is order-level, not person-level.** There are 99,441 unique `customer_id` values but only 96,096 unique `customer_unique_id` values — meaning ~3,345 orders belong to a customer who ordered more than once. Any repeat-customer or cohort analysis must group on `customer_unique_id`.
- **Statistical significance vs. practical significance**: with a sample this large (95,824 orders used in the core delay analysis), even tiny effects become statistically significant. The freight-ratio and order-value correlations (-0.024 and -0.035) are technically non-zero but small enough to be practically negligible — they are reported honestly as "doesn't matter much" findings rather than inflated into a headline.
- **Geolocation was aggregated** from ~1,000,163 raw lat/lng points down to one average coordinate per zip-code prefix before any geographic joins, since the raw table has many points per zip and would otherwise multiply row counts on merge.
- **Reviews were deduplicated** to one row per `order_id` (keeping the earliest) before any merge, since a small number of orders had more than one review row.

## Limitations

- **No text analysis was performed** on the 58.7% of reviews that do have written comments. This leaves a real signal on the table — the *reasons* customers cite in their own words for 1-star reviews likely go beyond delivery alone (product quality, wrong item, customer service), and are not captured by the score-only analysis here.
- **Correlation, not causation.** Late delivery correlating strongly with low review scores does not rule out confounding factors — for example, orders that are late may also disproportionately involve products or sellers that are independently more likely to disappoint, and this analysis does not control for that with a multivariate model.
- **Geographic aggregation loses within-state variation.** A state like São Paulo (40,494 orders) contains both the seller hub cities with near-instant delivery and outlying rural areas — the state-level average masks that spread.
- **Seller-level rankings can be skewed by product mix.** A seller who happens to sell a chronically low-rated category (e.g. office furniture) may look worse than a seller with genuinely worse service, since this analysis does not separate seller effect from category effect.
- **Review timing vs. delivery timing edge cases exist.** A small number of reviews are created before the recorded delivery date (e.g., customer reviewed based on tracking/shipping experience rather than final delivery), which were not specifically filtered out and could add minor noise to the delay-vs-score relationship.
- **The dataset covers 2016–2018 only.** Findings reflect Olist's logistics and marketplace conditions in that window and may not generalize to current operations.

## Recommendations

- Prioritize logistics investment in the Northeast (AL, MA, SE, PI, CE) — these five states have late-delivery rates 3–5x higher than the Southeast hub states, and the delay-to-review-score relationship shows this converts directly into lost satisfaction.
- Audit the 64 sellers averaging below 3.5 stars (of 887 with sufficient review volume) — this is a small, actionable list rather than a systemic seller-quality problem.
- Freight pricing and general price-tier strategy do not need satisfaction-driven adjustment — the data does not support the intuition that "expensive shipping" or "expensive orders" upset customers.
- Investigate the diapers/hygiene and office furniture categories specifically — office furniture in particular has enough volume (1,677 reviews) that its 3.49 average is not noise, and likely reflects a real product or fulfillment issue (damage in transit, sizing/fit mismatch expectations) worth a follow-up.

## Suggested Extensions

- **NLP on review text**: sentiment/keyword analysis on the 41.3% of reviews with written comments, focused on 1–2 star reviews, to surface complaint themes beyond delivery
- **Multivariate model**: a regression or random forest predicting review score from delay, freight ratio, category, seller, and state simultaneously, to isolate each factor's independent effect rather than relying on separate bivariate correlations
- **Repeat-customer cohort analysis**: using `customer_unique_id`, compare 1st-order vs. 2nd-order review scores for the ~3,345 customers who ordered more than once, as an early churn signal
- **Interactive dashboard**: rebuild the core charts in Streamlit or Plotly Dash for a clickable, filterable version of this analysis