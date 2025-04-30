---
layout: article
title: 2D FSI - Stable Fluid
mathjax: true
pageview: true
show_subscribe: true
chart: true
mermaid: true
lightbox: true
theme: dark
sharing: false
---

# Stable Fluid

Actually, CFD is a good way to simulate the fluid dynamics.
But, the Navier-Stokes equation is not stable in complex cases with high cells number.
Stable Fluid is a method to solve the Navier-Stokes equation in 2D used by semi-Largian-advection scheme in computer graphics, which is absolutely stable, althought it is not very accurate.
See [Paper](https://www.dgp.toronto.edu/public_user/stam/reality/Research/pdf/ns.pdf) for details.
In this project, I this method is made by FDM in 2D.

##  Theory

The Navier-Stokes equation is a partial differential equation, which is a partial derivative of the velocity field, which as follows:

$$ \nabla \cdot \vec{u} = 0 $$

$$ \frac{\partial u}{\partial t} + \frac{\partial (u \cdot u)}{\partial x} + \frac{\partial (v \cdot u)}{\partial y} = -\frac{\partial p}{\partial x} + \nu \left( \frac{\partial^2 u}{\partial x^2} + \frac{\partial^2 u}{\partial y^2} \right) $$

$$ \frac{\partial v}{\partial t} + \frac{\partial (u \cdot v)}{\partial x} + \frac{\partial (v \cdot v)}{\partial y} = -\frac{\partial p}{\partial y} + \nu \left( \frac{\partial^2 v}{\partial x^2} + \frac{\partial^2 v}{\partial y^2} \right) $$

The stable fluid method aimto split the equation into two parts, one for the velocity and the other for the pressure, which can ingore  the pressure effect on the velocity, which are follows:

$$ w = u + \nabla p $$

which means, any vector field can be decomposed into a divergence free part and a curl free part.
So, how to solve velocity field from any vector field? 
Projection operator can do this:

$$ u = P(w) = w - \nabla p $$

which contains a relation : 

$$ P(u) = u, P(w) = u, P(w-u) = P(\nabla p) = 0 $$.

This is also means no divgence free part of Pressure Gradient.

So, pressure should be solved first, as follows:

$$ \nabla^2 p = \nabla \cdot w $$

which contians the mass conservation relation.

Now, we take projection operator into N-S equation:

$$ \frac{\partial u}{\partial t}  = P  \  ( -(u \cdot \nabla) u + \mu \nabla^2 u + f ) $$

, by doing this, we split the pressure part from the Navier-Stokes equation.

Now, for convenience, we take above equation into three parts:

$$ \frac{\partial u}{\partial t} = \frac{u^{n+1} - u^n}{ \delta t}  = \frac{u^{n+1} - u^n_2}{ \delta t}  + \frac{u^{n}_2 - u^n_1}{ \delta t} + \frac{u^{n}_1 - u^n}{ \delta t} $$

which means time iteration is split into three parts:

$$ \frac{u^{n}_1 - u^n}{ \delta t} = P (f) $$

which is about body force iteration.

$$ \frac{u^{n}_2 - u^n_1}{ \delta t} = P (-(u \cdot \nabla) u)    $$

which is about advection iteration.

$$ \frac{u^{n+1} - u^n_2}{ \delta t} = P (\mu \nabla^2 u)  $$

which is about diffusion iteration.

###  Force Iteration

We cancel the Projection term from the equation to get the force iteration

$$ \frac{u^{n}_1 - u^n}{ \delta t} = P (f) $$

$$ \frac{w_1 - w_0}{ \delta t} = f $$

$$ w_1 = w_0 + \delta t f $$

where $u^n = P(w_0)$ is the previous pressure and $u^{n}_1 = P(w_1)$ is the current pressure.
Then we get the force iteration:$w_1$



