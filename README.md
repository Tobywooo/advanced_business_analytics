# advanced_business_analytics
Group 1 Project in DSBA Advanced Business Analytics 6211
https://fred.stlouisfed.org/

## Correlation Analysis ###
SKU sales show a stronger relationship with housing completions (COMPUTSA) than with housing starts (HOUST). The most useful signals are generally smoothed trends (roll3) and year-over-year changes (pct_12m), rather than raw monthly levels or one-month changes. The most common winning timing pattern is a 3- to 6-month lag, especially 6 months, which suggests sales tend to respond to housing activity with a delay.

At the SKU level, many of the best-fit correlations are moderately strong, but they should be treated as exploratory rather than final, because each SKU was optimized across many feature combinations. The clearest portfolio-level takeaway is that COMPUTSA, especially in smoothed or YoY form, is the better macro driver to use as a benchmark signal. If you want one standardized macro relationship to carry forward, COMPUTSA_roll3_lag6 is the strongest candidate from this analysis.

Use best_beta as the SKU-level scaling coefficient and best_corr as the confidence check.
For each SKU, best_beta tells you the estimated change in sales_units associated with a 1-unit increase in the selected housing driver (COMPUTSA or HOUST). For example, if a SKU has best_beta = 0.18, then a 1-unit increase in the chosen housing series is associated with an estimated increase of 0.18 sales units for that SKU. best_corr should not be used as the scaling factor; instead, it tells you how strong and reliable that relationship appears to be. Higher absolute values mean a stronger relationship.

In practice, use best_beta for planning or sensitivity calculations, and use best_corr plus observation count as a filter before trusting the result. A simple rule is to rely more on coefficients where abs(best_corr) is at least moderate and the SKU has enough history. So the workflow is: identify the SKU, check its best_driver, use best_beta as the coefficient, and confirm that best_corr is strong enough to make the relationship worth using.
