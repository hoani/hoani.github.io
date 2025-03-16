---
title: "Understanding Quaternion Rotations"
excerpt: "How quaternions work"
permalink: /guides/engineering/mathematics/quaternion-basics
classes: wide
categories:
  - guide
  - mathematics
---

An object in 3-space can go from any initial orientation to any final orientation by: 
* defining exactly one vector in 3-space
* rotating that object about that vector by a specific angle

### The anatomy of a quaternion

A quaternion is defined:

$$ \widehat{q} = a + b\hat{i} + c\hat{j} + d\hat{k} $$

Where $$ \hat{i} $$, $$ \hat{j} $$ and $$ \hat{k} $$ are all unit vectors along the three spatial axes. 

### Significance of the 3-Space Component

To understand quaternion rotation intuitively, let's ignore the "real" part and only pay attention to the 3-space component:

$$ \widehat{q}_{ijk} = b\hat{i} + c\hat{j} + d\hat{k} $$

The magnitude $$ \Vert \widehat{q}_{ijk} \Vert $$ defines a fraction from $$ 0.0 $$ to $$ 1.0 $$ which represents a $$ 0.0^{\circ} $$ to $$ 180.0^{\circ} $$ rotation about the vector $$ \widehat{q}_{ijk} $$.

*Note: The relationship between $$ \Vert \widehat{q}_{ijk} \Vert $$ is not linear. $$ \Vert \widehat{q} \Vert = 1.0 $$ therefore, a rotation of $$ 90.0^{\circ} $$ maps to $$ \Vert \widehat{q}_{ijk} \Vert = 0.7071, a = 0.7071 $$.* 

### What about a?

The mathematics of quaternions are a little bit more involved than what I described above. 

Thier inventer, William Hamilton describes them as quotents of two vectors, which requires $$ \Vert \widehat{q} \Vert = 1.0 $$. Therefore, $$ a $$ effectively describes how much a vector should not rotate in order to point in the direction of another vector. 

A value of $$ a = 1.0 $$ indicates the two vectors making up the quaternion pointed in the same direction.

### Calculating a quaternion from a vector and rotation

Given a unit vector:

$$ \widehat{n} = \begin{bmatrix} x \\ y \\ z \end{bmatrix} $$

And an angle which we will rotate around the unit vector by, $$ \theta $$.

$$ \widehat{q} = \cos\left(\frac{\theta}{2}\right) + \sin\left(\frac{\theta}{2}\right) (x\hat{i} + y\hat{j} + z\hat{k}) $$
