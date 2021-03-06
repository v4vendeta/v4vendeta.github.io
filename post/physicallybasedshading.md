# physically based shading

传统的光照模型有很多局限性，很多材质无法表示，如今不管是在实时渲染还是离线渲染所使用的是基于物理的着色模型，以精确描述材质和光的传输过程中能量变化

**辐射度量学**


一些物理量的定义

| terms             |                                   description                                   |   符号   |                            计算方式                            |        单位 |
| :---------------- | :-----------------------------------------------------------------------------: | :------: | :------------------------------------------------------------: | ----------: |
| Radiant Energy    |                                  total energy                                   |  $Q$   |                                                                |       $J$ |
| Radiant Flux      |                            the energy per unit time                             | $\Phi$ |                     $\Phi=\frac{dQ}{dt}$                     |       $W$ |
| Radiant Intensity |                         the energy per unit solid angle                         |  $I$   |         $I=\frac{d\Phi}{d\omega}$       |    $W/sr$ |
| Irradiance        | the energy per (perpendicular/projected) unit area incident on a surface point. |  $E$   |                 $E(p) =\frac {d\Phi(p)}{dA}$                 |   $W/m^2$ |
| Radiance          |               energy per unit solid angle per projected unit area               |  $L$   | $L(p,\omega)=\frac{d^2\Phi(p,\omega)}{d\omega dA\cos\theta}$ | $W/m^2sr$ |



一些转换关系
$d\omega=dA*r^2$  
$L(p,\omega)=\frac{dE(p)}{d\omega\\cos\theta}\\ 
E(p)=\int_{\omega\in\Omega} L(p,\omega)\cos\theta d\omega $


### Rendering Equation & Reflectance Equation

按照wiki的定义
> In computer graphics, the rendering equation is an integral equation in which the equilibrium radiance leaving a point is given as the sum of emitted plus reflected radiance under a geometric optics approximation. The various realistic rendering techniques in computer graphics attempt to solve this equation.



我们在渲染中计算的量是某个点x沿d方向的radiance，屏幕上的像素是距离相机最近点沿到相机方向的radiance

所要计算的radiance $L_i(c,-v)$，c是相机位置，-v是光线方向

不考虑paticipating media，相机接受的$L_i$就是最近点的$L_o$，$L_i(c,-v)=L_o(p,v)$


**rendering equation**
$L_o(p,v)=L_e(p,v)+\int_{l\in\Omega}f(l,v)L_i(p,l)(n·l)dl$


**reflectance equation**

上式去掉自发光项$L_e(p,v)$

$L_o(p,v)=\int_{l\in\Omega}f(l,v)L_i(p,l)(n·l)dl$

现在看相机c沿-v方向的radiance=距离c最近点的，某个点的radiance=单位半球面上其他方向入射光的 projected radiance*brdf函数的和

简洁起见，省略p
$L_o(v)=\int_{l\in\Omega}f(l,v)L_i(l)(n·l)dl$

**转换成球面积分**

用$\theta$,$\phi$表示方向$l$
$dl=\sin\theta d\theta d\phi\\ 
n·l=\cos\theta\\ 
L_o(\theta_o,\phi_o)=\int_{\phi_i=0}^{2\pi}\int_{\theta_i=0}^{\frac{\pi}{2}}f(\theta_i,\phi_i,\theta_o,\phi_o)\cos\theta_i\sin\theta_i d\theta_i d\phi_i$


# BRDF

同样是wiki的定义

>The bidirectional reflectance distribution function (BRDF; ${\displaystyle f_{\text{r}}(\omega _{\text{i}},\,\omega _{\text{r}})}f_{\text{r}}(\omega _{\text{i}},\,\omega _{\text{r}}) $) is a function of four real variables that defines how light is reflected at an opaque surface. It is employed in the optics of real-world light, in computer graphics algorithms, and in computer vision algorithms. The function takes an incoming light direction, ${\displaystyle \omega _{\text{i}}}$, and outgoing direction, ${\displaystyle \omega _{\text{r}}}$ (taken in a coordinate system where the surface normal ${\displaystyle \mathbf {n} }$  lies along the z-axis), and returns the ratio of reflected radiance exiting along ${\displaystyle \omega _{\text{r}}}$$\omega _{\text{r}}$ to the irradiance incident on the surface from direction ${\displaystyle \omega _{\text{i}}}$. Each direction $\omega$  is itself parameterized by azimuth angle $\phi$  and zenith angle $\theta$ , therefore the BRDF as a whole is a function of 4 variables. The BRDF has units sr^−1^, with steradians (sr) being a unit of solid angle.

**物理性对brdf的约束**

1. Helmholtz reciprocity principle
    $f(l,v)=f(v,l)$
    

2. conservation of energy
    $∀l,\int_{Ω}f(l,v)(n·v) dω_0 ≤ 1$
   能量守恒，反射光能量不能大于入射光

精确的Helmholtz reciprocity和能量转换对于实时渲染算法不是必须的

---
# BRDF Models


### lambert diffuse

rtr4中定义了directional-hemispherical reflectance $R(l)$描述brdf能量转换的程度

lambertian漫反射的brdf函数是常数，与l，v无关，n·l决定了着色的强度

$R(l)=c\\ 
=\int_{}f(l,v)(n·l)dl\\ 
=f(l,v)\int_{l\in\Omega}(n·l)dl\\ 
=f(l,v)\int_{0}^{2\pi}\int_{0}^{\frac{\pi}{2}}\cos\theta\sin\theta d\theta d\phi\\ 
=\pi f(l,v)$
$f(l,v)=\frac{c}{\pi}$

注:lambert diffuse是经验模型，not physically correct

---

### fresnel reflectance

- external reflection

光疏->光密，n1<n2，只有反射
例：空气->金属

反射夹角与入射夹角相同，方向$r=2(n·l)n-l$

![](https://www.realtimerendering.com/figures/thumb/RTR4.09.19.jpg)
反射光的强度由fresnel reflectance系数$F(\theta)$决定，对于一种材质，$F(\theta)$只由入射光方向与表面法线方向的夹角$\theta$决定，$\theta$越大，$F(\theta)$越大，通常，物体边缘的反射强度更高。

有无fresnel reflectance的片对比（unity）

![](https://docs.unity3d.com/uploads/Main/StandardShaderFresnelGraduationTable.jpg)



schlick给出了一种$F(\theta)$的近似方法

$F(n,l)\approx F_0+(1-F_0)(1-(n·l))^5$
更通用的形式是这一种

$F(n,l)\approx F_0+(F_{90}-F_0)(1-(n·l))^{\frac{1}{p}}$

其中$F_0=(\frac{n1-n2}{n1+n2})^2$，与折射率有关，若从空气入射到材质中一般为$F_0=(\frac{n-1}{n+1})^2$

几种材质的$F(\theta)$曲线，分别是glass，copper，aluminum，chrominum，iron，zinc。分为RGB三个通道，实线是fresnel equation的计算结果，虚线是schlick方法近似的结果，可以看出效果还不错，部分材质需要用更准确的模型描述

![](https://www.realtimerendering.com/figures/thumb/RTR4.09.22.jpg)

银（Ag）和水银（Hg）在rgb三通道的fresnel value都很高，一般用来做镜子，



- internal reflectance

光密->光疏，n1>n2，发生反射+折射，只会发生在电介质中

![](https://www.realtimerendering.com/figures/thumb/RTR4.09.23.jpg)

折射方向由snell law计算

$sin_{\theta_i}\cdot n_1=sin_{\theta_t}\cdot n_2$
$\theta_t$始终大于$\theta_i$，小于1
$\theta_i>\theta_c$时发生全反射

反射光强度将external的$F_0\to F_{90}$映射到了$F_0\to F_{\theta_c}$，一张对比图

![](https://www.realtimerendering.com/figures/thumb/RTR4.09.24.jpg)

internal reflection总是偏大些（Ex：水下的气泡总是有很强的反射）

**参数化**

一般diffuse用$\rho_{ss}$，specular用$F_0$，一些厂商在实现上由于和美术对接/性能等原因使用了不同的参数化方法，包括

1. A Realistic Lighting Model for Computer Animators
用metalness表示F项
最新版
Reflection Model Design for WALL-E and Up
unity standard shader的参数是smoothness和metalness

2. Physically Based Shading at Disney
disney shading model中specular scalar参数来限制$F_0$范围，几个变种，Real Shading in Unreal Engine 4，Moving Frostbite to Physically Based Rendering，Rendering of Call of Duty Infinite Warfare,
3. 对于$F_0>0.02$这个特性，一些厂商还做了优化

---

### microgemotry

不是在建模时将模型建成微表面！

**NDF ${D(m)}$**

表面法线方向不唯一，呈随机正态分布，聚集在$n$附近，视微表面法线方向为$m$，法线分布函数NDF定义为$D(m)$
$D(m)$在球面上积分为

$\int_{m\in\Theta}D(m)(m\cdot n)dm=1$

更通用的形式

$\int_{m\in\Theta}D(m)(v\cdot m)dm=v\cdot n$


**几何函数 ${G(l,v,h)}$**

考虑反射量的组成，只有那些有光源照射且反射方向=视线方向的微表面才对反射量有贡献。几何函数 $G(l,v,h)$ 描述了这一属性，即 m = h 的未被遮蔽的表面点的百分比

- masking function $G_1$

![](https://www.realtimerendering.com/figures/thumb/RTR4.09.32.jpg)
渲染计算量只考虑可视表面，即$dot(m,n)>0$的面，背面不可见

$\int_{m\in{\Theta}}G_1(m,v)D(m)(v,m)^+dm=v\cdot n$

Smith masking function 是最常见的$G_1$函数

$G_1(m,v)=\frac{\lambda^+(m,v)}{1+\Lambda(v)}\\
\lambda^+(x)=\begin{cases}
    1, where x>0\\
    0, where x<=0
\end{cases}$

- joint masking-shadow function $G_2$

$G_2$由$G_1$推导而来，描述了可视微表面对光源可见的比例（本文不做具体推导，详见heitz）

最终的$G_2$项
$G_2(l,v,m)=\frac{\lambda^+(m,v)\lambda^+(m,l)}{1+\Lambda(v)+\Lambda(l)}$


### 反射项brdf（cook-torrance）

$f(l,v)=\frac{F(h,l)G(l,v,h)D(h)}{4|n\cdot l||n\cdot v|}$


<!-- ### subsurface scattering
 
brdf模型只考虑光线/视线在点的上半球面（光不穿过物体），

 -->

# Real Shading in Unreal Engine 4

## GOAL:
- real-time performance
- reduce complexity
- intutive interface
- Perceptually Linear
- easy to master
- robust
- experssive
- flexible

## Shaidng Model

### Diffuse
disney pricipled brdf里diffuse没有用lambertian diffuse，而是加上给你了Fresnel factor，计算形式如下
$f_d=\frac{c_{diffuse}}{\pi}(1+(F_{D90}-1)(1-\cos\theta_v))^5)(1+(F_{D90}-1)(1-\cos\theta_v=l))^5)$
$F_{D90}=0.5+2\cdot roughness\cdot\cos^2_d\theta_d$

### Specular 
GGX做NDF，Smith-GGX做Geometry Function，Spherical Gaussian approximation做Fresnel Term

## IBL:

用MC方法近似渲染方程，pdf是$p(l_k,v)$
![](https://www.zhihu.com/equation?tex=L_%7Bo%7D%28v%29+%3D+%5Cint_%7BH%7DL_%7Bi%7D%28l%29f%28l%2Cv%29cos%5Ctheta_%7Bl%7Ddl%5Capprox+%5Cfrac%7B1%7D%7BN%7D%5Csum_%7Bk%3D1%7D%5E%7BN%7D%5Cfrac%7BL_%7Bi%7D%28l_%7Bk%7D%29f%28l_%7Bk%7D%2Cv%29cos%5Ctheta_%7Bl_%7Bk%7D%7D%7D%7Bp%28l_%7Bk%7D%2C+v%29%7D)
# Split Sum Approximation

将上面的求和式写成如下形式
![](https://www.zhihu.com/equation?tex=%5Cfrac%7B1%7D%7BN%7D%5Csum_%7Bk%3D1%7D%5E%7BN%7D%5Cfrac%7BL_%7Bi%7D%28l_%7Bk%7D%29f%28l_%7Bk%7D%2C+v%29cos%5Ctheta_%7Bl_%7Bk%7D%7D%7D%7Bp%28l_%7Bk%7D%2C+v%29%7D+%5Capprox+%28%5Cfrac%7B1%7D%7BN%7D%5Csum_%7Bk%3D1%7D%5E%7BN%7DL_%7Bi%7D%28l_%7Bk%7D%29%29%28%5Cfrac%7B1%7D%7BN%7D%5Csum_%7Bk%3D1%7D%5E%7BN%7D%5Cfrac%7Bf%28l_%7Bk%7D%2Cv%29cos%5Ctheta_%7Bl_%7Bk%7D%7D%7D%7Bp%28l_%7Bk%7D%2C+v%29%7D%29+)

对不同的roughness，可以把$\frac{1}{N}\sum^N_{k=1}L_i(l_k)$预计算成纹理


### 其他

roughness，specularness，metalness，smoothness，glossiness的概念


### Reference

《Real-Time Rendering, 4th Edition》
[https://www.scratchapixel.com/lessons/3d-basic-rendering/phong-shader-BRDF](https://www.scratchapixel.com/lessons/3d-basic-rendering/phong-shader-BRDF)

[https://en.wikipedia.org/wiki/List_of_common_shading_algorithms](https://en.wikipedia.org/wiki/List_of_common_shading_algorithms)
[https://en.wikipedia.org/wiki/Phong_reflection_model](https://en.wikipedia.org/wiki/Phong_reflection_model)

<!-- [flat shading](https://www.cs.drexel.edu/~david/Classes/Papers/p443-newell.pdf)
[phong shading ](http://www.cs.northwestern.edu/~ago820/cs395/Papers/Phong_1975.pdf)
[gouraud shading](https://ieeexplore.ieee.org/abstract/document/1671906) -->

<!-- [](http://blog.sina.com.cn/s/blog_415a66f001018bud.html)

[](http://web.cs.wpi.edu/~emmanuel/courses/cs563/S05/projects/surface_reflection_losasso.pdf) -->

https://www.brown.edu/research/labs/mittleman/sites/brown.edu.research.labs.mittleman/files/uploads/lecture13_0.pdf


