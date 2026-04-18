


## Correlation Analysis
Make sure to create the config.json file and include your FRED API key. Use the format below
{ 
  "FRED_API_KEY": "(Enter your key here)" 
}

### Correlation Analysis Data Dictionary
  -sku_nbr: Unique product number
  -sku_name: Unique Product name
  -correlation: Percent strength of relationship
  -abs_correlation: absolute value percent strength of relationship
  -sales_feature: The sales metric that was tested against each FRED feature.

    Possible values:

    sales_level = raw sales_units
    sales_pct_1m = month-over-month percent change in sales_units
    sales_pct_12m = year-over-year percent change in sales_units
    sales_roll3 = 3-month rolling average of sales_units

    Loop tests each sales_feature against each fred_feature and stores which pairing had the strongest correlation for that SKU.
  -fred_feature: The database and window used
  -n_obs: Number of observations to use in the rolling average calculation
  -best_beta: the regression slope for the best driver - ####This is the coefficent####
  -best_driver: Which Fred Database performed best 
  -best_corr: The correlation best between the best driver and the target variable

Running the full correlation_analysis notebook will give you the graph outputs for each sku

# Sales Forecast

## SKU-Level Forecasting Pipeline
This forecasting pipeline is designed to produce reliable monthly sales forecasts for many SKUs while accounting for the fact that different SKUs may behave differently over time. Instead of applying one forecasting method to every product, the process evaluates several model types for each SKU and selects the model that performs best for that SKU’s historical sales pattern. The code supports this by separating the data SKU by SKU, validating candidate models through rolling time-based evaluation, and saving the best future forecast for each item. 

### Purpose of the Process
The main purpose of the workflow is to forecast future sales_units at the SKU level. Since each SKU can have its own demand pattern, seasonality, trend, or volatility, the process avoids using a one-size-fits-all model. A product with stable demand may be best handled by exponential smoothing, while a product with seasonal or more complex behavior may benefit from SARIMA or Prophet.
The process also creates an auditable output. It does not only return forecasts; it also records which model was selected, how well it performed during validation, and whether any SKUs were skipped due to insufficient data or model failures.

## Data Preparation Process
Before any modeling happens, the data is converted into a clean monthly time series. This matters because time series forecasting models expect observations to be ordered consistently over time.
The month column is converted into a proper datetime format and normalized to the first day of each month. This ensures that dates such as mid-month or end-of-month timestamps are treated consistently as monthly periods.
For each SKU, the data is aggregated to one row per month. This is important because the models need a single sales value for each time period. If a SKU has multiple rows in the same month, those rows are summed into one monthly sales_units value.
The process also fills in missing months with zero sales. This creates a continuous monthly timeline, which is necessary for SARIMA and exponential smoothing models. Without this, the model may misinterpret gaps in time or fail because the data does not have a regular monthly frequency.
The reason for this preparation is to make every SKU’s history structurally consistent before model comparison begins.

## SKU-Specific Modeling Process
The pipeline treats each SKU as its own forecasting problem. This is important because SKU-level sales patterns can vary greatly. Some products may sell consistently every month, some may have seasonal patterns, and some may have irregular demand.
By looping through each SKU individually, the process allows each product to receive its own model selection. This is more flexible than fitting one global model to all SKUs because it respects the individual behavior of each item.
The temporary SKU-level dataset is only used as a workspace. The important outputs are the final forecast, the selected model type, and the performance metrics.

## Rolling Validation Process
The workflow uses rolling validation instead of a single train/test split. This is one of the most important parts of the process.
In normal machine learning, data is often randomly split into training and testing sets. That is not appropriate for time series forecasting because future data should never be used to predict past data. Time order must be preserved.
Rolling validation solves this by repeatedly training on earlier months and validating on later months. For example, with 36 months of data, the process trains on the first 24 months and validates on the next 3 months. Then it expands the training window and validates on the next 3 months, and so on.
The reason for this is to simulate how forecasting works in real life: use the past to predict the future, then move forward and repeat. This gives a more reliable estimate of model performance than testing on only one fixed time period.
Rolling validation also reduces the risk of choosing a model that only performed well during one unusual test window.

## Model Selection Process
The pipeline uses different model families depending on the amount of available history.
For SKUs with enough history, the process compares Prophet and SARIMA. These models are more advanced and can capture trend and seasonal structure, but they generally require more data to perform reliably.
For SKUs with less history, the process compares simpler exponential smoothing models: SES, DES, and TES. These models are often more stable when less data is available.
This split is used because complex models are not always better. With limited history, Prophet or SARIMA may overfit, fail to converge, or produce unstable forecasts. Simpler models can often perform better when the available pattern is limited.

## Prophet Process
Prophet is included because it can model trend changes and yearly seasonality. It is useful when a SKU’s sales pattern may shift over time or contain seasonal structure.
The workflow tests multiple Prophet configurations using grid search. This allows the process to evaluate different levels of trend flexibility and seasonal strength rather than assuming one Prophet setup is best.
The reason for tuning Prophet is that its performance can vary depending on how flexible the trend and seasonality settings are. A highly flexible model may fit historical data closely but overreact to noise. A less flexible model may be more stable but miss real changes in demand.

## SARIMA Process
SARIMA is included because it is a traditional statistical time series model that can capture autocorrelation and seasonality. It is especially useful when sales patterns depend on previous months or repeat annually.
The process tests multiple SARIMA parameter combinations using grid search. This is necessary because SARIMA performance depends heavily on the chosen order and seasonal order.
The seasonal period is set to 12 because the data is monthly. That allows SARIMA to look for yearly seasonal behavior.
SARIMA can be powerful, but it can also be computationally expensive and sensitive to the data. That is why the code catches errors and continues running if certain SARIMA configurations fail.

## Exponential Smoothing Process
For shorter histories, the workflow compares SES, DES, and TES.
  SES, or single exponential smoothing, is used for relatively stable demand without clear trend or seasonality.
  DES, or double exponential smoothing, adds a trend component. It is better suited for SKUs where sales are increasing or decreasing over time.
  TES, or triple exponential smoothing, adds trend and seasonality. It is useful when a SKU appears to have repeating yearly patterns.

The reason for comparing all three is that shorter-history SKUs may still have different structures. Some may be flat, some may trend upward or downward, and some may show seasonal movement. The model selection process lets the data decide which pattern is most useful.
TES is only attempted when enough seasonal history exists because seasonal models need enough data to estimate repeating patterns. For monthly data, at least two years of training history is usually needed to estimate yearly seasonality meaningfully.

## MAPE Optimization Process
The pipeline uses MAPE, or Mean Absolute Percentage Error, to compare models.
MAPE measures forecast error as a percentage of actual sales. This makes it easier to compare SKUs with different sales volumes because the error is scaled relative to actual demand.
The process calculates MAPE for each rolling validation fold and then averages the results. The model with the lowest average MAPE is selected as the best model for that SKU.
The reason for using average rolling MAPE is that it rewards models that perform consistently across multiple validation periods, rather than models that happen to perform well in only one period.
The code also protects against zero actual sales by excluding those rows from the MAPE calculation. This prevents divide-by-zero errors, although it also means zero-sales periods are not directly penalized in the MAPE score.

## Final Refit Process
After the best model is selected, the pipeline refits that model on the SKU’s full historical dataset.
This is important because the rolling validation phase is only used to compare models. Once the best model has been chosen, the final forecast should use all available history so that the model has the most complete information possible.
For example, if a SKU has 36 months of history, rolling validation uses subsets of that history to test model performance. But the final selected model is trained on all 36 months before forecasting the next 12 months.
This gives the final forecast the best chance of capturing the SKU’s full historical pattern.

## Forecast Output Process
The final forecast output stores future monthly predictions for each SKU. The output keeps the same structure as the original dataframe, with the addition of model_type.

  The final columns are:
    sku_nbr
    sub_class
    sales_units
    month
    sku_name
    model_type

In this output, sales_units represents the forecasted value for each future month.
The model_type column identifies whether the forecast came from Prophet, SARIMA, SES, DES, or TES. This makes the forecast explainable and easier to audit.

## Metrics and Audit Process
In addition to the forecast dataframe, the process creates a metrics dataframe. This is important because forecast values alone do not explain how the model was chosen.
The metrics output records the selected model, average rolling MAPE, fold-level MAPE values, model parameters, number of candidate models tested, and whether the SKU was successfully forecasted or skipped.
This gives you a way to evaluate the overall quality of the forecasting process. You can identify which SKUs had high error, which models won most often, and which SKUs lacked enough usable data.

## Error Handling Process
The pipeline is built to continue running even if individual models fail. This is important because time series modeling across many SKUs often produces exceptions.
Some SKUs may have flat sales, sparse sales, all-zero validation periods, or patterns that cause SARIMA or Prophet to fail. Instead of stopping the entire forecasting process, the code skips failed model attempts and continues testing other candidates.
If no model works for a SKU, that SKU is marked as skipped in the metrics dataframe.
This makes the pipeline more robust and prevents one problematic SKU from interrupting forecasts for all other SKUs.

## Why This Design Makes Sense
The overall design is practical because it balances accuracy, flexibility, and reliability.
It is flexible because each SKU receives its own model comparison.
It is statistically appropriate because validation respects time order.
It is reliable because it handles model failures and short-history SKUs differently from long-history SKUs.
It is auditable because it saves not only the forecast but also the selected model and performance metrics.
The main tradeoff is runtime. Prophet and SARIMA grid searches can be slow when applied to many SKUs and multiple rolling validation folds. However, the benefit is a more thoughtful and data-driven forecast selection process.

## Summary
This pipeline forecasts monthly sales at the SKU level by preparing each SKU as a clean monthly time series, evaluating multiple model types through rolling validation, selecting the model with the lowest average MAPE, refitting the selected model on all available history, and producing a 12-month future forecast.
The process is designed this way because SKU demand patterns vary, time series validation must preserve chronological order, and different models perform better under different data conditions. The final result is not just a forecast, but a structured forecasting system that explains which model was used and why it was chosen.

