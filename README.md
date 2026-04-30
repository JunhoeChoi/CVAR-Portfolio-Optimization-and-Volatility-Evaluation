# CVaR Portfolio Optimization & Stress Testing

> **Quantitative Risk Management | Python | Portfolio Optimization | Stress Testing**

---

## Overview

This project builds a **CVaR-minimizing portfolio** using linear programming and evaluates its performance against Max-Sharpe and MVO portfolios. It also implements a full volatility forecasting pipeline using GARCH-family models — and rigorously stress tests the optimized portfolio under crisis scenarios.

The core motivation: traditional mean-variance optimization ignores tail risk. This project directly addresses that by minimizing **Conditional Value-at-Risk (CVaR)** at the 95% confidence level using the Rockafellar–Uryasev LP formulation — the industry-standard approach for tail risk optimization.

---

### Skills Demonstrated

- Tail risk optimization (CVaR minimization via linear programming)
- Rolling-window portfolio backtesting and VaR violation tracking
- Risk decomposition (Marginal ES, Component ES, ES Share)
- Volatility forecasting and model selection (GARCH, GJR-GARCH, SVR-GARCH)
- Stress testing under correlated crisis scenarios (Cholesky Monte Carlo, EWMA)
- Portfolio performance benchmarking (CVaR-Min vs. Max-Sharpe)

---

### Tech Stack

- **Python**: numpy, pandas, scipy, matplotlib, yfinance, arch, sklearn
- **Optimization**: `scipy.optimize.linprog` (HiGHS), `scipy.optimize.minimize`
- **Volatility Models**: GARCH(1,1), GJR-GARCH, SVR-GARCH
- **Stress Testing**: Cholesky decomposition, EWMA covariance, Monte Carlo simulation

---

## Portfolio Universe

15 assets across 4 groups:

| Group | Assets |
|---|---|
| Mega-cap Tech / Growth | NVDA, AAPL, MSFT, GOOGL, AMZN, META, AVGO, TSLA |
| Financial | BRK-B, JPM |
| Healthcare (Defensive) | UNH, JNJ |
| Hedge / Cash | BIL, SHV, GLD |

**Data**: Yahoo Finance, 2020-01-01 – 2025-01-01

---

## CVaR Methodology

### CVaR-Based Portfolio Optimization (Linear Programming)
- Minimizes CVaR at the **95% confidence level**
- Uses the **Rockafellar–Uryasev LP formulation** (no parametric assumptions)
- Portfolio weights optimized ex ante based on historical tail behavior
- For the theoretical derivation, see:

[CVaR-Based Portfolio Optimization Theory Note](CVaR_Portfolio_Optimization.pdf)

### Constraints (Long-only)
| Asset Group | Weight Bound |
|---|---|
| Core mega-cap (AAPL, MSFT) | Min 2%, Max 20% |
| High-vol growth (NVDA, TSLA) | Max 15% |
| Large-cap growth (GOOGL, AMZN, META, AVGO) | Max 18% |
| Financial (BRK-B, JPM) | Max 15% |
| Hedge / Cash (BIL, SHV, GLD) | Max 5% |

### Weekly Rebalancing Strategy (26-Week Lookback)
- Weights updated every Friday using past 26 weeks of daily returns
- Daily returns tracked within each week
- Avoids look-ahead bias by excluding current week from estimation window

---

## Results

### CVaR-Min vs. Max-Sharpe Portfolio

| Metric | CVaR-Min | Max-Sharpe |
|---|---|---|
| Annualized Return | **20.32%** | 26.33% |
| Annualized Volatility | **13.64%** | 18.98% |
| Sharpe Ratio | **1.49** | 1.39 |
| VaR (95%) | **1.36%** | 1.84% |
| CVaR / ES (95%) | **1.90%** | 2.67% |

**CVaR-minimizing portfolio achieved a higher Sharpe Ratio (1.49 vs 1.39) with significantly lower tail risk (CVaR 1.90% vs 2.67%) compared to the Max-Sharpe portfolio.**

<img src="plots/Portfolio Optimization/Max sharpe vs CvaR.png">
<img src="plots/Portfolio Optimization/Compounded Return.png">
<img src="plots/Portfolio Optimization/Port Weights by Sectors.png">

---

## Risk Decomposition

### Marginal ES
Measures how much the portfolio's Expected Shortfall changes if the weight of an asset increases infinitesimally — captures tail riskiness independent of current weight.

### Component ES
Measures each asset's actual contribution to portfolio tail loss — incorporates both tail behavior and portfolio weight.

| Asset | Mean Weight | Marginal ES | Component ES | ES Share |
|---|---|---|---|---|
| NVDA | 0.006 | 0.04382 | 0.000282 | 1.48% |
| AAPL | 0.062 | 0.02902 | 0.001785 | 9.40% |
| MSFT | 0.077 | 0.03085 | 0.002463 | 12.97% |
| GOOGL | 0.045 | 0.03096 | 0.001466 | 7.72% |
| AMZN | 0.053 | 0.03582 | 0.001901 | 10.01% |
| META | 0.016 | 0.03300 | 0.000506 | 2.67% |
| AVGO | 0.052 | 0.02684 | 0.001661 | 8.74% |
| TSLA | 0.008 | 0.04211 | 0.000336 | 1.77% |
| BRK-B | 0.150 | 0.01605 | 0.002407 | 12.67% |
| JPM | 0.094 | 0.02052 | 0.002220 | 11.68% |
| UNH | 0.135 | 0.01617 | 0.002237 | 11.78% |
| JNJ | 0.150 | 0.01022 | 0.001533 | 8.07% |
| BIL | 0.050 | -0.00003 | -0.000001 | -0.01% |
| SHV | 0.050 | 0.00002 | 0.000001 | 0.00% |
| GLD | 0.050 | 0.00403 | 0.000202 | 1.06% |
| **Total** | | | **0.018999** | **100%** |

---

## Volatility Forecasting (GARCH Family)

| Model | RMSE |
|---|---|
| GARCH(1,1) | 0.003999 |
| GJR-GARCH | 0.003396 |
| **SVR-GARCH** | **0.002809** |

SVR-GARCH outperformed classical GARCH models in out-of-sample volatility forecasting.

<img src="plots/Portfolio Optimization/GARCH.png">
<img src="plots/Portfolio Optimization/Forecasting_GARCH.png">
<img src="plots/Portfolio Optimization/Forecasting_GJR_GARCH.png">
<img src="plots/Portfolio Optimization/Forecasting_SVR_GARCH.png">

---

## Stress Testing

Two stress testing frameworks applied to the latest CVaR-optimized portfolio weights:

### Framework 1: Monte Carlo (Cholesky Decomposition)
- Simulated **10,000 daily portfolio returns** using Cholesky decomposition of the historical covariance matrix
- Correlated random shocks: `correlated_returns = L @ Z`
- Computed 95% VaR, 99% VaR, and 99% CVaR

<img src="plots/Stress Test results/Monte Carlo Cholesky distribution result.png">

---

### Framework 2: Crisis Stress Scenarios
Four simultaneous stress shocks applied:

| Stress Factor | Assumption |
|---|---|
| Correlation Breakdown | All cross-asset correlations forced to 0.9 |
| Volatility Shock | Asset volatilities doubled |
| EWMA Covariance | Latest covariance via EWMA (λ=0.94) |
| Equity Shock | Additional -10% deterministic shock to equity sector |

Compared Normal vs. Stressed 99% VaR to quantify tail risk amplification under crisis conditions.

#### Stress Test Results

* Variance with Normal distribution
<img src="plots/Stress Test results/EWMA stress test result.png">

* EWMA Variance
<img src="plots/Stress Test results/Stress test result.png">

---
