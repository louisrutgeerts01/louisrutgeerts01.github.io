---
title: "Nelder–Mead in a Nutshell (with Equations)"
date: 2025-08-26 14:00:00 +0200
categories: [Optimization, Derivative-free]
tags: [nelder-mead, simplex, unconstrained, derivative-free]
description: A quick, math-first tour of the Nelder–Mead simplex method: reflection, expansion, contraction, and shrink steps—when to use it and when it struggles.
math: true
pin: false
toc: true
---

**Goal.** Minimize a black-box objective \( f:\mathbb{R}^n\to\mathbb{R} \) *without gradients* by iteratively moving a simplex.

---

## Algorithm (core steps)

We keep a simplex with \(n+1\) vertices \( \{x_1,\dots,x_{n+1}\} \) and values \( f_i=f(x_i) \). Each iteration:

**1) Order the vertices**

$$
f(x_1) \le f(x_2) \le \cdots \le f(x_{n+1}) .
$$

Best \(x_b=x_1\), second worst \(x_w=x_n\), worst \(x_h=x_{n+1}\).

**2) Centroid (excluding the worst)**

$$
x_c \;=\; \frac{1}{n}\sum_{i=1}^{n} x_i .
$$

**3) Reflection**

$$
x_r \;=\; x_c + \alpha\,(x_c - x_h), \qquad \alpha = 1.
$$

If \( f(x_b) \le f(x_r) < f(x_w) \), accept \(x_h \leftarrow x_r\).

**4) Expansion** (if reflection is excellent)

$$
x_e \;=\; x_c + \gamma\,(x_r - x_c), \qquad \gamma = 2.
$$

If \( f(x_e) < f(x_r) \), set \(x_h \leftarrow x_e\); else accept \(x_r\).

**5) Contraction** (if reflection is not good)

Outside (when \( f(x_b) \le f(x_r) < f(x_h) \)):

$$
x_{oc} \;=\; x_c + \beta\,(x_r - x_c), \qquad \beta = \tfrac{1}{2}.
$$

Inside (when \( f(x_r) \ge f(x_h) \)):

$$
x_{ic} \;=\; x_c - \beta\,(x_r - x_c) \;=\; x_c + \beta\,(x_h - x_c).
$$

Accept the contracted point if it improves on \(x_h\); otherwise **shrink**.

**6) Shrink**

$$
x_i \leftarrow x_b + \delta\,(x_i - x_b), \quad i=2,\dots,n+1, 
\qquad \delta = \tfrac{1}{2}.
$$

---

## Stopping criteria (common)

Function spread small:

$$
\sqrt{\frac{1}{n+1}\sum_{i=1}^{n+1}\big(f(x_i)-\bar f\big)^2} \;\le\; \varepsilon_f,
\qquad
\bar f \;=\; \frac{1}{n+1}\sum_{i=1}^{n+1} f(x_i).
$$

Or simplex size small:

$$
\max_{i=1,\dots,n+1} \lVert x_i - x_b \rVert \;\le\; \varepsilon_x .
$$

---

## When it shines (and when it struggles)

- ✅ **Low dimensions** (\(n \lesssim 10\)), **noisy** or **nonsmooth** objectives, quick prototyping.
- ⚠️ Can stall in **ill-conditioned** valleys (e.g., Rosenbrock) and scale poorly with dimension. Pre-scaling variables and restarts help.

---

### Quick start (SciPy)

```python
from scipy.optimize import minimize

res = minimize(f, x0, method="Nelder-Mead",
               options={"xatol": 1e-6, "fatol": 1e-6, "maxfev": 2000})
x_star, f_star = res.x, res.fun