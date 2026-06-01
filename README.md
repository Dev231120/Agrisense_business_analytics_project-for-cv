AgriSense: Crop Price Prediction Across Indian States
Business + Analytics | Time-Series Forecasting | Power BI Dashboards |

An integrated platform that forecasts retail and wholesale crop prices across Indian states and translates those forecasts into actionable stocking, procurement, and risk-management decisions for farmers, traders, wholesalers, and policymakers.


Table of Contents-

Project Overview
Tech Stack
Repository Structure
Dataset Description
Methodology
Model Results & Accuracy Scores
Business Intelligence Layer
Power BI Dashboards
Key Insights & Recommendations
Stakeholder Framework
Future Roadmap
How to Run


Project Overview-
AgriSense is an end-to-end agricultural market intelligence system designed for the Indian agri-economy. It addresses a critical gap: crop prices in India are highly volatile, shaped by rainfall cycles, harvest timelines, logistics bottlenecks, and festival-driven demand spikes — yet most stakeholders lack tools to anticipate and respond to these movements.
The platform covers the full analytics pipeline:

Market mapping — identifying key stakeholders and demand-supply drivers
Data engineering — cleaning, merging, and reshaping retail and wholesale price datasets across all Indian states/UTs
Time-series forecasting — benchmarking Rolling Average, ARIMA, and SARIMA models for both retail and wholesale price prediction
Business intelligence enrichment — overlaying inventory signals, procurement windows, volatility zones, and final trade recommendations
Visual delivery — two Power BI dashboards surfacing forecasts, margin opportunities, and state-wise performance indicators

The pilot crop and state is Rice in Uttar Pradesh, chosen for data completeness and economic significance, with the architecture designed to scale to any crop-state combination.

Tech Stack-

Data Processing — pandas, numpy
Visualization — matplotlib, seaborn
Statistical Testing — statsmodels (ADF test, ACF/PACF)
Forecasting Models — statsmodels ARIMA, SARIMAX; Rolling Window
Evaluation Metrics — sklearn.metrics (MSE, MAE, RMSE)
Business Intelligence — Microsoft Power BI Desktop
CV / VQA Architecture — DETR (Facebook Research), GroundingDINO, CLIP
Notebook Environment — Google Colab / Jupyter Notebook


Repository Structure-
agrisense/
│
├── agrisense_business_analytics.ipynb   # Main analysis notebook (114 cells)
│
├── data/
│   ├── wholesale_price_data.csv         # Raw wholesale prices by state/date
│   ├── retail_price_data.csv            # Raw retail prices by state/date
│   ├── df_time_series.csv               # Enriched time-series (intermediate)
│   └── df_time_series_final.csv         # Final enriched dataset (Power BI source)
│
├── dashboards/
│   ├── up_rice_business_analysis.pbix              # Dashboard 1: UP Rice Business Analysis
│   └── AgriSense_Price_Forecasting_Dashboard.pbix  # Dashboard 2: Price Forecasting
│
└── README.md
Notebook breakdown — 114 total cells:

109 code cells
5 markdown cells (section headers and notes)


Dataset Description-
Source Data

wholesale_price_data.csv

Format — wide, 38 states/UTs as rows
Date range — 732 unique dates (Aug 2022 – Aug 2024)
Coverage — all Indian states/UTs


retail_price_data.csv

Format — wide, 38 states/UTs as rows
Date range — 732 unique dates (Aug 2022 – Aug 2024)
Coverage — all Indian states/UTs



After Reshaping (Long Format)
Both datasets are melted from wide to long format using pd.melt() on crop columns, producing three core fields per record: Date, States/UTs, Crop, and price.
Merged Dataset (df_full)
The two long-format frames are inner-joined on ['Date', 'States/UTs', 'Crop'], producing a unified dataset with:

retail_price — in ₹/kg
wholesale_price — originally ₹/quintal, converted to ₹/kg by dividing by 100
price_spread — retail minus wholesale, used as a gross margin proxy

Pilot Time-Series: Rice in Uttar Pradesh

Date range — 2022-08-31 to 2024-08-30
Total daily records — 730 entries
Missing dates filled — 1 (forward-filled using ffill)
Train set (80%) — 583 records
Test set (20%) — 147 records
Index type — DatetimeIndex, daily frequency


Methodology-
1-Data Cleaning & Feature Engineering

Removed metadata rows ('Average Price', 'Maximum Price', 'Minimum Price', 'Modal Price') that were incorrectly appearing in the States/UTs column
Parsed dates from %d-%m-%Y string format to datetime64
Converted wholesale prices from ₹/quintal to ₹/kg by dividing by 100 for cross-series comparability
Reindexed to a complete daily date range and forward-filled the single missing date

2-Stationarity Testing (ADF Test)
The Augmented Dickey-Fuller test was applied before and after first-order differencing:

Raw retail price series

ADF Statistic — -0.4719
p-value — 0.8974
Result — ❌ Non-stationary


First-differenced retail price series

ADF Statistic — -9.2054
p-value — 1.95 × 10⁻¹⁵
Result — ✅ Stationary (p < 0.05)



This confirmed d = 1 for ARIMA/SARIMA model configuration.
3-ACF / PACF Analysis
ACF and PACF plots were computed on the differenced retail series with 30 lags, informing the parameter selection of AR(p) = 1 and MA(q) = 1.
4-Model Configuration

Rolling 7-day Average

Window — 7-day, with 1-day lag
Target — Retail & Wholesale


ARIMA

Order — (1, 1, 1), trend='t'
Target — Retail & Wholesale


SARIMA

Order — (1, 1, 1)
Seasonal order — (1, 0, 0, 7), trend='t'
Seasonal period — 7 (weekly agricultural price cycle)
Target — Retail & Wholesale



5-Business Enrichment Signals
The final dataset (df_time_series_final.csv) contains 16 columns, including the following derived signals:

inventory_action — 'Hold Inventory' if forecast > current price, else 'Sell Inventory'
procurement_signal — 'Buy Window' if current price < 30-day rolling average, else 'Wait'
volatility_30d — 30-day rolling standard deviation of retail price
risk_zone — 'Low Risk' if volatility ≤ 33rd percentile, 'Medium Risk' if 33–66th percentile, 'High Risk' if above 66th percentile
final_recommendation — 'Strong Hold' when Hold + Low Risk, 'Urgent Sell' when Sell + High Risk, 'Cautious' for all other combinations


Model Results & Accuracy Scores-
Retail Price Forecasting — MSE

Rolling 7-day Average — MSE: 0.021546  (Best short-term)
SARIMA (1,1,1)(1,0,0,7) — MSE: 0.105443
ARIMA (1,1,1) — MSE: 0.113025

Wholesale Price Forecasting — MSE

Rolling 7-day Average — MSE: 0.029592 (Best short-term)
ARIMA (1,1,1) — MSE: 0.096126
SARIMA (1,1,1)(1,0,0,7) — MSE: 0.161965

Extended Metrics on Test Set — Retail Price (147 test samples)

Rolling 7-day Average

MAE — 0.0918 ₹/kg  Best
RMSE — 0.1243 ₹/kg  Best


SARIMA (1,1,1)(1,0,0,7)

MAE — 0.2765 ₹/kg
RMSE — 0.3247 ₹/kg


ARIMA (1,1,1)

MAE — 0.2815 ₹/kg
RMSE — 0.3362 ₹/kg




Key finding: For short-term (day-ahead) prediction, the 7-day rolling average outperforms both ARIMA and SARIMA by a wide margin. ARIMA and SARIMA are better suited for longer-range structural forecasting (30–100 day horizon).

100-Day Forward Forecast
Both SARIMA models (retail and wholesale) were used to project prices 100 days beyond the last observed date, providing actionable forward guidance for procurement and inventory planning.

Business Intelligence Layer-
Price Statistics — Rice, Uttar Pradesh

Start of series (Aug 2022)

Retail price — ₹32.06/kg
Wholesale price — ₹28.05/kg
Price spread — ₹4.01/kg


End of series (Aug 2024)

Retail price — ₹39.77–39.92/kg
Wholesale price — ₹34.88–35.09/kg
Price spread — ₹4.68–5.04/kg



The price spread (retail minus wholesale) represents the gross margin available to traders and retailers. An increasing spread over the 2-year period signals widening margins — a key alert signal for wholesaler and retailer strategy.
Sample Enriched Data (Last Rows)

2024-08-26 — Retail: ₹39.77 | Wholesale: ₹35.09 | Spread: ₹4.68
2024-08-27 — Retail: ₹39.92 | Wholesale: ₹34.88 | Spread: ₹5.04
2024-08-28 — Retail: ₹39.81 | Wholesale: ₹34.91 | Spread: ₹4.90


Power BI Dashboards-
Dashboard 1 — UP Rice Business Analysis (up_rice_business_analysis.pbix)

Created — 2025-12-30
File size — ~281 KB
Pages — 1 (single-page operational dashboard)
Data tables — df_time_series, df_time_series_final

Visuals included:

Slicer — filters by Crop and States/UTs
Card — Latest Retail Price
Card — Latest Forecast Price
Card — Inventory Action signal
Card — Risk Zone classification
Donut Chart — inventory_action distribution (Hold vs Sell breakdown)
Line Chart — final_forecast, retail_price, rolling_30_avg, volatility_30d over Date hierarchy (Year → Quarter → Month → Day)
100% Stacked Column Chart — risk_zone composition over time

DAX measures used:

Sum(df_time_series_final.final_forecast)
Sum(df_time_series_final.retail_price)
Sum(df_time_series_final.rolling_30_avg)
Sum(df_time_series_final.volatility_30d)
Min(df_time_series_final.procurement_signal)
CountNonNull(df_time_series_final.Date)
df_time_series_final.risk_zone
df_time_series.Final Recommendation


Dashboard 2 — AgriSense Price Forecasting Dashboard (AgriSense_Price_Forecasting_Dashboard.pbix)

Created — 2025-12-29
File size — ~8.3 MB (includes fully embedded data model)
Pages — 1 (single-page strategic dashboard)
Data tables — historical_prices_data, price_spread_data, forecast_prices_7d, results, Calendar

Visuals included:

Card — Sum of retail_price (historical)
Card — Sum of wholesale_price (historical)
Card — Sum of price_spread
Line Chart — actual_price vs forecast_price from forecast_prices_7d over Date hierarchy
Line Chart — retail_price, wholesale_price, price_spread from historical_prices_data over YearMonth (Calendar table)
Clustered Column Chart — model comparison showing results.Model vs Sum(results.mean_squared_error) for Rolling7d, ARIMA, and SARIMA
Slicer — filters by historical_prices_data.Crop and historical_prices_data.States/UTs

Data model tables:

historical_prices_data — full retail and wholesale price history
price_spread_data — derived margin and spread metrics
forecast_prices_7d — 7-day rolling forecast vs actual on the test set
results — model comparison table with MSE values for all 3 models
Calendar — date dimension table with YearMonth for time intelligence


Key Insights & Recommendations-

7-day rolling average is the best short-term predictor — MSE of 0.0216 vs 0.1130 for ARIMA and 0.1054 for SARIMA on retail prices. For day-ahead decisions, this is the preferred signal.
ARIMA/SARIMA are better for structural planning — Despite higher MSE on 1-step predictions, these models capture long-range trends and seasonality, making them suitable for 30–100 day procurement and policy planning horizons.
Price spread is widening — The retail-wholesale margin grew from ~₹4.01/kg (Aug 2022) to ~₹4.68–5.04/kg (Aug 2024), indicating either improving trader margins or growing supply chain inefficiency — an alert signal for policymakers.
Risk zones enable tiered action — Classifying market periods into Low / Medium / High Risk using 30-day volatility percentiles enables differentiated responses: strong holds during stable periods and urgent sell signals during high-volatility windows.
Procurement windows are identifiable — When current retail price dips below the 30-day rolling average, a Buy Window signal fires — a simple but effective trigger for inventory stocking decisions.


Stakeholder Framework-

Farmers — Know optimal harvest-sale timing; avoid selling during low-price risk zones
Traders / Wholesalers — Act on Buy Window signals; hold inventory during Strong Hold periods
Retailers — Monitor price spread trends; time procurement using the procurement_signal column
Policymakers — Identify High Risk zones for targeted MSP intervention; track state-level price divergence
Logistics Providers — Align transport capacity with harvest-cycle peaks captured by the weekly SARIMA seasonal component


Future Roadmap-

Multi-crop, multi-state scaling — Parameterize the pipeline to run across all crops in df_full (38 states × N crops)
External factor integration — Incorporate rainfall data (IMD), mandi arrivals, and diesel/logistics prices as SARIMAX exogenous variables
Festival demand calendar — Add Diwali, Eid, and harvest festival dummy variables to capture demand spikes
DETR / GroundingDINO / CLIP integration — Visual crop quality assessment from mandi images; CLIP-based cross-modal search
VQA module — Natural language interface for farmers to query price forecasts in Hindi using Visual Question Answering
Real-time data pipeline — Connect to Agmarknet / data.gov.in APIs for live price ingestion
Automated alert system — Push notifications to stakeholders when risk zone transitions to High Risk or a procurement signal fires


How to Run-
Prerequisites
bashpip install pandas numpy matplotlib seaborn statsmodels scikit-learn
Steps

Clone the repository and place wholesale_price_data.csv and retail_price_data.csv in /content/ (or update the paths at the top of the notebook)
Open agrisense_business_analytics.ipynb in Google Colab or Jupyter
Run all cells sequentially — the notebook is end-to-end linear
Output files df_time_series.csv and df_time_series_final.csv will be saved to the working directory
Load these CSVs into the Power BI dashboards via Transform Data → Change Source to refresh the visuals

Key Output Files

df_time_series.csv — 730-row daily time-series with ARIMA, SARIMA, and Rolling forecasts attached
df_time_series_final.csv — same as above plus all business signals: inventory_action, procurement_signal, risk_zone, final_recommendation
