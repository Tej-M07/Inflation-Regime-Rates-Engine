# Inflation Regime Rates Engine

A quantitative risk engine designed to model US Treasury yield volatility specifically within the post-2022 high-inflation regime.

### Project Context
Standard risk models often rely on long historical lookbacks (5-10 years), which include the **"Zero Interest Rate Policy" (ZIRP)** era. Including pre-2022 data in today's calibration "pollutes" the model, leading to underestimated volatility and mean-reversion levels that are structurally disconnected from the current Fed stance.

This project implements a **Regime-Aware Risk Model** that isolates the hiking cycle (post-March 2022) to quantify interest rate risk more accurately.

---

### Core Features

* **Regime Filtering:** Automatically filters input data to start strictly from the Fed "Lift-off" (March 16, 2022), ensuring risk parameters ($\sigma$, $\kappa$) reflect the current volatility regime.
* **PCA Factor Decomposition:** Decomposes yield curve movements into **Level** (~82%), **Slope** (~10%), and **Curvature** (~7%) factors, validating the "Litterman-Scheinkman" framework.
* **Hybrid Vasicek Calibration:** Implements a forward-looking calibration. While volatility and speed are derived from history, the long-term equilibrium ($\theta$) is anchored to the **current 10Y Market Yield**, preventing "backward-looking bias."
* **FRTB Compliance:** Calculates **99% Expected Shortfall (ES)** alongside Value-at-Risk (VaR), capturing the "fat tail" losses that standard VaR metrics fail to detect.

---

### Technical Methodology

#### 1. Data Ingestion & Robustness
The engine fetches daily Treasury Constant Maturity rates (3M, 2Y, 5Y, 10Y, 30Y) from the **Federal Reserve Economic Data (FRED)** API.
* *Auto-Fallback:* Includes a mock data generator that produces synthetic Brownian motion if the FRED API connection times out, ensuring the pipeline remains demonstrable offline.

#### 2. Principal Component Analysis (PCA)
The engine performs PCA on daily yield changes ($dX$) to identify risk drivers:
* **PC1 (Level):** Parallel shifts in the curve (Fed Policy).
* **PC2 (Slope):** Curve rotation (Recession/Inflation signals).
* **PC3 (Curvature):** Mid-curve convexity (Supply/Demand dynamics).

#### 3. Hybrid Calibration Logic
Standard OLS calibration on historical data often suggests a mean reversion level ($\theta$) that is merely the historical average (e.g., 5.0% during hikes).
* **The Fix:** We anchor $\theta$ to the current market-implied long-term rate ($r_{10Y}$).
* **The SDE:** $dr_t = \kappa(\theta_{market} - r_t)dt + \sigma dW_t$

---

### Libraries

**Dependencies**
```bash
pip install numpy pandas pandas-datareader matplotlib seaborn scikit-learn

```

---

### Output

The script generates a 4-panel risk dashboard:

* **Regime Plot:** Visualizes the "Inflation Era" vs. historical ZIRP.
* **PCA Loadings:** Factor sensitivities for each tenor (3M to 30Y).
* **Monte Carlo Forecast:** 10,000 rate paths projected 1 year forward.
* **Tail Risk Distribution:** Histogram comparing 99% VaR vs. 99% Expected Shortfall.

---

### Limitations & Discussion

* **One-Factor Constraint:** The Vasicek model assumes the entire curve is perfectly correlated (driven by one source of uncertainty). In reality, short rates (Fed driven) and long rates (Macro driven) can decorrelate. A **2-Factor Hull-White model** would be the next logical upgrade.
* **Normality Assumption:** The simulation assumes Gaussian shocks. Since financial markets exhibit excess kurtosis (fat tails), this model may underestimate "Black Swan" probability compared to a **Student-t** or **Jump Diffusion** model.
* **Negative Rates:** While unlikely in the current US regime, the Vasicek framework mathematically permits negative rates, unlike the Cox-Ingersoll-Ross (CIR) model.

