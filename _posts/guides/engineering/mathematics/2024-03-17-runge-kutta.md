---
title: "Runge-Kutta Integrators"
excerpt: "Formulas for Runge-Kutta Integrators"
permalink: "/guides/mathematics/runge-kutta"
toc: true
categories:
  - guide
  - mathematics
---

Given a differential equation:
$$
\dot{x} = f\left(x, t\right)
$$
We can solve this with varying accuracy.

Notes:
* The derivation of Runge-Kutta uses Taylor series
* Software like Matlab uses an adaptive Runge-Kutta, known as RK2,3 or RK4,5. This will adapt the timestep within its parameters to reduce errors in high dynamics, or reduce computation time in low dynamics.

## Forward Euler

$$x_{k+1} = x_k + \Delta t f\left(x_k, t_k\right) $$

* Local error is $$O(\Delta t^2)$$ 
* Global error is $$O(\Delta t)$$ - i.e. It increases linearly.

## Runge-Kutta 2nd Order (RK2)

Conceptually this finds the gradient halfway along the next euler step - and uses this to take a full step.
$$
\begin{align}
x_{k+1} &= x_k + \Delta t f_2 \\
\\
f_1 &= f\left(x_k, t_k\right) \\
f_2 &= f\left(x_k + \frac{\Delta t}{2} f_1, t_k + \frac{\Delta t}{2}\right)
\end{align}
$$
* Local error is $$O(\Delta t^3)$$
* Global error is $$O(\Delta t^2)$$

## Runge-Kutta 4th Order (RK4)

This expands the Taylor series further, taking additional gradients into account.

$$
\begin{align}
x_{k+1} &= x_k + \frac{\Delta t}{6}\left(f_1 + 2 f_2 + 2 f_3 + f_4\right) \\
\\
f_1 &= f\left(x_k, t_k\right) \\
f_2 &= f\left(x_k + \frac{\Delta t}{2}f_1, t_k + \frac{\Delta t}{2}\right) \\
f_3 &= f\left(x_k + \frac{\Delta t}{2}f_2, t_k + \frac{\Delta t}{2}\right) \\
f_4 &= f\left(x_k + \Delta t f_3, t_k + \Delta t\right)
\end{align}
$$

* Local error is $$O(\Delta t^5)$$ 
* Global error is $$O(\Delta t^4)$$ 