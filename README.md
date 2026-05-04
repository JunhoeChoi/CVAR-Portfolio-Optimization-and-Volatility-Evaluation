# CVaR Portfolio Optimization & Stress Testing

> **Quantitative Risk Management | Python | Portfolio Optimization | Stress Testing**

---

## Overview

Built a full quantitative portfolio management pipeline in Python — from tail-risk optimization through risk attribution, volatility forecasting, and stress testing — across a 15-asset U.S. multi-asset universe (2020–2025). Implemented and backtested three competing strategies (CVaR minimization, Max Sharpe, MVO) under an identical weekly rebalancing framework with strict look-ahead bias controls. Validated risk models using GARCH-family volatility models, VaR backtesting, and Monte Carlo stress testing with Cholesky decomposition.

---

## Executive Summary
 
This project constructs and evaluates a **CVaR-minimizing portfolio** across a 15-asset U.S. multi-asset universe (2020–2025), benchmarked against Max Sharpe and mean-variance strategies. Portfolio weights are re-optimized weekly using a 26-week rolling window via the Rockafellar–Uryasev linear programming formulation, with strict look-ahead bias controls. Over the 5-year backtest — spanning COVID-19, the 2022 Fed rate shock, and the 2023–24 AI-driven rally — the CVaR strategy delivered a **Sharpe ratio of 1.49** and **annualized volatility of 13.64%**, outperforming the Max Sharpe benchmark on both metrics despite lower absolute returns (20.32% vs. 26.33%).
 
Risk is measured and validated at multiple layers: historical VaR/CVaR backtesting, GARCH-family conditional volatility models (GARCH(1,1), GJR-GARCH, SVR-GARCH), and a Monte Carlo stress testing framework using Cholesky decomposition. The SVR-GARCH hybrid achieves the best out-of-sample volatility forecast accuracy (RMSE: 0.002809), reducing error by ~30% versus the GARCH(1,1) baseline. Under a fully stressed scenario — correlation breakdown to 0.90, 2× volatility scaling, and a −10% equity shock — the 99% portfolio VaR widens from −2.58% to −12.06%, quantifying the tail amplification that historical VaR alone cannot capture.

---

## Motivation
 
Tail-risk management is a core concern for institutional investors, risk managers, and regulators, particularly after the 2008 financial crisis accelerated the industry's shift from variance-based to shortfall-based risk measures. This project implements a **full production-style pipeline** — from optimization through risk attribution, volatility forecasting, and stress testing — to answer a concrete question:
 
* Does explicitly minimizing CVaR at the portfolio construction stage deliver meaningfully better risk-adjusted outcomes than Sharpe-based or mean-variance approaches, over a backtest covering COVID-19, the 2022 rate shock, and the 2023–24 equity rally?

---

### Skills Demonstrated

- Tail risk optimization (CVaR minimization via linear programming)
- Rolling-window portfolio backtesting and VaR violation tracking
- Risk decomposition (Marginal ES, Component ES, ES Share)
- Volatility forecasting and model selection (GARCH, GJR-GARCH, SVR-GARCH)
- Stress testing under correlated crisis scenarios (Cholesky Monte Carlo, EWMA)
- Portfolio performance benchmarking (CVaR-Min vs. Max-Sharpe vs. MVO)

---

### Tech Stack

- **Python**: numpy, pandas, scipy, matplotlib, yfinance, arch, sklearn
- **Optimization**: `scipy.optimize.linprog` (HiGHS), `scipy.optimize.minimize`
- **Volatility Models**: GARCH(1,1), GJR-GARCH, SVR-GARCH
- **Stress Testing**: Cholesky decomposition, EWMA covariance, Monte Carlo simulation

---

## 1. Assets

15 assets across 4 groups:

| Group | Assets |
|---|---|
| Mega-cap Tech / Growth | NVDA, AAPL, MSFT, GOOGL, AMZN, META, AVGO, TSLA |
| Financial | BRK-B, JPM |
| Healthcare (Defensive) | UNH, JNJ |
| Hedge / Cash | BIL, SHV, GLD |

**Data**: Yahoo Finance, 2020-01-01 – 2025-01-01

---

## 2. CVaR Minimization via Linear Programming

The portfolio is optimized by minimizing 95% Conditional Value-at-Risk (Expected Shortfall) using the **Rockafellar–Uryasev (2000)** LP reformulation. No parametric distributional assumptions are made; the full empirical return distribution is used directly.

### 2.1. Key portfolio design choices
- Minimizes CVaR at the **95% confidence level**
- Uses the **Rockafellar–Uryasev LP formulation** (no parametric assumptions)
- Portfolio weights optimized ex ante based on historical tail behavior
- For the theoretical derivation, see:

[CVaR-Based Portfolio Optimization Theory Note](CVaR_Portfolio_Optimization.pdf)

### 2.2. Constraints (Long-only)
| Asset Group | Weight Bound |
|---|---|
| Core mega-cap (AAPL, MSFT) | Min 2%, Max 20% |
| High-vol growth (NVDA, TSLA) | Max 15% |
| Large-cap growth (GOOGL, AMZN, META, AVGO) | Max 18% |
| Financial (BRK-B, JPM) | Max 15% |
| Hedge / Cash (BIL, SHV, GLD) | Max 5% |

### 2.3. Weekly Rebalancing Strategy (26-Week Lookback)
- **Look-ahead bias prevention:** At each Friday rebalance, weights are estimated using only the prior 26 weeks of daily returns. The resulting weights are applied forward to the next week's daily returns.
- **Rebalancing frequency:** Weekly (Friday), generating a daily return series for portfolio analysis.

**Benchmark:** Max Sharpe and Mean-Variance portfolio built with the identical rebalancing framework and constraints.

### 2.4. Portfolio Weight Dynamics (Weekly Rebalancing)

#### Dynamics of CVaR Portfolio Weights
<img src="plots/Portfolio Optimization/Portfolio Weight Dynamics.png">

*CVaR optimizer consistently overweights defensive names (UNH, JNJ, BRK-B) and rotates away from high-vol growth during stressed periods.*

#### Optimal Asset Allocation by CVaR Optimization
<img src="plots/Portfolio Optimization/CVaR Optimal Asset Allocation.png">



---

## 3. Backtest Results

### 3.1. CVaR-Min vs. S&P 500

#### Cumulative Return vs. S&P 500
<img src="plots/Portfolio Optimization/Cumulative Return of CVaR and SnP.png">

### 3.2. Strategy Comparison 

#### CVaR-Min vs. Max-Sharpe vs. MVO Portfolio

| Metric | CVaR-Min | Max Sharpe | MVO |
|---|---|---|---|
| Annualized Return | 20.32% | **26.33%** | 17.84% |
| Annualized Volatility | 13.64% | 18.98% | **13.30%** |
| Sharpe Ratio | **1.49** | 1.39 | 1.34 |
| Average VaR (95%) | 1.99% | **1.84%** | 1.95% |
| Average CVaR / ES (95%) |2.36% | 2.67% | **2.27%** |

CVaR-Min delivers the **best risk-adjusted return** (Sharpe **1.49** vs. 1.39 vs. 1.34) with substantially lower tail risk than Max Sharpe (CVaR: 2.36% vs. 3.22%). While MVO achieves marginally lower average VaR (1.95% vs. 1.99%) and CVaR (2.27% vs. 2.36%), it breaches the VaR threshold significantly more often — 18 exceptions vs. **13** over the same 886-day window. 

This distinction matters: MVO controls average variance well but leaves the tail inadequately managed, whereas CVaR-Min explicitly targets the loss distribution beyond the threshold, resulting in fewer extreme-day breaches. Taken together, CVaR-Min demonstrates that explicitly targeting the tail at the optimization stage produces more robust downside protection than variance minimization or Sharpe maximization alone.

<img src="plots/Portfolio Optimization/Max Sharpe vs CvaR vs MVO.png">

---

## 4. Risk Decomposition

For each rebalancing period, the portfolio's Expected Shortfall is decomposed at the asset level using finite-difference numerical differentiation:

- ** Marginal ES: ** Measures how much the portfolio's Expected Shortfall changes if the weight of an asset increases infinitesimally — captures tail riskiness independent of current weight.

- **Component ES:** Measures each asset's actual contribution to total portfolio ES — incorporates both tail behavior and portfolio weight.

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

## 5. VaR Backtesting
 
#### Historical VaR — CVaR-Min Portfolio (99%, 250-day window)
<img src="plots/Portfolio Optimization/99 percent historical VaR (CVaR Opt).png">
 
#### Historical VaR — Max Sharpe Portfolio (99%, 250-day window)
<img src="plots/Portfolio Optimization/99 percent historical VaR (Max Sharpe).png">
 
#### Historical VaR — MVO Portfolio (99%, 250-day window)
<img src="plots/Portfolio Optimization/99 percent historical VaR (MVO).png">

*MVO's VaR line sits between CVaR-Min and Max Sharpe — variance minimization reduces overall volatility but leaves the tail exposure less tightly controlled than explicit CVaR targeting.*

## 6. Volatility Modeling & GARCH-Based VaR

Three volatility models are fitted on the CVaR portfolio's daily return series:

| Model | Specification | Key Feature |
|---|---|---|
| **GARCH(1,1)** | Constant mean, Student-t errors | Baseline; captures volatility clustering |
| **GJR-GARCH** | BIC-selected (p,q), normal errors | Asymmetric response — larger vol spike on negative shocks (leverage effect) |
| **SVR-GARCH** | SVR mean + GJR-GARCH on residuals, Student-t errors | Hybrid ML/econometric; separates mean and variance estimation |
 
All models validated with the **Kupiec Proportion of Failures (POF) test**.

#### GARCH(1,1) Estimated Volatility — CVaR vs. Max Sharpe
<img src="plots/Portfolio Optimization/GARCH.png">

*CVaR portfolio (orange) consistently exhibits lower GARCH-estimated volatility than the Max Sharpe portfolio (blue), particularly during the COVID spike (2020) and rate shock (2022).*

**Result Summary**

| Model          | Backtest Days | Exceptions | Avg. VaR | Avg. CVaR |
| -------------- | ------------: | ---------: | -------: | --------: |
| Historical VaR |           886 |         13 |    1.99% |     2.36% |
| GARCH(1,1) VaR |         1,135 |         19 |    1.84% |     2.12% |
| GJR-GARCH VaR  |         1,135 |         17 |    1.84% |       N/A |
| SVR-GARCH VaR  |         1,135 |         13 |    2.01% |     3.03% |

#### GARCH(1,1) VaR Backtesting
<img src="plots/Portfolio Optimization/99 percent GARCH(1,1)-VaR.png">
 
#### GJR-GARCH VaR Backtesting
<img src="plots/Portfolio Optimization/99 percent gjr-GARCH VaR.png">
 
#### SVR-GARCH VaR Backtesting
<img src="plots/Portfolio Optimization/99 percent SVR-GARCH VaR.png">
 

### 7. Volatility Forecasting (Out-of-Sample RMSE)

| Model | RMSE |
|---|---|
| GARCH(1,1) | 0.003999 |
| GJR-GARCH | 0.003396 |
| **SVR-GARCH** | **0.002809** |

*SVR-GARCH reduces forecast error by ~30% vs. GARCH(1,1). The gain comes from better conditional mean estimation: removing SVR-predicted returns before fitting the variance model lets GARCH capture pure volatility dynamics more cleanly.*

#### GARCH(1,1) Forecast vs. Realized Volatility
<img src="plots/Portfolio Optimization/Forecasting_GARCH.png">
 
#### GJR-GARCH Forecast vs. Realized Volatility
<img src="plots/Portfolio Optimization/Forecasting_GJR_GARCH.png">
 
#### SVR-GARCH Forecast vs. Realized Volatility
<img src="plots/Portfolio Optimization/Forecasting_SVR_GARCH.png">

---

## 8. Monte Carlo Stress Testing

A 15-asset stress testing framework using Cholesky decomposition on 10,000 simulated return paths across two covariance regimes:

Stressed scenario (4-stage shock):

Correlation breakdown: All pairwise correlations forced to 0.90 — replicates crisis-era contagion (e.g., March 2020, October 2008)
Volatility scaling: Asset-level volatility doubled (2× historical)
Deterministic equity shock: −10% injected into all 10 equity positions
EWMA variant: Steps 1–3 repeated using EWMA covariance (λ = 0.94) for a more regime-responsive baseline

### Baseline Monte Carlo: 99% VaR = −2.67%, CVaR = −3.15%

- Simulated **10,000 daily portfolio returns** using Cholesky decomposition of the historical covariance matrix
- Correlated random shocks: `correlated_returns = L @ Z`
- Computed 95% VaR, 99% VaR, and 99% CVaR

#### Result

| Metric | CVaR | MVO |
|---|---|---:|
| Monte Carlo VaR (95%) | 1.89% | 1.86% |
| Monte Carlo VaR (99%) | 2.67% | 2.68% |
| Monte Carlo CVaR (99%) | 3.15% | 3.15% |

**CVaR**
<img src="plots/Stress Test results/Simulated Portfolio return (CVaR).png">

**MVO**
<img src="plots/Stress Test results/Simulated Portfolio return (MVO).png">


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

| Scenario | 99% Stress VaR | 99% Baseline Monte Carlo VaR | Interpretation |
|---|---:|---:|:---:|
| Benchmark Stress Test | 12.85% | 2.67% | Stress scenario using historical volatility before applying stress factors |
| EWMA Volatility-Based Stress Test | 11.08% | 2.67% | Stress scenario with EWMA volatility to handle recent volatility dynamics |

The stress-test results show that portfolio downside risk increases materially under stressed covariance assumptions. The baseline 99% Monte Carlo VaR is 2.67%, while the estimated 99% Stress VaR rises to 12.85% under the stressed Monte Carlo Cholesky scenario and 11.08% under the EWMA-based stress scenario. This indicates that tail losses become significantly larger when the portfolio return distribution is shifted under stressed volatility and correlation conditions.

* Historical Volatility-Based Stress Test
<img src="plots/Stress Test results/Monte Carlo Cholesky distribution result.png">

* EWMA Volatility-Based Stress Test
<img src="plots/Stress Test results/EWMA stress test result.png">

---

## Appendix:
Using lastest portfolio optimized weights, invest in three portfolios. 
- S&P 500
- CVaR Min
- MVO

Result

<img src="plots/Stress Test results/Cumulative Return of CVaR vs MVO vs SnP (2025).png">
