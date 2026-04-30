---
layout: post
title: Optimal Control 2
date: 2026/4/29 23:28
tags: [Optimal Control]
author: Henry Han
---

## Discrete-Time Dynamics

### "Explicit" Form
$$x_{k+1} = f_d(x_k, u_k)$$
* $f_d$ is the "discrete" dynamics.

### Simplest Discretization: Forward Euler Integration
$$x_{k+1} = \underbrace{x_k + h f(x_k, u_k)}_{f_d}$$
* $h$: time step

### Pendulum Simulation
* $l=1, m=1, h=0.1, 0.01$ (**blows up**)

---

## Stability of Discrete-Time Systems

* **In continuous time:** $Re[eig(\frac{\partial f}{\partial x})] < 0 \Rightarrow$ stable.
* **In discrete time, dynamics is an iterated map:**
    $x_N = f_d(f_d(f_d(...f_d(x_0)...)))$

* **Linearize + apply chain rule:**
    $\frac{\partial x_N}{\partial x_0} = \prod_{i=0}^{N-1} \frac{\partial f_d}{\partial x} \big|_{x_0} = A_d^N$

* **Assume equilibrium is at $x_0 = 0$:**
    Stable $\Rightarrow \lim_{k \to \infty} A_d^k x_0 = 0 \quad \forall x_0$
    $\Rightarrow \lim_{k \to \infty} A_d^k = 0$
    $\Rightarrow |eig(A_d)| < 1$ (**inside unit circle in complex plane**)

### Applying to Pendulum + Forward Euler:
$$x_{k+1} = x_k + h f(x_k) = f_d(x_k)$$
$$A_d = \frac{\partial f_d}{\partial x_k} = I + hA = I + h \begin{bmatrix} 0 & 1 \\ -\frac{g}{l} \cos\theta & 0 \end{bmatrix}$$
* $eig(A_d\big|_{\theta=0}) \approx 1 \pm 0.313i$ (for $h=0.1$)
* **Plot $|eig(A_d)|$ vs. $h$:** $\Rightarrow$ only (marginally) stable in the limit $h \rightarrow 0$.

### Intuition:
Linear approximation always overshoots $\sin(t)$.
[Image showing forward euler overshooting a sine wave curve]

### Takeaway Message:
* Be careful when discretizing ODEs.
* Sanity check based on energy, momentum, etc. (Conservation and/or dissipation).
* **Don't use Forward Euler integration.**

## Advanced Numerical Integration and Control Discretization

### A Better Explicit Integrator
While Forward Euler is simple, it often lacks the necessary accuracy for complex systems.

* **4th Order Runge-Kutta Method (RK4):** The "industry standard" for explicit integration.
* **Intuition:**
    * **Euler** fits a line segment over each timestep.
    * **RK4** fits a cubic polynomial $\Rightarrow$ much better accuracy!
    * **Pseudo-code:**
    $x_{k+1} = f_{RK4}(x_k)$
    
    $k_1 = f(x_k)$
    $k_2 = f(x_k + \frac{h}{2} k_1)$
    $k_3 = f(x_k + \frac{h}{2} k_2)$
    $k_4 = f(x_k + h k_3)$
    
    $x_{k+1} = x_k + \frac{h}{6}(k_1 + 2k_2 + 2k_3 + k_4)$

> **Takeaway:** The accuracy win far outweighs the additional computational cost. However, even sophisticated integrators have issues—always sanity check your results!

---

### "Implicit" Form
Unlike explicit methods, implicit methods define the future state as a function of itself.

* **Form:** $f_d(x_{k+1}, x_k, u_k) = 0$
* **Simplest Version: Backward Euler**
    $x_{k+1} = x_k + h f(x_{k+1})$ 
    *(Evaluates $f$ at the future time)*
* **How do we simulate?**
    * Write as: $f_d(x_{k+1}, x_k, u_k) = x_k + h f(x_{k+1}) - x_{k+1} = 0$
    * Solve a **root-finding problem** for $x_{k+1}$ (e.g., using Newton's method).
* **Pendulum Simulation:**
    * Opposite energy behavior from Forward Euler.
    * Discretization adds **numerical damping**.
    * While technically "unphysical," this effect allows simulators to take big steps and remain stable, which is often convenient.
    * Very common in "low-fi" simulators in graphics and robotics.

> **Takeaway:** Implicit methods are often "more stable" than Forward Euler. While solving the implicit equation is more expensive for forward simulation, they are not more expensive in many "direct" control methods.

---

### Discretizing Controls
So far, we have discretized the state $x(t)$. We also need to discretize the control input $u(t)$.

#### 1. Simplest Option: Zero-Order Hold (ZOH)
$u(t) = u_k$ for $t_k \leq t < t_{k+1}$
* **Pros:** Easy to implement.
* **Cons:** May require many "knot points" to accurately capture a continuous $u(t)$.

#### 2. Better Option: First-Order Hold (FOH)
$u(t) = u_k + \left( \frac{u_{k+1} - u_k}{h} \right)(t - t_k)$
* Can approximate continuous $u(t)$ with fewer knot points.
* Not much extra work compared to ZOH.
* Super common (e.g., classic **DIRCOL** - Direct Collocation).
