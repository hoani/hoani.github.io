---
title: "Eigen Values, Eigen Vectors"
excerpt: "Some notes on eigen values/vectors"
toc: true
categories:
  - guide
  - mathematics
  - octave-guide
---

## Overview

If we have a matrix $$ A \in \mathbb{R}^n $$, then its eigen vectors are defined:

$$ Av = \lambda v $$

Where:
* $$ \lambda $$ is a scalar
* $$ v $$ is a vector

That is, $$ v $$ are vectors that are only scaled by $$ A $$, but do not change direction.

$$ \lambda $$ are the eigen values of $$ A $$.

## Calculating eigen vectors and eigen values

To find the eigen values:

$$ det(\lambda I_n - A) = 0 $$

To find the eigen vectors (once we know the eigen values):

$$ (\lambda I_n - A) v = \boldsymbol{0} $$

### 2x2 matrix example

$$ A = \left[ \begin{array}{ccc} 1 & 1 \\ 4 & -2 \end{array} \right] $$

**Finding eigen values**

Take the determinent of $$ (\lambda I_n - A) $$ and set to zero:

$$ det\left(\left[ \begin{array}{ccc} \lambda - 1 & -1 \\ -4 & \lambda + 2 \end{array} \right]\right) = 0 $$

$$ (\lambda - 1)(\lambda + 2) - 4 = 0 $$

Has solutions $$ \lambda = 2, -3 $$

**Finding eigen vectors**

For $$ \lambda = 2 $$:

$$ \left[ \begin{array}{ccc} 1 & -1 \\ -4 & 4 \end{array} \right] v = \boldsymbol{0} $$

Let:

$$ v = \left[ \begin{array}{ccc} v_1 \\ v_2 \end{array} \right] $$

Then:

$$ v_1 - v_2 = 0 $$

$$ \therefore v = \left[ \begin{array}{ccc} 1 \\ 1 \end{array} \right] $$

For $$ \lambda = -3 $$:

$$ \left[ \begin{array}{ccc} -4 & -1 \\ -4 & -1 \end{array} \right] v = \boldsymbol{0} $$

Then:

$$ 4 v_1 + v_2 = 0 $$

$$ \therefore v = \left[ \begin{array}{ccc} 1 \\ -4 \end{array} \right] $$

## Python

```python
import numpy as np

A = np.array([[1, 1],[4, -2]])
w, v = np.linalg.eig(A)

print(f"Eigen values = {w[0], w[1]}") 
print(f"Eigen vectors = {v[:,0]/v[0,0], v[:,1]/v[0,1]}") 
```

With output:

```
Eigen values = (2.0, -3.0)
Eigen vectors = (array([1., 1.]), array([ 1., -4.]))
```

## Octave/Matlab

```r
A = [1, 1; 4, -2];
[v, w] = eigs(A);
disp("Eigen values")
disp(w)
disp("Eigen vectors")
disp(v(:, 1)/v(1, 1))
disp(v(:, 2)/v(1, 2))

```

With output:

```
Eigen values
Diagonal Matrix
  -3   0
   0   2

Eigen vectors
   1
  -4
   1
   1

```
 