---
title: "Lightweight Physics"
excerpt: "Adding a physics simulator to an embedded project"
permalink: "/guides/mathematics/embedded-physics"
toc: true
categories:
  - guide
  - mathematics
  - gamedev
---

[Trick Shot II](/trick-shot-2) runs on a microcontroller simulating real-time 2D physics at 60Hz.

The key parts of this engine were:
* A solver (integrator)
* Equations of motion
* Collision detection
* Collision handling

## Simulation Structure

The `Simulation` was a simple `struct` made up of two lists of objects:
* `Dynamic` - objects which can move around
* `Static` - objects which have *infinite* mass

At the beginning of each level in `Trick Shot`, the simulation is cleared and then populated with these objects.

On each update step, the `Simulation` follows the following procedure:
* For all `Dynamic` objects, `Solve` for your next state
* For all `Dynamic` objects, check for `DynamicCollision` with all _other_ `Dynamic` objects
  * This sound pricey because it is an `O(N^2)` operation, but in practice I rarely had more than one dynamic object; and when I did there were no more than 4.
* For all `Dynamic` objects, check for `StaticCollison` with all `Static` objects
* For all `Dynamic` objects, `Commit` the next state

Calling `Solve` results in two states:
* `current`
* `next` - what our integrator would choose as the next state without any collisions

Then we determine changes in `next` state based on collision interactions.

Finally, we `Commit` the `next` state by making `current = next`.

## Solver and Equations of Motion

The selected solver was [Runge-Kutta 2nd Order](/guides/mathematics/runge-kutta), this solver is quite simple; but so were the physics in `Trick Shot`.

I wanted a solver which had a deterministic execution time. Afterall, this was for a Jam, and I didn't want something complex to debug if it broke.

The model used was a simple projectile model. I've derived this in [Lagrange's Equations](/guides/engineering/mathematics/lagranges-equations), but the general formula are:

$$ \begin{aligned} \ddot{x} &= -\frac{1}{2M}k_D\dot{x} \\ \ddot{y} &= -g - \frac{1}{2M}k_D\dot{y} \end{aligned} $$

For the balloons, I set gravity $$g$$ to 0, and bumped the drag right up, so they kind of just float around and help bounce the ball around as they get pushed.

## Collisions

Collision detection is discussed in [Geometry and Collisions](/guides/mathematics/geometry-and-collisions). 

An important part of the collision detection is determining the points of contact, which the "closest point to" methods are very helpful for doing.

### Momentum Equations

We have two objects colliding where:
* $$p_1$$, $$v_1$$, $$m_1$$ are the position, velocity and mass of object 1
* $$p_2$$, $$v_2$$, $$m_2$$ are the position, velocity and mass of object 2

Compute the collision normal:

$$ 
n = \frac{p_2 - p_1}{\|p_2 - p_1\|} 
$$ 

Compute the projected velocity magnitude on that normal:

$$
v_{collide} = (v_2 - v_1) \cdot n
$$

If $$v_{collide} >= 0$$ the points are moving away from each other, so no collision occurs.

Compute the impulse:

$$
j = -(1 + e)\frac{vrel}{\frac{1}{m_1} + \frac{1}{m_2}} 
$$

Where:
* `e` is the restitution, $$e \in [0, 1]$$
	* if `e` is 0, totally inelastic, maximum energy is lost in the collision
	* if `e` is 1, totally elastic, no energy is lost and they bounce

Finally, we can compute new velocities:

$$
\begin{aligned}
v_1' &= v_1 - (j/m_1) \cdot n \\
v_2' &= v_2 - (j/m_2) \cdot n \\
\end{aligned}		
$$

### Handling Collisions

The momentum equations work well for two dynamic point masses, but in the physics engine, we were dealing with shaped objects, some static and some dynamic.

#### Dynamic on Static Collisions

Dynamic/Static collisions are easier to handle, I used the momentuum equations but with the following changes:
* Place dynamic object in the `next` state
* Check if there is a collision. If there is:
  * Find the closest point $$p_{c2}$$ on the other object.
  * The normal vector is given by $$ n = \frac{p_{c2} - p_1}{\|p_{c2} - p_1\|} $$
    * This works because the dynamic objects were circles
    * If the dynamic objects were not circles, we would also need to compute $$p_{c1}$$ the closest point on the dynamic body _to_ the static body
  * Compute $$v_{collide}$$
  * Place dynamic object in the `current` state
  * Pass all of this to the static body to determine $$j$$
  * If $$j$$ is not zero, apply it to the `next` state

The main reason that the `static` body determines $$j$$ is to allow bouncy, springy and other such obstacles to all affect the dynamic object differently.

In some cases, static objects (like springs) will break the laws of physics and inject energy into the system... it is a game afterall.

#### Dynamic on Dynamic Collisions

My implementation for Dynamic on Dynamic collisions was similar to Dynamic on Static. The main difference being that both objects adjust thier momentum.

I barely tested how these interactions worked, and I have a feeling they are probably quite buggy under many circumstances.

Like many things in game design, I avoided complexity by not exposing the player to it.

