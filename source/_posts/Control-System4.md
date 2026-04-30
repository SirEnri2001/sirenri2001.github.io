---
layout: post
title: Optimal Control 4
date: 2026/4/29 23:28
tags: [Optimal Control]
author: Henry Han
---

# Optimal Control

## Introduction
[Diagram: Reference -> (+) -> (Measured error) -> [Controller] -> (System Input) -> [System] -> System output]
                          ^                                           |
                          |---------- [Sensor] <--- (Measured output) -|

## Calculus of Variations
$$\min_{x(t)} J(x(t)) = \int_{t_0}^{t_f} L(t, x(t), \dot{x}(t)) dt$$

### * Deterministic Optimal Control
- **Continuous time:**
  $$\min_{x(t), u(t)} J(x(t), u(t)) = \underbrace{\int_{t_0}^{t_f} l(x(t), u(t)) dt}_{\text{"stage cost"}} + \underbrace{l_f(x(t_f))}_{\text{"terminal cost"}}$$
  
  s.t. $\dot{x}(t) = f(x(t), u(t))$  $\leftarrow$ "dynamics constraints"
  (possibly other constraints)
  
  - "This is an infinite dimensional" problem in the following sense:
    [Graph showing a curve with sample points $u_1, u_2, \dots, u_N$]
    $u_{1:N} = \begin{bmatrix} u_1 \\ u_2 \\ \dots \\ u_N \end{bmatrix}$
    $u(t) = \lim_{N \to \infty} u_{1:N}$

- Solutions as open-loop trajectories
- There are a handful of problems with analytic solutions but not many
- We will focus on the discrete-time setting which leads to tractable algorithms.

### * Discrete time
$$\min_{x_{1:N}, u_{1:N-1}} J(x_{1:N}, u_{1:N-1}) = \sum_{k=1}^{N-1} l(x_k, u_k) + l_f(x_N)$$
s.t. $x_{k+1} = f(x_k, u_k)$
$u_{min} \le u_k \le u_{max}$  $\leftarrow$ "torque limits"
$C(x_k) \le 0$  $\leftarrow$ "Obstacle / safety constraints"

- This is now finite dimensional
- Samples $x_k, u_k$ are often called "knot points"
- Continuous -> discrete time using integration (e.g., Runge-Kutta etc.)
- Discrete -> continuous using interpolation

---

### * Pontryagin's Minimum Principle
- Also called "Maximum Principle" if you maximize a reward.
- First-order necessary condition for deterministic optimal control problem.
- In discrete time, just a special case of KKT condition.

#### KKT Condition
$\min_x f(x)$
s.t. $c(x) = 0 : \lambda$
$g(x) \le 0 : \mu$

$L(x, \lambda, \mu) = f(x) + \lambda^T c(x) + \mu^T g(x)$ (Lagrangian)

- **Stationarity:** $\nabla_x L = \nabla_x f(x) + (\frac{\partial c}{\partial x})^T \lambda + (\frac{\partial g}{\partial x})^T \mu = 0$
- **Primal feasibility:** $c(x) = 0, g(x) \le 0$
- **Dual feasibility:** $\mu \ge 0$
- **Complementary slackness:** - $\mu_i \cdot g_i(x) = 0 \quad \forall i$
  - $\mu \odot g(x) = 0$
  - $\mu^T g(x) = 0$

---

- **Given:**
  $$\min_{x_{1:N}, u_{1:N-1}} \sum_{k=1}^{N-1} l(x_k, u_k) + l_f(x_N)$$
  s.t. $x_{k+1} = f(x_k, u_k)$

- **We can form the Lagrangian:**
  $$L = \sum_{k=1}^{N-1} l(x_k, u_k) + \lambda_{k+1}^T (f(x_k, u_k) - x_{k+1}) + l_f(x_N)$$

- **This result is usually stated in terms of the "Hamiltonian":**
  $$H(x, u, \lambda) = l(x, u) + \lambda^T f(x, u)$$

- **Plug it into L:**
  $$L = H(x_1, u_1, \lambda_2) + \left[ \sum_{k=2}^{N-1} H(x_k, u_k, \lambda_{k+1}) - \lambda_k^T x_k \right] + l_f(x_N) - \lambda_N^T x_N$$
  *(Note change to indexing)*

- **Take derivatives w.r.t. x and $\lambda$:**
  $\frac{\partial L}{\partial \lambda_k} = \frac{\partial H}{\partial \lambda_k} - x_{k+1} = f(x_k, u_k) - x_{k+1} = 0$
  $\frac{\partial L}{\partial x_k} = \frac{\partial H}{\partial x_k} - \lambda_k^T = \frac{\partial l}{\partial x_k} + \lambda_{k+1}^T \frac{\partial f}{\partial x_k} - \lambda_k^T = 0$
  $\frac{\partial L}{\partial x_N} = \frac{\partial l_f}{\partial x_N} - \lambda_N^T$

- **For u, we write the min explicitly to handle torque limits:**
  $u_k = \text{argmin}_{\tilde{u}} H(x_k, \tilde{u}, \lambda_{k+1})$
  s.t. $\tilde{u} \in U$ (shorthand for "in feasible set", e.g., $u_{min} \le \tilde{u} \le u_{max}$)

### In summary:
- $x_{k+1} = \nabla_\lambda H(x_k, u_k, \lambda_{k+1})$  $\to$ Just the dynamics
- $\lambda_k = \nabla_x H(x_k, u_k, \lambda_{k+1})$  $\to$ Backwards in time
- $u_k = \text{argmin}_{\tilde{u}} H(x_k, \tilde{u}, \lambda_{k+1})$ s.t. $\tilde{u} \in U$
- $\lambda_N = \frac{\partial l_f}{\partial x_N}$
(Shooting method)

- **Those can be stated almost identically in continuous time:**
  - $\dot{x} = \nabla_\lambda H(x, u, \lambda)$
  - $-\dot{\lambda} = \nabla_x H(x, u, \lambda)$
  - $u = \text{argmin}_{\tilde{u}} H(x, \tilde{u}, \lambda)$ s.t. $\tilde{u} \in U$
  - $\lambda(t_f) = \frac{\partial l_f}{\partial x}$

### * Some Notes:
- Historically many algorithms were based on integration the continuous ODE, forward/backward to do gradient descent on $u(t)$.
- Those are called "indirect" and/or "shooting" methods.
- In continuous time $\lambda(t)$ is called "co-state" trajectory.
- These methods have largely fallen out of favor as computers have improved.

---

# Linear Quadratic Regulator (LQR) Notes

## Table of Contents
- LQR Intro
- LQR via Shooting (Pontryagin's)
- LQR as a QP
- Riccati Recursion

---

## LQR Problem
Any non-linear control **local** problem $\to$ LQR Problem

$$\min_{x_{1:N}, u_{1:N-1}} J = \sum_{k=1}^{N-1} \left( \frac{1}{2} x_k^T Q_k x_k + \frac{1}{2} u_k^T R_k u_k \right) + \frac{1}{2} x_N^T Q_N x_N$$

**s.t.** $x_{k+1} = A_k x_k + B_k u_k$

**Linear Quadratic Regulator**
- $Q \ge 0$
- $R > 0$ (strictly positive definite)

### Characteristics:
- Can (locally) approximate many nonlinear problems.
- Computationally tractable.
- Many extensions e.g., finite vs infinite horizon, stochastic, etc.
- **"Time invariant"** if $A_k = A, B_k = B, Q_k = Q, R_k = R \quad \forall t$.
- **"Time varying"** otherwise.

---

## LQR with Indirect Shooting
- $x_{k+1} = \nabla_{\lambda} H(x_k, u_k, \lambda_{k+1}) = A x_k + B u_k$
- $\lambda_k = \nabla_x H(x_k, u_k, \lambda_{k+1}) = Q x_k + A^T \lambda_{k+1}$
- $\lambda_N = Q_N x_N$
- $u_k = \text{argmin}_u H(x_k, u, \lambda_{k+1}) = -R^{-1} B^T \lambda_{k+1}$

### Procedure:
1. Start with an initial guess $u_{1:N-1}$ trajectory.
2. Simulate (**"rollout"**) to get $x_{1:N}$.
3. Backward pass to get $\lambda$ and $\Delta u$.
4. Rollout with line search on $\Delta u$.
5. Go to (3) until convergence.

### Example: "Double Integrator"
$$\dot{x} = \begin{bmatrix} \dot{q} \\ \ddot{q} \end{bmatrix} = \begin{bmatrix} 0 & 1 \\ 0 & 0 \end{bmatrix} \begin{bmatrix} q \\ \dot{q} \end{bmatrix} + \begin{bmatrix} 0 \\ 1 \end{bmatrix} u$$
- $\dot{q}$: velocity
- $\ddot{q}$: acceleration
- $q$: position
- $u$: Force

*Think of this as a brick sliding on ice (no friction).*

**Discrete time:**
$$x_{k+1} = \underbrace{\begin{bmatrix} 1 & h \\ 0 & 1 \end{bmatrix}}_{A} \begin{bmatrix} q_k \\ \dot{q}_k \end{bmatrix} + \underbrace{\begin{bmatrix} \frac{1}{2}h^2 \\ h \end{bmatrix}}_{B} u_k$$
*(where $h$ is the time step)*

---

## LQR as a QP
- Assume $x_1$ (initial state) is given (not a decision variable).
- Define $z = \begin{bmatrix} u_1 \\ x_2 \\ u_2 \\ x_3 \\ \vdots \\ x_N \end{bmatrix}$ (decision variable).
- Define $H = \text{diag}(R_1, Q_2, R_2, Q_3, \dots, Q_N)$ such that $J = \frac{1}{2} z^T H z$.
- Define $C$ and $d$:
$$\underbrace{\begin{bmatrix} B_1 & -I & 0 & 0 & \cdots \\ A_2 & B_2 & -I & 0 & \cdots \\ & & \ddots & & \\ & & A_{N-1} & B_{N-1} & -I \end{bmatrix}}_{C \text{ Matrix}} \underbrace{\begin{bmatrix} u_1 \\ x_2 \\ u_2 \\ \vdots \\ x_N \end{bmatrix}}_{z} = \underbrace{\begin{bmatrix} -A_1 x_1 \\ 0 \\ 0 \\ \vdots \\ 0 \end{bmatrix}}_{d}$$

Now we can write the LQR problem as a standard **QP**:
$$\min_z \frac{1}{2} z^T H z$$
$$\text{s.t. } Cz = d$$

### The Lagrangian of this QP is:
$$L(z, \lambda) = \frac{1}{2} z^T H z + \lambda^T (Cz - d)$$

### KKT Condition:
- $\nabla_z L = Hz + C^T \lambda = 0$
- $\nabla_{\lambda} L = Cz - d = 0$

$$\Rightarrow \begin{bmatrix} H & C^T \\ C & 0 \end{bmatrix} \begin{bmatrix} z \\ \lambda \end{bmatrix} = \begin{bmatrix} 0 \\ d \end{bmatrix}$$

We get the **exact solution** by solving one linear system!
- **Example:** Double integrator.
- **Note:** Much better than shooting!

---

## A closer look at the LQR QP:
- The KKT system for LQR is very sparse (lots of zeros) and has a lot of structure.

### Row-by-Row Analysis:
- **Terminal condition (Row 6):**
  $Q_N x_N - \lambda_N = 0 \quad \to \text{terminal condition from Pontryagin}$
  $\Rightarrow \lambda_N = Q_N x_N$

- **Control Step (Row 5):**
  $R u_3 + B^T \lambda_4 = R u_3 + B^T Q_N x_4 = 0$ (Plugin dynamics for $x_4$)
  $\Rightarrow R u_3 + B^T Q_N (A x_3 + B u_3) = 0$
  $\Rightarrow u_3 = -\underbrace{(R + B^T Q_N B)^{-1} B^T Q_N A}_{K_3} x_3$

- **Recursion step (Row 4):**
  $Q x_3 - \lambda_3 + A^T \lambda_4 = 0$
  *Plugin* $\lambda_4$: $Q x_3 - \lambda_3 + A^T Q_N x_4 = 0$
  *Plugin Dynamics*: $Q x_3 - \lambda_3 + A^T Q_N (A x_3 + B u_3)$
  *Plugin* $u_3 = K_3 x_3$: $\lambda_3 = \underbrace{(Q + A^T Q_N (A - B K_3))}_{P_3} x_3$

### Riccati Equation / Recursion:
Now we have a recursion for $K$ and $P$:
1. $P_N = Q_N$
2. $K_k = (R + B^T P_{k+1} B)^{-1} B^T P_{k+1} A$
3. $P_k = Q + A^T P_{k+1} (A - B K_k)$

### Summary:
- We can solve the QP by doing a **backward Riccati** followed by a **forward rollout** to compute $x_{1:N}$ and $u_{1:N-1}$ from initial conditions.
- General (dense) QP has complexity $O(N^3 (n+m)^3)$.
- Riccati solution complexity: $O((n+m)^3 N)$.