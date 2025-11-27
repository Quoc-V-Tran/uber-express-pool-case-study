# Uber Express Pool: Wait Time Experiment

This project analyzes data from a switchback A/B test on Uber Express Pool, comparing a 2-minute wait time (control) vs a 5-minute wait time (treatment). The goal is to understand how longer wait times affect:

- Driver payout (Uber cost)
- Trip volume and matches
- Rider cancellations
- Profitability across rush-hour vs non–rush-hour periods

The analysis started as a UCLA Anderson case study and was turned into a reproducible workflow.

## Project Questions

1. Was the experiment properly randomized across rush-hour and non–rush-hour periods?
2. How does moving from a 2-minute to 5-minute wait change driver payout, trips, cancellations, matches, revenue, and profit?
3. Should Uber use different wait times during rush hour vs off-peak?

## Data and Derived Metrics

The dataset includes, among other fields:

- `treatment` – 0 for 2-minute wait, 1 for 5-minute wait
- `commute_period` – rush-hour vs non–rush-hour indicator
- `trips_pool`, `trips_express`
- `total_driver_payout`
- `rider_cancellations`
- `total_matches`

To translate trips and payouts into economics, I created:

- `total_revenue` using the case assumptions:
  - Pool trips ≈ \$15 each
  - Express trips ≈ \$9 each
  - `total_revenue = 15 * trips_pool + 9 * trips_express`
- `profit = total_revenue - total_driver_payout`

## Methods

### 1. Randomization checks

Because this is a switchback experiment, it is important to confirm that treatment is balanced across commute types.

- Histograms and summary tables show that roughly 51.7% of non–rush-hour periods received the 5-minute treatment, which is close to a 50/50 split.
- A linear regression of `treatment ~ commute_period` produces a statistically insignificant coefficient on `commute_period` (p = 0.881), indicating that rush-hour vs non–rush-hour does not predict treatment assignment.

This supports treating the experiment as properly randomized across commute types.

### 2. Treatment-effect models

I then ran linear regressions of various outcomes on the 5-minute treatment indicator, first without and then with an interaction for commute period. Outcomes include:

- `total_driver_payout`
- `trips_express`
- `rider_cancellations`
- `total_matches`
- `total_revenue`
- `profit`

The baseline (2-minute control) outcomes per testing period are approximately:

- Total driver payout ≈ \$31,047
- Express trips ≈ 2,718
- Rider cancellations ≈ 169
- Total matches ≈ 2,757

Under the 5-minute treatment, holding other factors fixed:

- Total driver payout decreased by about \$3,163
- Express trips decreased by about 315
- Rider cancellations increased by about 29
- Total matches decreased by about 389

These results align with the case narrative: longer waits reduce cost per ride via fuller vehicle utilization, but at the cost of more cancellations and fewer matches.

### 3. Rush-hour vs non–rush-hour (interaction models)

To explicitly separate rush-hour from non–rush-hour effects, I modeled each outcome as a function of `treatment`, `commute_period`, and their interaction.

Key results:

**Total Uber Cost (`total_driver_payout`)**

- Constant ≈ \$28,058 represents expected cost for a 2-minute wait during non–rush-hour.
- During non–rush-hour, moving to the 5-minute wait reduces cost by about \$3,077.
- Holding wait time at 2 minutes, rush hour increases cost by about \$12,879.
- The interaction term is not statistically significant, so the cost effect of 5-minute waits does not differ strongly between commute types.

**Total Revenue (`total_revenue`)**

- Constant ≈ \$44,403 represents expected revenue under a 2-minute wait.
- Rush hour increases total revenue by about \$11,298.
- The 5-minute treatment and its interaction with commute period are not statistically significant, suggesting no clear revenue effect from longer waits.

**Profit (`profit`)**

- Constant ≈ \$16,346 represents profit for a 2-minute wait in non–rush-hour periods.
- In non–rush-hour, moving to a 5-minute wait increases profit by about \$1,813.
- Holding the 2-minute wait fixed, rush hour decreases profit by about \$1,581.
- The interaction between 5-minute treatment and rush-hour is statistically significant at about –\$2,336.
- Overall, a 5-minute wait during rush hour reduces baseline profit by roughly \$2,104 per testing period.

## Recommendation

**Non–rush-hour**

- The 5-minute wait improves profitability by roughly \$1.8k per testing period.
- Riders and drivers are more tolerant of longer waits when they are not commuting, allowing better pooling and lower cost per ride.

**Rush-hour**

- The 5-minute wait hurts profitability by about \$2.1k per testing period, alongside fewer matches and more cancellations.
- Riders and drivers are more time-sensitive in heavy traffic, so longer waits reduce both utilization and experience.

**Recommended policy:** adopt a dynamic wait-time strategy:

- Use 5-minute waits during non–rush-hour periods to capture pooling efficiencies.
- Maintain 2-minute waits during rush-hour periods to protect profit and rider experience.

A natural extension is to further model `rider_cancellations ~ treatment * commute_period` and attach an explicit dollar value to cancellation-driven revenue loss.

## Repository structure (example)

- `data/`: cleaned case data (or a synthetic sample)
- `R/`: scripts for randomization checks, revenue/profit construction, and regression models
- `notebooks/`: an Rmd notebook with the full analysis
- `README.md`: this file

## How to run

1. Clone the repo and open it in RStudio.
2. Install required packages such as `tidyverse`, `broom`, and `ggplot2`.
3. Run the scripts in `R/` in order, or knit the main notebook in `notebooks/` to reproduce the analysis and charts.

