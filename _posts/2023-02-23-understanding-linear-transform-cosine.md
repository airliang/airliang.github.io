---
title: Understanding Linear Transform Cosine
tags: Rendering
---

## Overview
I will not focus the details on the paper and pay more attention to the whole understanding of the LTC.The most important thing is to under relationships with the cosine, brdf, and the transform cosine. What is linear transform cosine want to do? Why can it approximate the area lighting? And this note is based on the fact that we have already understood the basic knowledge of BRDF, physical-based lighting and realtime rendering.

## Basic understanding
We should under the images which are shown on the paper.

![](post_img/LTC/lobe.PNG)
![](post_img/LTC/lobe_brdf.PNG)

The top image is the 2D lobe of the BRDF and the bottom image represents the 3D lobe of the BRDF. The more red the color is the more radiance it reflects. 

What does the image below tell us?

![](post_img/LTC/moving_lobe.gif)

The left side is the reflect brdf lobe and the varying input lighting radiance direction.The right side is the LTC, they look very similar.

Now we assume we have a shading point which is lighted by an area light, because of the bi-directional property of the BRDF function, so we can use the lobe as the input radiance directions.

![](post_img/LTC/poly_light_shading.PNG)

Assuming the BRDF lobe and the area light position above, if we calculate the reflect radiance of the arealight, we should use the formula:

$\int_{area} f(\omega)L(\omega)\rm d\omega$

Obviously, we have no analytic solution.
So what should we do?

In the slides, the author introduces some lighting models that can approximate the BRDF, and also tells us what the problems these models have. For example, spherical gaussian have no analytic solutions, phong integration is not constant and so on.
We can see the slides on this website:

[Real-Time Polygonal-Light Shading with Linearly Transformed Cosines](https://drive.google.com/file/d/0BzvWIdpUpRx_Z2pZWWFtam5xTFE/view?resourcekey=0-K9rJBtyrgGtxfP3XHDUCyQ)

If we use the linearly transformed consines to approximate the BRDF, we will have a perfect result of appearance and performance.We can see the picture below, it tells us about each result approximated by different models.

![](post_img/LTC/shapes_of_parametric_brdfs.PNG)

## Linearly Transform Cosines
### Concept
We should know that the original function is cosine. And linearly transform cosines means that we transform the original function linearly, usually by a Matrix.

We will show the cosine function and some of its linearly transforms in the image below:

![](post_img/LTC/ltc_examples.png)

As the image shows that a is the original cosine function on a sphere, b,c,d are the transformed cosines.

### Key point
The most important thing to be understood here is that, the LTCs can approximate the BRDF, so what we should do is calculate the integral of the polygon light irradiance by the LTCs.

But how do we calculate? In the LTCs? No! In fact, we calculate the irradiance in the original cosine distribution functions. Because it is simple.
What does the image below tell us?

![](post_img/LTC/transform_process.gif)

It tells us that integral in LTCs is equivalent to the integral in original cosine function. The left side is the LTC which approximates the BRDF, the middle is the transform process, and the right side is the original cosine distribution.

We could see not only the cosine function has been transformed, but also the polygon light has.

We express the image above using this formula:

$\omega_p = \frac{M \omega_{p_o}}{\left \| M \omega_{p_o}\right \|}$

![](post_img/LTC/integral_formula.png)


Shown in the figure below:

![](post_img/LTC/area_transform.png)

We see D as LTC, and Do as the original cosine. After we get the M^(-1), we can get the new polygon in Do, so we compute the irradiance of the new polygon in Do so that we can calculate the reflect radiance at the shading point.

So far, we have understood the theory of the LTC. But it is not over, we still have 2 things to do.
- First, we must calculate the area of the new polygon in Do.
- Second, we must find the Matrix M and its inverse matrix, to transform the polygon vertex to the original cosine space.
The first problem to be solved can be seen in this paper:

[Geometric Derivation of the Irradiance of Polygonal
Lights](https://hal.science/hal-01458129/document)

The second problem will be more difficult. We should precompute the matrices that provide the best fits for GGX microfacet BRDF with varying incident angle θ and roughness α. I think it uses some knowledge of numerical analysis or something about convex optimization.

## Reference
[Geometric Derivation of the Irradiance of Polygonal
Lights](https://hal.science/hal-01458129/document)

[Real-Time Polygonal-Light Shading with Linearly Transformed Cosines](https://drive.google.com/file/d/0BzvWIdpUpRx_Z2pZWWFtam5xTFE/view?resourcekey=0-K9rJBtyrgGtxfP3XHDUCyQ)
