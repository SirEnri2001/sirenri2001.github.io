---
layout: post
title: Optimal Control 1
date: 2026/4/29 23:28
tags: [Optimal Control]
author: Henry Han
---

# CMU 16-745: Optimal Control

## Continuous-Time Dynamics
* **Most general/generic for smooth systems:**
    $$\dot{x} = f(x, u)$$
    * $x \in \mathbb{R}^n$ is the **state**
    * $u \in \mathbb{R}^m$ is the **input**
    * $f$ represents the **dynamics**

* **For a mechanical system:**
    $x = \begin{bmatrix} q \\ v \end{bmatrix}$
    * $q$: configuration (not always a vector)
    * $v$: velocity

### Example: Pendulum
* **Equation of Motion:** $ml^2 \ddot{\theta} + mgl \sin\theta = \tau$ (where $\tau = u$)
* **State variables:** $q = \theta$, $v = \dot{\theta}$
* **State vector:** $x = \begin{bmatrix} \theta \\ \dot{\theta} \end{bmatrix} \Rightarrow \dot{x} = \begin{bmatrix} \dot{\theta} \\ \dots \end{bmatrix} = \underbrace{\begin{bmatrix} \dot{\theta} \\ -\frac{g}{l} \sin\theta + \frac{1}{ml^2} u \end{bmatrix}}_{f(x, u)}$
* **Manifold:** $q \in S^1$ (circle), $x \in S^1 \times \mathbb{R}$ (cylinder)

---

## Control-Affine Systems
$$\dot{x} = f_0(x) + B(x)u$$
* $f_0(x)$: "**drift**"
* $B(x)$: "**input Jacobian**"
* Most systems can be put in this form.
* **Pendulum Example:**
    $f_0(x) = \begin{bmatrix} \dot{\theta} \\ -\frac{g}{l} \sin\theta \end{bmatrix}, \quad B(x) = \begin{bmatrix} 0 \\ \frac{1}{ml^2} \end{bmatrix}$

---

## Manipulator Dynamics
$$M(q)\dot{v} + C(q, v) = B(q)u + F$$
* $M(q)$: "**Mass matrix**" or "**Inertia tensor**"
* $C(q, v)$: "**Dynamic Bias**" (Coriolis + gravity)
* $B(q)$: "**Input Jacobian**"
* $F$: "**External forces**"

**Velocity Kinematics:** $\dot{q} = G(q)v$

**State-Space Form:**
$$\dot{x} = f(x, u) = \begin{bmatrix} G(q)v \\ M(q)^{-1}(B(q)u - C) \end{bmatrix}$$

* **Pendulum Example:** $M(q) = ml^2, \quad C(q, v) = mgl \sin\theta, \quad B=1, \quad G=I$
* All mechanical systems can be written like this.
* Just rewriting **Euler-Lagrange equations**: $L = \frac{1}{2} v^T M(q) v - U(q)$

---

## Linear Systems
$$\dot{x} = A(t)x + B(t)u$$
* Called "**time-invariant**" if $A(t) = A$ and $B(t) = B$.
* Called "**time-variant**" otherwise.
* We often approximate non-linear systems with linear ones:
    $$\dot{x} = f(x, u) \Rightarrow A = \frac{\partial f}{\partial x}, \quad B = \frac{\partial f}{\partial u}$$

---

## Equilibria
* A point where the system will "remain at rest": $\dot{x} = f(x, u) = 0$.
* Algebraically, these are the **roots of the dynamics**.
* **Pendulum Example:**
    $\dot{x} = \begin{bmatrix} \dot{\theta} \\ -\frac{g}{l} \sin\theta \end{bmatrix} = \begin{bmatrix} 0 \\ 0 \end{bmatrix} \Rightarrow \dot{\theta} = 0, \quad \theta = 0, \pi$

### First Control Problem:
* **Can I move the equilibria?**
    Example: Hold pendulum at $\theta = \pi/2$
    $\dot{x} = \begin{bmatrix} \dot{\theta} \\ -\frac{g}{l} \sin(\pi/2) + \frac{1}{ml^2} u \end{bmatrix} = \begin{bmatrix} 0 \\ 0 \end{bmatrix}$
    $\frac{1}{ml^2} u = \frac{g}{l} \sin(\pi/2) \Rightarrow u = mgl$

## Stability of Equilibrium

* **When will we stay "near" an equilibrium point under perturbations?**

* **Look at 1D system ($x \in \mathbb{R}$):**
        * $\frac{\partial f}{\partial x} < 0 \Rightarrow$ **stable**
    * $\frac{\partial f}{\partial x} > 0 \Rightarrow$ **unstable**

* **In higher dimensions:**
    * $\frac{\partial f}{\partial x}$ is a **Jacobian matrix**.
    * Take an eigen decomposition $\Rightarrow$ decouple into $n \times 1D$ systems.
    * $Re[eig(\frac{\partial f}{\partial x})] < 0 \Rightarrow$ **stable**
    * Otherwise $\rightarrow$ **unstable**

---

### Example: Pendulum
$$f(x) = \begin{bmatrix} \dot{\theta} \\ -\frac{g}{l} \sin\theta \end{bmatrix} \Rightarrow \frac{\partial f}{\partial x} = \begin{bmatrix} 0 & 1 \\ -\frac{g}{l} \cos\theta & 0 \end{bmatrix}$$

* **At $\theta = \pi$ (Upward):**
    $\frac{\partial f}{\partial x}\big|_{\theta=\pi} = \begin{bmatrix} 0 & 1 \\ g/l & 0 \end{bmatrix} \Rightarrow eig(\frac{\partial f}{\partial x}) = \pm \sqrt{\frac{g}{l}} \Rightarrow$ **unstable**

* **At $\theta = 0$ (Downward):**
    $\frac{\partial f}{\partial x}\big|_{\theta=0} = \begin{bmatrix} 0 & 1 \\ -g/l & 0 \end{bmatrix} \Rightarrow eig(\frac{\partial f}{\partial x}) = \pm i \sqrt{\frac{g}{l}} \Rightarrow$ **marginally stable** (undamped oscillations)

* **Marginally Stable:** The pure imaginary case results in undamped oscillations.
* **Adding Damping:** (e.g., $u = -k_d \dot{\theta}$) results in a strictly negative real part.
