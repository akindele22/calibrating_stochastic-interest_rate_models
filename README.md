# Calibration of Stochastic Interest & Volatility Models

## Overview

This project implements the calibration of the **Bates (1996) stochastic volatility model with jumps** to SM Energy (SM) option data. The calibration process uses the **Carr-Madan (1999) Fourier-transform** based approach for model pricing. The calibration window focuses on a **60-day option maturity slice**, covering 5 Heston parameters + 3 jump parameters.


## Models Implemented

### Heston (1993) Stochastic Volatility Model

The base stochastic volatility model with parameters:
- `v₀` – Initial variance
- `κ` – Mean reversion speed
- `θ` – Long-run average variance
- `σ` – Volatility of variance (vol-of-vol)
- `ρ` – Correlation between stock price and variance

### Bates (1996) Model

Extends Heston by adding compound Poisson jump process:
- `λⱼ` – Jump intensity (jumps per year)
- `μⱼ` – Mean log-jump size
- `δⱼ` – Jump volatility (standard deviation)

## Pricing Methods

| Method | Description |
|--------|-------------|
| **Carr-Madan (1999)** | FFT-based option pricing using characteristic functions |
| **Lewis (2001)** | Semi-analytical Fourier approach |
| **Monte Carlo Simulation** | Euler-Maruyama discretization with full truncation |

## Calibration Strategy

### Two-Stage Optimisation Procedure

1. **Differential Evolution (Global Search)**
   - Population size: 20
   - Generations: up to 2000
   - Latin Hypercube seeding with Seed 42

2. **L-BFGS-B (Local Search)**
   - Quasi-Newton gradient-based optimization
   - Box constraints for parameter bounds

### Parameter Boundaries (Step 2 – Bates)

| Parameter | Lower | Upper | Description |
|-----------|-------|-------|-------------|
| v₀ | 0.05 | 0.80 | Initial variance (22%-89% vol) |
| κ | 0.10 | 20.0 | Mean-reversion speed |
| θ | 0.05 | 0.80 | Long-run variance |
| σ | 0.05 | 2.00 | Volatility of variance |
| ρ | -0.99 | 0.99 | S-V correlation |
| λⱼ | 0.00 | 10.0 | Jump intensity |
| μⱼ | -0.50 | 0.50 | Mean log-jump |
| δⱼ | 0.01 | 0.50 | Jump volatility |

## Key Results

### Step 1 – Heston Calibration (15-day options)

| Parameter | Lewis (2001) | Carr-Madan (1999) |
|-----------|--------------|-------------------|
| v₀ | 0.0963 | 0.1292 |
| κ | 0.1000 | 1.4829 |
| θ | 0.0100 | 0.0010 |
| σᵥ | 0.9082 | 1.9978 |
| ρ | -0.9500 | -0.9104 |

**RMSE (Lewis):** $0.9505

### Step 2 – Bates Calibration (60-day options)

| Parameter | Value |
|-----------|-------|
| κ | 2.50 |
| θ | 0.040 |
| ξ | 0.30 |
| ρ | -0.60 |
| v₀ | 0.040 |
| λ | 0.40 |
| µⱼ | -0.06 |
| σⱼ | 0.18 |

**RMSE:** ~$0.04

### Step 3 – CIR Interest Rate Model

Calibrated to Euribor rates:

| Parameter | Value |
|-----------|-------|
| κ | 0.5020 |
| θ | 0.0557 |
| σ | 0.1000 |

**Monte Carlo Simulation (100,000 paths):**
- Expected 12-month Euribor: 3.89%
- 95% CI: [1.51%, 7.26%]

## Option Pricing Results

### Asian Call Option (20-day ATM)
- Fair price: $3.42
- Bank fee (4%): $0.137
- **Client price: $3.56**

### European Put Option (70-day, 95% moneyness)
- Fair price: $8.91
- Bank fee (4%): $0.36
- **Client price: $9.27**

## Technical Implementation Details

### FFT Configuration
- Grid size: N = 4096
- Spacing: η = 0.25
- Damping factor: α = 1.5
- Simpson weights for integration accuracy

### Important Fix Applied
**Issue:** Initial FFT implementation returned constant prices ($231.90) regardless of strike.

**Solution:** Center log-strike grid on forward price `F = S₀ × e^(rT)` instead of spot price.

### Monte Carlo Settings
- Paths: 100,000 – 200,000
- Time steps: 20 (daily)
- Euler-Maruyama with full truncation for variance


