---
title: "Powell’s Derivative-Free Optimization Methods: From Conjugate Directions to Trust Regions"
date: 2025-08-26 12:00:00 +0200
categories: [Blog Post, Optimization]
tags: [derivative-free-optimization, powell, trust-region, conjugate-directions, quadratic-models]
description: A look at Michael J. D. Powell’s pioneering work in derivative-free optimization, from his early conjugate direction methods in the 1960s to his model-based trust-region algorithms like COBYLA, NEWUOA, and BOBYQA.
math: true
toc: true
---

Powell’s work sits at the foundation of **derivative-free optimization (DFO)**: situations where we can evaluate a function but do not have (or do not trust) its derivatives. This post focuses on the two bookends of his contributions that matter most in practice: the early **direction-set** ideas that became known as *Powell’s method* and, later, the **model-based trust-region** algorithms that interpolate function values to build local surrogates. We begin with the 1964 conjugate-direction approach.

# Powell’s Conjugate Direction Method (1964)

Powell’s insight was that consistent progress is possible even without derivatives by combining a sequence of **line searches** with adaptive updates of the search directions. Starting from a base point $$ x_k $$, the method minimizes along each direction in turn:
\begin{equation}
x_{k,i} = \arg \min_{\alpha} f(x_{k,i-1} + \alpha d_i), \quad i=1,\dots,n,
\end{equation}
and then computes the **net displacement**
\begin{equation}
d_{n+1} = x_{k,n} - x_k,
\end{equation}
which is added as a new search direction after dropping the oldest one. A final line search is carried out along this direction:
\begin{equation}
x_{k+1} = \arg \min_{\alpha} f(x_{k,n} + \alpha d_{n+1}).
\end{equation}

Each cycle of \(n+1\) line searches implicitly captures curvature information, avoiding the zig-zag inefficiency of steepest descent and adapting to the geometry of the problem. Intuitively, it is like hiking in the dark: you move downhill along several directions, then look back at your path—the net displacement points more directly toward the valley floor. This balance of **exploration** (cycling through directions) and **adaptation** (adding new conjugate-like directions) made Powell’s method a landmark in derivative-free optimization and a precursor to his later trust-region algorithms such as COBYLA, NEWUOA, and BOBYQA.

---

## Python-style pseudocode (direction-set Powell)
    def powell_direction_set(f, x0, tol=1e-6, maxiter=200):
        """
        Derivative-free Powell (1964) direction-set method (pseudocode).
        - f: objective function R^n -> R
        - x0: initial numpy-like vector
        Returns: x_best, f(x_best)
        """
        import numpy as np

        n = len(x0)
        x = x0.copy()
        # Initial direction set: coordinate axes
        D = [np.eye(n)[:, i] for i in range(n)]

        def line_search(phi, a0=0.0):
            """
            1D derivative-free line search along alpha -> phi(alpha).
            Use any bracketing + golden/Brent routine. Placeholder here.
            """
            # pseudo-implementation: return argmin_alpha phi(alpha)
            alpha_star = brent_minimize(phi)  # assume available
            return alpha_star

        for k in range(maxiter):
            x_start = x.copy()
            f_start = f(x_start)

            # 1) Successive line minimizations along current directions
            for i in range(n):
                d = D[i]
                phi = lambda alpha: f(x + alpha * d)
                alpha_i = line_search(phi)
                x = x + alpha_i * d

            # 2) Net displacement direction
            d_new = x - x_start

            # If no progress, stop
            if np.linalg.norm(d_new) < tol * (1.0 + np.linalg.norm(x_start)):
                break

            # 3) Drop oldest direction, append new one
            D = D[1:] + [d_new]

            # 4) Extra line search along new direction
            phi_new = lambda alpha: f(x + alpha * d_new)
            alpha_new = line_search(phi_new)
            x = x + alpha_new * d_new

            # Convergence check (function or step)
            if abs(f_start - f(x)) < tol and np.linalg.norm(x - x_start) < tol:
                break

        return x, f(x)
 