# Hierarchical Beam Search: Optimization and Scaling

**Companion page for the paper:**
*Hierarchical Beam Search: Optimization and Scaling* — Olga Galinina, Sergey Andreev, Robert Heath (under review, 2026)

A self-contained HTML page with an interactive calculator for finding the optimal hierarchical beam search configuration under Rician fading, given a link budget and 5G NR numerology.

---

## What this is

Hierarchical beam search (HS) reduces beam alignment overhead by sweeping from wide to narrow beams across L layers, testing N beams per layer. The configuration choices — how many layers and how many beams per layer — are typically set by intuition or grid search. This paper derives them in closed form.

The page presents the key results and lets you compute the optimal delay for your own system parameters.

---

## Sections

### Overview video
A 6-minute embedded YouTube introduction to the scaling laws. 
Created using Manim – Mathematical Animation Framework (Version v0.20.1), https://www.manim.community/




### Calculator
The main interactive element. Takes link budget, array, channel, and numerology parameters; computes the delay-optimal number of layers and plots total access delay vs. codebook size X across five curves.

### Key results cards
Two formula cards side by side:
- **Unconstrained optimum** (Propositions 1–3): closed-form L*, N*, R*, and Lambert-W corrections for inter-layer overhead.
- **With reliability constraint**: the exact per-layer misdetection probability formula under Rician fading.

### Resources
Links to the derivation note PDF, GitHub repository, and manuscript contact.

### Citation
Copyable BibTeX entry.

### Python code modal
A floating `{ } Python code` button opens a tabbed modal with four Python snippets matching the calculator's computation: exact Pₑ, harmonic approximation, optimal L, and constraint check.

---

## Calculator inputs

| Field | Default | Description |
|---|---|---|
| X min / X max | 3 / 100 000 | Codebook size range for the plot |
| Points | 120 | Number of log-spaced X values |
| fc [GHz] | 28 | Carrier frequency |
| Distance [m] | 20 | TX–RX distance (free-space path loss) |
| Ptx [dBm] | 10 | Transmit power |
| NF [dB] | 5 | Receiver noise figure |
| B [MHz] | 100 | Bandwidth |
| Rician K | 10 | K-factor (LOS-to-scatter power ratio; K=0 is Rayleigh) |
| Nant | 128 | Physical ULA elements (caps beamforming gain) |
| θ₀ [deg] | 120 | Initial sector width for the reliability constraint |
| ε | 0.01 | Allowed end-to-end misdetection probability |
| Numerology | 0 (66.67 µs) | 5G NR OFDM symbol duration; sets the inter-layer period c₁ |
| Tc burst [ms] | 10 | Fixed burst duration; c₁ = ⌊Tc / (2 · Tsym)⌋ |

SNR is derived from the link budget: `SNR = Ptx · (λ/(4πd))² / (kT₀ · 10^(NF/10) · B)`. The computed SNR and c₁ are displayed in the summary bar above the chart.

---

## Chart curves

All curves plot total access delay in milliseconds vs. codebook size X on a log scale. The delay unit is one beam transmission = 2 OFDM symbols; the scale factor `2·Tsym` converts to ms.

| Curve | Colour | Formula | Description |
|---|---|---|---|
| Lower bound (continuous) | Gray dashed | X^{1/L*} + c₁(L*−1) with real-valued L* | Continuous relaxation of the delay-optimal problem (Option B, Proposition 3). Unreachable in practice. |
| Unconstrained discrete | Blue solid | min_L ⌈X^{1/L}⌉ + c₁(L−1) | Best integer L near the continuous optimum. No reliability requirement. This is always the lowest discrete curve. |
| Exact constraint (ε) | Green dashed | Same delay, subject to ∏(1−Pₑ) ≥ 1−ε | Reliability enforced using the exact binomial-sum Pₑ. Shown only where a feasible L exists; gaps mean no L satisfies the constraint at that X. Numerically stable for N ≲ 48 beams per layer; skips (conservative) when the alternating sum is unstable. |
| Harmonic constraint | Orange dashed | Closed-form sufficient condition from Proposition 3 | Uses the high-SNR harmonic approximation Pₑ ≈ (K+1)e^{−K} H_{N−1} / (G·SNR) substituted into the geometric-sum reliability bound. Cheaper to evaluate; slightly more conservative than the exact check. |
| Harmonic Pₑ constraint (approx) | Purple dotted | Product ∏(1−Pₑ_harm) ≥ 1−ε | Same product structure as the exact constraint but with the harmonic Pₑ approximation in each layer. |

Constrained curves are never below the blue curve in the data. If they appear so, it is a rendering artifact from `spanGaps` spanning null regions — set `spanGaps: false` on those datasets.

---

## Core formulas implemented

**Beamforming gain** (boundary-centered placement, ULA):
```
G(N, θ) = min(Nant, 101.5° · (N−1) / θ)
```
Sector narrows geometrically between layers: `θₗ = θ₀ / (N−1)^{ℓ−1}`.

**Exact per-layer error probability** (Proposition 1, derivation note):
```
Pₑ(N, θ) = 1 − Σ_{m=0}^{N−1} C(N−1,m)(−1)^m · [1/(1+m·σ²_tot/N₀)] · exp(−m·ν/N₀ / (1+m·σ²_tot/N₀))
```
where `ν = G·Prx·K/(K+1)` (LOS power) and `σ²_tot = G·Prx/(K+1) + N₀` (total variance). Evaluated with a recursive binomial coefficient update and Kahan compensated summation to reduce floating-point cancellation. Returns NaN (skipped) when N ≳ 48 and the alternating sum is no longer reliable.

**Harmonic approximation** (Proposition 2, high-SNR regime G·SNR ≫ K+1):
```
Pₑ(N, θ) ≈ (K+1) · e^{−K} · H_{N−1} / (G·SNR)
```
Underestimates Pₑ by 10–20%; useful for analytical insight into the ln N scaling.

**Delay-optimal depth, Option B** (Proposition 3):
```
L*_B = ln(X) / (2 · W(√(c₁ ln X) / 2))
```
where W is the Lambert W function (principal branch), computed via 12-step Halley iteration.

**Reliability constraint** (sufficient condition):
```
(K+1) · e^{−K} · H_{N−1} · θ₀ / (101.5° · SNR) · Σ_{ℓ=1}^{L} 1/(N−1)^ℓ ≤ ε
```

---

## Dependencies (CDN, no install needed)

- [MathJax 3](https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-chtml.js) — formula rendering
- [Chart.js 4.4.1](https://cdn.jsdelivr.net/npm/chart.js@4.4.1/dist/chart.umd.min.js) — interactive plot

No build step. Open `index.html` directly in any modern browser.

---

## Repository structure

```
index.html              # The full page (single file, all JS inline)
pe_tables_v2.ipynb      # Python notebook: Pe tables and figures matching the paper
README.md               # This file
Per-layer-misdetection-formula.pdf   # Derivation note (channel model → exact Pe → harmonic approx)
```

The notebook (`pe_tables_v2.ipynb`) contains the reference Python implementations of all formulas and reproduces the numerical tables in the derivation note. The last cell contains a widget-mirror Python script that replicates the calculator's computation exactly, including the float-N generalized binomial and Kahan summation, for cross-validation.

---

## Known numerical limitations

- **Exact Pₑ, large N.** The alternating binomial sum catastrophically cancels for N ≳ 48–50 at typical SNR. The widget detects this (NaN guard) and skips those L values in the exact reliability check — a conservative decision. The Python notebook provides a quadrature-based `Pe_exact_quad` that is accurate for all N and can be used to fill these gaps offline.
- **Float N in reliability checks.** The optimal N = X^{1/L} is generally non-integer. The widget evaluates Pₑ at float N using the generalized (non-integer upper index) binomial, which is a smooth interpolation between integer cases. This matches the widget-mirror Python exactly and is intentional for plot continuity.
- **Harmonic approximation validity.** The approximation requires G·SNR ≫ K+1. At small N, wide θ, or low SNR it underestimates Pₑ by up to 80%. The harmonic constraint curve is therefore optimistic in that regime; use the exact constraint for tight numerical bounds.

---

## Bugs fixed

Two bugs were corrected relative to the original source:

1. **`Pe_harm` harmonic argument** — `harmonic(N − 1)` was called with float N, yielding H_{⌊N−1⌋} instead of H_{round(N)−1}. Fixed to `harmonic(Math.max(1, Math.round(N) − 1))`. Affected the purple curve.

2. **`spanGaps: true` on constrained curves** — when a constrained curve has null values over a range of X (no feasible L), Chart.js bridged the gap with a straight diagonal line that visually crossed below the blue unconstrained curve. Fixed by setting `spanGaps: false` on the three constrained datasets.

---

## Citation

```bibtex
@article{galinina2026hierarchical,
  title  = {Hierarchical Beam Search: Optimization and Scaling},
  author = {Galinina, Olga and Andreev, Sergey and Heath, Robert},
  year   = {2026},
  note   = {Under review}
}
```

Contact: olga.galinina [at] gmail [dot] com
