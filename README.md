# DS203 Regression Project – Inflation Forecasting

## Overview

This project focuses on predicting future U.S. year-over-year CPI inflation using macroeconomic data from FRED.

The aim is to forecast inflation **1, 3, 6 and 12 months ahead** using information available at the current month. The notebook covers the complete process, starting from loading and preparing the data, followed by feature engineering and exploratory data analysis, and finally model training and evaluation.

## Dataset

The project uses the following 11 FRED series:

| Series | Description |
|---|---|
| `CPIAUCSL` | Consumer Price Index |
| `T10Y2Y` | 10-year minus 2-year Treasury yield spread |
| `UNRATE` | Unemployment rate |
| `INDPRO` | Industrial Production Index |
| `FEDFUNDS` | Federal funds rate |
| `PAYEMS` | Nonfarm payroll employment |
| `PPIACO` | Producer Price Index |
| `HOUST` | Housing starts |
| `DCOILWTICO` | WTI crude oil price |
| `M2SL` | M2 money supply |
| `UMCSENT` | Consumer sentiment |

The common monthly dataset covers **January 1986 to September 2026**, with 489 observations before the final modelling filters.

The 10 macroeconomic predictor series are used to create 20 candidate features. These include the original variables, monthly changes and year-over-year growth rates.

## Target Variable

Year-over-year CPI inflation is calculated using:

$$
\text{Inflation}_t = 100 \times \left(\frac{\text{CPI}_t}{\text{CPI}_{t-12}} - 1\right)
$$

The target for each forecasting horizon is the inflation rate in a future month:

$$
\text{Target}_{n,t} = \text{Inflation}_{t+n}
$$

where $n \in \{1, 3, 6, 12\}$.

In other words, the model uses data available in month $t$ to predict inflation $n$ months later.

## Data Preparation and Feature Engineering

The daily series, `T10Y2Y` and `DCOILWTICO`, are converted to monthly frequency by taking the mean of the available trading-day values in each month. All series are then aligned on a common monthly date index.

The notebook creates additional features using monthly changes and year-over-year growth rates. For example, the monthly change is calculated as:

$$
\Delta X_t = X_t - X_{t-1}
$$

The year-over-year growth rate is calculated as:

$$
\text{Growth}_t = 100 \times \left(\frac{X_t}{X_{t-12}} - 1\right)
$$

Some of the engineered features are:

- `UNRATE_chg_1m`
- `INDPRO_yoy`
- `PAYEMS_yoy`
- `PPIACO_yoy`
- `DCOILWTICO_yoy`
- `M2SL_yoy`
- `HOUST_yoy`
- `FEDFUNDS_chg_1m`
- `T10Y2Y_chg_1m`
- `UMCSENT_chg_1m`

Only information available at the prediction month is used to construct the features.

## Exploratory Data Analysis

The notebook includes exploratory analysis of:

- Inflation over time
- Different inflation regimes
- Feature distributions and summary statistics
- Skewness and volatility
- Outliers
- Correlations between variables
- Lead-lag correlations between predictors and future inflation

The EDA helps understand how the macroeconomic variables behave over time and how they relate to inflation.

## Models Used

The following regression models are compared:

1. Linear Regression
2. Ridge Regression
3. Lasso Regression
4. Elastic Net
5. K-Nearest Neighbours (KNN) Regression

The preprocessing steps include IQR-based outlier capping, median imputation and feature standardisation. These steps are handled as part of the modelling pipeline, so the preprocessing is fitted using the training data rather than the complete dataset.

## Training and Evaluation

Since the task involves forecasting, the data is split chronologically instead of randomly shuffled.

The data is divided into:

- Training set
- Validation set
- Test set

A gap corresponding to the forecast horizon is maintained between the splits to avoid overlap between target periods.

Hyperparameters and models are selected using validation performance. The selected model for each horizon is then refitted using the training and validation data and evaluated on the test set.

The evaluation metrics are:

- **RMSE (Root Mean Squared Error)**
- **MAE (Mean Absolute Error)**
- **R² (Coefficient of Determination)**

A training-mean baseline is also included for comparison.

The notebook additionally uses a no-change baseline. It assumes that inflation $n$ months ahead will remain equal to the inflation rate observed in the current month:

$$
\widehat{\text{Inflation}}_{t+n} = \text{Inflation}_t
$$

This provides a simple reference point to check whether the regression models actually improve the forecast.

## Results

The models selected using validation RMSE are:

| Forecast horizon | Selected model |
|---|---|
| 1 month | Lasso |
| 3 months | Elastic Net |
| 6 months | Lasso |
| 12 months | Elastic Net |

The test-set results are:

| Horizon | Model | MAE | RMSE | R² |
|---|---|---:|---:|---:|
| 1 month | Lasso | 1.326 | 2.039 | 0.112 |
| 3 months | Elastic Net | 1.607 | 2.303 | -0.135 |
| 6 months | Lasso | 1.711 | 2.524 | -0.345 |
| 12 months | Elastic Net | 2.096 | 2.940 | -0.824 |

The results show that forecasting performance generally becomes worse as the prediction horizon increases.

One important observation is that the no-change baseline performs better than the selected regression models at all four horizons. This means that, with the features and models used in this project, the regression models do not improve upon simply using the current inflation rate as the forecast.

## Feature Selection

Lasso is also used to examine which features are retained after regularisation.

Some features that appear across the different forecasting horizons include:

- `FEDFUNDS`
- `PAYEMS`
- `PPIACO_yoy`
- `DCOILWTICO_yoy`

The selected features can differ depending on the forecast horizon. The coefficients indicate relationships learned by the models and should not be interpreted as proof of causation.

## How to Run the Notebook

Keep the notebook and all 11 CSV files in the same folder:

- `Project_Code.ipynb`
- `CPIAUCSL.csv`
- `T10Y2Y.csv`
- `UNRATE.csv`
- `INDPRO.csv`
- `FEDFUNDS.csv`
- `PAYEMS.csv`
- `PPIACO.csv`
- `HOUST.csv`
- `DCOILWTICO.csv`
- `M2SL.csv`
- `UMCSENT.csv`

Open `Project_Code.ipynb` using Jupyter Notebook, JupyterLab or VS Code, and run the cells in order.

The notebook uses the following Python libraries:

- NumPy
- Pandas
- Matplotlib
- Seaborn
- Scikit-learn

If Seaborn is not installed, the notebook installs it in the setup cell.

## Conclusion

This project compares different regression models for forecasting U.S. inflation at multiple time horizons. The results show that the selected macroeconomic features are not enough to outperform the no-change baseline on the test set. The gap in performance also becomes more noticeable for longer forecast horizons.

Although the regression models do not beat the baseline, this comparison is useful because it shows how difficult inflation forecasting can be and why a simple benchmark should always be included when evaluating a forecasting model.
