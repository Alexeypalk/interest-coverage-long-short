# Interest Coverage Long-Short

A sector-neutral, systematic long-short equity strategy that ranks stocks on **interest coverage**, a measure of a firm's ability to service its debt, and tests whether high-coverage firms outperform low-coverage firms on a risk-adjusted basis. The project spans the full research pipeline: point-in-time data construction, signal ranking, portfolio construction, risk budgeting, and performance attribution.

---

## Thesis

**Interest coverage** (EBIT / interest expense) measures how comfortably a company can pay the interest on its debt from operating earnings. It is a core solvency and credit-quality signal: firms with high interest coverage are financially robust, while firms with low coverage carry elevated default and distress risk.

The hypothesis is that this credit-quality signal is priced inefficiently enough to earn a risk-adjusted premium, that going **long high-coverage firms and short low-coverage firms**, neutralized by sector, produces positive alpha. This sits in the "quality" family of systematic factors alongside profitability and low-leverage signals.

---

## Methodology

**Universe.** Four sectors: Technology, Finance, Energy, Pharmaceuticals, drawn from Compustat/CRSP, ~2006–2025, monthly frequency.

**Signal & ranking.** Each month, within each sector, stocks are ranked into deciles on trailing-twelve-month interest coverage. The strategy goes **long the top decile** (high coverage) and **short the bottom decile** (low coverage). Ranking is done **within sector** to neutralize sector exposure, so the strategy isolates the credit-quality signal rather than betting on which industries do well.

**Timing.** Rankings from month *t−1* are used to trade month *t*, so the strategy only ever acts on information available at the point of trading.

**Portfolio construction.** Two constructions are implemented:
1. **Equal-weighted deciles**: a simple, transparent baseline.
2. **Volatility-budgeted long-short**: inverse-volatility weighting within each leg (equal risk contribution), with the overall position scaled to a target dollar volatility using the full long/short covariance (`Var = Var_long + Var_short − 2·Cov`), subject to a leverage cap.

**Cross-sector allocation.** A **Two-Fund Separation** (analytical max-Sharpe) overlay allocates across the four sector strategies using a rolling estimate of the tangency portfolio, `w ∝ Σ⁻¹·E[Rᵉ]`.

**Attribution.** Performance is evaluated against the S&P 500 with a full regression-based attribution: Jensen's alpha and beta (with t-stats and p-values), R², residual volatility, Sharpe ratio, and the **appraisal ratio** (α / σ_residual), which isolates reward per unit of *idiosyncratic* risk, the appropriate lens for a market-neutral long-short strategy.

---

## Data integrity

This project takes point-in-time correctness seriously, because the most common way fundamental backtests overstate performance is by quietly using information before it was available.

- **No look-ahead bias.** Fundamentals are stamped by their **actual public report date** (Compustat `rdq`), not the fiscal period-end date. A quarter ending in June is not treated as knowable until it was actually reported (typically weeks later). This ensures the strategy never trades on undisclosed financials.
- **No survivorship bias.** Returns come from **CRSP with delisting returns included**, so firms that went bankrupt or were delisted contribute their final (often sharply negative) returns rather than silently disappearing. This matters especially for the short leg, which targets exactly the distressed, low-coverage firms most likely to delist.
- **Transaction costs.** A per-rebalance transaction-cost model is applied so reported performance is net of trading frictions, not an idealized paper return.

**On the results:** correcting these biases *lowers* the backtested performance relative to a naive implementation. That is the intended, honest outcome — a realistic estimate of the signal's value is more useful than an inflated one.

---

## Repository structure

```
├── interest_coverage_data_pull_code.ipynb          # Compustat + CRSP pull; builds the point-in-time monthly panel
├── Interest_Coverage_alpha.ipynb                   # Ranking, portfolio construction, risk budgeting, attribution
├── data/                                           # (gitignored) output CSV from the data pull
└── README.md
```

---

## Key results

Reported per sector and for the combined portfolios: annualized return, annualized volatility, Sharpe ratio, maximum drawdown, win rate, average leverage, alpha, beta, and appraisal ratio: all net of transaction costs and benchmarked against the S&P 500.

*(See `strategy.ipynb` for the full performance tables and charts: per-sector cumulative returns, volatility-budgeted portfolio value and leverage paths, the Two-Fund Separation allocation, and the appraisal-ratio attribution.)*

---

## Stack

Python · pandas · NumPy · statsmodels · Matplotlib · Compustat/CRSP (via WRDS)

---

## Limitations & extensions

- **Transaction costs** are modeled as a flat per-side cost; a liquidity-aware (square-root market-impact) model would be more realistic, particularly for the small-cap, low-coverage short leg.
- **Universe** is four sectors; broadening to the full cross-section would test generality.
- **Signal** is a single ratio; combining interest coverage with other quality metrics (profitability, leverage, accruals) would likely improve robustness.
- **Validation** uses the full historical sample; a strict walk-forward / out-of-sample split would further guard against overfitting.

---

*This is a research and educational project, not investment advice.*
