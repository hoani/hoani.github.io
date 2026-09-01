---
title: "Geometry And Collisions"
excerpt: "Collision and intersection algorithms"
permalink: "/guides/mathematics/geometry-and-collisions"
toc: true
categories:
  - guide
  - mathematics
---

For both [Throw a Watermelon](/throw-a-watermelon) and [Trick Shot II](trick-shot-2), I wrote my own collision methods to determine closest points to shapes, collisions, overlaps etc. For trick-shot which can run on a microcontroller, it was especially important that the alogorithms were efficient and ran with deterministic complexity.

I'm documenting the selected algorithms here.

# Lines

## Line Intersection

I found documentation from Bryce Boe here [bryceboe.com - line segment intersection algorithm](https://bryceboe.com/2006/10/23/line-segment-intersection-algorithm/).

Given we have two lines AB and CD, we look at the direction of all four sets of three points. This works because the directions change as the lines intersect.

<figure>
    <img src="/assets/images/posts/guides/geometry/line-intersection.png">
</figure>

We take three points, say A, B from the first line and C from the second. We calculate whether the points are arranged in a clockwise or counter clockwise direction. We then switch point C with D and then determine if points A, B, D are clockwise or counter clockwise. If both checks have the same direction, then the line segments do not cross. 

We then do the same check for points C, D, A and C, D, B - this prevents a false positive where one line points _into_ the other, but doesn't intersect with it.

To determine orientation:

$$
ccw_{abc} = (C_y-A_y)(B_x-A_x) \gt (B_y-A_y)(C_x-A_x)
$$

Then to determine if lines intersect:

$$
intersect_{AB-CD} = \left(ccw_{abc} \neq ccw_{abd}\right) \&\& \left(ccw_{cda} \neq ccw_{cdb}\right)
$$

## Closest Point On Line

If we have line AB and a point P, we want to find Q - a point on AB that is closest to P.

<figure>
    <img src="/assets/images/posts/guides/geometry/line-closest-point.png">
</figure>

This is achieved by finding $$ P_{\infty}$$ the projected point which would lie on AB, if AB were an infinite line.
If $$P_{\infty}$$ is inside AB, then $$Q=P_{\infty}$$ otherwise we clamp it to the closest end.

$$
\begin{align}
u_{AB} &= \frac{B - A}{||B - A||}  \\
d &= P - A \cdot u_{AB} \\
p_{\infty} &= A + u_{AB} d \\
Q &= \operatorname{clamp}(p_{\infty}, A, B)
\end{align}
$$

Where:
* $$u_{AB}$$ is the unit vector of the line $$AB$$
* $$d$$ is the distance that the projected $$P$$ lies across the line $$AB$$
