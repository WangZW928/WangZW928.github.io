---
layout: article
title: 2D FSI - Stable Fluid 1.0
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
In this project, this method is implemented using finite difference methods (FDM) in 2D.

##  Theory

The Navier-Stokes equation is a partial differential equation, which is a partial derivative of the velocity field, which as follows:

$$ \nabla \cdot \vec{u} = 0 $$

$$ \frac{\partial u}{\partial t} + \frac{\partial (u \cdot u)}{\partial x} + \frac{\partial (v \cdot u)}{\partial y} = -\frac{\partial p}{\partial x} + \nu \left( \frac{\partial^2 u}{\partial x^2} + \frac{\partial^2 u}{\partial y^2} \right) $$

$$ \frac{\partial v}{\partial t} + \frac{\partial (u \cdot v)}{\partial x} + \frac{\partial (v \cdot v)}{\partial y} = -\frac{\partial p}{\partial y} + \nu \left( \frac{\partial^2 v}{\partial x^2} + \frac{\partial^2 v}{\partial y^2} \right) $$

The stable fluid method aimto split the equation into two parts, one for the velocity and the other for the pressure, which can ingore  the pressure effect on the velocity, which are follows:

$$ w = u + \nabla p $$

According to Helmholtz–Hodge decomposition, any vector field can be uniquely decomposed into a divergence-free (solenoidal) component and a gradient field (curl-free) component.
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
Then we get the force iteration:$w_1$.


### Advection Iteration

For advection iteration, smeils-Largian method is used, meaning we trace the velocity field backward in time (semi-Lagrangian advection),
which is as follows:

$$ u^{n}_2(x) =  u^n_1(p(x,-\delta t))$$

,where $p(x,-\delta t)$ is the position of the previous time step.

Thus, we get the advection iteration:$w_2 = w^_1(p(x,-\delta t))$.

### Diffusion Iteration

We cancel the Projection term from the equation to get the diffusion iteration:

$$ w_3 = w_2 + \delta t \mu \nabla^2 w_2 $$

, while, implict method is used to solve the diffusion iteration:

$$ ( I - \delta t \mu \nabla^2)w_3 =  w_2 $$

, this equation is solved as a sparse linear system, often with Conjugate Gradient (CG) or Multigrid methods, which is noconditional stable.

### Projection step

According to above statements, we get the projection step:

$$ \nabla^2 p = \nabla \cdot w_3, w_4 = w_3 - \nabla p $$

, which is the projection step, and here $w_4$ is the current velocity field.