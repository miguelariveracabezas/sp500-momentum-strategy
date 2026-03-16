# S&P 500 Momentum Strategy

A research-grade cross-sectional equity momentum backtest built in Python and implemented in Jupyter.

The project studies whether a constrained large-cap momentum portfolio can outperform SPY after incorporating realistic frictions such as transaction costs, turnover limits, and sector concentration controls. The final notebook also includes walk-forward out-of-sample validation, Fama-French factor attribution, reconciliation checks, and a literature-based survivorship-bias estimate.

## Project status

**Current flagship notebook:** `momentum_strategy_v4.ipynb`

This is the strongest version of the project and the one intended for review. Earlier notebooks are retained to show the research progression from prototype to more defensible backtest.

## Strategy overview

- **Universe:** 100 large-cap U.S. equities in a current S&P 500-style universe
- **Benchmark:** SPY
- **Signal:** Vol-adjusted 12-1 momentum
- **Selection:** Top 20 names by signal at each rebalance
- **Rebalance frequency:** Quarterly
- **Sizing methods tested:**
  - Mean-variance optimization using Ledoit-Wolf covariance shrinkage and a Black-Litterman-style expected return prior
  - Equal weight
  - Inverse volatility
- **Constraints:**
  - 25% sector cap
  - 15% max position weight
  - 50% max one-way turnover per rebalance
- **Risk controls:**
  - 15% stop-loss
  - transaction cost model with square-root market impact
- **Validation layers:**
  - internal return-series reconciliation
  - walk-forward out-of-sample testing
  - Fama-French 6-factor regression
  - survivorship-bias quantification

## Why this project matters

The point of the project is not just to show a high return number. It is to show the full research process:

- formulate a signal
- build a portfolio construction engine
- model implementation frictions
- test robustness out of sample
- decompose performance into systematic factors
- identify where the apparent edge weakens under scrutiny

That is the real value of the notebook.

## Headline results from `momentum_strategy_v4.ipynb`

### In-sample / full-period summary

| Metric | Strategy | Bias-Adjusted | SPY |
|---|---:|---:|---:|
| Annual return | 13.23% | 12.97% | 10.66% |
| Sharpe ratio | 0.43 | 0.42 | 0.33 |
| FF6 alpha | 5.56% | 5.31% | -0.50% |
| FF6 alpha t-stat | 1.52 | 1.52 | -1.82 |
| Information ratio | 0.37 | 0.37 | -0.23 |
| Market beta | 0.737 | — | 0.994 |
| Momentum loading | 0.428 | — | -0.011 |
| Estimated survivorship bias | 0.25%/yr | — | — |

### Walk-forward out-of-sample summary

| Metric | OOS MVO |
|---|---:|
| Annual return | 20.38% |
| Sharpe ratio | 0.73 |
| Max drawdown | -34.81% |
| Test windows | 28 |
| Positive alpha windows | 61% |

## Interpretation

The final notebook produces encouraging performance, but the most important conclusion is that the strategy is **not yet proven to generate statistically significant standalone alpha**.

The Fama-French 6-factor regression shows:

- annualized alpha of **5.56%**
- **t-statistic of 1.52**
- roughly **49% of return variance explained by FF6 factors**

That means the strategy is interesting and reasonably well engineered, but the results are still substantially explained by systematic factor exposure rather than clearly isolated alpha.

## Methodology

### 1. Data
Daily adjusted close data is downloaded with `yfinance` from **2008-01-01 to 2025-01-01**.

### 2. Signal construction
The core signal is 12-1 momentum:

- long lookback: 252 trading days
- skip month: 21 trading days

The project then volatility-adjusts the raw momentum score using trailing realized volatility to reduce crash sensitivity.

### 3. Portfolio construction
At each rebalance date, the strategy:

1. ranks the universe on the momentum signal
2. selects the top 20 names
3. estimates the covariance matrix with Ledoit-Wolf shrinkage
4. forms expected returns using a Black-Litterman-style prior plus momentum tilt
5. solves for constrained weights under position, turnover, and sector limits

Equal-weight and inverse-volatility portfolios are included as comparison baselines.

### 4. Transaction costs
Implementation costs are not ignored. The backtest includes:

- bid-ask style slippage assumptions
- square-root market impact
- turnover-aware rebalancing

### 5. Validation
The notebook includes several layers of validation:

- cumulative NAV and return-series reconciliation
- walk-forward training and testing splits
- factor regression on FF6 daily factors
- literature-based survivorship-bias adjustment

## Known limitations

This section matters as much as the performance table.

1. **Survivorship bias is not fully eliminated.** The universe is a current large-cap S&P 500-style basket, not a true point-in-time constituent history from CRSP/Compustat/FactSet.
2. **The Black-Litterman implementation is hybrid rather than fully formal.** It is better described as a Black-Litterman-style expected return framework than a textbook posterior setup.
3. **Transaction costs are stylized.** They are more realistic than zero-cost backtests, but still not a full execution model.
4. **Momentum crash risk remains.** Vol-adjustment helps, but does not eliminate reversal risk.
5. **The sector cap logic should be treated carefully.** The notebook improves concentration control materially, but the implementation can still be tightened further.
6. **Results are research outputs, not live-trading claims.**

## Repository structure

```text
.
├── momentum_strategy.ipynb        # initial prototype
├── momentum_strategy_v2.ipynb     # improved architecture and stronger headline performance
├── momentum_strategy_v3.ipynb     # extended backtest and robustness improvements
├── momentum_strategy_v3(1).ipynb  # near-final variant
├── momentum_strategy_v4.ipynb     # flagship notebook / best overall version
└── README.md
```

## Running the notebook

### 1. Clone the repository

```bash
git clone <your-repo-url>
cd <your-repo-folder>
```

### 2. Install dependencies

```bash
pip install pandas numpy yfinance matplotlib scipy scikit-learn statsmodels jupyter
```

### 3. Launch Jupyter

```bash
jupyter notebook
```

Then open:

```text
momentum_strategy_v4.ipynb
```

## Best way to present this project

For recruiting or academic review, the strongest framing is:

- independent quantitative research project
- full backtest pipeline with realistic frictions
- explicit robustness checks
- honest assessment of where the signal weakens

The weakest framing is claiming that this notebook has already discovered deployable alpha.

## Next improvements

The highest-value next steps are:

- replace the current universe with true point-in-time S&P 500 constituent history
- harden the sector cap implementation so it is strictly binding after renormalization
- formalize the Black-Litterman posterior construction
- expand robustness testing across alternative universes and parameter sets
- convert the notebook into modular research scripts and reusable functions

## Disclaimer

This repository is for research and educational purposes only. It does not constitute investment advice, an offer to manage capital, or evidence of live tradable performance.
