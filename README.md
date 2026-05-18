# Using Smoothing Functions to Obtain Guarantees on the Tightness of Global Solutions in Non-Convex Optimization

**Authors:** Chen Zakaim, Aharon Ben-Tal — Technion, Israel Institute of Technology

---

## Overview

> **Note:** This repository provides a high-level overview and selected results from the thesis. The full thesis, including detailed proofs and extended numerical experiments, is in preparation.

This work presents a new method for solving the **sum-of-max problem**, a class of non-convex optimization problems known to be **NP-hard**:

$$\max_{x \in \mathcal{X}} F(x) := \sum_{t=1}^{T} \max_{k \in [K]} \{ f_{t,k}(x) \}$$

where $\mathcal{X}$ is a compact convex set and each $f_{t,k} \in C^2$ is convex. The key challenge is that $F(x)$ is non-differentiable, which prevents direct application of standard smooth optimization algorithms.

The sum-of-max problem is NP-hard because it can encode combinatorial problems such as the maximum clique problem [3]. Intuitively, with $T$ terms and $K$ options each, there are $K^T$ possible combinations of which function "wins" the max in each term — each combination defines a different smooth region with its own local optimum, leading to an exponential number of local optima that cannot be efficiently explored.

---

## Background: The COMAX and MDFC Algorithms

**COMAX** is an algorithm for maximizing a convex objective over a convex feasible set. It operates in two phases:

- **Phase 1 – Finding a good starting point:** Computes a second-order Taylor expansion of the objective around its minimizer, then approximates the feasible set with inscribed and circumscribing ellipsoids. A line-search step finds a feasible interior point.
- **Phase 2 – Convergence:** Applies gradient ascent starting from the point found in Phase 1.

**MDFC** is an extension of COMAX that handles objectives of the form $f(x) - g(x)$ (Difference-of-Convex, or D.C.), where both $f$, $g$, and the constraint functions are convex and $C^2$.

---

## The Smoothing Approach

Since $F(x) \notin C^2$, COMAX cannot be applied directly. To overcome the non-differentiability of the max operator, we employ a pair of smooth surrogate functions that serve as **certified upper and lower bounds** on the true objective.

The surrogates are parameterized by a smoothing parameter $\rho > 0$ — larger values yield tighter approximations, and both converge to the true max as $\rho \to \infty$. A key property is that the upper-bound surrogate **preserves convexity** when the underlying functions are convex, making it directly compatible with COMAX.

By running the algorithm on the upper bound and evaluating the lower bound at the solution, we obtain a **tightness certificate** on solution quality — without enumerating all possible function combinations.

---

## Weakly Convex Functions and D.C. Decomposition

For objectives that are not convex, a **weakly convex** reformulation is used. Given a $C^2$ function $f$ with weak convexity parameter $\lambda$, define $g(x) = \frac{\lambda}{2} \|x\|^2$. Then:

$$f(x) = \underbrace{[f(x) + g(x)]}_{\text{convex}} - g(x)$$

This D.C. decomposition allows MDFC to be applied even to non-convex objectives, enabling global maximum search beyond the standard convex setting.

---

## Numerical Results

### Example 1 — Sum-of-Max (n = 150 dimensions, T = 4 terms, K = 2)

We consider a sum-of-max objective of the form:

$$F(x) = \max(f_{11}(x), f_{12}(x)) + \max(f_{21}(x), f_{22}(x)) + \max(f_{31}(x), f_{32}(x)) + \max(f_{41}(x), f_{42}(x))$$

where $x \in \mathbb{R}^{150}$ and each $f_{t,k}$ is a convex $C^2$ function. A naïve approach would require running COMAX separately for all $2^4 = 16$ possible function combinations to find the global maximum. Instead, the smoothing approach runs the algorithm **once** and verifies the result once, with the upper and lower bound surrogates at the solution being nearly identical — certifying near-optimality without enumeration.

| Best combination found | Objective value |
|------------------------|-----------------|
| $f_{12} + f_{21} + f_{32} + f_{42}$ | 86,770.93 |

### Example 2 — Non-Convex Quadratic Optimization over the Simplex

We consider a standard quadratic optimization problem from Bomze [3]: maximizing a non-convex quadratic objective $B(x) := x^\top Q x$ over the standard simplex $\Delta \subset \mathbb{R}^5$, where $Q$ is an indefinite symmetric matrix. Since the problem is non-convex, standard methods risk getting trapped at local maxima.

By reformulating it as a D.C. program and applying MDFC, the method successfully recovers the known global optimum:

$$\max_{x \in \Delta} \ x^\top Q x = 16.3333$$

This demonstrates the framework's ability to handle genuinely non-convex problems beyond the sum-of-max structure.

### Example 3 — MLE for Laplace Distribution

Given $N = 40$ i.i.d. observations $x_1, \ldots, x_{40}$ from a Laplace distribution with PDF:

$$f(x \mid \mu, b) = \frac{1}{2b} \exp\left( -\frac{|x - \mu|}{b} \right)$$

the log-likelihood is:

$$\ell(\mu, b) = \sum_{i=1}^{40} \left[ -\log(2b) - \frac{|x_i - \mu|}{b} \right]$$

Since $|x_i - \mu| = \max(x_i - \mu,\ \mu - x_i)$, this can be rewritten as a sum-of-max problem:

$$\ell(\mu, b) = -\sum_{i=1}^{40} \max\left( \log(2b) - \frac{x_i - \mu}{b},\ \log(2b) + \frac{x_i - \mu}{b} \right)$$

which fits directly into our framework. Applying the D.C. / MDFC approach yields:

| Parameter | Optimal | Recovered |
|-----------|---------|-----------|
| μ | 8.1125 | 8.1100 |
| b | 1.6549 | 1.6413 |
| Objective | −87.8753 | −87.9276 |

The recovered parameters are very close to the true optimum, demonstrating practical effectiveness.

---

## Benchmark Results

The algorithm was tested against known global optima from the optimization literature. In every tested instance, the method matched or closely approximated the known optimal value.

| Problem | Source | n | Known Optimum | Our Result | Match |
|---------|--------|---|---------------|------------|-------|
| Sum-of-Max | Thesis | 150 | 86,770.93 | 86,770.93 | ✓ |
| Quadratic over Simplex | Bomze [3] | 5 | 16.3333 | 16.3333 | ✓ |
| Laplace MLE | Thesis | 2 | −87.8753 | −87.9276 | ≈ |

**Note on scalability:** The examples above use $n = 150$ dimensions for demonstration purposes. The algorithm scales to higher dimensions — all experiments were run on a MacBook Air M1 (CPU only). The computational bottleneck is $O(n^3)$ per iteration, dominated by the Hessian computation and the hidden convexity subproblem.

---

## Summary

| Component | Role |
|-----------|------|
| Smooth upper bound surrogate | Convex, differentiable approximation of the max — enables COMAX |
| Smooth lower bound surrogate | Provides a certificate on solution quality |
| COMAX / MDFC | Underlying solver for smooth convex / D.C. problems |
| Weak convexity | Enables D.C. decomposition of non-convex objectives |

The combined framework provides a principled, efficient way to tackle NP-hard sum-of-max problems — as well as broader non-convex programs — with **certified solution quality**.

---

## References

1. **Ben-Tal, A. & Roos, E.** (2022). *An Algorithm for Maximizing a Convex Function Based on Its Minimum.* INFORMS Journal on Computing, 34(6), 3200–3214. [DOI](https://doi.org/10.1287/ijoc.2022.1238)

2. **Ben-Tal, A. & Tetruashvili, L.** (2025). *New Algorithms for Maximizing the Difference of Convex Functions.* Optimization Online. [PDF](https://optimization-online.org/wp-content/uploads/2025/04/comaxdc1.pdf)

3. **Bomze, I.M., de Klerk, E.** (2002). *Solving Standard Quadratic Optimization Problems via Linear, Semidefinite and Copositive Programming.* Journal of Global Optimization, 24, 163–185. [DOI](https://doi.org/10.1023/A:1020209017701)
