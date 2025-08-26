---
title: "Line Search vs Trust Region: General Theory"
date: 2025-08-26 12:00:00 +0200
categories: [Blog Post, Optimization]
tags: [line-search, trust-region, optimization, gradient-methods]
description: A gentle theoretical overview of line search and trust region methods for nonlinear optimization.
math: true
toc: true
---

In smooth nonlinear optimization, two major frameworks dominate the design of iterative algorithms: **line search** and **trust region**. Both aim to find a local minimizer of a function

$$
\min_{x \in \mathbb{R}^n} f(x)
$$

given that we can compute function values and gradients (sometimes Hessians).

---

## 1. Common Iterative Template

We update from a current iterate $$x_k$$ to a new iterate:

$$
x_{k+1} = x_k + s_k
$$

where $$s_k$$ is a *step* chosen to reduce $$f$$. The two frameworks differ in how they choose this step.

---

## 2. Line Search Methods

Line search methods pick a **search direction** $$d_k$$ (often gradient descent direction or a quasi-Newton direction) and then select a **step length** $$\alpha_k > 0$$.

**General form:**

$$
x_{k+1} = x_k + \alpha_k d_k
$$

### Search direction
- Steepest descent: $$d_k = -\nabla f(x_k)$$
- Newton / quasi-Newton: $$d_k = -B_k^{-1} \nabla f(x_k)$$

### Step length
Chosen to ensure sufficient decrease in $$f$$:
- **Exact line search**: minimize $$\phi(\alpha) = f(x_k + \alpha d_k)$$ exactly (rare in practice).
- **Inexact line search**: enforce conditions such as the **Armijo condition**:

$$
f(x_k + \alpha d_k) \le f(x_k) + c_1 \alpha \nabla f(x_k)^T d_k,
$$

with $$0 < c_1 < 1$$, ensuring enough decrease.

Often combined with the **Wolfe conditions** or **strong Wolfe conditions** to balance sufficient decrease with step length not being too small.

---

## 3. Trust Region Methods

Instead of moving along a fixed direction, trust region methods build a **local model** of $$f$$ near $$x_k$$ and restrict steps to a region where the model is trusted.

### Model
A common quadratic model:

$$
m_k(s) = f(x_k) + \nabla f(x_k)^T s + \tfrac{1}{2} s^T B_k s
$$

where $$B_k$$ approximates the Hessian.

### Subproblem
We solve:

$$
\min_{s \in \mathbb{R}^n} m_k(s) \quad \text{subject to} \quad \|s\| \le \Delta_k
$$

where $$\Delta_k > 0$$ is the **trust region radius**.

- If the step produces sufficient reduction in $$f$$, accept it and possibly enlarge $$\Delta_k$$.
- If not, reject or shrink $$\Delta_k$$ and resolve.

---

## 4. Comparison

| Aspect             | Line Search                                    | Trust Region                                  |
|--------------------|-----------------------------------------------|-----------------------------------------------|
| Step choice        | Direction $$d_k$$ + step length $$\alpha_k$$  | Any $$s_k$$ inside ball $$\|s\| \le \Delta_k$$ |
| Global convergence | Needs conditions (e.g., Wolfe)                 | Natural via ratio test $$\rho_k$$             |
| Robustness         | Sensitive to step length choice                | More robust when models are good              |
| Cost per iter      | Cheaper (1D search)                            | Harder (quadratic subproblem)                 |
| When used          | Large-scale, cheap gradient evaluations        | Expensive problems, need strong robustness    |

---

## 5. Key Insights

- **Line search** is simple and effective when gradients are cheap and the function is well-behaved.
- **Trust region** is more reliable when models are uncertain or Hessians are indefinite.
- Modern solvers often combine both (e.g., trust-region Newton with line search fallback).

---

## 6. References (classic texts)

- Nocedal & Wright, *Numerical Optimization* (2nd ed., 2006).
- Conn, Gould & Toint, *Trust-Region Methods* (2000).

---

📌 **Categories**: This article sits under `[Blog Post, Optimization]`.  
For code-heavy implementations, you can use `[Code Project, Optimization]` instead.