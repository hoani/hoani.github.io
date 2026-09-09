---
title: "Geometry And Collisions"
excerpt: "Collision and intersection algorithms"
permalink: "/guides/mathematics/geometry-and-collisions"
toc: true
categories:
  - guide
  - mathematics
---

For both [Throw a Watermelon](/throw-a-watermelon) and [Trick Shot II](/trick-shot-2), I wrote my own collision methods to determine closest points to shapes, collisions, overlaps etc. For trick-shot which can run on a microcontroller, it was especially important that the alogorithms were efficient and ran with deterministic complexity.

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
d &= (P - A) \cdot u_{AB} \\
p_{\infty} &= A + u_{AB} d \\
Q &= \operatorname{clamp}(p_{\infty}, A, B)
\end{align}
$$

Where:
* $$u_{AB}$$ is the unit vector of the line $$AB$$
* $$d$$ is the distance that the projected $$P$$ lies across the line $$AB$$

# Polygons

Polygon algorithms generalize a bunch of different shapes, I used these to handle triangles and rectangles. The algorithms here also don't care if the polygon is convex or concave, which is quite handy.

## Point in Polygon

[geeksforgeeks.org](https://www.geeksforgeeks.org/dsa/how-to-check-if-a-given-point-lies-inside-a-polygon/) provided an excellent algorithm for determining if a point is in a polygon.

The general idea is:
* Convert your point P into a line PQ, where Q is a point far away from P 
  * for example, Q could be at point $$P + (0, 10e^6)$$
  * the important part is that Q must be outside of the polygon.
* For each line that makes up the polygon, determine if that line intersects line PQ (see [Line Intersection](#line-intersection))
* If the number of intersections is odd, P is inside the polygon, otherwise it is outside the polygon.

<figure>
    <img src="/assets/images/posts/guides/geometry/point-in-polygon.png">
</figure>

## Line intersects Polygon

There are two cases to detect:
* The line intersects the polygon's outer lines
* Line is contained within the polygon

To detect line intersection:
* For each line of the polygon, does that line AB intersect (see [Line Intersection](#line-intersection))
* If there are no line intersections, check if point A of line AB is in the polygon (see [Point in Polygon](#point-in-polygon)) 

## Polygon intersects Polygon

There are two cases to detect here too:
* polygon contained in polygon
* polygon crosses polygon perimeter

If we have polygon A and polygon B:
* For each line in polygon A, check if it intersects polygon B (see [Line intersects Polygon](#line-intersects-polygon))

This handles both cases since we also check if a line is contained in the polygon

# Circles

## Point in Circle

If we have a circle A and a point P, check that the distance between P and the circle's center is less or equal to the circle's radius.

For efficiency, I compare the distance squared instead:

$$
r^2 \leq (C_x - P_x)^2 + (C_y - P_y)^2
$$

Where:
* $$r$$ is the circle's radius
* $$C$$ is the center point of the circle
* $$P$$ is the point of interest

This saves on computing a square root which can be computationally inefficient.

## Line Intersects Circle

Given a line AB and a circle C:
* Find the closest point P on line AB to the circle C's center point (see [Closest Point On Line](#closest-point-on-line))
* Check if P is in the circle (see [Point in Circle](#point-in-circle))

## Polygon Intersects Circle

If we have polygon A, and circle B:
* For each line of A check if that line intersects the circle (see [Line Intersects Circle](#line-intersects-circle))

## Circle Intersects Circle

If we have circle A and circle B, this is very similar to point in circle, but we sum the radius's together:

$$
(r_A + r_B)^2 \leq (A_x - B_x)^2 + (A_y - B_y)^2
$$

Where:
* $$r_A$$ and $$r_B$$ are the radii of circles A and B respectively
* $$A$$ is the center point of circle A
* $$B$$ is the center point of circle B
