# Data-Driven Asset Allocation Strategies

Open Avenues Foundation — Quantitative Finance Track

---

## Research Question

Does the Kelly Criterion improve risk-adjusted portfolio performance compared to
equal-weight allocation, and how do transaction costs and rebalancing frequency
affect net-of-cost returns?

---

## Project Structure

├── Data-Driven_Asset_Allocation_Strategies.ipynb
├── README.md
└── data/                  # Downloaded via yfinance at runtime

---

## Asset Universe
14 publicly traded securities spanning five sectors, selected for liquidity, factor diversity, and Kelly stability:

| Ticker | Sector       | Ticker | Sector       |
|--------|--------------|--------|--------------|
| AAPL   | Technology   | SLV    | Commodities  |
| GOOGL  | Technology   | XOM    | Commodities  |
| MSFT   | Technology   | FCX    | Commodities  |
| NEE    | Clean Energy | TLT    | Fixed Income |
| CEG    | Clean Energy | IEF    | Fixed Income |
| ABBV   | Healthcare   | V      | Financials   |
| UNH    | Healthcare   |        |              |
| LLY    | Healthcare   |        |              |

Data source: Yahoo Finance via yfinance. Hourly prices from 2024-12-01 to
2025-12-01. The annual risk-free rate is set to 2.90%, converted to an
equivalent hourly rate.

---

## Methodology

### 1. Data Download & Preprocessing

Hourly closing prices are downloaded for all 20 assets plus SPY as a benchmark.
Forward-fill handles occasional intraday gaps. Hourly returns and excess returns
are computed and aligned to a shared datetime index.

### 2. Kelly Weight Estimator

For each rolling window the Kelly weight vector is estimated via the
mean-variance approximation:

f ∝ Σ⁻¹μ

Covariance is estimated using Ledoit-Wolf shrinkage rather than the sample
covariance matrix, improving stability when the estimation window is close to
the number of assets. Weights are constrained to be long-only, capped at 21%
per asset, and normalized to sum to one.

### 3. Rebalancing Frequency Analysis

Kelly weights are recomputed at a configurable frequency (REBAL_FREQ). Three
frequencies are tested and compared:

| Frequency | Ann. Return | Sharpe |
|-----------|-------------|--------|
| Hourly    | 453%        | 15.6   |
| Daily     | 651%        | 22.2   |
| 48-hour   | 668%        | 23.0   |

### 4. Transaction Costs

A 10 basis point one-way transaction cost is applied at each rebalance based on
portfolio turnover. This reflects realistic trading frictions and is the primary
driver of performance differences across rebalancing frequencies.

### 5. Strategy Evaluation

Three strategies are compared on net-of-cost annualized performance:

| Strategy     | Ann. Return | Ann. Volatility | Sharpe | Sortino | Max Drawdown |
|--------------|-------------|-----------------|--------|---------|--------------|
| Kelly        | 651%        | 29.13%          | 22.24  | 36.38   | -3.18%       |
| Equal Weight | 755%        | 27.99%          | 26.88  | 40.92   | -4.92%       |
| SPY          | 311%        | 25.93%          | 11.88  | 15.85   | -5.29%       |

---

## Key Findings

- Transaction cost drag, not the allocation signal, was the primary performance
  headwind for Kelly. Reducing rebalancing frequency from hourly to daily
  improved annualized returns by ~200 percentage points.
- Equal-weight outperformed Kelly on returns and Sharpe ratio net of costs,
  likely reflecting both lower turnover and a stock universe tilted toward
  large-cap winners where dispersion is limited.
- Kelly achieved the lowest maximum drawdown of all three strategies (-3.18%
  vs -4.92% for equal-weight and -5.29% for SPY), suggesting the allocation
  signal adds value in limiting downside risk even when it trails on returns.
- Both active strategies significantly outperformed the SPY benchmark.

---

## Libraries

yfinance
pandas
numpy
matplotlib
sklearn (LedoitWolf)

Install all dependencies:

pip install yfinance pandas numpy matplotlib scikit-learn

---

## How to Run

1. Clone the repository.
2. Install dependencies (see above).
3. Open Data-Driven_Asset_Allocation_Strategies.ipynb in Jupyter Notebook or
   JupyterLab.
4. Run all cells sequentially. Price data is downloaded automatically from
   Yahoo Finance.

---

## Acknowledgments

Built as part of the Open Avenues Foundation Data-Driven Asset Allocation
project.
