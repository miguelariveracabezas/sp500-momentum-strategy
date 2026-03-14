# S&P 500 Cross-Sectional Momentum Strategy

A fully systematic quant equity strategy backtested over 10 years on the top 50 S&P 500 stocks by market cap. Combines academic momentum signals with institutional-grade portfolio construction and realistic execution simulation.

---

## Strategy Summary

| Parameter | Value |
|-----------|-------|
| **Universe** | Top 50 S&P 500 stocks by market cap |
| **Signal** | 12-1 month cross-sectional momentum |
| **Holdings** | Top 20 momentum winners each month |
| **Sizing** | Mean-variance optimization (Ledoit-Wolf + Black-Litterman) |
| **Rebalance** | Monthly |
| **Max position** | 15% per stock |
| **Benchmark** | SPY (buy-and-hold) |

---

## Methodology

### Signal: 12-1 Month Momentum
At each monthly rebalance, every stock is scored by its cumulative return from 12 months ago to 1 month ago:

Mom_t = P(t-21) / P(t-252) - 1

The 1-month skip avoids short-term reversal (Jegadeesh & Titman, 1993). The top 20 stocks by momentum score are selected for the portfolio.

### Portfolio Sizing: MVO + Ledoit-Wolf + Black-Litterman
**Covariance:** Ledoit-Wolf shrinkage on trailing 252-day returns.

**Expected returns:** Black-Litterman equilibrium returns Pi = delta * Sigma * w_eq, tilted by momentum z-scores:

mu_BL = Pi + alpha * z_mom

**Optimization:** Maximise Sharpe ratio subject to long-only, 15% position cap.

### Execution Costs
- Bid-ask spread: 5bps one-way per position traded
- Market impact: 10bps one-way per position traded
- Applied on absolute weight change (turnover-scaled)

### Risk Management
**Per-position stop-loss:** Exit any stock down >12% from entry price.

**Portfolio circuit breaker:** Liquidate all positions if drawdown exceeds 20%. Resume when drawdown recovers above 10%.

---

## Performance Tearsheet

8-panel output:
1. Cumulative return (log scale) vs SPY
2. Drawdown — strategy vs benchmark
3. Rolling 12M Sharpe ratio
4. Rolling 12M volatility
5. Daily return distribution
6. Monthly returns heatmap (year x month)
7. Monthly portfolio turnover
8. Number of holdings / circuit breaker activity

---

## Setup

```bash
git clone https://github.com/YOUR_USERNAME/sp500-momentum-strategy.git
cd sp500-momentum-strategy
pip install -r requirements.txt
jupyter notebook momentum_strategy.ipynb
```

---

## Dependencies

```
yfinance>=0.2.0
pandas>=2.0
numpy>=1.24
scipy>=1.10
scikit-learn>=1.3
matplotlib>=3.7
jupyter
```

---

## Project Structure

```
sp500-momentum-strategy/
├── momentum_strategy.ipynb    # Main strategy notebook
├── README.md                  # This file
└── requirements.txt           # Python dependencies
```

---

## Limitations

- **Survivorship bias:** Universe uses current S&P 500 constituents — delisted stocks not included
- **Transaction costs:** Estimated; real impact scales with AUM
- **Momentum crashes:** Strategy is exposed to sharp reversals during market recoveries
- **Look-ahead bias:** None — all signals use strictly prior data

---

## References

- Jegadeesh & Titman (1993). *Returns to Buying Winners and Selling Losers.* Journal of Finance.
- Black & Litterman (1992). *Global Portfolio Optimization.* Financial Analysts Journal.
- Ledoit & Wolf (2004). *A well-conditioned estimator for large-dimensional covariance matrices.*
- Daniel & Moskowitz (2016). *Momentum Crashes.* Journal of Financial Economics.
