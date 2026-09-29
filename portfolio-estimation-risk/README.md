# Estimation Risk in Mean-Variance Portfolio Optimization

**Can a portfolio manager trust an optimizer built on historical data?** This project stress-tests classic mean-variance optimization on the 11 U.S. sector ETFs, from how reliably its inputs can be estimated to how it performs with real frictions in a full walk-forward backtest. It ends with a recommendation to a decision-maker.

![Walk-forward strategy vs. benchmarks](images/walk_forward_vs_benchmarks.png)

## Recommendation

**Don't fund the strategy as built.** The unconstrained walk-forward strategy posts a Sharpe ratio of **0.19**, compared with **0.76** for an equal-weight portfolio and **0.85** for SPY. Because nothing limits its leverage, its wealth curve drops below zero twice, which would mean a total loss of capital in live trading. The underlying framework works. Before the strategy is considered again, it needs:

1. **Leverage or long-only constraints.** In this analysis they cost only 19–35% of the theoretical objective and prevent the catastrophic drawdowns.
2. **Shrinking expected returns toward zero** before each re-optimization.
3. **A shrinkage covariance estimator** (e.g., Ledoit-Wolf) to stabilize month-to-month weights and reduce turnover.

## Key findings

- **Expected returns are nearly impossible to pin down.** Even with ten years of daily data, the estimated means carry about 48% error relative to the true values. In a bootstrap, 8 of the 11 sectors' mean returns can't be statistically distinguished from zero.
- **Correlations aren't stable over time.** Rolling estimates show distinct regimes, most visibly around COVID in 2020. This is why the backtest uses a rolling estimation window instead of an expanding one.
- **More assets make the estimates worse.** With one year of data, the estimated covariance matrix degrades as the asset universe grows. It becomes singular once the number of assets equals the number of observations, which pushes required leverage past 1,000,000%.

  ![Estimation error vs. universe size](images/estimation_error_vs_universe_size.png)

- **Risk aversion and volatility only change the scale of positions; correlations and expected returns change which sectors the portfolio favors.** This matches the closed-form solution, and the notebook verifies it numerically.
- **Monthly rebalancing works best.** Across every trading-cost level tested (1–20 bps), monthly rebalancing produces the highest after-cost Sharpe ratio. It balances the cost of trading on noise against holding stale weights.

## Approach

| Section | What it does |
|---|---|
| 1. Data pipeline | Caches daily adjusted prices and checks freshness against the NYSE trading calendar. Sets the sample start from the data itself so every ticker has a complete history. |
| 2. Parameter and model uncertainty | Monte Carlo simulation with known true parameters, CAPM vs. sample moments, bootstrap confidence intervals, rolling-window regime analysis, and a dimension-vs.-sample-size stress test |
| 3. Weight sensitivity | Tracks how optimal weights change with risk aversion, shrunken expected returns, perturbed correlations, and scaled volatility. Adds long-only and leverage-capped constraints solved with SLSQP. |
| 4. Rebalancing and costs | Chronological 60/40 train/test split with no lookahead. Tests rebalancing frequency against trading costs, with weight drift and management fees. |
| 5. Walk-forward backtest | Monthly re-estimation on a rolling 252-day window, a performance tearsheet vs. equal-weight and SPY benchmarks, and the final recommendation |

**Data:** 11 SPDR sector ETFs plus SPY, June 2018 – September 2026 (the start date is set by XLC's 2018 launch), from Yahoo Finance via `yfinance`.

## Running it

```bash
pip install -r requirements.txt
jupyter notebook sector_portfolio_estimation_risk.ipynb
```

Price data downloads automatically on the first run and is cached to `data/`. Results will shift slightly as new trading days are added.

## Tools

Python · pandas · NumPy · SciPy · statsmodels · matplotlib · yfinance · pandas_market_calendars
