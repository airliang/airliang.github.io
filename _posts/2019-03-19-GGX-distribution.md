---
title: Microfacet GGX/Trowbridge-Reitz Distribution PDF Derivation
tags: Rendering Math
---

The real name of "GGX" is "Trowbridge–Reitz".

## Derivation
The normal distribution of microsurfaces is defined as follows:

$ \int _{H^2(n)}D(\omega_h)\cos \theta_h d\omega_h = 1$

So we can get the pdf of $\omega_h$:

$ p(\omega_h) = D(\omega_h)\cos \theta_h$

According to the definition of Beckmann–Spizzichino, the isotropic distribution is define as:

$D(\omega_h) = \frac{\alpha^2}{\pi \left( \cos^2\theta_h (\alpha^2 - 1) + 1 \right)^2}$

We know $p(\omega) = \sin\theta p(\theta, \phi) $, this is because $d\omega = \sin\theta d\theta d\phi $, see below:

![Solid Angle](post_img/math/solid_angle_differential.png)



$p(\omega) = D(\omega) \cos \theta_h = \frac{\alpha^2 \cos \theta_h}{\pi \left( (\cos \theta_h)^2 (\alpha^2 - 1) + 1 \right)^2}$

Thus, we have:

$p(\theta, \phi) = \frac{\alpha^2 \cos \theta_h \sin\theta}{\pi \left( (\cos \theta_h)^2 (\alpha^2 - 1) + 1 \right)^2}$

Then we could use Marginal probability density function of $\theta$:

$p(\theta) = \int_0^{2\pi} p(\theta, \phi) d\phi = \int_0^{2\pi} \frac{\alpha^2 \cos \theta \sin \theta}{\pi \left( (\cos \theta)^2 (\alpha^2 - 1) + 1 \right)^2} d\phi = \frac{2 \alpha^2 \cos \theta \sin \theta}{\left( (\cos \theta)^2 (\alpha^2 - 1) + 1 \right)^2}$

Conditional probability desnity funciton of $\phi$:

$p(\phi | \theta)=\frac{p(\theta, \phi)}{p(\theta)}= \frac{1}{2\pi}$

Then we can calculate the CDF of $\theta$ and $\phi|\theta$ separately.

$\int_0^{\theta}p(\theta) = \int_0^{\theta} \frac{2 \alpha^2 \cos \theta \sin \theta}{\left( (\cos \theta)^2 (\alpha^2 - 1) + 1 \right)^2} d\theta = \frac{\alpha^2}{(\alpha^2-1)(1+\cos^2\theta(\alpha^2-1))} -\frac{1}{\alpha^2-1} = \xi_0$

I solve the integrate from [symbolab](https://www.symbolab.com/) since I can't solve it manually.

Then we get:


$\cos^2\theta = \frac{1-\xi_0}{1+(\alpha^2 - 1)\xi_0}$

$\theta = \cos^{-1}\sqrt{\frac{1-\xi_0}{1+(\alpha^2 - 1)\xi_0}}$

Let's see $\phi|\theta$:

$\int_0^{\phi}p(\phi | \theta)d\phi = \frac{\phi}{2\pi} = \xi_1$

$\phi = 2\pi\xi_1$

## Reference
[Understanding the concept of Solid Angle](https://www.youtube.com/watch?v=VmnkkWLwVsc&ab_channel=EngineeringStreamlined)

[pbrbook 13.5.3 Spherical Coordinates](https://www.pbr-book.org/3ed-2018/Monte_Carlo_Integration/Transforming_between_Distributions)
