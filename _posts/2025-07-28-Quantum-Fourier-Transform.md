---
layout: article
title: Quantum Computing - Quantum Fourier Transform (QFT)
mathjax: true
pageview: true
show_subscribe: true
chart: true
mermaid: true
lightbox: true
theme: dark
sharing: false
---

# Quantum Fourier Transform

Quantum Fourier Transform (QFT) is a key subroutine in many quantum algorithms such as Shor’s algorithm. It is the quantum analogue of the classical discrete Fourier transform, but it operates on the amplitudes of a quantum state in superposition.

The formal definition of QFT is:

$$
QFT(|j\rangle) = \frac{1}{\sqrt{N}} \sum_{k=0}^{N-1} e^{\frac{2 \pi i j k}{N}} |k\rangle
$$

## Tensor Product Representation

Let us derive a tensor product form of the QFT:

$$
\begin{aligned}
QFT(|j\rangle) 
&= \frac{1}{\sqrt{N}} \sum_{k=0}^{N-1} e^{\frac{2 \pi i j k}{N}} |k\rangle \\
&= \frac{1}{\sqrt{2^n}} \sum_{k_1=0}^{1} \cdots \sum_{k_n=0}^{1} e^{\frac{2\pi i j \sum_{l=1}^{n} k_l 2^{n-l}}} |k_1 k_2 \cdots k_n\rangle \\
&= \frac{1}{\sqrt{2^n}} \sum_{k_1=0}^{1} \cdots \sum_{k_n=0}^{1} \bigotimes_{l=1}^{n} e^{2\pi i j k_l 2^{-l}} |k_l\rangle \\
&= \frac{1}{\sqrt{2^n}} \bigotimes_{l=1}^{n} \sum_{k_l=0}^{1} e^{2\pi i j k_l 2^{-l}} |k_l\rangle \\
&= \frac{1}{\sqrt{2^n}} \bigotimes_{l=1}^{n} \left( |0\rangle + e^{2\pi i j 2^{-l}} |1\rangle \right)
\end{aligned}
$$

This derivation is based on the relationship between binary and decimal representations:

- For integers: \( |j\rangle = |\overline{j_1 j_2 \cdots j_n}\rangle \), where \( j = \sum_{l=1}^{n} 2^{n-l} j_l \)
- For binary fractions: \( \overline{0.j_1 j_2 \cdots j_n} = \sum_{l=1}^{n} 2^{-l} j_l \)

## Simple Examples

### 1. Single-Qubit Fourier Transform

$$
|\psi\rangle = \alpha|0\rangle + \beta|1\rangle
$$

Then,

$$
U_{QFT}|\psi\rangle = \frac{1}{\sqrt{2}}(\alpha+\beta)|0\rangle + \frac{1}{\sqrt{2}}(\alpha - \beta)|1\rangle = H|\psi\rangle
$$

Which is exactly the Hadamard transform.

### 2. Two-Qubit Fourier Transform

$$
\begin{aligned}
U_{QFT}|10\rangle 
&= \frac{1}{\sqrt{4}} (|0\rangle + e^{2\pi i \cdot 2 / 2}|1\rangle)(|0\rangle + e^{2\pi i \cdot 2 / 4}|1\rangle) \\
&= \frac{1}{2} (|0\rangle + e^{2\pi i}|1\rangle)(|0\rangle + e^{\pi i}|1\rangle) \\
&= \frac{1}{2} (|0\rangle + |1\rangle)(|0\rangle - |1\rangle) \\
&= \frac{1}{2}|00\rangle - \frac{1}{2}|01\rangle + \frac{1}{2}|10\rangle - \frac{1}{2}|11\rangle
\end{aligned}
$$

## Constructing the Quantum Circuit

By observing the tensor product expression, we can construct the QFT circuit step by step.

### Tensor Form

$$
QFT(|j\rangle) = \frac{1}{\sqrt{2^n}} (|0\rangle + e^{2\pi i \cdot 0 \cdot j_n}|1\rangle)(|0\rangle + e^{2\pi i \cdot 0 \cdot j_{n-1} j_n}|1\rangle) \cdots (|0\rangle + e^{2\pi i \cdot 0 \cdot j_1 j_2 \cdots j_n}|1\rangle)
$$

### With Hadamard Gates

Introducing Hadamard gates, we rewrite as:

$$
QFT(|j\rangle) = \frac{1}{\sqrt{2^n}} (H |j_n\rangle) (|0\rangle + e^{2\pi i j_n 2^{-2}} e^{2\pi i \cdot 0 \cdot j_{n-1}}|1\rangle) \cdots (|0\rangle + \prod_{l=2}^{n} e^{2\pi i j_{n+2-l} 2^{-l}}|1\rangle)
$$

### With Controlled Rotation Gates

We introduce controlled phase rotation gates:

$$
R_l = 
\begin{bmatrix}
1 & 0 \\
0 & e^{2\pi i / 2^l}
\end{bmatrix}, \quad l=2,3,\ldots,n
$$

Why controlled? Because the rotation only applies when the control qubit (like \(j_{n+2-l}\)) is \(1\), i.e., not zero.

### Final Tensor Expression of QFT Circuit

$$
QFT(|j\rangle) = (H|j_n\rangle) \otimes (R_2 H|j_{n-1}\rangle) \otimes \cdots \otimes (R_n \cdots R_2 H|j_1\rangle)
$$

This results in the well-known QFT circuit composed of:

- A Hadamard gate on each qubit
- A series of controlled-phase rotations
- Optional final swap gates to reverse qubit order (not shown here)

## Code Implementation

You can find my working Qiskit example of Quantum Fourier Transform at:

[GitHub Repository - Qiskit Code Example](https://github.com/WangZW928/Qiskit-code-examples/blob/master/code-3.ipynb)

---
