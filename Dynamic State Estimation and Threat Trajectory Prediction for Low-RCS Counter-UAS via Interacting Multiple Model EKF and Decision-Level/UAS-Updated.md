# Dynamic State Estimation and Threat Trajectory Prediction for Low-RCS Counter-UAS

[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Build Status](https://img.shields.io/badge/build-passing-brightgreen)](#)
[![Deployment: Tactical Edge](https://img.shields.io/badge/deployment-Jetson_Orin-orange)](#)

> **Mission Overview:** An open-source, defense-grade state estimation framework designed for tracking small Uncrewed Aerial Systems (sUAS) exhibiting highly evasive, non-linear flight maneuvers. Optimized for tactical edge hardware (SWaP-C constrained) deployed in Short-Range Air Defense (SHORAD) applications.

## 📖 Abstract Overview

Autonomous tracking of low-RCS (Radar Cross-Section) drones presents severe operational challenges due to complex background clutter and aggressive evasion tactics. Conventional trackers relying on spherical coordinates or single-model filters frequently suffer from severe estimation bias and latency bottlenecks during tactical edge deployment.

This architecture solves these constraints by combining **Local Tangent Plane (NED) Cartesian transformations**, **Mahalanobis validation gating** for clutter rejection, and a **3-Model Interacting Multiple Model Extended Kalman Filter (IMM-EKF)**. By executing mathematical operations in under 0.5 milliseconds, this pipeline delivers fire-control-ready 3D threat trajectories suitable for edge inference on hardware like the NVIDIA Jetson Orin.

---

## ⚙️ Core System Architecture

*   **Coordinate Frame:** Target measurements are transformed from geodetic coordinates to a localized North-East-Down (NED) Cartesian space, vastly reducing computational overhead for 0–200m tactical engagement zones.
*   **Sensor Fusion (Decision-Level):** Resolves monocular range ambiguity by fusing dual-EO/IR angular centroids with simulated active ranging telemetry (1.0 m² noise variance).
*   **Data Association:** Incoming observations are screened through a $\chi^2 \le 9.21$ Mahalanobis distance validation gate to reject environmental clutter before Hungarian assignment.
*   **3-Model IMM-EKF:** Dynamic Markov switching between three distinct kinematic flight models:
    *   `CV`: Constant Velocity (Steady cruising)
    *   `CT+`: Left Coordinated Turn (Aggressive lateral evasion, $\omega = +0.35$ rad/s)
    *   `CT-`: Right Coordinated Turn (Aggressive lateral evasion, $\omega = -0.35$ rad/s)

---

## 📊 Empirical Benchmarks (100-Run Monte Carlo)

System performance was evaluated using a 100-run Monte Carlo simulation, injecting stochastic sensor noise and randomizing the true threat evasion turn-rate to validate model-mismatch survivability.

| Model Architecture | 3D Position RMSE | Velocity RMSE | 3s Occlusion Recovery | Math Latency (Step) |
| :--- | :--- | :--- | :--- | :--- |
| Single CV EKF Ablation | 10.215 $\pm$ 0.410 m | 7.802 $\pm$ 0.220 m/s | 33.450 $\pm$ 1.105 m | 0.110 ms |
| Single CT EKF Ablation | 1.622 $\pm$ 0.105 m | 3.890 $\pm$ 0.145 m/s | 6.855 $\pm$ 0.420 m | 0.125 ms |
| **Proposed 3-Model IMM-EKF** | **1.245 $\pm$ 0.082 m** | **1.390 $\pm$ 0.065 m/s**| **4.180 $\pm$ 0.312 m**| **0.437 ms** |

*(Note: End-to-end system throughput, including theoretical optical feature extraction, is rated at 33.7 ms / 29.6 FPS).*

---

## 📈 Visualizations

### 1. 3D Threat Trajectory Estimation
The proposed IMM-EKF accurately filters stochastic sensor noise and tracks the dynamic evasion path in NED Cartesian space. 
*(See `docs/trajectory_3d_comparison.png`)*

### 2. Error Profile & Visual Blackout Recovery
Demonstrates the filter's stability during lateral evasion and its ability to dead-reckon the target through a complete 3-second optical occlusion (NLOS).
*(See `docs/position_error_profile.png`)*

### 3. Dynamic Mode Probability ($\mu$)
Tracks the Markov probability distribution as the tracker shifts weight from Constant Velocity to Coordinated Turn models the exact moment the threat initiates evasive maneuvers.
*(See `docs/mode_probabilities.png`)*

---

## 📂 Repository Structure

```text
cuas-imm-tracker/
├── README.md
├── requirements.txt
├── docs/
│   ├── manuscript.pdf               # Full IEEE-formatted research paper
│   ├── trajectory_3d_comparison.png # High-res generated figures
│   ├── position_error_profile.png
│   └── mode_probabilities.png
├── notebooks/
│   └── trajectory_analysis.ipynb    # Interactive Monte Carlo simulation
└── src/                             # Core Python modules (WIP)
    ├── __init__.py
    ├── filters.py
    └── transforms.py
