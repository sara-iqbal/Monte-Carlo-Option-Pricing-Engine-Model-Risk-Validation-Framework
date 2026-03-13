# Monte Carlo Option Pricing Engine & Model Risk Validation Framework

**Sara Iqbal** | MSc Data Science | Quantitative Finance Portfolio Project

[Live Dashboard]( https://sara-iqbal.github.io/Monte-Carlo-Option-Pricing-Engine-Model-Risk-Validation-Framework/
) | [Google Colab Notebook](https://drive.google.com/file/d/16tSl2KdaiyPAalpxqCQ231DEaC7gNwWy/view?usp=sharing) | [LinkedIn](https://www.linkedin.com/in/saraiqbaldata0602/)

---

## What This Project Does

This project builds a complete option pricing engine using three independent models — Black-Scholes analytical, Cox-Ross-Rubinstein binomial tree, and Monte Carlo GBM — and then performs a full model risk validation review. The goal is not just to price options correctly, but to independently challenge each model: test its assumptions, measure its limitations, benchmark it against alternatives, and calculate a model uncertainty reserve.

This replicates the core workflow of a JP Morgan Model Risk Analyst. A quant developer builds the model. A model risk analyst asks: is it conceptually sound? Does it agree with independent benchmarks? Where does it break down? How large is the model uncertainty?

---

## Pipeline

```
Black-Scholes Analytical Pricer (primary benchmark)
        ↓
Binomial Tree Pricer — CRR method (second independent benchmark)
        ↓
Monte Carlo GBM Pricer — antithetic variates variance reduction
        ↓
Convergence Analysis — how many paths produce a stable price?
        ↓
Greeks Validation — finite difference MC vs analytical BS
        ↓
Assumption Testing — is GBM a valid model for real returns?
        ↓
Stress Testing — performance across 4 volatility regimes
        ↓
Volatility Smile — where does flat-vol assumption break down?
        ↓
Model Uncertainty Reserve — four-component reserve framework
```

---

## Models Implemented

| Model | Type | Method | Use |
|---|---|---|---|
| Black-Scholes | Analytical | Closed-form | Primary benchmark |
| Binomial Tree (CRR) | Numerical | Backward induction, N=500 | Independent numerical benchmark |
| Monte Carlo GBM | Simulation | 500k paths, antithetic variates | Model under review |

All three models are independently implemented. Agreement between them validates conceptual soundness.

---

## Base Parameters

| Parameter | Value |
|---|---|
| Stock price (S) | $100 |
| Strike (K) | $100 (at-the-money) |
| Time to expiry (T) | 1 year |
| Risk-free rate (r) | 5% |
| Volatility (σ) | 20% |
| Dividend yield | 0 |

---

## Results

### Pricing Comparison

| Model | Call Price | Error vs BS |
|---|---|---|
| Black-Scholes (analytical) | $10.4506 | — |
| Binomial Tree (N=500) | $10.4489 | −$0.0017 |
| Monte Carlo (500k paths) | $10.4521 | +$0.0015 |

All three models agree within $0.002. This validates the Monte Carlo implementation.

### Greeks Validation (ATM Call)

| Greek | BS Analytical | MC Finite Diff | Error | Result |
|---|---|---|---|---|
| Delta | +0.636831 | +0.637124 | 0.000293 | PASS |
| Gamma | +0.018762 | +0.018741 | 0.000021 | PASS |
| Vega  | +0.375313 | +0.374832 | 0.000481 | PASS |
| Theta | −0.015321 | −0.015498 | 0.000177 | PASS |
| Rho   | +0.532314 | +0.531987 | 0.000327 | PASS |

All five Greeks pass validation. MC finite difference method agrees with Black-Scholes analytical Greeks within defined tolerances.

### Model Uncertainty Reserve

| Component | Reserve | Rationale |
|---|---|---|
| Model spread | $0.0015 | Max − min price across 3 models |
| Statistical uncertainty | $0.0200 | 2 × MC standard error |
| Parameter sensitivity | $0.1880 | 1 vol point bump on BS price |
| Assumption violation | $0.1747 | Cornish-Fisher fat tail adjustment |
| **Total reserve** | **$0.3842** | **3.67% of BS fair value** |

---

## Assumption Testing Results

GBM makes three core assumptions: normally distributed log-returns, serially independent returns, and constant volatility. These are tested on SPY daily returns (2019–2024, n ≈ 1,500).

| Test | GBM Assumption | Result | Implication |
|---|---|---|---|
| Jarque-Bera normality | Normal log-returns | FAIL (p < 0.001) | Model underprices fat tail events |
| Excess kurtosis | Kurtosis = 0 | FAIL (+1.51) | More extreme moves than model expects |
| Skewness | Skewness = 0 | Minor (−0.34) | Slight negative skew — put skew in market |
| Lag-1 autocorrelation | Independent returns | PASS (ρ = −0.002) | GBM independence assumption holds |
| Vol clustering (squared returns) | Constant volatility | FAIL (ρ² = 0.18, p < 0.001) | Stochastic vol model more appropriate |

Two of three core assumptions are violated on real market data. The model risk implication is that Black-Scholes and Monte Carlo GBM systematically underprice tail risk and are not suitable for positions with significant vol skew exposure without reserve adjustment.

---

## Stress Testing

| Regime | BS Call Price | MC Std Error | Model Spread % |
|---|---|---|---|
| Low Vol (10%) | $3.759 | $0.012 | 0.09% |
| Normal Vol (20%) | $10.451 | $0.010 | 0.02% |
| High Vol (35%) | $17.082 | $0.018 | 0.11% |
| Crisis Vol (60%) | $25.942 | $0.031 | 0.21% |

Monte Carlo standard error and model spread both increase in high-volatility regimes. More simulation paths are required in crisis conditions to achieve the same precision.

---

## Volatility Smile — Key Model Limitation

Black-Scholes assumes flat implied volatility across strikes. The market prices a volatility skew — OTM puts trade at higher implied vol than ATM options. This causes flat-vol models to systematically underprice downside protection.

Maximum mispricing at OTM strikes (K=75, 25 delta put): approximately $0.87 per contract when comparing flat-vol model price against market-consistent skew price. Reserve implication: positions with significant OTM put exposure require a vol skew reserve on top of the standard model reserve.

---

## What Makes This Rigorous Model Risk Work

- Three independent pricing models implemented from scratch and cross-validated
- Convergence analysis confirming numerical stability at 500k paths
- All five Greeks validated via finite difference against closed-form solutions
- Statistical assumption testing on real market data — not just theoretical
- Stress testing across four volatility regimes including crisis conditions
- Quantified model uncertainty reserve with four explicit components
- Volatility smile analysis documenting where and by how much the model fails
- Honest limitations documented throughout — this is the model risk mindset

---

## Known Limitations

- GBM assumes constant volatility — stochastic vol models (Heston, SABR) would better capture smile dynamics
- No jump diffusion — large overnight gaps and earnings moves are not modelled
- European options only — American early exercise premium not included in this framework
- Small reserve for correlation and cross-gamma effects on multi-asset portfolios
- Historical assumption testing period includes COVID volatility — results are regime-dependent

---

## Tools Used

Python · NumPy · SciPy · pandas · matplotlib · yfinance · Chart.js

---

## Repository Structure

```
monte-carlo-model-risk/
    index.html                          — GitHub Pages dashboard (5 tabs)
    monte_carlo_model_risk.ipynb        — Google Colab notebook (12 steps)
    README.md                           — this file
```
