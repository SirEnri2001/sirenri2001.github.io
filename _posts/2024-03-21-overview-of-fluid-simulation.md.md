---
layout: post
title: Fluid Simulation - Concept and Practice
subtitle: Taichi implementation for Fluid Simulation
cover-img: /assets/img/fluid-simulation.jpg
share-img: /assets/img/fluid-simulation.jpg
share-img: /assets/img/fluid-simulation.jpg
tags: [papers, fluid simulation]
author: Henry Han
---

# Taichi implementation for Fluid Simulation

## Algorithm Overview

1. Initialize Grid with some Fluid
2. for ( i from 1 to n )
3. Let $t = 0.0$
- While $t<t_{frame}$
- - Calculate $\Delta t$
- - Advect Fluid
- - Pressure Projection (Pressure Solve)
- - Advect Free Surface
- - $t = t + \Delta t$
- Write frame i

---
## 1. Linear Interpolation and Bilinear Interpolation
---

```python
#####################
#   Bilinear Interpolation function
#####################
@ti.func
def sample(vf, u, v, shape):
    i, j = int(u), int(v)
    # Nearest
    i = ti.max(0, ti.min(shape[0] - 1, i))
    j = ti.max(0, ti.min(shape[1] - 1, j))
    return vf[i, j]


@ti.func
def lerp(vl, vr, frac):
    # frac: [0.0, 1.0]
    return (1 - frac) * vl + frac * vr


@ti.func
def bilerp(vf, u, v, shape):
    # use -0.5 to decide where bilerp performs in cells
    s, t = u - 0.5, v - 0.5
    iu, iv = int(s), int(t)
    a = sample(vf, iu + 0.5, iv + 0.5, shape)
    b = sample(vf, iu + 1.5, iv + 0.5, shape)
    c = sample(vf, iu + 0.5, iv + 1.5, shape)
    d = sample(vf, iu + 1.5, iv + 1.5, shape)
    # fract
    fu, fv = s - iu, t - iv
    return lerp(lerp(a, b, fu), lerp(c, d, fu), fv)
```

---
## 2. Advection
---

Advection can be informally described as follows: "Given some quantity $Q$ on our simulation grid, how will $Q$ change $\Delta t$ later?"

Therefore, we have:

$$Q^{n+1}=\text{advect}(Q^n,\Delta t, \frac{\partial Q^n}{\partial t})$$

### Forward Euler

In the code below, *Forward Euler* time integrator is used, which consists of three steps:

1. Calculate $-\frac{\partial Q}{\partial t}$
2. Sample position $\vec{X} = Q(i, j)$ 
3. Calculate $\vec{X}_{prev} = \vec{X} - \frac{\partial Q}{\partial t}*\Delta t$
4. Set the gridpoint for $Q^{n+1}(i,j):=Q(i,j)$ that is nearest to $\vec{X}_{prev}$

We can consider using more accurate time integrator, such as RK-2 or implicit euler.

```python
@ti.kernel
def advection(vf: ti.template(), qf: ti.template(), new_qf: ti.template()):
    for i, j in vf:
        coord_cur = ti.Vector([i, j]) + ti.Vector([0.5, 0.5])
        vel_cur = vf[i, j]
        coord_prev = coord_cur - vel_cur * eulerSimParam["dt"]
        q_prev = bilerp(qf, coord_prev[0], coord_prev[1], (eulerSimParam["shape"]))
        new_qf[i, j] = q_prev
```

---
## 3. **Lagrangian** v.s. **Eulerian**
---

[<a href="https://www.youtube.com/watch?v=0Vp0wU7czBM">reference</a>]

By default, **Navier-Stokes** equation is defined in **Lagrangian** viewpoint, which is based on particle movements:

$$\frac{d\vec{u}}{dt}=\vec{g}-\frac{1}{\rho}\nabla p+\nu\nabla\cdot\nabla\vec{u}$$ 

For each particle $p=p(x,y,z,t)$, it has velocity $u=u(x,y,z,t)$ and acceleration 

$a_{\text{particle}}=\frac{d\vec{u}}{dt}=a_{\text{particle}}(x,y,z,t)$

For **Eulerian** viewpoint, the volume domain is fixed, which means point of reference is stationary:

$$\vec{a}(x,y,z,t)=\vec{a}_{\text{particle}}(x,y,z,t)$$

This equation says the acceleration field at this location and time(**Eulerian** viewpoint) equals to the acceleration of the fluid particle occupying this location at this time(**Lagrangian** viewpoint). 

Therefore, the acceleration at a field variable (**Eulerian** description) can be calculated as:

$$\vec{a}(x,y,z,t)=\frac{d\vec{u}}{dt}=\frac{\partial\vec{u}}{\partial t}+\frac{\partial\vec{u}}{\partial x}\frac{\partial x}{\partial t}+\frac{\partial\vec{u}}{\partial y}\frac{\partial y}{\partial t}+\frac{\partial\vec{u}}{\partial z}\frac{\partial z}{\partial t}$$

which can be simplified as

$$\vec{a}=\frac{D\vec{u}}{Dt}=\frac{\partial\vec{u}}{\partial t}+(\vec{u}\cdot\vec{\nabla})\vec{u}$$

### Material Derivative

Material derivative $\frac{DQ}{Dt}$ is a general form of the acceleration of Eulerian description:

$$\frac{DQ}{Dt}=\frac{\partial Q}{\partial t}+\vec{u}\cdot\nabla Q$$

For this equation, we have $Q = Q(x,y,z,t)$, the quantity at a blob of fluid moving with $\vec{u}$, and $\nabla Q= \left[\frac{\partial Q}{\partial x},\frac{\partial Q}{\partial y},\frac{\partial Q}{\partial z}\right]$. $\frac{\partial Q}{\partial t}$ is the local change due to unsteadiness(related with changes in time). $\vec{u}\cdot\nabla Q$ is the change due to movement to a different part of the flow(related with changes in position). This means we can have acceleration in a steady flow.

Therefore, for Navier Stokes equation 

$$\frac{D\vec{u}}{Dt}=\vec{g}-\frac{1}{\rho}\nabla p+\nu\nabla\cdot\nabla\vec{u}$$ 

yields the standard form of the momentum equation:

$$\frac{\partial\vec{u}}{\partial t}=-\vec{u}\cdot\nabla\vec{u}+\vec{g}-\frac{1}{\rho}\nabla p+\nu\nabla\cdot\nabla\vec{u}$$

```python
def advection_step():
    advection(velocities_pair.cur, color_pair.cur, color_pair.nxt)
    advection(velocities_pair.cur, velocities_pair.cur, velocities_pair.nxt)
    color_pair.swap()
    velocities_pair.swap()
    apply_vel_bc(velocities_pair.cur)
```
