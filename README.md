
# Yield Curve Reconstruction using the Cox-Ingersoll-Ross (CIR) Model

## Project Overview

This project reconstructs the government bond yield curve using the Cox–Ingersoll–Ross (CIR) stochastic interest rate model, calibrated on real market data. Starting from only the **3-month short rate**, the model reconstructs the full yield curve across nine maturities — from 3 months out to 30 years.

The project covers theoretical grounding, parameter calibration, out-of-sample evaluation, and two extensions (Two-Factor CIR and Polynomial Ridge Regression) that improve upon the base model's accuracy.

\---

## Problem Statement

Given historical government bond yield data, estimate the parameters of the CIR model and use them to reconstruct the entire yield curve. Evaluate the quality of reconstruction using statistical metrics and compare the model's predictions with observed market yields across normal and inverted yield curve regimes.

\---

## CIR Model

The CIR short-rate model is given by:

```
dr(t) = κ(θ − r(t)) dt + σ√r(t) dW(t)
```

|Symbol|Meaning|
|-|-|
|`r(t)`|Short-term interest rate|
|`κ`|Mean-reversion speed — how fast rates return to θ|
|`θ`|Long-run mean interest rate|
|`σ`|Volatility parameter|
|`W(t)`|Wiener process (standard Brownian motion)|

The model guarantees non-negative interest rates (under the Feller condition `2κθ > σ²`) and provides a **closed-form bond pricing formula** — no simulation required.

### Yield Formula

```
y(τ) = [ B(τ) · r₀ − ln A(τ) ] / τ
```

where `A(τ)` and `B(τ)` are functions of the calibrated parameters and maturity `τ`.

\---

## Methodology

1. Load and clean historical government bond yield data (train/test split)
2. Handle missing values via linear interpolation and clip outliers using the IQR method
3. Use the 3-month Treasury rate as the short-rate proxy `r₀`
4. Calibrate CIR parameters `(κ, θ, σ)` via Cross-Sectional GMM (Nelder-Mead optimiser), minimising:

```
L = (1/N) Σ [ y_CIR(τᵢ) − y_obs(τᵢ) ]² + λ · max(0, σ² − 2κθ)
```

The second term is a Feller penalty ensuring rates stay positive.

5. Reconstruct yields for maturities: `3M, 6M, 9M, 1Y, 2Y, 5Y, 10Y, 20Y, 30Y`
6. Evaluate performance using R², RMSE, and bias analysis
7. Visualise results via yield curve plots, time-series comparisons, and residual histograms

\---

## Models

|Model|Type|Parameters|
|-|-|-|
|Base CIR (1-Factor)|Stochastic / Closed-form|κ, θ, σ|
|Two-Factor CIR|Stochastic / Closed-form|κ\_X, θ\_X, σ\_X, κ\_Y, θ\_Y, σ\_Y, α|
|Polynomial Ridge|Machine Learning|Degree-2 poly + Ridge (per maturity)|

\---

## Results

The calibrated CIR model successfully captures the overall shape of the yield curve. Key observations:

* **Short and medium maturities (3M–2Y)** are fitted well — the 3M short rate has strong predictive power over the near end of the curve.
* **Long maturities (10Y–30Y)** are harder to fit — at large τ, the CIR yield converges to a structural long-run level and loses sensitivity to `r₀`.
* The model shows **systematic bias during inverted yield curves** — it underestimates short yields and overestimates long yields when rates are above the long-run mean.
* The **Two-Factor CIR** reduces this bias by decoupling the level and slope of the curve.
* The **Polynomial Ridge extension** achieves the best out-of-sample R² by capturing non-linear relationships between the 3M rate and longer maturities.

Performance is evaluated using R² and RMSE reported per maturity in the notebook.

\---

## Extensions Explored

* **Two-Factor CIR** — splits `r(t)` into a slow-reverting level factor and a fast-reverting slope factor, enabling better fit across all curve shapes including inversions
* **Polynomial Ridge Regression** — fits a degree-2 polynomial from `r₃M` to each maturity independently, with Ridge regularisation and recency-based sample weights
* Discussion of further extensions: jump-diffusion CIR, time-dependent parameters, and alternative calibration methodologies (MLE vs GMM)

\---

## Technologies Used

* Python
* NumPy
* Pandas
* SciPy
* Matplotlib
* Scikit-learn
* Jupyter Notebook / Google Colab

\---

## Repository Structure

```
FinClub/

├── FinCLUB PS1.ipynb

├── README.md

├── Problem\_Statement.pdf

└── data/

   ├── train\_data.csv

   ├── test\_data.csv

   ├── test\_data\_3M.csv

   └── yield\_curve\_predictions.csv
```

\---

## How to Run

How to Run

1. Clone/download the repository.
2. Ensure the data folder remains in the repository root.
3. Open Finclub_PS1.ipynb.
4. Run all cells sequentially.
5. Or just click 'Open in Colab' on the top left of the "Finclub_PS1.ipynb" and run the cells sequentially.

The notebook loads datasets from the data/ directory included in this repository.

\---

## References

1. **Cox, Ingersoll \& Ross (1985) — The Original Paper**
*A Theory of the Term Structure of Interest Rates*, Econometrica, 53(2), 385–407.
Read Sections 1–3 and the bond pricing derivation — everything in this project flows from this result.
[JSTOR Link](https://www.jstor.org/stable/1911242)
2. **Brigo \& Mercurio — Interest Rate Models: Theory and Practice (Chapters 3–4)**
The standard quant textbook for interest rate models. Chapters 3–4 cover CIR calibration and the CIR++ extension in full detail.
[Springer Link](https://link.springer.com/book/10.1007/978-3-540-34604-3)
3. **MIT OpenCourseWare 18.S096 — Lectures 17 \& 18**
Free MIT lecture notes and videos on stochastic processes and interest rate models. The clearest academic treatment of the CIR SDE and bond pricing at an accessible level.
[YouTube Playlist](https://youtube.com/playlist?list=PLUl4u3cNGP63ctJIEC1UnZ0btsphnnoHR&si=aOyXivdn4Wl3cIsV)
4. **Normalizing Constant — Stochastic Calculus Playlist**
YouTube playlist covering Brownian motion, Itô's lemma, and SDEs from scratch. Recommended before reading the papers if stochastic calculus is new to you.
[YouTube Playlist](https://youtube.com/playlist?list=PLD5zpXPUmJv1X4uWIBCRqFd0ZcxP9y12g&si=Zwm9qQMu6AMiYRQX)

\---

## Author

**Mohd Saif**

Project submitted as part of the Stochastic Interest Rate Modelling project — Finance Club, IIT Roorkee


