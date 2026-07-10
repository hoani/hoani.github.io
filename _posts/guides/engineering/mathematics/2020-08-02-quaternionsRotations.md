---
title: "Quaternion Rotation Math"
excerpt: "Useful mathematics for quaternion based rotations"
permalink: /guides/engineering/mathematics/quaternion-rotations
classes: wide
categories:
  - guide
  - mathematics
---

As mentioned in [Understanding Quaternion Rotations](/guides/engineering/mathematics/quaternion-basics), to construct a quaternion, we need to know:
* a vector in 3-space to rotate about
* the shortest angle to rotate about that vector

### Calcualting a quaternion from two vectors

Given two vectors $$ \widehat{v} $$ and $$ \widehat{w} $$.

We can construct the vector to rotate about from $$ \widehat{v} $$ to $$ \widehat{w} $$ by taking thier cross product:

$$ \widehat{r} = \widehat{v} \times \widehat{w} $$

To determine the smallest angle to rotate about $$ \widehat{r} $$, take the arccosine of the dot product:

$$ \theta = \cos^{-1} ( \widehat{v} \cdot \widehat{w} ) $$

The quaternion can then be calculated directly with $$ \widehat{r} $$ and $$ \theta $$:

$$ \widehat{q} = \cos\left(\frac{\theta}{2}\right) + \sin\left(\frac{\theta}{2}\right) \frac{\widehat{r}}{\parallel\widehat{r}\parallel} \cdot \begin{bmatrix} \hat{i} \\ \hat{j} \\ \hat{k} \end{bmatrix} $$

### Quaternion Conjugates

If a quaternion $$ \widehat{q}_{vw} $$ represents a rotation from $$ v $$ to $$ w $$, then the conjugate $$ \widehat{q}_{vw}* $$ represents the rotation from $$ w $$ to $$ v $$.

$$ \widehat{q}_{vw} = a + b\hat{i} + c\hat{j} + d\hat{k} $$

$$ \widehat{q}_{wv} = \widehat{q}_{vw}* = a - b\hat{i} - c\hat{j} - d\hat{k} $$

Intuitively, the conjugate flips the vector in 3-space which we rotate about.

### Rotating a vector using a quaternion

Given a vector $$ \widehat{v} $$ and a quaternion $$ \widehat{q} $$ which describes the rotation from $$ \widehat{v} $$ to $$ \widehat{w} $$:

$$ \widehat{w} = \widehat{v} + 2\widehat{q}_{ijk} \times (\widehat{q}_{ijk} \times \widehat{v} + q_a \widehat{v}) $$

### Making a rotation matrix

Given a quaternion:

$$ \widehat{q} = a + b\hat{i} + c\hat{j} + d\hat{k} $$

A rotation matrix from frame A to frame B is given by:

$$ R^{A/B} = 
\begin{bmatrix} 
  1 - 2(q_{c}^{2} + q_{d}^{2})  & 2(q_{b}q_{c} + q_{a}q_{d})    & 2(q_{b}q_{d} - q_{c}q_{a})    \\
  2(q_{b}q_{c} - q_{a}q_{d})    & 1 - 2(q_{b}^{2} + q_{d}^{2})  & 2(q_{c}q_{d} + q_{b}q_{a})    \\ 
  2(q_{b}q_{d} + q_{c}q_{a})    & 2(q_{c}q_{d} - q_{b}q_{a})    & 1 - 2(q_{b}^{2} + q_{c}^{2}) 
\end{bmatrix} $$

### Combining Quaternions with Quaternion Multiplication

Given:
* a quaternion $$ \widehat{q}_{1} $$ which represents the rotation from space $$ A \to B $$
* a quaternion $$ \widehat{q}_{2} $$ which represents the rotation from space $$ B \to C $$

We can multiply the quaternions to get a quaternion $$ \widehat{q}_3 $$ which represents a rotation from space $$ A \to C $$:

$$ \widehat{q}_{3} = \widehat{q}_{1} \widehat{q}_{2} $$

The real part of $$ \widehat{q}_{3} $$ is given by:

$$ \widehat{q}_{3a} = q_{1a} q_{2a} - \widehat{q}_{1ijk} \cdot \widehat{q}_{2ijk} $$

The vector of $$ \widehat{q}_{3} $$ is given by:

$$ \widehat{q}_{3ijk} = q_{2a} \widehat{q}_{1ijk} + q_{1a} \widehat{q}_{2ijk} + \widehat{q}_{1ijk} \times \widehat{q}_{2ijk} $$
