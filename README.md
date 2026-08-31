# Rolling Mean-Variance Asset Allocation

Open Avenues Foundation — Data-Driven Asset Allocation Strategies

---

## Research Question

Does mean-variance optimization with covariance shrinkage improve risk-adjusted
portfolio performance over naive equal-weight allocation, once transaction costs
are accounted for and the result is validated out of sample?

**Short answer: no.** On this sample the optimized portfolio matched equal weight
on Sharpe and underperformed it on drawdown, both in sample and out.

---

## Project Structure
├── Kelly_Asset_Allocation.ipynb
├── README.md
└── prices_hourly.parquet # cached price data, created on first run



---

## Asset Universe

14 securities across six sectors, chosen for liquidity and factor diversity:

| Ticker | Sector       | Ticker | Sector       |
|--------|--------------|--------|--------------|
| AAPL   | Technology   | SLV    | Commodities  |
| GOOGL  | Technology   | CPER   | Commodities  |
| MSFT   | Technology   | FCX    | Commodities  |
| NEE    | Clean Energy | TLT    | Fixed Income |
| CEG    | Clean Energy | IEF    | Fixed Income |
| ABBV   | Healthcare   | V      | Financials   |
| UNH    | Healthcare   |        |              |
| LLY    | Healthcare   |        |              |

Hourly closing prices from Yahoo Finance via `yfinance`, 2024-12-01 to 2025-12-01
(1,731 bars). Risk-free rate of 2.90% annual, converted to an hourly rate over
1,638 trading hours per year.

---

## Methodology

**Data.** Hourly closes for the 14 assets plus SPY as benchmark. Forward-fill
handles intraday gaps; downloads are validated for missing or all-NaN columns and
cached to parquet.

**Weight estimator.** For each rolling 504-hour window (~3 weeks), weights are
estimated as w ∝ Σ⁻¹μ


with Σ estimated via Ledoit-Wolf shrinkage rather than the sample covariance
matrix, which is near-singular at 504 observations across 14 assets. Weights are
constrained long-only, capped at 15% per asset, and normalized to sum to one.

Note: normalizing to full investment discards the magnitude of Σ⁻¹μ, which is what
distinguishes the Kelly Criterion — the total exposure it prescribes. What this
implements is the long-only capped tangency portfolio, not Kelly.

**Costs.** 10bps one-way, charged on turnover at each weekly rebalance.

**Validation.** Walk-forward across 13 folds, training on ~6 months and testing on
the following period, with identical mechanics for both strategies.

---

## Results

**In-sample** (net of costs, annualized):

| Strategy      | Return | Volatility | Sharpe | Sortino | Max Drawdown |
|---------------|--------|------------|--------|---------|--------------|
| Mean-variance | 31.24% | 16.31%     | 1.738  | 2.549   | −14.86%      |
| Equal weight  | 30.81% | 15.92%     | 1.753  | 2.602   | −12.80%      |
| SPY           | 28.02% | 18.77%     | 1.338  | 1.981   | −14.24%      |

**Out-of-sample** (13 walk-forward folds):

| Strategy      | Return | Volatility | Sharpe | Sortino |
|---------------|--------|------------|--------|---------|
| Mean-variance | 48.75% | 14.01%     | 3.273  | 4.916   |
| Equal weight  | 44.39% | 11.32%     | 3.665  | 5.789   |

Optimized beat equal weight in 8 of 13 folds (62%), which is not distinguishable
from chance (p = 0.29). Out-of-sample Sharpes are not comparable to the in-sample
table — the folds cover a later, lower-volatility stretch of the sample.

---

## Key Findings

**The optimization added no risk-adjusted value.** Equal weight matched or beat it
on Sharpe, Sortino, and max drawdown in sample, and on pooled Sharpe out of sample.
The 43bps of extra raw return came with higher volatility and two additional points
of drawdown.

**It wins often but loses big.** The optimized portfolio outperformed in 62% of
out-of-sample folds while trailing on pooled Sharpe — frequent small wins offset by
larger losses.

**Diversification, not weighting, drove the benchmark gap.** Both portfolios beat
SPY by roughly 3 points of return at lower volatility. That came from the asset mix
spanning equities, commodities, and Treasuries, and is unaffected by how the weights
within it are chosen.

**An earlier version of this project reported a Sharpe of 1.938.** That result came
from a bug in the weight-capping routine: assets rejected by the long-only
constraint had zero weight and therefore maximum slack, so they received the largest
share of redistributed capital. Roughly 4.4% of the portfolio went to positions the
optimizer had explicitly declined. Fixing it eliminated the apparent edge.

**This is the expected outcome.** With 504 observations and 14 assets, the
covariance matrix has 105 free parameters. Mean-variance optimization is known to
amplify estimation error in μ, and Ledoit-Wolf shrinkage stabilizes Σ but does
nothing for the mean estimates — the noisier input. Equal weight estimates nothing,
which is why it is hard to beat on short samples.

---

## Limitations

One year of hourly data through a sustained bull market. Flat 10bps costs with no
slippage or market impact. No tax modeling; weekly rebalancing in a taxable account
would generate short-term gains that materially reduce after-tax returns.

## Next Steps

Extend to a full market cycle including a drawdown — this requires daily rather than
hourly bars, since `yfinance` serves roughly two years of hourly history. Test
sensitivity to window length and rebalance frequency, neither of which was tuned.
Add minimum-variance and risk-parity baselines, which drop the μ estimate entirely
and would isolate how much of any edge comes from the return forecast versus the
covariance structure.

---

## Setup

```bash
pip install yfinance pandas numpy matplotlib scikit-learn pyarrow
```

Open `Kelly_Asset_Allocation.ipynb` and run all cells. Price data downloads on first
run and caches to `prices_hourly.parquet`.

---

## Acknowledgments

Built as part of the Open Avenues Foundation Data-Driven Asset Allocation project.
