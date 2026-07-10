---
title: "Python Lambda"
excerpt: "Simple Lambda examples"
permalink: /guides/software/python/lambda
categories:
  - guide
  - python
---

## Syntax

```py
sum = lambda x, y: x + y
sum(1, 4)                    # 5
(lambda x, y: x + y)(3, 7)   # 10
```

Lambdas contain:
* A single expression
* Statements are not allowed


## Simple Use Case - a First Order RC Model

![](/assets/images/posts/guides/pyOde/000_RC.png){: .rounded-corners .two-thirds}

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



