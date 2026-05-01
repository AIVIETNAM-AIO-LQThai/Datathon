# Clean No-Leakage Feature Plan for 2023–2024 Revenue and COGS Forecasting

## 1. Core rule

For the final forecast period, we do **not** know actual 2023–2024 values for orders, customers, product mix, payments, inventory, returns, or promotion outcomes.

The model predicts only:

```text
Revenue
COGS
```

Auxiliary variables are **not predicted alongside Revenue and COGS** in this plan.

A feature is allowed only if it can be reproduced for every 2023–2024 date using:

```text
1. the Date itself,
2. official historical data up to the training cutoff,
3. previous recursive predictions of Revenue/COGS.
```

If a feature requires actual 2023–2024 auxiliary activity, it is not allowed.

---

## 2. Cutoff rule

Historical expectations must be rebuilt separately for each training cutoff.

### Validation setup

```text
Train cutoff: 2020-12-31
Validation:   2021-01-01 to 2022-12-31
```

All historical expectations must be learned only from data up to `2020-12-31`.

### Final forecast setup

```text
Train cutoff: 2022-12-31
Forecast:     2023-01-01 to 2024-07-01
```

Historical expectations may use official data from 2012–2022.

No feature may use actual 2021–2022 auxiliary values during validation if those values are beyond the validation training cutoff. No feature may use actual 2023–2024 auxiliary values during final forecasting.

---

## 3. Naming rule

Use names that make the source obvious.

Use:

```text
hist_
hist_recent3y_
hist_trend_
scheduled_
lag_
roll_
```

Avoid names that sound like actual future observations:

```text
order_count
age_share
product_mix
payment_value
inventory_level
promo_active
previous_month_orders
previous_month_age_share
```

Examples of clear names:

```text
hist_recent3y_expected_order_count_by_monthofyear
hist_recent3y_expected_aov_by_monthofyear
hist_trend_projected_category_share_streetwear_by_monthofyear
scheduled_promo_active
revenue_lag_7
cogs_ratio_roll_mean_28
```

---

## 4. Feature priority tiers

### Tier 1 — strongest and should be made first

These features are reproducible, business meaningful, and likely useful.

1. Date/calendar features
2. Revenue/COGS lags, rolling statistics, YoY anchors
3. Revenue/COGS historical expectations by calendar period
4. COGS ratio, GPM, ROI-proxy lag and historical expectation features
5. Order pressure historical expectations
6. AOV historical expectations
7. Product category mix historical expectations
8. Scheduled promotion calendar features, if schedule recurrence is inferable from official `promotions.csv`

### Tier 2 — useful but must be tested by validation

1. Recent/regime-aware promotion effect features
2. Payment method and installment-share historical expectations
3. Inventory pressure historical expectations
4. Customer lifecycle historical expectations
5. Trend-projected product mix

### Tier 3 — weak or optional

1. Age-group share expectations
2. Gender share expectations
3. Very granular demographic interactions
4. Broad all-history promotion lift averages
5. Broad all-history inventory averages
6. Raw payment value sums

These should be added only if validation improves.

---

# Feature Groups

## 5. Date/calendar features

These are always safe because they come directly from `Date`.

Recommended:

```text
year
month
quarter
dayofweek
dayofyear
weekofyear
day_of_month
days_to_month_end
days_since_month_start
month_progress
is_weekend
is_odd_year
flag_q1
flag_q2
flag_q3
flag_q4
flag_q3_odd_margin_risk
flag_q4_peak
flag_q2_peak
```

Fourier features:

```text
sin_y1, cos_y1, ..., sin_y5, cos_y5
sin_w1, cos_w1
sin_m1, cos_m1
```

Do not add external holidays. Do not manually create future promo schedules unless the recurrence is derived from official `promotions.csv`.

---

## 6. Sales target-history features

These are safe because they are derived from Revenue/COGS history.

During forecasting, these are recursively updated using previous predicted Revenue/COGS.

### Lags

Apply to both Revenue and COGS:

```text
lag_1
lag_7
lag_14
lag_28
lag_56
lag_91
lag_182
lag_364
lag_365
lag_371
lag_728
```

### Rolling statistics

Windows:

```text
7, 14, 28, 56, 91, 182, 365 days
```

Metrics:

```text
mean
median
std
min
max
range
IQR
CV
```

### Momentum, spike, and drop features

Safe only if computed from shifted values.

Recommended:

```text
revenue_delta_1
revenue_delta_7
revenue_pct_change_7
revenue_spike_score_14
revenue_spike_score_28
revenue_just_spiked_14
revenue_just_spiked_28
days_since_revenue_spike
days_since_revenue_drop
post_spike_cooldown
post_drop_rebound
```

Same pattern may be applied to COGS.

A spike should be defined using yesterday compared with a prior rolling median/IQR. Do not use same-day true Revenue/COGS.

---

## 7. Sales historical expectations

These are built from training data only and attached to future dates by calendar keys.

Recommended:

```text
hist_expected_revenue_by_monthofyear
hist_expected_revenue_by_quarter
hist_expected_revenue_by_weekofyear
hist_expected_revenue_by_dayofweek
hist_expected_revenue_by_dayofmonth
hist_expected_revenue_by_month_dayofweek
hist_recent3y_expected_revenue_by_monthofyear
hist_revenue_recent_vs_all_ratio_by_monthofyear
```

Same for COGS:

```text
hist_expected_cogs_by_monthofyear
hist_expected_cogs_by_quarter
hist_expected_cogs_by_weekofyear
hist_expected_cogs_by_dayofweek
hist_recent3y_expected_cogs_by_monthofyear
hist_cogs_recent_vs_all_ratio_by_monthofyear
```

These are safe because they are historical calendar baselines, not future actual values.

---

## 8. Margin, GPM, COGS ratio, and ROI-proxy features

Do **not** use same-day target-derived values.

Unsafe:

```text
gpm_today
roi_today
cogs_ratio_today
gross_profit_today
```

Safe lagged features:

```text
cogs_ratio_lag_1
cogs_ratio_lag_7
cogs_ratio_roll_mean_28
cogs_ratio_roll_median_56
gpm_lag_1
gpm_lag_7
gpm_roll_mean_28
gpm_roll_median_56
roi_proxy_lag_1
roi_proxy_roll_mean_28
```

Safe historical expectations:

```text
hist_expected_cogs_ratio_by_monthofyear
hist_recent3y_expected_cogs_ratio_by_monthofyear
hist_expected_cogs_ratio_by_quarter
hist_expected_gpm_by_monthofyear
hist_recent3y_expected_gpm_by_monthofyear
hist_expected_gpm_by_quarter
hist_expected_roi_proxy_by_monthofyear
hist_expected_q3_odd_cogs_ratio
hist_expected_q3_odd_gpm
```

Definitions:

```text
COGS ratio = COGS / Revenue
GPM        = (Revenue - COGS) / Revenue
ROI proxy  = (Revenue - COGS) / COGS
```

These are high-priority because COGS is structurally tied to Revenue.

---

## 9. Orders-derived features

Do **not** use actual future order count. Do **not** use previous forecast-month order count.

Use only historical calendar expectations.

### Strong order-pressure features

```text
hist_recent3y_expected_order_count_by_monthofyear
hist_expected_order_count_by_monthofyear
hist_expected_order_count_by_weekofyear
hist_expected_order_count_by_dayofweek
hist_expected_order_count_by_month_dayofweek
hist_order_count_recent_vs_all_ratio_by_monthofyear
hist_order_count_trend_slope_by_monthofyear
```

The recent3y and trend versions are preferred over plain all-history averages because order volume can drift by year.

### AOV features

```text
hist_recent3y_expected_aov_by_monthofyear
hist_expected_aov_by_monthofyear
hist_expected_aov_by_weekofyear
hist_expected_aov_by_dayofweek
hist_aov_recent_vs_all_ratio_by_monthofyear
hist_aov_trend_slope_by_monthofyear
```

AOV is likely more useful than raw order count because it captures basket value and product-price drift.

### Order quality features

Only if order statuses are present and reliable:

```text
hist_recent3y_cancel_rate_by_monthofyear
hist_recent3y_return_rate_by_monthofyear
hist_recent3y_delivered_rate_by_monthofyear
hist_recent3y_paid_rate_by_monthofyear
hist_order_status_entropy_by_monthofyear
```

Avoid raw count sums unless normalized as rates or daily averages.

---

## 10. Customer-derived features

Customer features must be historical expectations only. They are not updated with future actual customer behavior.

### Keep only if validation improves

Age-group features are optional because age mix may be stable and weak.

Possible features:

```text
hist_recent3y_age_18_24_share_by_monthofyear
hist_recent3y_age_25_34_share_by_monthofyear
hist_recent3y_age_35_44_share_by_monthofyear
hist_recent3y_age_45_54_share_by_monthofyear
hist_recent3y_age_55_plus_share_by_monthofyear
hist_recent3y_young_customer_share_by_monthofyear
hist_recent3y_age_mix_entropy_by_monthofyear
```

Remove gender-share features unless validation proves they help:

```text
hist_expected_female_share_by_monthofyear
hist_expected_male_share_by_monthofyear
hist_expected_gender_mix_entropy_by_monthofyear
```

Reason: if male/female Revenue contribution is nearly equal and stable, these columns add little signal.

### Customer lifecycle features

Use only as historical expectations:

```text
hist_recent3y_expected_new_customer_count_by_monthofyear
hist_expected_new_customer_count_by_weekofyear
hist_recent3y_returning_customer_share_by_monthofyear
hist_recent3y_repeat_customer_share_by_monthofyear
hist_recent3y_reactivation_count_by_monthofyear
hist_recent3y_dormant_365_pressure_by_monthofyear
hist_recent3y_repeat_purchase_gap_median_by_monthofyear
```

These are medium priority. They should not be used as previous-month actual customer values.

---

## 11. Product mix features

Product mix can be useful because category share changes over time and affects Revenue and COGS ratio.

Do **not** use actual future product mix.

Preferred features are recent/regime-aware or trend-aware:

```text
hist_recent3y_category_share_streetwear_by_monthofyear
hist_recent3y_category_share_outdoor_by_monthofyear
hist_recent3y_category_share_casual_by_monthofyear
hist_recent3y_category_share_genz_by_monthofyear
hist_category_share_recent_vs_all_ratio_streetwear_by_monthofyear
hist_category_share_recent_vs_all_ratio_outdoor_by_monthofyear
hist_category_mix_entropy_by_monthofyear
hist_streetwear_minus_outdoor_share_by_monthofyear
```

Trend-projected category mix can be tested:

```text
hist_trend_projected_streetwear_share_by_monthofyear
hist_trend_projected_outdoor_share_by_monthofyear
```

How to make trend-projected category share:

```text
For each category and month-of-year:
1. compute historical category share by year using training data only,
2. fit share ~ year,
3. project to forecast year,
4. clip to [0, 1].
```

Use this only if validation improves. Trend projection can overfit if category history is noisy.

---

## 12. Promotion features

Promotions need special handling.

If promotions are repeated by an official schedule/pattern in `promotions.csv`, future scheduled promotion features are allowed because they are known from the official recurring pattern, not from actual future outcomes.

### Strong scheduled-promo features

Use these when schedule recurrence can be derived from official data:

```text
scheduled_promo_active
scheduled_promo_type_onehot
scheduled_promo_discount_depth
scheduled_promo_day_index
scheduled_promo_days_remaining
scheduled_promo_window_position
scheduled_pre_promo_7d
scheduled_post_promo_7d
scheduled_promo_x_month_end
scheduled_promo_x_q3_odd
```

These are stronger than generic historical promo probabilities because they represent the known scheduled calendar.

### Weak promo features to remove or deprioritize

Remove broad all-history lift averages such as:

```text
hist_expected_promo_revenue_lift_by_monthofyear
hist_expected_promo_cogs_lift_by_monthofyear
hist_expected_promo_aov_lift_by_monthofyear
hist_expected_promo_margin_impact_by_monthofyear
```

Reason: promotion lift changes by year, discount depth, product mix, inventory, and customer regime. A plain all-history month average mixes different business regimes and can be misleading.

### If promotion-effect features are used, make them recent and promo-type-specific

Safer alternatives:

```text
hist_recent3y_promo_lift_ratio_by_promo_type
hist_recent3y_promo_margin_impact_by_promo_type
hist_promo_lift_recent_vs_all_ratio_by_promo_type
hist_promo_lift_trend_slope_by_promo_type
hist_recent3y_post_promo_cooldown_by_promo_type
```

Lift should be calculated relative to a comparable local baseline, not raw Revenue difference.

Example:

```text
promo_lift_ratio = Revenue_on_promo_day / expected_nonpromo_revenue_for_similar_calendar_period
```

The baseline should use only training history and should match similar month, day-of-week, and recent regime when possible.

---

## 13. Payment features

Do not use actual future payment data.

Avoid raw summed payment values because they mostly duplicate Revenue or reflect data volume.

Prefer shares, rates, and averages:

```text
hist_recent3y_credit_payment_share_by_monthofyear
hist_recent3y_cod_payment_share_by_monthofyear
hist_recent3y_bank_payment_share_by_monthofyear
hist_payment_method_entropy_by_monthofyear
hist_avg_installment_count_by_monthofyear
hist_share_installment_payment_by_monthofyear
hist_share_high_installment_payment_by_monthofyear
hist_high_value_payment_share_by_monthofyear
```

Do not use:

```text
hist_expected_payment_value_by_monthofyear
hist_sum_installments_by_monthofyear
```

Reason: sums are often scale proxies, not behavior. Counts, shares, and averages are more interpretable and stable.

---

## 14. Inventory features

Inventory features are risky because future actual inventory is unknown and inventory policy may change by year.

Use only if validation improves.

Prefer recent/regime-aware rates:

```text
hist_recent3y_stockout_rate_by_monthofyear
hist_recent3y_stockout_rate_by_category_monthofyear
hist_recent3y_sell_through_rate_by_monthofyear
hist_recent3y_overstock_rate_by_monthofyear
hist_inventory_pressure_recent_vs_all_ratio_by_monthofyear
```

Avoid broad all-history inventory averages:

```text
hist_expected_inventory_pressure_by_monthofyear
hist_expected_inventory_volatility_by_monthofyear
```

Reason: inventory is an operational decision, not a stable natural seasonality. A plain historical average can become stale quickly.

---

## 15. Safe interaction features

Only create interactions between features that are already safe and useful.

Recommended first:

```text
flag_q3_odd_margin_risk_x_hist_expected_cogs_ratio_by_quarter
flag_q4_peak_x_hist_recent3y_expected_aov_by_monthofyear
flag_q4_peak_x_hist_recent3y_category_share_streetwear_by_monthofyear
is_last_3_days_x_hist_recent3y_expected_order_count_by_monthofyear
is_last_3_days_x_hist_recent3y_expected_aov_by_monthofyear
scheduled_promo_active_x_hist_recent3y_promo_margin_impact_by_promo_type
scheduled_promo_active_x_hist_recent3y_expected_aov_by_monthofyear
hist_recent3y_category_mix_entropy_by_monthofyear_x_hist_expected_cogs_ratio_by_monthofyear
```

Avoid interactions with weak demographics unless validation proves value.

---

## 16. Features to avoid

### Actual or previous-month auxiliary activity

```text
actual_order_count_2023
previous_month_order_count_2023
actual_age_share_2023
previous_month_age_share_2023
actual_new_customers_2023
previous_month_new_customers_2023
actual_product_mix_2023
previous_month_product_mix_2023
actual_payment_value_2023
previous_month_payment_value_2023
actual_inventory_2023
previous_month_inventory_2023
```

### Same-day target-derived leakage

```text
gpm_today
roi_today
cogs_ratio_today
gross_profit_today
```

### Weak all-history averages for changing business actions

```text
hist_expected_promo_revenue_lift_by_monthofyear
hist_expected_promo_margin_impact_by_monthofyear
hist_expected_inventory_pressure_by_monthofyear
hist_expected_payment_value_by_monthofyear
hist_expected_female_share_by_monthofyear
hist_expected_male_share_by_monthofyear
```

### External data

```text
external holiday calendars
manually hard-coded future promotion schedules not derived from official data
web-scraped economic indicators
weather
Google Trends
non-official data
```

---

## 17. Clean implementation order

### Phase 1: core clean features

```text
sales lags/rolling/YoY/spike features
sales historical expectations
COGS ratio / GPM / ROI-proxy features
order count historical expectations with recent3y + recent_vs_all + trend slope
AOV historical expectations with recent3y + recent_vs_all + trend slope
product category share historical expectations with recent3y + recent_vs_all
scheduled promotion calendar features from official recurrence
```

### Phase 2: tested business enrichment

```text
recent3y promo lift by promo type using local baseline
payment method shares and installment shares
customer lifecycle expectations
inventory pressure rates
trend-projected product category shares
```

### Phase 3: optional only if validation improves

```text
age-group share expectations
demographic × product interactions
inventory × category interactions
complex promo-effect interactions
```

---

## 18. Final safety checklist

Before including a feature, answer these questions:

```text
1. Can it be computed for 2023–2024 without actual future auxiliary data?
2. Is it built only from official data up to train_end?
3. Is it either calendar-derived, historical/regime expectation, scheduled from official recurrence, or recursive from previous Revenue/COGS predictions?
4. Does it avoid same-day target leakage?
5. Is it not just a stale all-history average of a changing business action?
```

If any answer is no, remove or downgrade the feature.

The clean feature plan should favor:

```text
known calendar structure
recursive target history
recent/regime-aware historical expectations
scheduled promotion features derived from official data
rates/shares/averages instead of raw sums
```

It should avoid:

```text
future auxiliary actuals
previous-month future auxiliary values
same-day target-derived features
broad all-history averages for changing business actions
weak stable demographic columns
```
