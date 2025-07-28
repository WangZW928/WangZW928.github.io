---
layout: article
title: Implicit FVM Jacobian Assembly (Cell-Based Loop)
mathjax: true
pageview: true
show_subscribe: true
chart: true
mermaid: true
lightbox: true
theme: dark
sharing: false
---

# Implicit Finite Volume Method (FVM): Jacobian Assembly with Cell-Based Loop

In implicit CFD solvers using Finite Volume Method (FVM), residual and Jacobian assembly strategies vary depending on whether we loop over **faces** or **cells**. This note documents how Jacobians are assembled when **cells** are the primary loop object.

## Residual Construction

For a given cell $i_0$, the total residual is the sum of fluxes across all faces shared with neighboring cells:

$$
\mathbf{R}_{i_0} = \sum_{j \in \{1, 2, 3\}} \mathbf{F}_{i_0,i_j}(\mathbf{U}_0, \mathbf{U}_j)
$$

Here, $\mathbf{F}_{i_0,i_j}$ is the numerical flux from cell $i_0$ to its neighboring cell $i_j$.

## Flux Derivative Symmetry (Face Loop Only)

Assuming $\mathbf{F}_{ij}$ is positive from $i$ to $j$, then for the opposing direction (from $j$ to $i$), the flux is negative. By the chain rule:

$$
\frac{\partial \mathbf{R}_{ji}}{\partial \mathbf{U}_i} = -\frac{\partial \mathbf{R}_{ij}}{\partial \mathbf{U}_i}, \quad
\frac{\partial \mathbf{R}_{ji}}{\partial \mathbf{U}_j} = -\frac{\partial \mathbf{R}_{ij}}{\partial \mathbf{U}_j}
$$

These symmetric relations hold **only** when looping over faces. In a **cell-based loop**, each face is processed **twice** (once for each adjacent cell), so the symmetry cannot be directly used.

## Implicit Euler Time Integration

Using first-order implicit Euler:

$$
\frac{V_0}{\Delta t} \Delta \mathbf{U}_0 = -\mathbf{R}^{n+1}_{0}
= \sum_{j \in \{1,2,3\}} -\mathbf{F}_{i_0,i_j}(\mathbf{U}_0^n + \Delta \mathbf{U}_0,\mathbf{U}_j^n + \Delta \mathbf{U}_j)
$$

Expanding the RHS with a first-order Taylor expansion (ignoring second-order terms):

$$
\mathbf{R}^{n+1}_{0} \approx \mathbf{R}^n_{0} +
\frac{\partial \mathbf{R}_{0}}{\partial \mathbf{U}_0} \Delta \mathbf{U}_0 +
\sum_{j \in \{1,2,3\}} \frac{\partial \mathbf{F}_{i_0,i_j}}{\partial \mathbf{U}_j} \Delta \mathbf{U}_j
$$

So the full implicit system becomes:

$$
\left(\frac{V_0}{\Delta t} + \frac{\partial \mathbf{R}_{0}}{\partial \mathbf{U}_0} \right)\Delta \mathbf{U}_0
+ \sum_{j \in \{1,2,3\}} \frac{\partial \mathbf{F}_{i_0,i_j}}{\partial \mathbf{U}_j} \Delta \mathbf{U}_j = -\mathbf{R}^{n}_{0}
$$

## Jacobian Matrix Structure

This corresponds to one **row** in the global Jacobian matrix:

$$
\begin{bmatrix}
\left(\frac{V_0}{\Delta t} + \frac{\partial \mathbf{R}_{0}}{\partial \mathbf{U}_0} \right)
& \frac{\partial \mathbf{F}_{i_0,i_1}}{\partial \mathbf{U}_1}
& \frac{\partial \mathbf{F}_{i_0,i_2}}{\partial \mathbf{U}_2}
& \frac{\partial \mathbf{F}_{i_0,i_3}}{\partial \mathbf{U}_3}
\end{bmatrix}
\begin{bmatrix}
\Delta \mathbf{U}_0 \\
\Delta \mathbf{U}_1 \\
\Delta \mathbf{U}_2 \\
\Delta \mathbf{U}_3
\end{bmatrix}
=
-\mathbf{R}^{n}_{0}
$$

Each row is constructed independently per cell — there is no interference between the Jacobian rows from different cells.

## Local Assembly Logic

For each cell, we assemble the local Jacobian as follows:

- Diagonal (self term):

  $$
  \mathbf{A}_{i_0, i_0} \leftarrow \frac{V_0}{\Delta t} + \frac{\partial \mathbf{R}_0}{\partial \mathbf{U}_0}
  $$

- Off-diagonal (neighbor terms):

  $$
  \mathbf{A}_{i_0, i_j} \leftarrow \frac{\partial \mathbf{F}_{i_0, i_j}}{\partial \mathbf{U}_j}, \quad j = 1, 2, 3
  $$

This pattern forms a sparse Jacobian structure where each row corresponds to one cell and columns point to the cell and its neighbors.

## Summary

- Residual $\mathbf{R}_i$ for each cell is assembled from fluxes across its faces.
- When looping over cells, each face is processed twice — symmetry in flux derivatives is not directly applicable.
- Jacobian is built one row at a time, corresponding to one cell.
- This formulation is friendly for parallel computation and block-sparse matrix storage.

