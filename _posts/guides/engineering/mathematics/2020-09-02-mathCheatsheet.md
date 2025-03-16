---
title: "Math cheatsheet"
excerpt: "Useful formula"
toc: true
permalink: /guides/engineering/mathematics/cheatsheet
categories:
  - guide
  - mathematics
---

## Calculus

| Product Rule | $$ \left( uv \right)' = u'v + uv' $$ |
| Quotent Rule | $$ \left( \frac{u}{v} \right)' = \frac{vu' - uv'}{v^2} $$ |
| Chain Rule | $$ \frac{d}{dt}\left[ u\left( v(t) \right)\right] = u'(v(t))v'(t) $$ |
| Integration by Parts | $$ \int u dv = uv - \int vdu $$ |

Useful derivatives

| $$ \frac{d}{dt}t^n $$ | $$ = $$ | $$ nt^{n-1} $$ |
| $$ \frac{d}{dt}e^t $$ | $$ = $$ | $$ e^{t} $$ |
| $$ \frac{d}{dt}ln(t) $$ | $$ = $$ | $$ \frac{1}{t} $$, $$ t > 0 $$ |
| $$ \frac{d}{dt}ln(u(t)) $$ | $$ = $$ | $$ \frac{u'(t)}{u(t)} $$ |
| $$ \frac{d}{dt}sin(t) $$ | $$ = $$ | $$ cos(t) $$ |
| $$ \frac{d}{dt}cos(t) $$ | $$ = $$ | $$ -sin(t) $$ |

## Trig Identities

| $$ \tan \theta $$ | $$ = $$ | $$ \frac{\cos\theta}{\sin\theta} $$ |
| $$ \sin^2 \theta + \cos^2 \theta $$ | $$ = $$ | $$ 1 $$ |
| $$ \sin \frac{\theta}{2} $$ | $$ = $$ | $$ \pm \sqrt{\frac{1 - \cos\theta}{2}} $$ |
| $$ \cos \frac{\theta}{2} $$ | $$ = $$ | $$ \pm \sqrt{\frac{1 + \cos\theta}{2}} $$ |
| $$ \tan \frac{\theta}{2} $$ | $$ = $$ | $$ \frac{1 - \cos\theta}{\sin\theta} $$ |

## 3-space vectors

Multiplying unit vectors:
$$
\begin{align}
\hat{i}\hat{j} &= -\hat{j}\hat{i} = \hat{k} \\
\hat{j}\hat{k} &= -\hat{k}\hat{j} = \hat{i} \\
\hat{k}\hat{i} &= -\hat{i}\hat{k} = \hat{j}
\end{align}
$$

Given two vectors:
$$
\begin{align}
\hat{a} &= a_0\hat{i} + a_1\hat{j} + a_2\hat{k} \\
\hat{b} &= b_0\hat{i} + b_1\hat{j} + b_2\hat{k}
\end{align}
$$

The cross product is given by:
$$
\begin{align}
\hat{a} \times \hat{b} &= \begin{vmatrix} 
	\hat{i} & \hat{j} & \hat{k} \\
	a_0 & a_1 & a_2 \\
	b_0 & b_1 & b_2
\end{vmatrix} \\
&= (a_1b_2 - a_2b_1)\hat{i} + (a_2b_0 - a_0b_2)\hat{j} + (a_0b_1 - a_1b_0)\hat{k}
\end{align}
$$

## Quadratic formula

$$ 
\begin{align}
x &= \frac{-b \pm \sqrt{b^2 - 4ac}}{2a} \\
\end{align} 
$$


