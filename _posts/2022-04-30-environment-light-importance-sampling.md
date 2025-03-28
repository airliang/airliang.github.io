---
title: Environment light importance sampling
tags: Math Rendering
---

## Importance sampling overview
Two key points:
1. What is the sample?
2. The probability density value of the sample.

If we want to estimate the integral of function f, we can use the following Monte Carlo formula to estimate it:

$F_N=\frac{1}{N}\sum_{i=1}^{N}\frac{f(X_i)}{p(X_i)}$, where X is the estimated sample and N is the number of samples.

What we need to estimate is the contribution of incident rays reaching a point within the hemisphere, and only environment light is discussed here.

So what is the sample?

## Build distribution
Only by constructing the distribution can we obtain the probability density of the sample. Since the distribution is discrete, here the probability density is understood as the probability mass function.

When sampling environment light in the direction, the three-dimensional direction can be converted into two-dimensional polar coordinates $(\theta, \phi)$.

Therefore, latitude and longitude diagrams can be used to create environment light and construct distribution.

Longitude and latitude diagram description:

![](post_img/environment_light_importance_sampling/longitude_latitude_map.png)

Example image:

![](post_img/environment_light_importance_sampling/example.png)

Using the brightness of each pixel in the ambient light map to construct the distribution, the distribution can be constructed using the following code:

```c++
float[] distributions = new float[width * height];
Color[] pixels = envmap.GetPixels(mipmap);
for (int v = 0; v < height; ++v)
{
    float vp = ((float)(height - 1 - v) + 0.5f) / (float) height;
    float sinTheta = Mathf.Sin(Mathf.PI * vp);
    for (int u = 0; u < width; ++u)
    {
        float y = pixels[u + v * width].ToVector3().magnitude;
        float distribution = y;
        if (distribution == 0)
             distribution = float.Epsilon;
        distributions[u + v * width] = distribution * sinTheta;
    }
 }
 ```

 >Why multiply by a $\sin\theta$?

Imagine wrapping the longitude and latitude diagram into a sphere. In fact, the area of each pixel on the sphere is different and not evenly spread. Pixels at the equator must occupy the largest area on the sphere, while pixels at the poles only have one point.

However, when unfolded into a latitude and longitude diagram, both the equator and the pole occupy a row of texture pixels.

So we need to multiply sinTheta to change the distribution: the polar coordinate $\theta = 0$ at the pole, and $\theta = 90$ at the equator.

![](post_img/environment_light_importance_sampling/times_sintheta_explain.png)

Perform the first transformation of the probability density from the direction to the polar coordinates.

$p(\theta, \phi) = p(\omega) \times \sin\theta$

## Probability density conversion from UV coordinates to polar coordinates

Mapping relationship between texture UV coordinates and polar coordinates:

$
\begin{bmatrix}
\theta \\ 
\phi
\end{bmatrix}  = \begin{bmatrix}
\pi \times (v - 0.5) \\ 
2 \pi \times (u - 1)
\end{bmatrix}$

Obtain Jacobian matrix by taking partial derivatives

$\begin{bmatrix}
\frac{\partial \theta}{\partial v}  & \frac{\partial \phi}{\partial v}\\ 
\frac{\partial \theta}{\partial u} & \frac{\partial \phi}{\partial u}
\end{bmatrix} = 
\begin{bmatrix}
\pi  & 0\\ 
0 & 2\pi
\end{bmatrix}$

According to the conversion formula of probability density of random variables:
$y = T(x)$

$p_y(y) = p_y(T(x))=\frac{p_x}{|J_T|}$

So there are:

$p(\theta, \phi) = \frac{p(u,v)}{2\pi^2}$

$p(\omega)=\frac{p(\theta, \phi)}{\sin \theta} = \frac{p(u, v)}{2\pi^2\sin \theta}$

Sampling code:

```c++

float3 ImportanceSampleEnviromentLight(float2 u, out float pdf, out float3 wi)
{
    float mapPdf = 0;
    pdf = 0;
    wi = 0;
    float2 uv = Sample2DContinuous(u, mapPdf);
    if (mapPdf == 0)
        return float3(0, 0, 0);
    
    float theta = (1.0 - uv.y) * PI;
    float phi = (0.5 - uv.x) * 2 * PI;
    float cosTheta = cos(theta);
    float sinTheta = sin(theta);
    float sinPhi = sin(phi);
    float cosPhi = cos(phi);
    //left hand coordinate and y is up
    float x = sinTheta * cosPhi;
    float y = cosTheta;
    float z = sinTheta * sinPhi;
    wi = float3(x, y, z);
    // Compute PDF for sampled infinite light direction
    pdf = mapPdf / (2 * PI * PI * sinTheta);
    if (sinTheta == 0)
    {
        pdf = 0;
        return 0;
    }
    
    return _LatitudeLongitudeMap.SampleLevel(_LatitudeLongitudeMap_linear_repeat_sampler, uv, 0) * 100;
}
```

## comparison
64spp importance sample envmap

![](post_img/environment_light_importance_sampling/64spp_importance_sampling.png)

64spp uniform sample envmap

![](post_img/environment_light_importance_sampling/64spp_uniform_sampling.png)

## Derivation of Jacobian determinant in probability density conversion
In order to better understand this Jacobian determinant, let's assume that the mapping relationship between two variables $X$ and $Y$ corresponding to UV is $X = X (U, V), Y = Y (U, V)$. This is similar to theta and phi above.

Partial derivatives can be understood geometrically as the tangent of a plane. If we view $X$ and $Y$ as just another 2D coordinate system, then partial derivatives can form a vector. The mapping of the plane composed of dudv to the xy plane is a parallelogram, as shown in the figure below.

If the size of the change in u is du, then the size of the change in x is $(\partial x/\partial u) du$, and the size of the change in y is $(\partial y/\partial u)$ du.

The vectors A and B in the above figure are:

$A = (\partial x / \partial u, \partial y / \partial u)\rm du$

$B = (\partial x / \partial v, \partial y / \partial v) \rm dv$

Why can AB be decomposed into the vector representation above? 

According to the multivariate differential formula, assuming that A is decomposed into x and y directions, the length is dx and dy.

$dx = \frac{\partial x}{\partial u}du + \frac{\partial x}{\partial v}dv$

$dy = \frac{\partial y}{\partial u}du + \frac{\partial y}{\partial v}dv$

Since uv is vertical, when du changes, dv is 0 , so the vector AB can be represented by the above expression.

Therefore, the area of the parallelogram formed by AB is:

$|A \times B| = 
\begin{vmatrix} 
\partial x / \partial u & \partial x / \partial v\\ 
\partial y / \partial u & \partial y / \partial v
\end{vmatrix} \rm du \rm dv$

The probability of X and Y in the area S formed by AB is equal to the probability of UV in the area R formed by dudv.

$P\{(X,Y)\in S \} = P\{(U,V) \in R \}$

The probability of a domain on the plane can be understood as probability = probability density function x area:

$f_{XY} \times S = f_{UV} \times R$

$f_{XY} \times |J|\rm d u dv = f_{UV}\rm dudv $

Finally, it is concluded that:

$f_{XY} = \frac{f_{UV}}{|J|}$

## Reference
[Pbrbook.org Sampling Light Sources](https://www.pbr-book.org/3ed-2018/Light_Transport_I_Surface_Reflection/Sampling_Light_Sources)
