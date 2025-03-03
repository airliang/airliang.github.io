---
title: Perspective-Correct Interpolation
tags: Math
---

## Mathematics
Look at the picture below:

![](post_img/math/perspective_correct_interpolation.PNG)

If we know the $point <x1, z1>$ and the $point <x2, z2>$, then project them to the z=-e plane and get the pixel point $p1$ and $p2$, how can we get the depth value of $p3$ point?
$P$ is the $x$ value of the point and $-e$ is the z value.
Let's see the similar triangle equation:

$$\frac{p}{x} = \frac{-e}{z}$$    (1)

Then we create a line equation which is formed by $<x1,z1>$ and $<x2, z2>$ as follow:

![](post_img/math/perspective_correct_interpolation_2.PNG)

$$ax + bz = c$$

$$a\frac{x}{z} + b = \frac{c}{z}$$    (2)

From the formula (1) we get $\frac{x}{z} = -\frac{p}{e}$. Then substain it into (2):

$$\frac{1}{z} = -\frac{ap}{ce} + \frac{b}{c}$$  (3)

$p_3$ is interpolated by $p_1$ and $p_2$.

$$p_3 = (1-t)p_1 + tp_2$$  (4)

Subtain (4) to (3), then we can calculate $z_3$:

$$\frac{1}{z_3} = \frac{1}{z_1}(1 - t) + \frac{1}{z_2}t$$   (5)

## Interpolation of homogeneous coordinates

If there are 2 points, we have homogeneous coordinates $(w_1x_1, w_2z_1, w1)$ and $(w_2x_2, w_2z_2, w_2)$.
Now we interpolate them to get a new point $(w_3x_3, w_3z_3, w_3)$ by $t$.

$$w_3 = w_1(1 - t) + w_2t$$

$$w_3z_3 = w_1 z_1(1 - t) + w_2 z_2t$$

We should prove that the last 2 equations can yield (5)

Now we look at the picture from games101:

![](post_img/math/perspective_correct_interpolation_3.PNG)

From the triangles above, we also get the y value of the unprojected point:

$$y = \frac{z}{n}y'$$

