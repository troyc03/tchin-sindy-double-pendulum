# Sparse Identification of the Double Pendulum

**Discovering the equations behind chaotic motion.**

A computational physics project exploring whether the governing dynamics of a nonlinear double pendulum can be recovered from noisy time-series data using **Sparse Identification of Nonlinear Dynamics (SINDy)**, with numerical simulations validated against a classical fourth-order Runge–Kutta (RK4) integrator.

---

## Overview

The double pendulum is a deceptively simple mechanical system with remarkably complex behavior. Two coupled pendulums governed by deterministic equations can exhibit extreme sensitivity to initial conditions, making their long-term motion difficult to predict.

This project investigates a fundamental question:

> **Can we reconstruct the underlying equations of motion of a nonlinear, potentially chaotic system using only noisy observational data?**

To explore this question, the project combines physics-based modeling, numerical integration, synthetic time-series generation, noise injection, and sparse regression.

The double pendulum serves as a testbed for **data-driven model discovery**: starting with observations of the system's motion, the goal is to identify a compact mathematical model that captures its underlying dynamics.

## The Pipeline

The project follows a physics-to-discovery workflow:

1. **Model the system:** Formulate the nonlinear equations governing the coupled pendulums.
2. **Simulate the dynamics:** Generate trajectories using numerical integration.
3. **Generate observations:** Construct time-series data from the simulated motion and introduce noise to approximate imperfect measurements.
4. **Build a candidate library:** Express the dynamics using a library of candidate nonlinear functions.
5. **Identify the governing equations:** Apply SINDy to select a sparse set of active terms and estimate their coefficients.
6. **Validate the discovered model:** Compare the identified dynamics against the reference simulation using trajectory comparisons and quantitative error metrics.

The central idea is simple: rather than assuming the complete governing equations are known, use observed behavior to infer a parsimonious mathematical description.

## The Physics

Consider a double pendulum consisting of two point masses connected by rigid, massless rods.

The generalized coordinates are

$$
\mathbf{q} =
\begin{bmatrix}
\theta_1 \\
\theta_2
\end{bmatrix},
$$

where \(\theta_1\) and \(\theta_2\) describe the angular positions of the two pendulums.

The system's state is

$$
\mathbf{x} =
\begin{bmatrix}
\theta_1 & \theta_2 & \dot{\theta}_1 & \dot{\theta}_2
\end{bmatrix}^{T}.
$$

Its dynamics can be written in first-order form as

$$
\dot{\mathbf{x}} = \mathbf{f}(\mathbf{x}),
$$

where \(\mathbf{f}\) represents the nonlinear vector field governing the evolution of the system.

Unlike a simple harmonic oscillator, the double pendulum contains coupled nonlinear terms involving angular velocities and trigonometric functions of the relative angle. Depending on its energy and initial conditions, its motion can range from relatively regular oscillations to strongly chaotic trajectories.

The project uses this system to investigate how well sparse regression can recover nonlinear dynamics from sampled observations.

## Numerical Simulation

A reliable reference trajectory is essential for evaluating any discovered model.

The project uses the **classical fourth-order Runge–Kutta method (RK4)** to numerically integrate the governing equations:

$$
\mathbf{x}_{n+1}
=
\mathbf{x}_n
+
\frac{\Delta t}{6}
\left(
\mathbf{k}_1
+2\mathbf{k}_2
+2\mathbf{k}_3
+\mathbf{k}_4
\right),
$$

where the intermediate stages are

$$
\mathbf{k}_1 &= \mathbf{f}(\mathbf{x}_n),\\
\mathbf{k}_2 &= \mathbf{f}\left(\mathbf{x}_n+\frac{\Delta t}{2}\mathbf{k}_1\right),\\
\mathbf{k}_3 &= \mathbf{f}\left(\mathbf{x}_n+\frac{\Delta t}{2}\mathbf{k}_2\right),\\
\mathbf{k}_4 &= \mathbf{f}\left(\mathbf{x}_n+\Delta t\,\mathbf{k}_3\right).
$$

RK4 provides a numerical reference against which the learned model can be evaluated. It is a numerical approximation, not an exact analytical solution, so validation also depends on the integration step size and numerical error.

## Sparse Identification of Nonlinear Dynamics (SINDy)

SINDy, introduced by Steven L. Brunton, Joshua L. Proctor, and J. Nathan Kutz, is a data-driven framework for discovering governing equations from measurements of a dynamical system.

Instead of fitting an unrestricted black-box model, SINDy searches for a sparse representation of the system's dynamics.

Given sampled states arranged into a data matrix

$$
\mathbf{X} =
\begin{bmatrix}
\mathbf{x}(t_1)^T\\
\mathbf{x}(t_2)^T\\
\vdots\\
\mathbf{x}(t_m)^T
\end{bmatrix},
$$

the corresponding time derivatives are represented by \(\dot{\mathbf{X}}\).

A candidate library of nonlinear functions is constructed:

$\Theta(\mathbf{X}) =
\begin{bmatrix}
1 & \mathbf{X} & \mathbf{X}^{(2)} & \cdots
\end{bmatrix}$

where the library may contain polynomial terms, trigonometric functions, and other physically motivated candidate features.

The dynamics are approximated as

$$
\dot{\mathbf{X}}
\approx
\Theta(\mathbf{X})\boldsymbol{\Xi}
$$

where \(\boldsymbol{\Xi}\) contains the coefficients of the candidate terms.

Sparse regression seeks a coefficient matrix with relatively few nonzero entries. In doing so, it attempts to distinguish the terms needed to explain the observed dynamics from unnecessary candidates.

**The objective:** recover a compact, interpretable model that reproduces the important behavior of the double pendulum without retaining every term in the candidate library.

### Why Sparsity Matters

A dense model may fit the observed data while containing redundant or spurious terms. A sparse model instead favors a smaller set of active terms, potentially revealing the structure of the underlying equations.

However, successful identification depends on several factors:

- The quality and duration of the observed trajectories.
- The sampling frequency and numerical differentiation method.
- The noise level in the measurements.
- The expressiveness of the candidate function library.
- The choice of sparsity threshold or regularization strength.
- Whether the available data sufficiently excite the system's relevant dynamics.

These considerations make the double pendulum a useful benchmark for studying the strengths and limitations of data-driven equation discovery.

## Noise and Robustness

Real measurements are rarely perfect. To investigate the effect of imperfect observations, the project introduces noise into simulated time-series data before applying the identification procedure.

This makes it possible to examine how measurement corruption affects the estimated derivatives, selected model terms, and recovered coefficients.

Noise robustness is particularly important for SINDy because derivative estimation can amplify high-frequency measurement errors.

A complete evaluation should distinguish between:

- **State reconstruction:** How closely does the model reproduce the observed state variables?
- **Coefficient recovery:** How closely do the identified coefficients match the reference equations?
- **Model structure recovery:** Does the algorithm select the correct governing terms?
- **Trajectory prediction:** How well does the identified model reproduce the system's subsequent evolution?

These are related but distinct measures of success. A model can fit a short trajectory well without recovering the correct governing equations.

## Validation Strategy

The identified model is evaluated against the RK4 reference simulation.

The validation workflow can include:

| Evaluation | Purpose |
|---|---|
| State trajectory comparison | Compare angular positions and velocities over time. |
| Derivative prediction | Evaluate the learned vector field against reference derivatives. |
| Coefficient comparison | Measure agreement between recovered and reference coefficients when the model representations are compatible. |
| Reconstruction error | Quantify discrepancies between predicted and reference states. |
| Noise sensitivity | Examine how identification changes as the noise level increases. |
| Long-term behavior | Investigate whether the discovered model reproduces the system's qualitative dynamics. |

A standard quantitative measure is the root mean squared error (RMSE):

$$

\mathrm{RMSE}
=
\sqrt{
\frac{1}{N}
\sum_{i=1}^{N}
\left(
y_i-\hat{y}_i
\right)^2
}

$$

Here, \(y_i\) denotes a reference value and \(\hat{y}_i\) denotes the corresponding model prediction.

For chaotic systems, small differences in initial conditions and numerical errors can eventually produce large trajectory divergence. Consequently, long-term pointwise agreement alone is not always a reliable measure of model quality. Short-horizon prediction, derivative accuracy, coefficient recovery, and qualitative dynamical behavior provide complementary evidence.

## Results

*Replace this section with the results produced by your implementation.*

Recommended figures and tables include:

- **Reference dynamics:** Double-pendulum angular trajectories generated using RK4.
- **Effect of noise:** Clean versus noisy observations.
- **Sparse recovery:** Identified coefficients compared with the reference model.
- **Trajectory validation:** Reference and discovered trajectories plotted together.
- **Error analysis:** Prediction error as a function of time or noise level.

A useful results table might look like this:

| Metric | Result |
|---|---:|
| Observation noise level | `[Insert value]` |
| Sampling interval | `[Insert value]` |
| Number of identified terms | `[Insert value]` |
| Derivative prediction RMSE | `[Insert value]` |
| State prediction RMSE | `[Insert value]` |

Report the experimental setup alongside each result so that the numbers are interpretable and reproducible.

## Getting Started

### Prerequisites

The implementation-specific requirements should be documented here. Depending on the libraries used in your repository, these may include:

- Python 3.x
- NumPy
- SciPy
- Matplotlib
- A SINDy implementation or custom sparse-regression routine

### Installation

Clone the repository:

```bash
git clone https://github.com/troyc03/tchin-sindy-double-pendulum
```

Create and activate a virtual environment if desired:

```bash
python -m venv .venv
```

On macOS or Linux:

```bash
source .venv/bin/activate
```

On Windows:

```powershell
.venv\Scripts\activate
```

Install the project's dependencies using the method supported by the repository. For example, if a `requirements.txt` file is provided:

```bash
pip install -r requirements.txt
```

### Running the Project

Add the actual entry point for your implementation. For example:

```bash
python <YOUR_MAIN_SCRIPT>.py
```

The script should document or demonstrate the relevant stages of the workflow, including simulation, data preparation, sparse identification, and validation.

## Project Structure

The following is an illustrative structure. Adapt it to match the actual files in your repository.

```text
sparse-identification-double-pendulum/
├── data/                 # Simulated or processed observations
├── src/
│   ├── dynamics.py       # Double-pendulum equations
│   ├── simulation.py    # Numerical integration
│   ├── sindy.py         # Sparse identification
│   └── validation.py    # Model evaluation
├── notebooks/            # Experiments and visualizations
├── figures/              # Generated plots
├── tests/                # Numerical and regression tests
├── requirements.txt
└── README.md
```

## Key Concepts

This project brings together several important ideas in computational science:

- **Nonlinear dynamics:** Modeling coupled systems whose evolution depends nonlinearly on their state.
- **Numerical integration:** Approximating continuous-time dynamics using discrete time steps.
- **Sparse regression:** Selecting a small subset of explanatory terms from a larger candidate library.
- **System identification:** Inferring mathematical models from observed input-output or state data.
- **Scientific computing:** Combining mathematical modeling, computational experiments, and quantitative validation.
- **Chaos and sensitivity:** Understanding why small perturbations can lead to substantially different trajectories.

## Future Work

Potential extensions include:

- Investigating how model recovery changes with increasing measurement noise.
- Comparing sparse regression thresholds and regularization strategies.
- Exploring polynomial versus physics-informed candidate libraries.
- Comparing finite-difference derivative estimates with smoothing or other differentiation methods.
- Testing whether identified equations generalize to unseen initial conditions.
- Studying the effects of sampling frequency and trajectory duration.
- Comparing discovered dynamics with alternative system-identification approaches.
- Extending the workflow to additional nonlinear mechanical systems.

## References

1. Brunton, S. L., Proctor, J. L., and Kutz, J. N. (2016). *Discovering governing equations from data by sparse identification of nonlinear dynamical systems.* Proceedings of the National Academy of Sciences, 113(15), 3932–3937. https://doi.org/10.1073/pnas.1517384113

## Author

[Troy Chin](https://github.com/troyc03/))

---

*The goal is not merely to simulate a complex system, but to discover the mathematical structure that makes its behavior possible.*
