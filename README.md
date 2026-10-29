# State Estimation of Vanadium Redox Flow Batteries

**Master's thesis:** *Development of an Observer Algorithm for State Estimation of Vanadium Redox-Flow Batteries*

**Author:** Hasmukhkumar Mukeshbhai Dhaduk  
**Institution:** Deggendorf Institute of Technology (Technische Hochschule Deggendorf)  
**Industrial research context:** Kermi GmbH, R&D  
**Tools:** MATLAB, Simulink

## Overview

This repository documents the mathematical modelling, simulation and observer-based state estimation of a **vanadium redox flow battery (VRFB)**. The work investigates how internal vanadium-ion concentrations and state of charge (SoC) can be estimated using electrical measurements and a dynamic electrochemical model.

Two estimation approaches were investigated:

- **Lyapunov-based nonlinear observer:** a model-based design using transformed concentration states and correction terms.
- **Unscented Kalman Filter (UKF):** a sigma-point nonlinear state estimator combining model predictions with voltage measurements.

The two methods were evaluated under charge/discharge operation and different electrolyte flow rates, with emphasis on convergence, estimation error and implementation complexity.

## Research objectives

1. Develop a dynamic eight-state model of vanadium-ion concentrations in the cell and electrolyte tanks.
2. Simulate concentration dynamics under measured battery current and electrolyte flow rate.
3. Estimate internal states from the negative and positive half-cell voltage signals.
4. Evaluate the Lyapunov observer and UKF using concentration-estimation error and SoC comparisons.
5. Assess the trade-off between observer accuracy and computational complexity.

## Repository structure

```text
MasterThesis/
├── Matlab_code/          # MATLAB modelling and estimation code
├── Model plot/           # Model simulation plots
├── Model_vs_observer/    # Model/observer comparison results
├── Simulink_model/       # Simulink-related model files
└── README.md
```

> The structure above reflects the top-level directories. Exact MATLAB entry-point filenames, dataset locations and software dependencies should be checked inside each folder before execution.

## VRFB electrochemical model

The model represents four vanadium species in both the electrochemical cell and external tanks. The state vector is

```math
\mathbf{x}=\begin{bmatrix}
c_2^c & c_3^c & c_4^c & c_5^c & c_2^t & c_3^t & c_4^t & c_5^t
\end{bmatrix}^{\!T},
```

where superscripts $c$ and $t$ denote **cell** and **tank** concentrations. The subscripts correspond to vanadium species V(II), V(III), V(IV) and V(V).

The concentration dynamics account for electrolyte circulation, applied current and membrane-related crossover effects. In compact form,

```math
\dot{\mathbf{x}}=\left(q\mathbf{A}_1+\frac{S}{d\,v_c}\mathbf{A}_2\right)\mathbf{x}
+\frac{1}{Fv_c}\mathbf{b}I_{\mathrm{eff}}.
```

Here $q$ is the electrolyte flow rate, $S$ the membrane area, $d$ the membrane thickness, $v_c$ the cell electrolyte volume, $F$ Faraday's constant, and $I_{\mathrm{eff}}$ the efficiency-adjusted current. The matrices $\mathbf{A}_1$ and $\mathbf{A}_2$ describe flow coupling and modeled crossover/reaction terms, respectively.

A forward-Euler discretisation was used in the MATLAB implementation:

```math
\mathbf{x}_{k+1}=\mathbf{x}_k+\Delta t\,f(\mathbf{x}_k,I_k,q_k).
```

### Voltage measurements

Negative and positive half-cell potentials provide information about tank concentration ratios via Nernst-type relationships:

```math
E_{\mathrm{neg}}=E'_{\mathrm{neg}}+\frac{RT}{F}\ln\left(\frac{c_2^t}{c_3^t}\right),
```

```math
E_{\mathrm{pos}}=E'_{\mathrm{pos}}+\frac{RT}{F}\ln\left(\frac{c_5^t}{c_4^t}\right).
```

The measured current, electrolyte flow rate and two half-cell potentials serve as inputs/observations for state estimation. Model assumptions include constant temperature and nominally equal electrolyte flow rates on both sides.

### State of charge

The state of charge is calculated from tank concentrations on each side:

```math
\mathrm{SoC}_{\mathrm{neg}}=\frac{c_2^t}{c_2^t+c_3^t},\qquad
\mathrm{SoC}_{\mathrm{pos}}=\frac{c_5^t}{c_4^t+c_5^t}.
```

## State-estimation algorithms

### 1. Lyapunov-based nonlinear observer

The nonlinear observer uses transformed concentration variables, including tank concentration ratios and inverse concentration sums. The observer correction law is designed around Lyapunov stability arguments, with gain and state-bound constraints to support convergence. Its main strengths are a structured design and relatively low computational burden; performance depends on observer tuning and initial-state mismatch.

### 2. Unscented Kalman Filter (UKF)

The UKF estimates the complete eight-state concentration vector. Sigma points are propagated through the nonlinear state model, and predicted voltage measurements are used in the update step. The implementation uses model/process covariance $Q$, measurement covariance $R$ and the initial covariance $P_0$ as tuning parameters.

Representative thesis settings:

| Parameter | Value |
|---|---:|
| State dimension | 8 |
| Sampling time | 1 s |
| UKF $\alpha$ | $10^{-3}$ |
| UKF $\beta$ | 2 |
| UKF $\kappa$ | 0 |
| $Q$ (nominal scalar setting) | $10^{-9}$ |
| $R$ (nominal scalar setting) | $10^{-2}$ |
| $P_0$ (nominal scalar setting) | $10^{-2}$ |

Scalar covariance settings are understood as the scale factors for the corresponding matrices, as implemented in the thesis code.

## Evaluation conditions and thesis findings

Two representative operating conditions were investigated:

| Condition | Flow rate | Approximate flow in L/s |
|---|---:|---:|
| FR60 | 60 mL/min | $1.00\times10^{-3}$ |
| FR75 | 75 mL/min | $1.25\times10^{-3}$ |

The following grouped RMSE values (in mol/L) summarise the UKF estimation results from the thesis:

| State group | FR60 UKF RMSE | FR75 UKF RMSE |
|---|---:|---:|
| Negative cell | 0.0607 | 0.0745 |
| Positive cell | 0.0300 | 0.0384 |
| Negative tank | 0.0565 | 0.0684 |
| Positive tank | 0.0264 | 0.0336 |

In the investigated cases, the **UKF generally achieved lower concentration estimation error** than the Lyapunov-based observer, particularly for the positive-side tank and cell states. The nonlinear observer remains attractive where simpler implementation and lower computational overhead are important.

Approximate Lyapunov-observer settling/convergence times reported in the thesis were **10,000 s (FR60)** and **12,500 s (FR75)**. Separate SoC verification comparisons reported errors of approximately **3.0% (FR60)** and **2.96% (FR75)**. These SoC results should not be interpreted as the grouped concentration RMSE values above.

> The values above summarise the thesis experiments rather than guaranteeing identical results for arbitrary inputs, initial conditions or MATLAB/Simulink versions.

## Getting started

1. Clone this repository:

   ```bash
   git clone https://github.com/hasmukh9697/MasterThesis.git
   cd MasterThesis
   ```

2. Open MATLAB and add the relevant repository folders to the MATLAB path.
3. Review `Matlab_code/` and `Simulink_model/` to identify the model, observer and test scripts/models.
4. Check the required input datasets and file paths in the chosen scripts before running the model.
5. Run the concentration model first, then the observer/UKF evaluation where provided; review the output plots and comparisons.

**Requirements:** MATLAB; Simulink for `.slx`/`.mdl` models. The exact MATLAB release and any additional toolbox requirements have not been verified.

## Applications

The methods explored here are relevant to battery monitoring, model-based SoC estimation, energy storage control research, and state observers for nonlinear electrochemical systems. The repository is a **research prototype**, not a validated production battery-management system.

## Acknowledgements

This work was carried out in the context of a master's thesis at Deggendorf Institute of Technology with industrial research involvement at Kermi GmbH. Experimental data contributions from the ISES Energy Storage Lab at Hochschule München are gratefully acknowledged.

## Citation and use

If using this work academically, please cite the master's thesis and identify the specific code or figures used. Check repository contents and institutional/data-sharing permissions before redistributing experimental datasets. No software license is implied unless a license file is added to the repository.

## Contact

Repository maintainer: [hasmukh9697](https://github.com/hasmukh9697)
