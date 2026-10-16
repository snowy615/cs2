# Machine Learning · Part 3: Optimisation

*Lectures 8–9. [← Part 2](ML-2-Generalisation-Regularisation-and-Bayes.md) · [Overview](ML-Summary.md) · [Part 4 →](ML-4-Classification.md) · [Formula sheet](ML-7-Formula-Sheet.md)*

Most ML problems have **no closed-form solution** (lasso, absolute loss, logistic regression, neural networks). Two routes: (a) express the problem as a standard **convex program** (e.g. a linear program) and hand it to a black-box solver; (b) use **gradient-based** iterative methods, which need care with hyperparameters such as the learning rate.

---

## Contents
10. [Convex sets, convex functions, convex programs](#10-convex-sets-convex-functions-convex-programs)
11. [Absolute loss as a linear program](#11-absolute-loss-as-a-linear-program)
12. [Gradients, Hessians, gradient descent](#12-gradients-hessians-gradient-descent)
13. [Subgradients](#13-subgradients)
14. [Newton's method](#14-newtons-method)
15. [SGD, minibatches, constrained problems](#15-sgd-minibatches-constrained-problems)

---

## 10. Convex sets, convex functions, convex programs

> [!NOTE]
> **📖 Key definitions**
> - **Convex set** $`C \subseteq \mathbb{R}^D`$: for all $`\mathbf{x}, \mathbf{y} \in C`$ and $`\lambda \in [0,1]`$, $`\lambda\mathbf{x} + (1-\lambda)\mathbf{y} \in C`$. *The line segment between any two points stays inside.*
> - **Convex function** $`f`$ (on a convex domain): for all $`\mathbf{x}, \mathbf{y}`$ and $`\lambda \in [0,1]`$,
> ```math
> f(\lambda\mathbf{x} + (1-\lambda)\mathbf{y}) \le \lambda f(\mathbf{x}) + (1-\lambda)f(\mathbf{y}).
> ```
> *The chord lies above the graph; bowl-shaped.*
> - **Positive semi-definite (PSD)** matrix: $`\mathbf{A} = \mathbf{A}^\top`$ and $`\mathbf{x}^\top\mathbf{A}\mathbf{x} \ge 0`$ for all $`\mathbf{x}`$ (equivalently all eigenvalues $`\ge 0`$). The set $`\mathbb{S}^D_+`$ of PSD matrices is a convex cone.
> - **Convex optimisation problem:** minimise convex $`f(\mathbf{x})`$ subject to $`g_i(\mathbf{x}) \le 0`$ ($`g_i`$ convex) and $`h_j(\mathbf{x}) = 0`$ ($`h_j`$ affine). The feasible set is convex.
> - **Optimal value** $`v^* = \min\{f(\mathbf{x}) : \text{constraints}\}`$ (may be $`+\infty`$ if infeasible, $`-\infty`$ if unbounded). **Globally optimal point:** $`f(\mathbf{x}^*) = v^*`$ (need not be unique). **Locally optimal:** feasible and $`f(\mathbf{x}) \le f(\mathbf{y})`$ for all feasible $`\mathbf{y}`$ within some ball of radius $`B > 0`$.

### 10.1 Catalogue of convex sets (each proved straight from the definition)
| Set | Why convex |
|---|---|
| $`\mathbb{R}^D`$ | trivially |
| Intersection $`\bigcap_i C_i`$ of convex sets | the segment lies in every $`C_i`$ |
| $`L^p`$ norm ball $`\{\mathbf{x} : \lVert\mathbf{x}\rVert \le 1\}`$ | $`\lVert\lambda\mathbf{x} + (1-\lambda)\mathbf{y}\rVert \le \lambda\lVert\mathbf{x}\rVert + (1-\lambda)\lVert\mathbf{y}\rVert \le 1`$ (triangle inequality) |
| Polyhedron $`\{\mathbf{x} : \mathbf{A}\mathbf{x} \le \mathbf{b}\}`$ | $`\mathbf{A}(\lambda\mathbf{x} + (1-\lambda)\mathbf{y}) \le \lambda\mathbf{b} + (1-\lambda)\mathbf{b} = \mathbf{b}`$ |
| PSD cone $`\mathbb{S}^D_+`$ | $`\mathbf{x}^\top(\lambda\mathbf{A} + (1-\lambda)\mathbf{B})\mathbf{x} = \lambda\mathbf{x}^\top\mathbf{A}\mathbf{x} + (1-\lambda)\mathbf{x}^\top\mathbf{B}\mathbf{x} \ge 0`$ |

### 10.2 Catalogue of convex functions (your toolbox)
| Function | Why convex |
|---|---|
| Affine $`f(\mathbf{x}) = \mathbf{b}^\top\mathbf{x} + c`$ | equality holds in the definition (expand both sides) |
| Quadratic $`f(\mathbf{x}) = \tfrac12\mathbf{x}^\top\mathbf{A}\mathbf{x} + \mathbf{b}^\top\mathbf{x} + c`$ with $`\mathbf{A}`$ symmetric PSD | Hessian $`\nabla^2 f = \mathbf{A}`$ has non-negative eigenvalues |
| Any norm $`f(\mathbf{x}) = \lVert\mathbf{x}\rVert`$ | triangle inequality + homogeneity, as for the norm ball |
| Non-negative weighted sum $`\sum_i w_if_i`$, $`w_i > 0`$, $`f_i`$ convex | add the inequalities |

**Toolbox use:** to show a loss is convex, decompose it into a non-negative sum of these pieces. E.g. ridge = quadratic (PSD $`\mathbf{X}^\top\mathbf{X}`$) + $`\lambda\lVert\mathbf{w}\rVert_2^2`$; lasso = quadratic + $`\lambda\lVert\mathbf{w}\rVert_1`$; absolute loss = sum of norms of affine functions. **All losses in this course are convex, except neural networks.** (Why we care: a loss should be *useful*, small when the prediction is close, e.g. predicting a price to within a few pence, **and** easy to optimise, i.e. convex. Deep learning breaks with this and settles for local minima.)

### 10.3 The key theorem
> **Theorem 1.** In a convex optimisation problem, **every locally optimal point is globally optimal.**

*Proof (by contradiction).* Suppose $`\mathbf{x}`$ is locally optimal (with radius $`B`$) but some feasible $`\mathbf{y} \ne \mathbf{x}`$ has $`f(\mathbf{y}) < f(\mathbf{x})`$. Build a point on the segment between them, very close to $`\mathbf{x}`$:
```math
\mathbf{z} = \lambda\mathbf{y} + (1-\lambda)\mathbf{x}, \qquad \lambda = \frac{B}{2\lVert\mathbf{x}-\mathbf{y}\rVert_2} \in (0, 1] \ (\text{shrink } B \text{ if needed}).
```
Then $`\mathbf{z}`$ is feasible (convex feasible set) and $`\lVert\mathbf{x} - \mathbf{z}\rVert_2 = \lVert\lambda(\mathbf{x}-\mathbf{y})\rVert_2 = B/2 \le B`$, so local optimality says $`f(\mathbf{x}) \le f(\mathbf{z})`$. But convexity of $`f`$ gives
```math
f(\mathbf{z}) \le \lambda f(\mathbf{y}) + (1-\lambda)f(\mathbf{x}) < \lambda f(\mathbf{x}) + (1-\lambda)f(\mathbf{x}) = f(\mathbf{x}),
```
a contradiction. ∎

**Why this matters:** gradient descent only finds local minima; for convex problems that's the global minimum, from any starting point. For **non-convex** problems (neural nets) nearby starts can end at different points with wildly different values (Fig. 4).

### 10.4 Standard classes of convex programs
- **Linear program (LP):** minimise $`\mathbf{c}^\top\mathbf{x} + d`$ subject to $`\mathbf{A}\mathbf{x} \le \mathbf{e}`$, $`\mathbf{B}\mathbf{x} = \mathbf{f}`$ (component-wise). No closed form, but polynomial-time algorithms and solvers that handle tens of thousands of variables. If an optimum exists, one is at a **vertex** of the feasible polytope (Fig. 1: the objective direction $`\mathbf{c}`$ and the optimal vertex; in degenerate cases a whole face is optimal). Equality constraints are redundant in principle ($`\mathbf{B}\mathbf{x} = \mathbf{f}`$ ⇔ $`\mathbf{B}\mathbf{x} \le \mathbf{f}`$ and $`-\mathbf{B}\mathbf{x} \le -\mathbf{f}`$), but how a solver handles equalities and non-negativity can change running time a lot.
- **Quadratically constrained quadratic program (QCQP):** quadratic objective and constraints (generalises LP). SVMs are quadratic programs (Part 4).
- **Semidefinite program (SDP):** minimise $`\mathrm{tr}(\mathbf{C}\mathbf{X})`$ subject to $`\mathrm{tr}(\mathbf{A}_i\mathbf{X}) = b_i`$, $`\mathbf{X}`$ PSD ($`\mathbf{C}, \mathbf{A}_i`$ symmetric). (Generalises QCQP; just be aware it exists.)

---

## 11. Absolute loss as a linear program
Objective $`\mathcal{L}(\mathbf{w}) = \sum_{i=1}^N\lvert\mathbf{x}_i^\top\mathbf{w} - y_i\rvert`$. Absolute values aren't linear, so introduce **slack variables** $`\zeta_1, \dots, \zeta_N`$ (one per data point) and solve the LP in $`D + N`$ variables:
```math
\text{minimise } \sum_{i=1}^N\zeta_i \quad\text{subject to}\quad \mathbf{w}^\top\mathbf{x}_i - y_i \le \zeta_i,\quad y_i - \mathbf{w}^\top\mathbf{x}_i \le \zeta_i \qquad (i = 1,\dots,N).
```
**Why this is correct.** The two constraints say $`\zeta_i \ge \lvert\mathbf{w}^\top\mathbf{x}_i - y_i\rvert`$ (so also $`\zeta_i \ge 0`$). Feasible solutions exist (take any $`\mathbf{w}`$ and $`\zeta_i = \lvert\mathbf{w}^\top\mathbf{x}_i - y_i\rvert`$). At the optimum $`(\mathbf{w}^*, \boldsymbol\zeta^*)`$, since we minimise $`\sum\zeta_i`$, each $`\zeta_i^*`$ is pushed down to equality $`\zeta_i^* = \lvert\mathbf{w}^{*\top}\mathbf{x}_i - y_i\rvert`$; hence the LP objective equals $`\mathcal{L}(\mathbf{w}^*)`$ and $`\mathbf{w}^*`$ minimises it. ∎

**Limits of the trick:** the lasso $`\sum_i(\mathbf{w}^\top\mathbf{x}_i - y_i)^2 + \lambda\sum_i\lvert w_i\rvert`$ has a **quadratic** part that can't be written with linear constraints, so we need gradient methods.

---

## 12. Gradients, Hessians, gradient descent

> [!NOTE]
> **📖 Key definitions** (for $`f(\mathbf{w})`$, $`\mathbf{w} \in \mathbb{R}^D`$)
> - **Gradient** (column vector of first partials): $`\nabla_{\mathbf{w}}f = \big[\partial f/\partial w_1, \dots, \partial f/\partial w_D\big]^\top`$. It is **orthogonal to the contour lines** and points in the direction of **steepest increase**.
> - **Hessian** (matrix of second partials): $`\mathbf{H}_{jk} = \partial^2 f/\partial w_j\partial w_k`$; symmetric when the second derivatives exist. Captures **curvature**.
> - **Gradient descent (GD):** from $`\mathbf{w}_0`$, iterate
> ```math
> \mathbf{w}_{t+1} = \mathbf{w}_t - \eta_t\,\mathbf{g}_t, \qquad \mathbf{g}_t = \nabla f(\mathbf{w}_t).
> ```
> $`\eta_t`$ is the **step size / learning rate**.
> - **Stationary point:** $`\nabla f = \mathbf{0}`$. Classified by the Hessian's eigenvalues (real, since the Hessian is symmetric): all positive ⇒ local **minimum**; all negative ⇒ local **maximum**; mixed signs ⇒ **saddle point**; a zero eigenvalue ⇒ degenerate (flat direction).

**Pictures:** Fig. 2 uses $`f = w_1^2 + w_2^2`$: circular contours with gradients pointing radially outwards, orthogonal to the contours. Fig. 3 uses $`f(w_1, w_2) = \tfrac12(w_1^2 - w_2)^2 + \tfrac12(w_1 - 1)^2`$, a curved valley.

**Choosing the step size** (Fig. 3): too large ⇒ erratic zig-zags across the valley or divergence; too small ⇒ painfully slow (maybe never converges in practice). Options: **line search** (minimise along the gradient direction), or treat $`\eta`$ as a hyperparameter chosen by validation (the ML norm).

**Convergence:** for convex $`f`$ with a suitable step size, GD converges to a global minimiser from **any** start. For non-convex $`f`$, it may stop at a local minimum or a saddle point.

---

## 13. Subgradients
Lasso's $`\lvert w_i\rvert`$ (and the hinge loss $`\max(0, w)`$) are **not differentiable at 0**. For a **convex** $`f`$, a vector $`\mathbf{g}`$ is a **subgradient** at $`\mathbf{w}_0`$ if
```math
f(\mathbf{w}) \ge f(\mathbf{w}_0) + \mathbf{g}^\top(\mathbf{w} - \mathbf{w}_0) \qquad \text{for all } \mathbf{w},
```
i.e. the "tangent" line/plane with slope $`\mathbf{g}`$ through $`(\mathbf{w}_0, f(\mathbf{w}_0))`$ lies **beneath** the graph. Where $`f`$ is differentiable the only subgradient is the gradient; at a kink there is a whole set (Fig. 5b). **Subgradient descent:** pick *any* subgradient and step.

- **Example 1:** $`f(\mathbf{w}) = \lvert w_1\rvert + \lvert w_2\rvert + \lvert w_3\rvert + \lvert w_4\rvert`$ at $`\mathbf{w} = [2, -3, 0, 1]^\top`$: subgradients are $`\{[1, -1, \gamma, 1]^\top : \gamma \in [-1, 1]\}`$ (sign of each non-zero coordinate; anything in $`[-1,1]`$ at a zero).
- **Example 2:** $`f(x) = \max(0, x)`$ at $`x = 0`$: the subdifferential is the interval $`[0, 1]`$.

---

## 14. Newton's method
GD approximates $`f`$ **linearly** at $`\mathbf{w}_t`$ and steps against the slope (a linear approximation has no minimum, hence the need for small steps). Newton's method uses a **quadratic** approximation (second-order Taylor):
```math
f_{\text{quad}}(\mathbf{w}) = f(\mathbf{w}_t) + \mathbf{g}_t^\top(\mathbf{w} - \mathbf{w}_t) + \tfrac12(\mathbf{w} - \mathbf{w}_t)^\top\mathbf{H}_t(\mathbf{w} - \mathbf{w}_t).
```
Its gradient is $`\nabla f_{\text{quad}} = \mathbf{g}_t + \mathbf{H}_t(\mathbf{w} - \mathbf{w}_t)`$; set it to $`\mathbf{0}`$ and solve:
> ```math
> \mathbf{w}_{t+1} = \mathbf{w}_t - \mathbf{H}_t^{-1}\mathbf{g}_t .
> ```
**Trade-off:** each step needs $`D + \binom D2`$ second derivatives **and a matrix inversion** (expensive for large $`D`$), but far fewer iterations than GD. **Caveat:** Newton jumps to the stationary point of the quadratic, which may be a maximum or saddle, and may not even decrease $`f`$. For convex $`f`$ this is harmless (all stationary points are global minima); for deep nets, saddle points are thought to be why second-order methods struggle.

---

## 15. SGD, minibatches, constrained problems
Typical ML objective: an **average of per-datapoint losses** plus a regulariser,
```math
\mathcal{L}(\mathbf{w}; \mathcal{D}) = \frac1N\sum_{i=1}^N\ell(\mathbf{w}; \mathbf{x}_i, y_i) + \lambda\mathcal{R}(\mathbf{w}), \qquad \nabla_{\mathbf{w}}\mathcal{L} = \frac1N\sum_{i=1}^N\nabla_{\mathbf{w}}\ell(\mathbf{w}; \mathbf{x}_i, y_i) + \lambda\nabla_{\mathbf{w}}\mathcal{R}(\mathbf{w}).
```
*Ridge example:* $`\ell = (\mathbf{w}^\top\mathbf{x}_i - y_i)^2`$, $`\mathcal{R} = \mathbf{w}^\top\mathbf{w}`$, so $`\nabla\mathcal{L}_{\text{ridge}} = \frac1N\sum_i 2(\mathbf{w}^\top\mathbf{x}_i - y_i)\mathbf{x}_i + 2\lambda\mathbf{w}`$.

> [!NOTE]
> **📖 Key definitions**
> - **Stochastic gradient descent (SGD):** pick $`i \in \{1, \dots, N\}`$ uniformly at random and step using $`\mathbf{g}_i = \nabla_{\mathbf{w}}\ell(\mathbf{w}; \mathbf{x}_i, y_i)`$ (plus the regulariser's gradient). It is an **unbiased estimate** of the full gradient: $`\mathbb{E}[\mathbf{g}_i] = \frac1N\sum_i\nabla\ell(\mathbf{w}; \mathbf{x}_i, y_i)`$.
> - **Minibatch SGD:** average the gradient over a random batch $`B`$ of $`b`$ points. **Reduces the variance** of the gradient estimate, so it's more stable than pure SGD. Shuffle the data, then cycle through batches.
> - **Online learning:** update the model as data arrive (no need to hold the whole dataset).
> - **Projected gradient descent:** for a constrained problem such as ridge in constrained form, minimise $`(\mathbf{X}\mathbf{w}-\mathbf{y})^\top(\mathbf{X}\mathbf{w}-\mathbf{y})`$ subject to $`\mathbf{w}^\top\mathbf{w} \le R`$ (lasso: $`\sum_i\lvert w_i\rvert \le R`$), a gradient step may leave the feasible set; so after each step **project** back to the nearest feasible point. Cheap for these two sets; in general the projection is itself a convex optimisation problem and can be expensive. (SGD has theoretical convergence guarantees; minibatching helps mostly in practice.)

**Cost comparison (Fig. 8):** SGD needed ~4× more iterations than GD, but each GD iteration costs $`O(ND)`$ versus $`O(D)`$ for SGD. SGD's path is noisy; GD's is smooth.

**Scaling the regulariser (footnote 4):** if the objective is an *average* loss, one sampled point + full regulariser gradient is fine; if it's a *sum*, scale the regulariser gradient down accordingly. Choose the learning rate by validation.

**Discussion:** "convex optimisation has become a technology, non-convex optimisation is still an art" (Boyd & Vandenberghe). Tricks of the trade (momentum, Adagrad, Adam, BFGS, …) matter most for non-convex problems like deep nets.

> [!TIP]
> **✅ Key takeaways**
> 1. **Convex ⇒ local = global.** Prove convexity by decomposing into affine + PSD-quadratic + norms + non-negative sums.
> 2. Absolute loss ⇒ LP via slack variables; quadratic terms ⇒ need gradient methods.
> 3. GD: $`\mathbf{w} \leftarrow \mathbf{w} - \eta\nabla f`$; learning rate is the critical hyperparameter. Newton: $`\mathbf{w} \leftarrow \mathbf{w} - \mathbf{H}^{-1}\nabla f`$; fewer steps, each expensive.
> 4. Subgradients handle kinks ($`\lvert w\rvert`$, hinge); any subgradient works for descent.
> 5. SGD/minibatch: unbiased, cheap, noisy gradients; the workhorse of deep learning.
> 6. Hessian eigenvalue signs classify stationary points.

> [!WARNING]
> **⚠️ Common misconceptions & errors**
> - ❌ "The gradient points along the contour." ✅ It's **orthogonal** to contours, pointing uphill; descend with the **minus** sign.
> - ❌ "Newton's method always decreases $`f`$." ✅ It goes to the stationary point of the quadratic model, which could be a saddle or maximum.
> - ❌ "SGD computes the true gradient." ✅ It's a random, unbiased **estimate**.
> - ❌ "Quadratics are always convex." ✅ Only when $`\mathbf{A}`$ is PSD.
> - ❌ Forgetting to add the regulariser's gradient in SGD, or mis-scaling it for sum-vs-average objectives.
> - ❌ "Linear programs can solve lasso." ✅ The squared term breaks linearity.
