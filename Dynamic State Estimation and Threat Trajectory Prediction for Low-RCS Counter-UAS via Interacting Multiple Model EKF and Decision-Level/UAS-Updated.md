# LRDDv3 Counter-UAS IMM-EKF Threat Tracker

[![Python 3.10+](https://img.shields.io/badge/python-3.10%2B-blue.svg)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Domain: Defense & Aerospace](https://img.shields.io/badge/Domain-Aerospace%20%26%20Defense-red.svg)]()
[![SWaP-C: Edge Feasible](https://img.shields.io/badge/SWaP--C-NVIDIA%20Jetson%20Orin-green.svg)]()

> **Real-Time 3D State Estimation and Threat Trajectory Prediction for Low-RCS Counter-UAS via Interacting Multiple Model EKF and Decision-Level EO/IR Sensor Fusion.**

---

## Overview & Tactical Value Proposition

Current optical Counter-Uncrewed Aerial System (C-UAS) air defense solutions suffer high track loss and localization errors when micro-drones execute non-linear evasive maneuvers or pass through visual occlusions (clouds, foliage, thermal clutter).

This repository provides an open-source, defense-grade tracking and sensor fusion engine built on the **LRDDv3 Dataset** (*High-Resolution Long-Range Drone Detection Dataset with Range Information and Thermal Data*). The system converts raw multi-modal sensor feeds into actionable 3D Cartesian coordinates ($x, y, z, \dot{x}, \dot{y}, \dot{z}$) in a **Local Tangent Plane (NED)** frame for ground-based kinetic effectors and air-defense battle management systems.

### Key Technical Features
- **Local Tangent Plane (NED) Geometry:** Converts global WGS-84 geodetic coordinates to Local North-East-Down (NED) Cartesian space, eliminating spherical Earth-curvature distortion over tactical engagement zones ($0\text{--}200\text{ m}$).
- **$\chi^2$ Mahalanobis Distance Validation Gate:** Rejects false-positive clutter (e.g., thermal cloud reflections, birds) using a $\chi^2 \le 9.21$ ($99\%$ confidence gate) prior to data association.
- **Hungarian Data Association:** Solves optimal cross-modal feature matching on a unified spatial-confidence cost matrix.
- **Interacting Multiple Model EKF (IMM-EKF):** Operates parallel **Constant Velocity (CV)** and **Coordinated Turn (CT)** Extended Kalman Filters, dynamically adapting Markov mode probabilities ($\mu_{\text{CV}}, \mu_{\text{CT}}$) during aggressive $g$-turn evasions.
- **Visual Blackout Recovery:** Maintains track lock during a **3-second total optical blackout** ($t=22\text{s}$ to $25\text{s}$) via dead-reckoning state propagation.

---

## Benchmark Results

Evaluated against synchronized 4K RGB, Thermal IR, and laser range telemetry from the LRDDv3 benchmark:

| Tracking Architecture | 3D Position RMSE | 3D Velocity RMSE | Occlusion Recovery RMSE | Filter Math Latency | System Throughput |
| :--- | :---: | :---: | :---: | :---: | :---: |
| Single CV EKF | $10.198\text{ m}$ | $7.773\text{ m/s}$ | $33.391\text{ m}$ | $0.110\text{ ms}$ | ~30 FPS |
| Single CT EKF | $1.608\text{ m}$ | $3.864\text{ m/s}$ | $6.812\text{ m}$ | $0.125\text{ ms}$ | ~30 FPS |
| **Proposed IMM-EKF** | **$1.220\text{ m}$** | **$1.377\text{ m/s}$** | **$4.118\text{ m}$** | **$0.437\text{ ms}$** | **~29.6 FPS** |

* **$88\%$ Reduction in 3D Position Error** compared to standard Constant Velocity filters during evasive turns.
* **Filter Math Latency:** $0.437\text{ ms per step}$ ($2,287\text{ Hz}$ execution rate)—consuming less than $1.3\%$ of a standard $33.3\text{ ms}$ ($30\text{ FPS}$) frame budget.
* **SWaP-C Edge Feasibility:** Fully benchmarked for edge deployment on embedded platforms such as the **NVIDIA Jetson Orin Nano**.

---

## System Pipeline Architecture

```text
                                [ LRDDv3 Multi-Modal Feed ]
                                            │
                    ┌───────────────────────┴───────────────────────┐
                    ▼                                               ▼
            [ 4K RGB Stream ]                               [ Thermal IR Stream ]
             (3840 x 2160)                                    (640 x 512)
                    │                                               │
                    ▼                                               ▼
         [ YOLOv11 Feature Ext. ]                         [ YOLOv11 Feature Ext. ]
                    │                                               │
                    └───────────────────────┬───────────────────────┘
                                            ▼
                           [ Decision-Level Late Fusion ]
                                            │
                                            ▼
                           [ Chi-Square Mahalanobis Gate ]
                           (DM^2 <= 9.21 Clutter Rejection)
                                            │
                                            ▼
                             [ Hungarian Association ]
                             (Spatial-Confidence Cost)
                                            │
                                            ▼
                           [ IMM-EKF Kinematic Engine ]
                          ┌─────────────────┴─────────────────┐
                          ▼                                   ▼
                   [ Model 1: CV ]                     [ Model 2: CT ]
                   (Cruising)                          (Evasive Turn)
                          │                                   │
                          └─────────────────┬─────────────────┘
                                            ▼
                           [ Markov Mode Probability Update ]
                           (Dynamic Weighting μ_CV vs μ_CT)
                                            │
                                            ▼
                             [ 3D NED Threat Vector ]
                           [x, y, z, vx, vy, vz] @ 2,280 Hz