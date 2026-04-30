---
layout: post
title: Optimal Control 3
date: 2026/4/29 23:28
tags: [Optimal Control]
author: Henry Han
---

### Linear Dynamical Systems (LDS)
* **Continuous-time ODE:** $\dot{x} = f(x, u)$
* **Discrete-time difference eq:** $x_{k+1} = f(x_k, u_k)$

#### Stability of $\dot{x} = Ax$
A linear system $\dot{x} = Ax$ is stable if $Re[eig(A)] \leq 0$.

#### Integration and the Matrix Exponential
For a simple linear system $\dot{x} = ax$:
1.  $\frac{dx}{dt} = ax$
2.  $\int \frac{1}{x} dx = \int a dt$
3.  $\ln x = at + C$
4.  $x = e^{at+C} = e^{at} \cdot e^C$
5.  $x = e^{at} \cdot C \quad \text{or} \quad x(t) = e^{at} x_0$


---

### Matrix Exponential and Discrete Dynamics
For a linear continuous-time system $\dot{x} = Ax$:

* **Solution:** $x(t) = (e^{At})x_0$, where $e^{At}$ is the **Matrix Exponential**.
* **Discrete Step:** $x_{k+1} = (e^{A\Delta t})x_k = A_d x_k$
    * $A_d$ is the **State Transition Matrix**.
* **Stability Condition:** The system is stable if $|eig(A_d)| < 1$ (all eigenvalues lie inside the unit circle in the complex plane).


#### Linear Systems with Input (ZOH)
Given $\dot{x} = Ax + Bu$ and assuming a **Zero-Order Hold (ZOH)** where $\dot{u} = 0$:
We can augment the state:
$$\begin{bmatrix} \dot{x} \\ \dot{u} \end{bmatrix} = \begin{bmatrix} A & B \\ 0 & 0 \end{bmatrix} \begin{bmatrix} x \\ u \end{bmatrix}$$
The discrete-time matrices are found via:
$$\begin{bmatrix} A_d & B_d \\ 0 & I \end{bmatrix} = e^{\begin{bmatrix} A & B \\ 0 & 0 \end{bmatrix}\Delta t}$$
* **Linear Discrete Dynamics:** $x_{k+1} = A_d x_k + B_d u_k$
* **Matrix Exponential Series:** $e^X = I + X + \frac{X^2}{2!} + \frac{X^3}{3!} + \dots = \sum_{k=0}^{\infty} \frac{X^k}{k!}$

---

### Derivative Formalisms
When dealing with multivariate functions $f: \mathbb{R}^n \rightarrow \mathbb{R}^m$:

* **Jacobian Matrix:** $\frac{\partial f}{\partial x} \in \mathbb{R}^{m \times n}$
    $$\frac{\partial f}{\partial x} = \begin{bmatrix} \frac{\partial y_1}{\partial x_1} & \dots & \frac{\partial y_1}{\partial x_n} \\ \vdots & \ddots & \vdots \\ \frac{\partial y_m}{\partial x_1} & \dots & \frac{\partial y_m}{\partial x_n} \end{bmatrix}$$
* **Gradient:** For a scalar function $y = f(x)$ where $f: \mathbb{R}^n \rightarrow \mathbb{R}$:
    * $\frac{\partial f}{\partial x}$ is a **row vector** ($1 \times n$).
    * $\nabla f(x) = (\frac{\partial f}{\partial x})^T$ is a **column vector** ($n \times 1$).
* **Hessian Matrix:** $D_x^2 f = \frac{\partial^2 f}{\partial x^2} \in \mathbb{R}^{n \times n}$ (Symmetric).

---

### Taylor Series for Dynamics
To linearize nonlinear dynamics $\dot{x} = f(x, u)$ about a nominal point $(\bar{x}, \bar{u})$:

1.  **Define Jacobians:** $A = \frac{\partial f}{\partial x} \big|_{\bar{x}, \bar{u}}, \quad B = \frac{\partial f}{\partial u} \big|_{\bar{x}, \bar{u}}$
2.  **Approximation:** $f(x, u) \approx f(\bar{x}, \bar{u}) + A(x - \bar{x}) + B(u - \bar{u})$
3.  **Delta Coordinates:** Let $x = \bar{x} + \Delta x$ and $u = \bar{u} + \Delta u$
    $$f(x, u) \approx f(\bar{x}, \bar{u}) + A\Delta x + B\Delta u$$


---

### Root Finding and Fixed Points
**Goal:** Given $f(x)$, find $x^*$ such that $f(x^*) = 0$.

* **Continuous-time:** Root finding finds the **equilibrium**.
* **Discrete-time:** Closely related to finding a **fixed point** where $f(x^*) = x^*$.

#### 1. Fixed Point Iteration
* **Method:** $x \leftarrow f(x)$
* **Constraint:** Only works for **stable** fixed points and typically has slow convergence.

#### 2. Newton's Method
* **Intuition:** Fit a linear approximation to $f(x)$ and solve for the root of the approximation.
* **Linearization:** $f(x + \Delta x) \approx f(x) + \frac{\partial f}{\partial x} \Delta x$
* **Step:** Set approximation to zero and solve for $\Delta x$:
    $$f(x) + \frac{\partial f}{\partial x} \Delta x = 0 \Rightarrow \Delta x = -\left(\frac{\partial f}{\partial x}\right)^{-1} f(x)$$

---

## Optimization and Minimization

### Root Finding with Newton’s Method (Continued)
* **Algorithm:**
    1.  Apply correction: $x \leftarrow x + \Delta x$
    2.  Repeat until convergence.
* **Example: Backward Euler**
    * Newton’s method provides very fast convergence for implicit integration.
* **Take-Away Messages:**
    * **Quadratic Convergence:** Errors decrease quadratically near the solution.
    * Can reach machine precision very quickly.
    * **Cost:** The most expensive part is solving the linear system $O(n^3)$. This can be improved by taking advantage of problem structure (sparsity).

---

### Minimization
**Goal:** $\min_x f(x)$, where $f: \mathbb{R}^n \rightarrow \mathbb{R}$

* **First-Order Condition:** If $f$ is smooth, $\frac{\partial f}{\partial x} \big|_{x^*} = 0$ at a local minimum.
* This transforms minimization into a root-finding problem for $Df(x) = 0$.
* **Newton's Step for Minimization:**
    Linearize the gradient: $\nabla f(x + \Delta x) \approx \nabla f(x) + \underbrace{\frac{\partial}{\partial x}(\nabla f(x))}_{\nabla^2 f(x)} \Delta x = 0$
    $$\Delta x = -(\nabla^2 f(x))^{-1} \nabla f(x)$$
* **Intuition:** Fit a quadratic approximation to $f(x)$ and exactly minimize that approximation.
* **Sufficiency:** $\nabla f(x) = 0$ is a necessary condition, but not sufficient.
    * In the scalar case: $\Delta x = -(\nabla^2 f)^{-1} \nabla f \Rightarrow$ "descent direction" $\times$ "learning rate/step size."

---

### Regularization and Robustness
* **Definiteness:**
    * $\nabla^2 f > 0 \Rightarrow$ descent (minimizing).
    * $\nabla^2 f < 0 \Rightarrow$ ascent (maximizing).
    * In $\mathbb{R}^n$, we require $\nabla^2 f \in S^n_{++}$ (strictly positive definite matrix).
* **Strong Convexity:** If $\nabla^2 f > 0$ everywhere, the problem is strongly convex and easily solved with Newton.
* **Regularization:** A practical solution when $\nabla^2 f$ is not positive definite:
    * While $H \neq \nabla^2 f$ not pos. def:
    * $H \leftarrow H + \beta I \quad (\beta > 0 \text{ is a scalar hyperparameter})$
    * This is called **"Damped Newton."** It guarantees a descent direction and shrinks the step size.

---

### Newton Optimization Variants
* **Least Squares:** $\min_x \|Ax - b\|^2_2 \approx (Ax - b)^T(Ax - b)$
    * Solution via pseudo-inverse: $x = (A^T A)^{-1} A^T b$.
* **Gauss-Newton Step:** Often used in robotics/least squares where the full Hessian is approximated by $J^T J$ to save computation.

---

### Line Search (Globalization)
Often, a full Newton step $\Delta x$ overshoots the minimum. To fix this, we use a **Line Search** strategy:

* **Strategy:** Check $f(x + \alpha \Delta x)$ and "backtrack" by reducing $\alpha$ until a "good" reduction is found.
* **Armijo Rule:**
    * $\alpha = 1$ (initial step length).
    * While $f(x + \alpha \Delta x) > f(x) + \beta \alpha \nabla f(x)^T \Delta x$:
    * $\alpha \leftarrow c\alpha \quad (c \approx 0.5, \beta \in [10^{-4}, 0.1])$.
* **Intuition:** Ensure the step agrees with the linearization within some tolerance $\beta$.

> **Takeaway:** Newton’s method with simple modifications (regularization and line search) is extremely effective at finding local optima.

---

### Equality Constraints
**Goal:** $\min_x f(x)$ s.t. $c(x) = 0$
* $f: \mathbb{R}^n \rightarrow \mathbb{R}$
* $c: \mathbb{R}^n \rightarrow \mathbb{R}^m$

**First-Order Necessary Conditions (KKT):**
1.  Need $\nabla f(x) = 0$ in "free" directions (directions tangent to the constraint).
2.  Need $c(x) = 0$.


* **Geometric Intuition:** At the optimum, any non-zero component of the gradient $\nabla f$ must be normal to the constraint surface $c(x) = 0$. This means $\nabla f$ must be a linear combination of the constraint gradients $\nabla c_i$.
