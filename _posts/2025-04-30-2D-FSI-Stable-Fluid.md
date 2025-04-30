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
See [Paper](https://kitian616.github.io/jekyll-TeXt-theme/samples.html#page-layout) for details.
In this project, I this method is made by FDM in 2D.

##  Theory

The Navier-Stokes equation is a partial differential equation, which is a partial derivative of the velocity field.
$$ \nabla \cdot \vec{u} = 0 $$
$$ \frac{\partial u}{\partial t} + \frac{\partial (u \cdot u)}{\partial x} + \frac{\partial (v \cdot u)}{\partial y} = -\frac{\partial p}{\partial x} + \nu \left( \frac{\partial^2 u}{\partial x^2} + \frac{\partial^2 u}{\partial y^2} \right) $$
