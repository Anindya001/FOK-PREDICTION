# FOK-PREDICTION: Fractional-Order Kinetics for Capacitor Degradation Forecasting

[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)

A physics-informed uncertainty quantification framework for aluminium electrolytic capacitor (AEC) degradation forecasting using **fractional-order kinetics** and **conformal prediction**.

---

## 🎯 Overview

FOK-PREDICTION combines mechanistic fractional-order differential equations with state-of-the-art uncertainty quantification to predict capacitor end-of-life with **rigorous statistical guarantees**. The framework addresses two critical limitations of classical prognostics:

1. **Physical Realism**: Fractional-order kinetics capture anomalous diffusion in porous electrolytes, providing interpretable parameters (α, k, f∞) that reflect actual degradation mechanisms
2. **Reliable Uncertainty**: Hybrid Bayesian + conformal prediction delivers both epistemic bounds and distribution-free finite-sample coverage guarantees

### Key Innovation

The fractional-order model describes capacitance decay as:

```
C(t) = C₀[f∞ + (1 - f∞)E_α(-kt^α)]
```

where:
- **E_α** is the Mittag-Leffler function (generalization of exponential)
- **α ∈ (0,1)** quantifies sub-diffusive transport (α=1 recovers classical Fickian diffusion)
- **k** is the characteristic rate constant
- **f∞** is the asymptotic capacitance retention fraction
- **C₀** is the initial capacitance

---

## ✨ Features

### Core Capabilities

- **Fractional-Order Physics Model**
  - Mittag-Leffler evaluation with robust fallbacks (SciPy → mpmath)
  - Constrained parameter estimation via transformed optimization
  - Monotonicity enforcement and numerical stability checks

- **Hybrid Uncertainty Quantification**
  - Laplace approximation for fast epistemic uncertainty
  - Full MCMC sampling (Metropolis-Hastings) for posterior exploration
  - Log-normal observation model for multiplicative errors
  - Posterior predictive distributions with epistemic/aleatoric decomposition

- **Conformal Prediction**
  - Split conformal calibration for finite-sample coverage
  - MAD-based conformity scores
  - Distribution-free guarantees: empirical coverage ≥ 1 - α - 1/(m+1)

- **Global Sensitivity Analysis**
  - Sobol indices via Saltelli sampling
  - Bootstrap confidence intervals
  - Multiple quantities of interest: Y(h), Δ(h), T(q)
  - Parameter screening for uncertainty reduction

- **Comprehensive Diagnostics**
  - Information criteria (AIC, BIC, WAIC)
  - Residual testing (Shapiro-Wilk, runs test)
  - Prequential cross-validation
  - Coverage calibration plots

- **Interactive GUI**
  - PyQt5 application (1,719 lines)
  - Multi-threaded analysis workers
  - Real-time progress tracking
  - Export functionality (figures, tables, posterior samples)

---

## 🚀 Quick Start

### Installation

```bash
# Clone the repository
git clone https://github.com/Anindya001/FOK-PREDICTION.git
cd FOK-PREDICTION

# Create virtual environment (recommended)
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### Basic Usage (Python API)

```python
import pandas as pd
from fractional_core import FractionalPICPCore, FractionalConfig

# Load capacitor degradation data
data = pd.read_csv("data/capacitor_C1.csv")  # columns: time, capacitance

# Configure pipeline
config = FractionalConfig(
    train_ratio=0.7,           # Use 70% for training
    confidence=0.9,            # 90% confidence intervals
    n_draws=2000,              # Posterior samples
    thresholds=[0.8, 0.7],     # Failure thresholds (80%, 70% of C₀)
    run_sensitivity=True,      # Enable Sobol analysis
)

# Run forecast
core = FractionalPICPCore(config)
result = core.run_forecast(data, time_column="time", target_column="capacitance")

# Access results
print(f"Estimated α: {result['fit']['params']['alpha']:.3f}")
print(f"RMSE (train): {result['metrics']['rmse_train']:.4f}")
print(f"Conformal coverage: {result['metrics']['conformal_coverage']:.2%}")

# Failure time predictions (median and 90% CI)
for q, quantiles in result['failure_time']['quantiles'].items():
    print(f"Time to {q*100:.0f}% capacity: {quantiles['median']:.1f} "
          f"[{quantiles['q05']:.1f}, {quantiles['q95']:.1f}] hours")
```

### GUI Application

```bash
python app_ui.py
```

Features:
1. **Data Loading**: CSV/Excel files with automatic column detection
2. **Model Selection**: FK, Classical Exponential, or KWW Stretched Exponential
3. **Analysis Modes**:
   - Forecast & UQ: Full pipeline with confidence bands
   - Sensitivity Study: Prior-based Sobol analysis
4. **Visualization**: Interactive plots with toggleable bands
5. **Export**: Save figures, tables, and posterior samples

---

## 📊 Project Structure

```
FOK-PREDICTION/
├── README.md                          # This file
├── requirements.txt                   # Python dependencies
├── setup.py                          # Package installation script
│
├── fractional_model.py               # FK model: C(t), E_α evaluation, T_q solver
├── fractional_estimation.py          # Constrained MLE with log-normal errors
├── fractional_prediction.py          # Deterministic forecasts and failure times
├── fractional_uq.py                  # Laplace/MCMC sampling, predictive distributions
├── fractional_conformal.py           # Split conformal calibration
├── fractional_sensitivity.py         # Sobol indices for QoIs
├── fractional_diagnostics.py         # Metrics, residual tests, prequential CV
├── fractional_core.py                # High-level orchestration (FractionalPICPCore)
│
├── math_utils.py                     # Numerical utilities (Mittag-Leffler, etc.)
├── weighted_conformal_prediction.py  # Phase-aware conformal predictor
├── surrogate_models.py               # Classical/KWW baseline models
├── models.py                         # Model interfaces
├── core.py                           # Legacy pipeline
├── excel_reader.py                   # Data I/O utilities
│
├── app_ui.py                         # PyQt5 GUI (1,719 lines)
├── main.py                           # CLI entry point
│
├── PUBLICATION_READINESS_EVALUATION.md  # Publication assessment
├── ACTION_TODO.md                       # Detailed action plan for publication
├── design.md                            # UI/UX design specification
├── fractional_upgrade_blueprint.md      # Technical roadmap
├── fk_conformal_impl_plan.md           # Implementation blueprint
│
└── data/                             # (Not included - add your datasets)
    ├── raw/
    ├── synthetic/
    └── processed/
```

---

## 📖 Documentation

### Core Modules

#### `fractional_model.py`
Implements the FK degradation model:
- `fractional_capacitance(t, params)`: Evaluate C(t)
- `fractional_derivative(t, params)`: Compute dC/dt
- `time_to_threshold(params, q)`: Solve for failure time T_q
- `ensure_monotonic(t, params)`: Verify physical plausibility

#### `fractional_estimation.py`
Parameter estimation via constrained optimization:
- Log-normal observation model
- Parameter transforms: k=exp(κ), α=logistic(a), f∞=logistic(b)
- Returns: `FKFitResult` with params, covariance, diagnostics

#### `fractional_uq.py`
Uncertainty quantification:
- `laplace_draws(fit, n_draws)`: Sample from posterior via Laplace approximation
- `mcmc_draws(fit, times, values)`: Metropolis-Hastings sampling
- `posterior_predictive(params_draws, sigma_draws, t)`: Generate predictive samples
- `failure_time_samples(params_draws, thresholds)`: Uncertainty in T_q

#### `fractional_conformal.py`
Distribution-free prediction intervals:
- `conformal_intervals(calibration_obs, calibration_samples, test_samples, alpha)`:
  Compute finite-sample guaranteed intervals

#### `fractional_sensitivity.py`
Global sensitivity analysis:
- `sobol_analysis(priors, qoi, n_samples)`: First-order and total Sobol indices
- Pre-defined QoIs: `qoi_capacitance(t_h)`, `qoi_deficit(t_h)`, `qoi_failure_time(q)`
- Bootstrap CIs for indices

#### `fractional_core.py`
High-level pipeline:
- `FractionalPICPCore.run_forecast(data, ...)`: End-to-end analysis
- Handles train/calibration/test splitting
- Returns comprehensive result dictionary with forecasts, UQ bands, metrics

### Configuration Options

`FractionalConfig` dataclass controls all pipeline parameters:

```python
FractionalConfig(
    train_ratio=0.7,              # Fraction for training (0.4-0.95)
    calibration_fraction=0.2,     # Fraction of training for conformal calibration
    confidence=0.9,               # Confidence level for intervals (0.0-1.0)
    n_draws=2000,                 # Number of posterior samples
    bootstrap_draws=512,          # Bootstrap samples for bias correction
    thresholds=[0.8, 0.7],        # Failure thresholds (as fractions of C₀)
    run_sensitivity=False,        # Enable Sobol analysis
    sensitivity_horizons=[200.0], # Mission horizons for sensitivity (hours)
    sobol_samples=2048,           # Saltelli sample size
    sobol_bootstrap=200,          # Bootstrap samples for Sobol CIs
    random_state=None,            # RNG seed for reproducibility
    use_mcmc=False,               # Use MCMC instead of Laplace (slower but more accurate)
    mcmc_draws=1000,              # MCMC posterior samples
    mcmc_burn_in=300,             # MCMC burn-in iterations
    mcmc_step_scale=0.5,          # MCMC proposal scale (tune for ~25% acceptance)
)
```

---

## 🔬 Example: Complete Analysis

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from fractional_core import FractionalPICPCore, FractionalConfig

# Generate synthetic data (replace with real data)
t = np.linspace(0, 500, 100)
true_params = {"C0": 100, "k": 0.001, "alpha": 0.7, "f_inf": 0.6}
C_true = true_params["C0"] * (
    true_params["f_inf"] +
    (1 - true_params["f_inf"]) * np.exp(-true_params["k"] * t**true_params["alpha"])
)
C_obs = C_true * np.exp(np.random.normal(0, 0.02, size=len(t)))
data = pd.DataFrame({"time": t, "capacitance": C_obs})

# Configure and run
config = FractionalConfig(
    train_ratio=0.7,
    confidence=0.95,
    n_draws=2000,
    thresholds=[0.8, 0.7],
    run_sensitivity=True,
    sensitivity_horizons=[200, 300, 400],
)
core = FractionalPICPCore(config)
result = core.run_forecast(data)

# Plot results
fig, ax = plt.subplots(figsize=(10, 6))
times = np.array(result["forecast"]["time"])
train_count = result["data"]["train_count"]

# Training data
ax.scatter(times[:train_count], result["data"]["values"][:train_count],
           c="#6b7c8c", s=20, label="Training", alpha=0.6)

# Forecast
ax.plot(times[train_count:], result["forecast"]["mean"][train_count:],
        c="#2f4858", lw=2, label="FK Forecast")

# Uncertainty bands
ax.fill_between(times[train_count:],
                result["forecast"]["epistemic_low"][train_count:],
                result["forecast"]["epistemic_high"][train_count:],
                alpha=0.3, color="#8ea8ba", label="Epistemic (95%)")
ax.fill_between(times[train_count:],
                result["forecast"]["conformal_low"][train_count:],
                result["forecast"]["conformal_high"][train_count:],
                alpha=0.2, color="#d9cbb0", label="Conformal (95%)")

# Test observations
ax.scatter(times[train_count:], result["data"]["values"][train_count:],
           c="#a0765b", s=30, marker="o", label="Test", zorder=5)

ax.axvline(times[train_count], ls="--", c="red", alpha=0.5, label="Forecast Boundary")
ax.set_xlabel("Time (hours)", fontsize=12)
ax.set_ylabel("Capacitance (µF)", fontsize=12)
ax.legend(loc="best", framealpha=0.9)
ax.grid(alpha=0.3)
plt.tight_layout()
plt.savefig("forecast_example.png", dpi=300)

# Print summary
print("\n" + "="*60)
print("FRACTIONAL-ORDER KINETICS FORECAST SUMMARY")
print("="*60)
print(f"Estimated Parameters:")
print(f"  α (fractional order): {result['fit']['params']['alpha']:.3f}")
print(f"  k (rate constant):    {result['fit']['params']['k']:.6f}")
print(f"  f∞ (retention):       {result['fit']['params']['f_inf']:.3f}")
print(f"  C₀ (initial cap):     {result['fit']['params']['C0']:.2f} µF")
print(f"\nModel Performance:")
print(f"  Train RMSE:           {result['metrics']['rmse_train']:.4f}")
print(f"  Test RMSE:            {result['metrics']['rmse_forecast']:.4f}")
print(f"  AIC:                  {result['metrics']['AIC']:.2f}")
print(f"  WAIC:                 {result['metrics']['WAIC']:.2f}")
print(f"\nUncertainty Quantification:")
print(f"  Conformal coverage:   {result['metrics'].get('conformal_coverage', 0):.1%}")
print(f"  Target coverage:      {config.confidence:.1%}")
print(f"\nFailure Time Predictions (hours):")
for q in config.thresholds:
    qt = result['failure_time']['quantiles'][q]
    print(f"  T({q:.0%}): {qt['median']:.1f} [{qt['q05']:.1f}, {qt['q95']:.1f}]")
print("="*60)
```

---

## 🧪 Testing

### Run Unit Tests

```bash
# Install test dependencies
pip install pytest pytest-cov

# Run all tests with coverage
pytest tests/ --cov=. --cov-report=html

# View coverage report
open htmlcov/index.html  # On macOS
# or: xdg-open htmlcov/index.html  # On Linux
```

### Verify Installation

```python
# Quick smoke test
from fractional_model import FKParams, fractional_capacitance
import numpy as np

params = FKParams(C0=100, k=0.001, alpha=0.7, f_inf=0.6)
t = np.array([0, 100, 200, 300])
C = fractional_capacitance(t, params)
print(f"C(t) = {C}")  # Should be monotonically decreasing
assert np.all(np.diff(C) <= 0), "Capacitance should decrease!"
print("✓ Installation verified!")
```

---

## 📚 Theoretical Background

### Fractional-Order Diffusion

The FK model derives from the Caputo fractional diffusion equation for electrolyte mass transport:

```
∂ₜC(x,t) = D₀ ᴰᵗᵅ[∂²ₓC(x,t)]
```

where ᴰᵗᵅ denotes the Caputo fractional derivative of order α. Under boundary conditions for evaporation at the capacitor seal, this yields the Mittag-Leffler decay:

```
C(t) = C₀[f∞ + (1-f∞)E_α(-kt^α)]
```

**Physical Interpretation:**
- **α = 1**: Classical Fickian diffusion (exponential decay)
- **α < 1**: Sub-diffusive transport (stretched exponential, heavy-tailed)
- **α → 0**: Ultra-slow diffusion (logarithmic-like decay)

For aluminium electrolytic capacitors, typical values are **α ∈ [0.5, 0.8]**, reflecting anomalous diffusion through porous aluminum oxide and tortuous electrolyte paths.

### Uncertainty Quantification

#### Bayesian Component (Epistemic)
- **Laplace Approximation**: Gaussian posterior centered at MLE with covariance from observed Fisher information
- **MCMC**: Metropolis-Hastings with adaptive proposals for full posterior exploration
- **Predictive Distribution**: Marginalizes over parameter uncertainty

#### Conformal Component (Coverage Guarantee)
- **Split Conformal Prediction** (Lei et al., 2018):
  - Calibration set: Compute conformity scores `S_i = |y_i - median(ŷ_i)| / MAD(ŷ_i)`
  - Quantile: `q̂ = Quantile(S, (1-α)(1 + 1/(m+1)))`
  - Prediction: `Ĉ(t) = [median - q̂·MAD, median + q̂·MAD]`
- **Finite-Sample Guarantee**: Coverage ≥ 1 - α - 1/(m+1) for any data distribution

### Sensitivity Analysis

**Sobol Decomposition:**

For a quantity of interest Y = f(α, k, f∞, C₀), the variance decomposes as:

```
Var[Y] = Σᵢ Vᵢ + Σᵢ<ⱼ Vᵢⱼ + ... + V₁₂₃₄
```

**Indices:**
- **First-order**: `Sᵢ = Vᵢ / Var[Y]` (main effect of parameter i)
- **Total-order**: `Sᵢᵀ = (Vᵢ + all interactions involving i) / Var[Y]`

**Interpretation:**
- `Sᵢᵀ < 0.01`: Parameter i is non-influential (can fix at nominal value)
- `Sᵢᵀ - Sᵢ > 0.2`: Strong interactions involving parameter i

---

## 🎓 Citation

If you use this code in your research, please cite:

```bibtex
@software{fok_prediction_2024,
  title = {FOK-PREDICTION: Fractional-Order Kinetics for Capacitor Degradation Forecasting},
  author = {{FOK-PREDICTION Contributors}},
  year = {2024},
  url = {https://github.com/Anindya001/FOK-PREDICTION},
  note = {Version 1.0}
}
```

**Related Publications:**
- (Manuscript in preparation for IEEE Transactions on Device and Materials Reliability)

---

## 🛠️ Dependencies

### Core Scientific Stack
- **Python** ≥ 3.8
- **NumPy** ≥ 1.21 (array operations)
- **SciPy** ≥ 1.7 (Mittag-Leffler, optimization)
- **Pandas** ≥ 1.3 (data handling)
- **scikit-learn** ≥ 1.0 (preprocessing utilities)

### Visualization
- **Matplotlib** ≥ 3.5
- **Seaborn** ≥ 0.11

### GUI
- **PyQt5** ≥ 5.15

### Optional
- **mpmath** (high-precision arithmetic fallback for Mittag-Leffler)
- **statsmodels** ≥ 0.13 (advanced statistical tests)
- **pytest** ≥ 6.0 (testing)

See `requirements.txt` for complete list.

---

## 📝 Development Roadmap

### Current Status (v1.0)
- ✅ Complete FK model implementation
- ✅ Hybrid UQ framework (Laplace + Conformal)
- ✅ Sobol sensitivity analysis
- ✅ Full-featured GUI
- ✅ Comprehensive diagnostics

### Planned Features (v1.1)
- [ ] Multi-output models (capacitance + ESR + leakage)
- [ ] Online updating for streaming data
- [ ] GPU acceleration for MCMC
- [ ] Transfer learning across capacitor types
- [ ] Web-based dashboard (Plotly Dash)

### Future Research Directions
- [ ] Adaptive conformal for non-stationary degradation
- [ ] Physics-informed neural networks as surrogate
- [ ] Hierarchical Bayesian models for fleet-level inference
- [ ] Optimal sensor placement via information theory

---

## 🤝 Contributing

Contributions are welcome! Please follow these guidelines:

1. **Fork** the repository
2. **Create** a feature branch (`git checkout -b feature/amazing-feature`)
3. **Commit** your changes (`git commit -m 'Add amazing feature'`)
4. **Push** to the branch (`git push origin feature/amazing-feature`)
5. **Open** a Pull Request

### Code Standards
- Follow PEP 8 style guide
- Add type hints to all functions
- Write docstrings (Google style)
- Include unit tests for new features
- Ensure all tests pass: `pytest tests/`

### Reporting Issues
Please use GitHub Issues and include:
- Python version
- Operating system
- Minimal reproducible example
- Error traceback (if applicable)

---

## 📄 License

This project is licensed under the **MIT License** - see the [LICENSE](LICENSE) file for details.

```
MIT License

Copyright (c) 2024 FOK-PREDICTION Contributors

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

## 🙏 Acknowledgments

- **Mittag-Leffler Implementation**: SciPy community for `scipy.special.mittag_leffler`
- **Conformal Prediction**: Inspired by work of Vovk, Shafer, Lei, and Wasserman
- **Fractional Calculus**: Based on theory by Podlubny, Kilbas, and Magin
- **UI Framework**: PyQt5 development team

### References

**Fractional Calculus:**
1. Podlubny, I. (1999). *Fractional Differential Equations*. Academic Press.
2. Magin, R. L. (2006). *Fractional Calculus in Bioengineering*. Begell House.

**Conformal Prediction:**
1. Vovk, V., Gammerman, A., & Shafer, G. (2005). *Algorithmic Learning in a Random World*. Springer.
2. Lei, J., G'Sell, M., Rinaldo, A., Tibshirani, R. J., & Wasserman, L. (2018). Distribution-free predictive inference for regression. *Journal of the American Statistical Association*, 113(523), 1094-1111.

**Prognostics:**
1. Saxena, A., Celaya, J., Saha, B., Saha, S., & Goebel, K. (2010). Metrics for offline evaluation of prognostic performance. *International Journal of Prognostics and Health Management*, 1(1), 4-23.

**AEC Reliability:**
1. Lahyani, A., Venet, P., Grellet, G., & Viverge, P. J. (1998). Failure prediction of electrolytic capacitors during operation of a switchmode power supply. *IEEE Transactions on Power Electronics*, 13(6), 1199-1207.

---

## 📧 Contact

**Project Maintainer**: [Anindya001](https://github.com/Anindya001)

**Issues & Questions**: [GitHub Issues](https://github.com/Anindya001/FOK-PREDICTION/issues)

**Research Collaboration**: For academic collaboration or industrial applications, please open a discussion on GitHub.

---

## 🌟 Star History

If you find this project useful, please consider giving it a ⭐ on GitHub!

[![Star History Chart](https://api.star-history.com/svg?repos=Anindya001/FOK-PREDICTION&type=Date)](https://star-history.com/#Anindya001/FOK-PREDICTION&Date)

---

**Last Updated:** 2025-10-21
**Version:** 1.0.0
**Status:** Research Software (Pre-Publication)
