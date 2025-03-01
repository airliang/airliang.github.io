---
title: Understanding Multiple Importance Sampling
tags: Math
---

## Direct Light Estimation
In path tracing, once the ray hits the surface, we need to estimate the direct light radiance reflectance in the outgoing direction. Considering an area light contribution of the incoming radiance to the hitting point, we need to integrate all the visible directions from the area light surface as the integral domain. See the image below:

![](post_img/understanding_multiple_importance_sampling/scattering_equation.png)

Mathematically, this is determined by the scattering equation:

$ L_o(X', \omega_o') = \int _{S^2}f_s(X', \omega_i' \rightarrow \omega_o')L_{e,i}(\omega_i')\rm d\sigma ^{\perp}(\omega_i') $

where $L_{e,i}$ represents the incident radiance due to the area light source $S$.

There are two common strategies for Monte Carlo evaluation of the scattering equation. One is sampling BSDF, the other is sampling light radiance. Since we can only use one integrand function in Monte Carlo method. The scattering equation contains at least two functions in the integrand, which results in choosing one of the functions to estimate.

## Sampling the BSDF

To sample the BSDF, we choose an incident direction $\omega_i'$ according to the pdf $p(\omega_i')$.  Normally, this density is chosen to be proportional to the BSDF.

$p(\omega_i') \propto f_s(X', \omega_i' \rightarrow \omega_o')$

Then we could estimate the outgoing Light 

$L_o(X', \omega_o') \approx \frac{f_s(X', \omega_i' \rightarrow \omega_o')L_{e,i}(\omega_i')}{p(\omega_i')}$

## Sampling the light source
To explain the other strategy, we first rewrite the scattering equation as an integral over the surface of the light source:

$$L_o(X' \rightarrow X'') = \int_{M}f_s(X \rightarrow X' \rightarrow X'')L_e(X \rightarrow X')G(X \leftrightarrow X')dA(X)$$

In this case, the sample is a point $X$ on the area light surface. Also, the pdf of this sample should be proportional to the Light source:

$$p(X) \propto L_{e,i}(X)\frac{|\cos(\theta_o)|}{|X-X'|^2}$$

With this strategy, the sample points $X$ are uniformly distributed within the solid angle subtended by the light source at the current point $X'$. 

>This provides us with the inspiration that we could build the light mesh triangle distribution with the $\cos\theta_o$.

## Reference
[Veach Multiple Importance Sampling](https://graphics.stanford.edu/courses/cs348b-03/papers/veach-chapter9.pdf)
