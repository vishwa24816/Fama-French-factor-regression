# Fama-French Factor Regression for a Multi-Stock Portfolio

This project runs a Fama-French three-factor regression on an equal-weighted portfolio of ten large-cap US stocks. It downloads 20 years of daily prices from Yahoo Finance, pulls the matching daily factor data from Kenneth French's data library, estimates the factor exposures and alpha of the portfolio, and checks how stable those exposures are over time.

The analysis is designed as a Jupyter notebook of ten cells that run top to bottom.

## Table of Contents

- [Background](#background)
- [What the Notebook Does](#what-the-notebook-does)
- [Requirements](#requirements)
- [Installation](#installation)
- [Configuration](#configuration)
- [Running the Notebook](#running-the-notebook)
- [Cell-by-Cell Walkthrough](#cell-by-cell-walkthrough)
- [Interpreting the Output](#interpreting-the-output)
- [Methodology](#methodology)
- [Known Limitations](#known-limitations)
- [Troubleshooting](#troubleshooting)
- [Extending the Project](#extending-the-project)
- [References](#references)
- [Disclaimer](#disclaimer)

## Background

The Fama-French three-factor model explains the excess return of a portfolio using three sources of systematic risk:

- **Market (Mkt-RF):** the excess return of the overall market over the risk-free rate.
- **Size (SMB):** "Small Minus Big," the return of small-cap stocks minus large-cap stocks.
- **Value (HML):** "High Minus Low," the return of high book-to-market stocks minus low book-to-market stocks.

The regression is:

$$R_p - R_f = \alpha + \beta_{M}(R_m - R_f) + \beta_{S}\,SMB + \beta_{H}\,HML + \varepsilon$$

The intercept, alpha, measures the return that is not explained by these three factors. A positive and statistically significant alpha suggests the portfolio earned returns beyond what its factor exposures would predict. The betas measure how sensitive the portfolio is to each factor.

## What the Notebook Does

1. Installs and imports the required Python libraries.
2. Defines the ten tickers, equal weights, and a 20-year date window ending today.
3. Downloads daily adjusted closing prices from Yahoo Finance.
4. Computes daily simple returns for each stock and the daily return of the equal-weighted portfolio.
5. Downloads the daily Fama-French three-factor dataset through `pandas-datareader`.
6. Aligns the portfolio returns with the factor data and computes excess returns.
7. Estimates the regression with heteroskedasticity- and autocorrelation-consistent (HAC) standard errors.
8. Extracts coefficients, t-statistics, p-values, annualized alpha, and R-squared into a summary table.
9. Estimates rolling 252-day factor betas to check whether exposures are stable.
10. Plots cumulative returns against the market and a residual diagnostic plot.

## Requirements

- Python 3.9 or newer
- Jupyter Notebook, JupyterLab, or VS Code with Jupyter support
- An internet connection, since the data is downloaded at runtime
- Roughly 200 MB of free memory for the data and regression objects

Python packages:

| Package | Purpose |
|---|---|
| `yfinance` | Downloads price history from Yahoo Finance |
| `pandas-datareader` | Downloads Fama-French factors from Kenneth French's library |
| `statsmodels` | Runs OLS regression with HAC standard errors |
| `pandas` | Data handling and alignment |
| `numpy` | Numerical operations |
| `matplotlib` | Plotting |

## Installation

Clone or download the notebook, then open a terminal in the project folder.

Create and activate a virtual environment (recommended):

```bash
python -m venv ff-env
source ff-env/bin/activate        # macOS or Linux
ff-env\Scripts\activate           # Windows
```

Install the dependencies:

```bash
pip install yfinance pandas-datareader statsmodels pandas numpy matplotlib
```

Alternatively, Cell 1 of the notebook installs these packages automatically.

## Configuration

All user-adjustable settings are in Cell 2.

| Setting | Default | Description |
|---|---|---|
| `tickers` | 10 large-cap US stocks | List of Yahoo Finance ticker symbols |
| `weights` | Equal weights (0.10 each) | Portfolio weights; must sum to 1.0 |
| `start_date` | 20 years before today | Start of the sample period |
| `end_date` | Today | End of the sample period |

To analyze a different portfolio, replace the tickers and make sure the weights sum to 1.0. For example:

```python
tickers = ["AAPL", "MSFT", "NVDA", "JPM", "XOM"]
weights = np.array([0.30, 0.25, 0.20, 0.15, 0.10])
```

Other settings you may want to change:

- **Rolling window length** in Cell 9 (`window = 252`). A shorter window reacts faster to changes in exposure but produces noisier estimates.
- **HAC lag length** in Cell 7 (`maxlags=5`). Increase this if you suspect longer-lasting autocorrelation in residuals.

## Running the Notebook

1. Launch Jupyter from the project folder:

   ```bash
   jupyter lab
   ```

2. Open the notebook and run the cells in order, from top to bottom. Each cell depends on variables created in earlier cells, so running them out of order will cause errors.

3. The Yahoo Finance download (Cell 3) takes a few seconds. The Fama-French download (Cell 5) may take longer depending on connection speed.

4. Check the printed shapes and date ranges after Cells 3, 5, and 6 before moving on to the regression.

## Cell-by-Cell Walkthrough

| Cell | Action | Output |
|---|---|---|
| 1 | Install and import libraries | None |
| 2 | Set tickers, weights, and date range | Printed ticker list and period |
| 3 | Download daily adjusted closes | Shape of the price table |
| 4 | Compute stock and portfolio returns | Summary statistics for portfolio returns |
| 5 | Download Fama-French 3-factor data | First rows of factor data |
| 6 | Merge data and compute excess returns | Count of overlapping observations |
| 7 | Run the OLS regression | Full statsmodels summary |
| 8 | Build the results table | Alpha, R-squared, and coefficient table |
| 9 | Estimate rolling betas | Line plot of rolling factor exposures |
| 10 | Plot cumulative returns and residuals | Two-panel figure |

## Interpreting the Output

**Coefficients.** The `Mkt-RF` coefficient near 1.0 indicates the portfolio moves roughly with the market. Values well above 1.0 indicate higher market sensitivity; values below 1.0 indicate lower sensitivity.

**SMB.** A positive coefficient means the portfolio behaves more like small-cap stocks. A negative coefficient means it behaves more like large-cap stocks. For a portfolio of mega-cap names, expect a negative value.

**HML.** A positive coefficient means a value tilt; a negative coefficient means a growth tilt.

**Alpha.** The `const` term is the daily alpha, and the notebook annualizes it by multiplying by 252. An alpha is statistically meaningful only if its p-value is below your chosen threshold (typically 0.05). Even when significant, alpha estimates over 20 years are sensitive to the sample period and to the choice of stocks.

**R-squared.** This shows the share of the portfolio's excess return variance explained by the three factors. For diversified equity portfolios, values often fall between 0.85 and 0.95. A low value suggests the portfolio has exposures the model does not capture, such as sector bets or idiosyncratic risk.

**Rolling betas.** Stable lines indicate a consistent factor profile. Large swings suggest the portfolio's exposures changed over time, which weakens the interpretation of any single full-sample estimate.

**Residual plot.** Residuals should scatter randomly around zero. A funnel shape suggests heteroskedasticity, and a curved pattern suggests a missing factor or nonlinearity.

## Methodology

**Returns.** Daily simple returns are computed as `P_t / P_{t-1} - 1` from adjusted closing prices. Adjusted prices account for dividends and splits.

**Portfolio construction.** The portfolio return is the weighted sum of stock returns with fixed weights, rebalanced to those weights each day. This is a convenient simplification. A true buy-and-hold portfolio would drift away from its target weights over time.

**Factor data.** The Fama-French daily file (`F-F_Research_Data_Factors_daily`) reports returns in percent. The notebook divides by 100 to convert to decimals. The `RF` column is the daily risk-free rate.

**Excess returns.** The dependent variable is the portfolio return minus `RF`.

**Estimation.** Ordinary least squares with Newey-West (HAC) standard errors, using five lags. HAC errors are robust to heteroskedasticity and to modest serial correlation, which is common in return data.

**Date alignment.** An inner join keeps only dates present in both the portfolio and factor datasets. Fama-French dates that fall on non-trading days, and trading days that lack factor data, are dropped.

## Known Limitations

- **Survivorship bias.** The ten tickers are chosen today. Companies that were removed from an index or went bankrupt during the period are not included, which tends to flatter historical returns.
- **Look-ahead in stock selection.** Choosing well-known winners and running a backtest on them overstates expected alpha.
- **Fixed weights.** Daily rebalancing differs from buy-and-hold, and both differ from what most investors actually hold.
- **Data gaps.** Stocks listed after the start date shrink the sample, because `dropna()` removes any date where any ticker is missing. Consider aligning on the shortest common history or handling missing data explicitly.
- **Factor lag.** Kenneth French's library updates with a delay, so the most recent months may be unavailable.
- **Model scope.** The three-factor model omits momentum, profitability, and investment factors. The five-factor model or a momentum-augmented model may give different alpha estimates.
- **Yahoo Finance reliability.** The `yfinance` library scrapes Yahoo's public endpoints, which are unofficial and can change without notice. Prices may occasionally contain errors or missing values.
- **Dividends and corporate actions.** Adjusted prices handle these, but the adjustment method is Yahoo's, and it is not audited.

## Troubleshooting

**`yf.download` returns an empty DataFrame or fewer than 20 years.** Yahoo Finance may limit daily history for some tickers, or one ticker symbol may be wrong. Check each symbol on finance.yahoo.com. Shorten `start_date` if needed.

**`KeyError: 'Close'` in Cell 3.** Recent versions of `yfinance` return a multi-level column index. Pass `auto_adjust=True` as shown, or select the column with `raw.xs("Close", axis=1, level=0)`.

**Fama-French download fails with an HTTP or SSL error.** Kenneth French's site occasionally blocks automated requests or goes offline for maintenance. Retry later, or download the CSV manually from the Kenneth French data library and load it with `pd.read_csv`, skipping the header rows.

**`ValueError` in Cell 5 when converting the index.** The factor index format can differ across `pandas-datareader` versions. Print `ff3.index[:5]` and adjust the conversion to `pd.PeriodIndex.to_timestamp()` or `pd.to_datetime()` as needed.

**Empty merged data in Cell 6.** The date ranges do not overlap. Confirm that `prices.index` and `ff3.index` are both `DatetimeIndex` objects with the same timezone setting (naive is recommended).

**Regression fails with a singular matrix error.** One of the factors is constant or perfectly collinear with another in the sample. This is unusual over 20 years but can occur if the merged dataset is very short.

**`seaborn-v0_8` style not found.** Older Matplotlib versions use a different style name. Replace it with `plt.style.use("ggplot")` or remove the line.

## Extending the Project

- **Five-factor model:** Download `F-F_Research_Data_5_Factors_2x3_daily` and add `RMW` and `CMA` to the regression in Cell 7.
- **Momentum:** Download the momentum factor file and add it as a fourth regressor.
- **Weight optimization:** Replace the fixed weights with minimum-variance or risk-parity weights estimated from a trailing window.
- **Buy-and-hold returns:** Compute portfolio value from initial shares and track drift over time.
- **Alternative benchmarks:** Compare the portfolio against an ETF such as SPY instead of only the factor returns.
- **Export results:** Save the summary table to CSV with `results.to_csv("ff3_results.csv")`.

## References

- Fama, E. F., & French, K. R. (1993). Common risk factors in the returns on stocks and bonds. *Journal of Financial Economics*, 33(1), 3–56.
- Kenneth R. French Data Library: https://mba.tuck.dartmouth.edu/pages/faculty/ken.french/data_library.html
- Newey, W. K., & West, K. D. (1987). A simple, positive semi-definite, heteroskedasticity and autocorrelation consistent covariance matrix. *Econometrica*, 55(3), 703–708.
- `yfinance` documentation: https://github.com/ranaroussi/yfinance
- `statsmodels` documentation: https://www.statsmodels.org

## Disclaimer

This project is for educational and research purposes. It is not investment advice, and the results should not be used as the sole basis for any financial decision. Historical factor exposures and alpha estimates do not guarantee future performance.
