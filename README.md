


## Correlation Analysis
Make sure to create the config.json file and include your FRED API key. Use the format below
{ 
  "FRED_API_KEY": "(Enter your key here)" 
}

## Correlation Analysis Data Dictionary
    sku_nbr: Unique product number
    sku_name: Unique Product name
    correlation: Percent strength of relationship
    abs_correlation: absolute value percent strength of relationship
    sales_feature: The sales metric that was tested against each FRED feature.

    Possible values:

    sales_level = raw sales_units
    sales_pct_1m = month-over-month percent change in sales_units
    sales_pct_12m = year-over-year percent change in sales_units
    sales_roll3 = 3-month rolling average of sales_units

    ## Loop tests each sales_feature against each fred_feature and stores which pairing had the strongest correlation for that SKU.
    fred_feature: The database and window used
    n_obs: Number of observations to use in the rolling average calculation
    best_beta: the regression slope for the best driver - ####This is the coefficent####
    best_driver: Which Fred Database performed best 
    best_corr: The correlation best between the best driver and the target variable

### Running the full correlation_analysis notebook will give you the graph outputs for each sku
