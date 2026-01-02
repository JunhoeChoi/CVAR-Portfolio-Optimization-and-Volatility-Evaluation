# CVAR-Portfolio-Optimization-and-Volatility-Evaluation
CVaR(Conditional Value at Risk) Minimizing Portfolio verses Max Sharpe Ratio Verses MVO portfolio. Check risk metrics with VaR, ES and GARCH modeling. Moreover, conducted volatility forecasting.

### Data
The portfolio consist of 12 major U.S. equities with defensive and diversifying assets such as cash, short-duration Treasuries, and gold.

* All data is from Yahoo Finance.
* Data Period: 2020-01-01 - 2025-01-01


## CVaR Methodology

### CVaR-Based Portfolio Optimization (Linear Programming)
* Construct a portfolio that minimizes Conditional Value-at-Risk (CVaR) at the 95% confidence level.
* Focus on downside tail risk.

### Framework
* Define portfolio losses as the negative of asset returns.
* Use the Rockafellar–Uryasev linear programming formulation of CVaR.
* Estimate one-shot CVaR optimization using the full empirical distribution of daily returns (no parametric assumptions).
* Portfolio weights are optimized ex ante based on historical tail behavior.

### Constraints
Asset-level weight bounds (Long-only portfolio)
* Core mega-cap stocks (e.g., AAPL, MSFT): Min 2%, Max 20%
* High-volatility growth stocks (e.g., NVDA, TSLA): Max 15%
* Large-cap growth stocks (e.g., GOOGL, AMZN, META, AVGO): Max 18%
* Financial stocks: cMax 15%
* Cash, short-duration Treasuries, and gold: Max 5%
* others : Max 15%

### Weekly Rebalancing CVaR Strategy (26-Week Lookback, Daily Return)
* This section implements a weekly rebalanced CVaR-minimizing strategy.
* This produces a daily return series for a strategy whose weights are updated weekly.
* At each rebalance date (Friday), the model:
    * estimates CVaR-minimizing weights using the past 26 weeks of daily returns, 
    * applies those weights to construct the next week’s portfolio returns on a daily basis (i.e., daily return within the week).
* Weights update weekly, but returns are measured daily.
<br>
<br>
**Why Weekly?** 
* More precise volatility / drawdown behavior analysis,
* better alignment with daily risk reporting.
