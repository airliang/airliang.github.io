---
title: Understanding Irradiance Integrals
tags: Math
---

## Question Description
Recently, I had a question from my friend. Here is his question:

![](post_img/math/divide_distance_square.png)

Why do indirect light radiance not need to divide the distance square?

We see the scattering equation here:

$L_o(X', \omega_o') = \int_{S^2}f_s(X', \omega_i' \rightarrow \omega_o')L_{e,i}(\omega_i'){\rm d}\sigma^{\perp}(\omega_i')$

Actually, the incoming radiance may not just from a semisphere, it might come from an area light source.

## Integrals over Projected Solid Angle
Irradiance at a point $p$ with surface normal $n$ due to radiance over a set of directions $\Omega$ is:

$E(p,n) = \int_{\Omega} L_i(p,\omega)|\cos \theta|\rm d\omega$

![](post_img/math/irradiance_semisphere.png)

*Irradiance at a point p is given by the integral of radiance times the cosine of the incident direction over the entire upper hemisphere above the point.*

The projected solid angle measure is related to the solid angle measure by

${\rm d}\omega^{\perp} =|\cos \theta|{\rm d}\omega$

Then the irradiance over the hemisphere at the point $p$ could be written by

$E(p,n) = \int_{H^2(n)} L_i(p,\omega)\rm d\omega ^{\perp}$

How about the light radiance comes from a surface?

## Integrals over Area
Differential area $\rm dA$ on a surface is related to differential solid angle as viewed from a point $p$ by

${\rm d}\omega = \frac{\rm d A \cos\theta}{r^2}$

See the picture:

![](post_img/math/radiance_from_area_light.png)

If we project ${\rm d}A$ to the the unit hemisphere along $\omega$, it's easy to understand the the area on the hemisphere needs to be divided by $r^2$.
See the definition of solid angle:

$\Omega = \frac{A}{r^2}$

Where $A$ is the area (or any shape) on the surface of the sphere and $r$ is the radius of the sphere.

Therefore, we can write the irradiance integral for the quadrilateral source as

$$E(p,n) = \int_A L_i(p,\omega) \cos \theta_i\frac{\rm dA \cos\theta_o}{r^2}$$

## Summary
Turning back to the question, the directional light radiance is sampled from a point in the light surface, so it needs be to divided by $r^2$. 

While the indirect light radiance is sampled by a direction so it doesn't need to be divided by $r^2$.