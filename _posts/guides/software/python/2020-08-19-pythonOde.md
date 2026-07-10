---
title: "Python ODE Solver"
excerpt: "Simple ODE Solver Examples"
permalink: /guides/software/python/ode
categories:
  - guide
  - python
---

## First Order RC Model

![](/assets/images/posts/guides/pyOde/000_RC.png){: .rounded-corners .two-thirds}

A resistor and capacitor in parallel model:

```python
import scipy.integrate as integrate
import matplotlib.pyplot as plt
import numpy as np

t = np.linspace(0.0, 10e-3, 1000)

# Parallel RC Model
R = 100
C = 10e-6
v_init = [10]
v = integrate.odeint(lambda v, t: -v/(R*C), v_init, t)

fig, ax = plt.subplots()
ax.plot(t, v)
```

## Second Order RCL Model

![](/assets/images/posts/guides/pyOde/001_RCL.png){: .rounded-corners .two-thirds}

A resistor, capacitor and inductor in parallel model:

```python
import scipy.integrate as integrate
import matplotlib.pyplot as plt
import numpy as np

t = np.linspace(0.0, 10e-3, 1000)

# Parallel RCL Model
R = 100
C = 10e-6
L = 12e-3
v_init = [10.0, 0.0]
v = integrate.odeint(
    lambda v, t: [v[1], -v[1]/(R * C) - v[0]/(C * L)],
    v_init, t)

fig, ax = plt.subplots()
ax.plot(t, v[:, 0])
plt.show()
```