# Lumina Oracle Finance

**A browser-based market fractal dynamics engine implementing the M-FDE equation.**

Single HTML file. No server. No dependencies to install. Open it and analyze any stock or crypto.

---

## The Equation

```
D^H P(t) = α(H±)·Caputo[P] + β·Σ_k P^D_f(ω_k)·cos(ω_k·t) + σ·dL_α^H(t)
```

Every symbol is computed from data and used in the simulation:

| Symbol | What it is | How it's computed |
|--------|-----------|-------------------|
| `D^H` | Fractional derivative operator | Hurst exponent H via R/S analysis or DFA |
| `α(H±)` | Asymmetric momentum coefficient | Separate Hurst H+ (upside) and H− (downside) via signed R/S |
| `Caputo[P]` | Memory-weighted return history | Power-law weighted sum of past returns, window = 40 steps |
| `β` | Forcing amplitude | HMM regime blending (4-state: Trending, Mean-Reverting, Volatile, Calm) |
| `D_f(ω_k)` | Scale-dependent fractal dimension | MF-DFA generalized Hurst h(q) for q ∈ {−4,−2,0,2,4} |
| `dL_α^H(t)` | Fractional α-stable motion | fBm correlation (Hosking) × α-stable draws (Chambers-Mallows-Stuck) |

This is the first formulation where all five properties of real market price paths are embedded simultaneously: **long memory**, **fat tails**, **multifractal scaling**, **asymmetric momentum**, and **multiple market cycles**.

---

## What It Computes

For any ticker (stocks, ETFs, crypto), Lumina estimates:

- **Hurst exponent H** — Is this market trending, random, or mean-reverting?
- **Multifractal spectrum Δh** — Does it behave differently at different timescales?
- **Asymmetric Hurst H+/H−** — Do rallies or crashes persist longer?
- **Leverage ratio L = H−/H+** — Quantifies the asymmetry as a single number
- **Lévy tail index α** — How fat are the tails vs Gaussian?
- **HMM regime** — Which of 4 dynamic regimes fits current behavior?
- **Monte Carlo forecast cone** — 30-step fSDE paths with 10/50/90% bands
- **3-fold walk-forward calibration** — Honest out-of-sample directional accuracy

---

## Quick Start

1. Download `lumina_v7_3.html`
2. Open it in any modern browser — no server needed
3. Type a ticker symbol (NVDA, SPY, BTC, AAPL, etc.)
4. Select a timeframe (6M, 1Y, or 5Y recommended)
5. The M-FDE equation runs entirely in your browser

**No API key required for the core analysis.** Price history (candles) is fetched from Stooq and Yahoo Finance (public endpoints). Crypto data from CoinGecko. The entire M-FDE engine — Hurst, MF-DFA, asymmetric H±, Lévy estimation, Monte Carlo cone — runs on that data with no key.

**Finnhub API key optional** — enables two additional features:
- Real-time stock quotes (live price display and auto-refresh)
- Market and asset news feed

A free Finnhub key is available at [finnhub.io](https://finnhub.io). Paste it into the key field in the app's settings panel. Without it, the analysis still runs fully using the last candle close price.

---

## What the Output Means

### Hurst Exponent H
- `H > 0.65` — Trending (persistent momentum)
- `0.55–0.65` — Mildly persistent
- `0.45–0.55` — Random walk (EMH-like)
- `0.42–0.45` — Mildly anti-persistent
- `H < 0.42` — Mean-reverting

### Asymmetric Hurst (H+ / H−)
- `L = H−/H+ > 1` — Bearish asymmetry: crashes persist more than rallies
- `L = H−/H+ < 1` — Bullish asymmetry: rallies persist more than crashes
- This signal is invisible to standard symmetric Hurst analysis

### Multifractal Spectrum Δh
- `Δh = h(−4) − h(4)` from MF-DFA
- `Δh < 0.05` — Monofractal (one D_f describes all scales)
- `0.05–0.15` — Mildly multifractal
- `Δh > 0.15` — Genuinely multifractal (different physics at small vs large moves)

### D_f Spectrum (small | medium | large)
Each frequency component in the forcing term uses its own fractal dimension:
- High-frequency cycles use D_f of small daily fluctuations
- Low-frequency cycles use D_f of large trend breakouts
- Wide spread = the asset behaves very differently at different scales

### Calibration Score
Three walk-forward folds, each testing on ~n/7 bars:
- Score = 0.70 × directional accuracy + 0.30 × (1 − RMSE_range)
- RMSE_range = model error / test-period price range (dimensionless, ~0.5–3.0 is normal)
- Structural breaks penalize 15% but do not zero the score

### Regime Labels
| Label | Meaning |
|-------|---------|
| TRENDING | H > 0.65, HMM confirms |
| LEANING-TREND | H 0.55–0.65, HMM says trending |
| CALM | Low vol, ambiguous direction |
| UNCERTAIN-BULL | HMM says trending but H disagrees (upside) |
| UNCERTAIN-BEAR | HMM says trending but H disagrees (downside) |
| LEANING-REVERT | H 0.42–0.45, mild mean-reversion |
| MEAN-REVERTING | H < 0.42, strong anti-persistence |
| VOLATILE | High vol, unreliable direction |

---

## Empirical Results (March 2026)

Four-asset analysis run simultaneously on 5Y data:

| Asset | H | Regime | H Asymmetry | Δh | Dir. Acc | Signal |
|-------|---|--------|-------------|-----|----------|--------|
| NVDA | 0.544 | CALM | L=1.313 bearish | 0.274 | 54.2% | neutral |
| SPY | 0.566 | LEANING-TREND | L=0.915 bullish | 0.175 | 49.3% | H+ watch |
| AAPL | 0.626 | LEANING-TREND | L=0.791 bullish | 0.463 | 43.8% | H+ watch |
| TSLA | 0.607 | LEANING-TREND | L=1.128 bearish | 0.097 | 59.7% | lean short |

**Key finding:** Standard symmetric Hurst analysis shows all four assets near H≈0.5 (random walk). Asymmetric Hurst reveals a clean split — NVDA and TSLA have bearish momentum asymmetry while SPY and AAPL have bullish recovery asymmetry. The market structure is differentiating even while all four assets are technically in correction (MA20 < MA50, RSI 36–52).

---

## Theoretical Background

### Connection to the Ψuniverse Framework

The M-FDE equation shares its mathematical architecture with the Ψuniverse physiological state-space equation developed for HRV analysis (patent pending, US20260022228A1). Both frameworks apply:

- Fractional calculus (Caputo derivative) to embed long-memory dynamics
- Multifractal geometry to capture scale-dependent complexity
- α-stable noise to handle fat-tailed extreme events

In the HRV context, the 108-dimensional state space characterizes the attractor of a physiological system. In the market context, M-FDE characterizes the attractor of a price process. The hypothesis is that complex adaptive systems — whether cardiac rhythm or market dynamics — share the same fractal attractor signature, and that signature is measurable and predictive.

### Honest Limitations

This is a principled heuristic, not a mathematically rigorous fSDE in the formal Caputo-Lévy sense. Specifically:

1. The Caputo derivative is approximated via a power-law weighted window, not solved analytically
2. The fαM noise combines fBm correlation structure with α-stable marginals — a practical approximation, not exact fractional α-stable motion
3. The MF-DFA h(q) estimation requires sufficient data; short windows (< 60 bars) degrade to monofractal R/S estimates
4. Calibration scores in the 30–55% range are typical for financial time series — do not interpret high calibration as a trading guarantee

These limitations are documented here because understanding what the tool does not know is as important as understanding what it does.

---

## Version History

| Version | Key Addition |
|---------|-------------|
| v6.0 | Initial fSDE: Gaussian Monte Carlo cone, R/S Hurst, HMM regime |
| v6.1 | Student-t ν=4 noise, MA20 anchor, ±25% return clamp |
| v6.2 | EMA-smoothed Hurst, DFA fallback, adaptive ω, signal backtest tab |
| v6.3 | Hosking exact fBm (Cholesky), Caputo D^H derivative, multi-frequency ω |
| v6.4 | Vol-adjusted RMSE, 5-tier H bands, H-override on regime label, adaptive lookback |
| v7.0 | MF-DFA multifractal spectrum, fαM correlated Lévy noise, asymmetric H±, scale-dependent D_f(ω) — true M-FDE |
| v7.1 | Chart y-axis fixed (price-based bounds, cone capped at 3× last price), parallel fetch Stooq+Yahoo |
| v7.2 | 3-fold walk-forward calibration, 70/30 directional/magnitude weighting |
| v7.3 | Range-normalized RMSE (honest calibration for all market conditions) |

---

## File Structure

```
lumina_v7_3.html    — Single-file application (128KB, 2,123 lines)
README.md           — This file
CITATION.cff        — Academic citation format
```

Everything runs in the browser. No npm, no Python, no server.

---

## Citing This Work

If you use Lumina Oracle Finance or the M-FDE framework in research, please cite using the format in `CITATION.cff` or:

> Quiroz, N.B. (2026). *Lumina Oracle Finance: A Market Fractal Dynamics Engine Implementing the M-FDE Equation*. GitHub. https://github.com/senseizuna/lumina-oracle-finance

The M-FDE framework is related to the Ψuniverse HRV analysis system (patent pending US20260022228A1, filed June 4, 2025).

---

## License

MIT License — use freely, cite honestly.

*Not financial advice. This tool is for research and educational purposes.*

---

**Nicolas B. Quiroz, MD**  
Independent researcher · nicolasquirozmd.com  
GitHub: github.com/senseizuna · github.com/BEPframework
