# A8 Probability · Part 2: Generating Functions and Joint Densities

*Chapters 3–4 of the Part A Probability notes (James Martin, MT 2025). [← Part 1](A8-1-Foundations-and-Limit-Theorems.md) · [Overview](A8-Probability-Summary.md) · [Part 3a →](A8-3a-Markov-Chains.md) · [Formula sheet](A8-4-Formula-Sheet.md)*

---

## Contents
3. [Generating functions](#3-generating-functions)
4. [Joint distribution of continuous random variables](#4-joint-distribution-of-continuous-random-variables)

---

## 3. Generating functions

> [!NOTE]
> **📖 Key definitions**
> - **Probability generating function (pgf)** of $`X`$ taking values in $`\{0,1,2,\dots\}`$:
> ```math
> G_X(s) = \mathbb{E}\big(s^X\big) = \sum_{k=0}^{\infty} p_X(k)\,s^k, \qquad s \in \mathcal{S} = \Big\{s : \sum_k \lvert s\rvert^k p_X(k) < \infty\Big\}.
> ```
> A power series with radius of convergence $`\ge 1`$.
> - **Moment generating function (mgf)** of any $`X`$: $`M_X(t) = \mathbb{E}\big(e^{tX}\big)`$, with values in $`[0, \infty]`$ (it may be infinite). For $`X \in \{0,1,2,\dots\}`$, $`M_X(t) = G_X(e^t)`$.
> - **Finite exponential moments:** $`\mathbb{E}\,e^{t_0\lvert X\rvert} < \infty`$ for some $`t_0 > 0`$, equivalently tails decay at least exponentially. Then $`M_X`$ is finite on $`(-t_0, t_0)`$ and all moments $`\mathbb{E}X^k`$ are finite.
> - **Characteristic function (cf):** $`\phi_X(t) = \mathbb{E}\big(e^{itX}\big) = \mathbb{E}\cos(tX) + i\,\mathbb{E}\sin(tX)`$, for $`t \in \mathbb{R}`$. **Always finite**: $`\lvert \phi_X(t)\rvert \le 1`$.

### 3.1 Probability generating functions

**Theorem 3.1.** (a) $`G^{(k)}(0) = k!\,p_X(k)`$. (b) $`G(1) = 1`$ and $`G^{(k)}(1) = \mathbb{E}\big[X(X-1)\cdots(X-k+1)\big]`$ (factorial moments). If the radius of convergence is exactly 1, read $`G^{(k)}(1)`$ as $`\lim_{s \uparrow 1}G^{(k)}(s)`$, which may be infinite.

So
```math
\mathbb{E}X = G'(1), \qquad \mathrm{var}(X) = G''(1) + G'(1) - \big[G'(1)\big]^2 .
```

**Theorem 3.2 (uniqueness).** If $`X`$ and $`Y`$ have the same pgf, they have the same distribution (immediate from 3.1(a)).

**Sums of independent variables.** If $`X, Y`$ are independent:
```math
G_{X+Y}(s) = \mathbb{E}\big(s^X s^Y\big) = \mathbb{E}\big(s^X\big)\,\mathbb{E}\big(s^Y\big) = G_X(s)\,G_Y(s).
```

**Random sums.** Let $`X_1, X_2, \dots`$ be i.i.d. (non-negative integer valued), $`N`$ non-negative integer valued and **independent of the $`X_i`$**, and $`S = X_1 + \dots + X_N`$. Then
```math
G_S(z) = G_N\big(G_X(z)\big).
```
*Proof* (law of total probability for expectations):
```math
\begin{aligned}
G_S(z) = \mathbb{E}\big(z^{X_1+\dots+X_N}\big) &= \sum_{n=0}^\infty \mathbb{E}\big(z^{X_1+\dots+X_N} \mid N = n\big)\,\mathbb{P}(N = n) \\
&= \sum_{n=0}^\infty \mathbb{E}\big(z^{X_1+\dots+X_n}\big)\,\mathbb{P}(N=n) \quad (N \text{ independent of the } X_i)\\
&= \sum_{n=0}^\infty \big(G_X(z)\big)^n\,\mathbb{P}(N=n) = \mathbb{E}\big[(G_X(z))^N\big] = G_N\big(G_X(z)\big). \qquad ∎
\end{aligned}
```
*(Differentiating at $`z=1`$ gives $`\mathbb{E}S = \mathbb{E}N\,\mathbb{E}X`$, a standard consequence.)*

### 3.2 Moment generating functions

**Why mgfs?** The pgf suits non-negative integer variables. The mgf works for any distribution, and it expands as a power series around $`t = 0`$ whose coefficients are the moments, whereas expanding $`G`$ around $`s = 0`$ gives no power series for general $`X`$.

**Theorem 3.3.** (a) If $`Y = aX + b`$ then $`M_Y(t) = e^{bt}M_X(at)`$. (b) If $`X_1, \dots, X_n`$ are independent then
```math
M_{X_1+\dots+X_n}(t) = \mathbb{E}\big(e^{tX_1}\cdots e^{tX_n}\big) = \mathbb{E}\big(e^{tX_1}\big)\cdots\mathbb{E}\big(e^{tX_n}\big) = M_{X_1}(t)\cdots M_{X_n}(t).
```
*((a): $`\mathbb{E}e^{t(aX+b)} = e^{bt}\mathbb{E}e^{(at)X}`$.)*

**The catch:** if the positive tail is too heavy, $`M_X(t) = \infty`$ for all $`t > 0`$ (similarly for the negative tail). The mgf is useful only when it is **finite on an interval around 0**.

**Theorem 3.4 (moments).** If $`M_X(t) < \infty`$ for $`\lvert t\rvert \le t_0`$ (some $`t_0 > 0`$), then
```math
M_X(t) = \sum_{k=0}^\infty \frac{t^k\,\mathbb{E}(X^k)}{k!} \quad (\lvert t\rvert \le t_0), \qquad M_X^{(k)}(0) = \mathbb{E}(X^k).
```
*Informal proof.* $`M_X(t) = \mathbb{E}\big(1 + tX + \tfrac{(tX)^2}{2!} + \cdots\big) = 1 + t\,\mathbb{E}X + \tfrac{t^2}{2!}\mathbb{E}X^2 + \cdots`$ by linearity. Swapping $`\mathbb{E}`$ with an infinite sum needs justification; it's fine here because $`\mathbb{E}\,e^{\lvert tX\rvert} < \infty`$. Differentiate at 0 for (b). ∎

**Theorem 3.5 (uniqueness).** If $`X, Y`$ have the same mgf, **finite on $`[-t_0, t_0]`$ for some $`t_0 > 0`$**, then $`X \overset{d}{=} Y`$.

**Theorem 3.6 (continuity).** If $`M_Y, M_{X_1}, M_{X_2}, \dots`$ are all finite on $`[-t_0, t_0]`$ and $`M_{X_n}(t) \to M_Y(t)`$ for all $`t \in [-t_0, t_0]`$, then $`X_n \xrightarrow{d} Y`$.

(Proofs of 3.5–3.6 are beyond the course; they come from Fourier inversion.)

**Example 3.7 (Exponential).** For $`X \sim \mathrm{Exp}(\lambda)`$:
```math
M(t) = \int_0^\infty \lambda e^{tx}e^{-\lambda x}\,dx = \frac{\lambda}{\lambda - t}\int_0^\infty (\lambda - t)e^{-(\lambda - t)x}\,dx = \frac{\lambda}{\lambda - t}, \qquad t \in (-\infty, \lambda).
```
The last integral is 1 because it integrates the $`\mathrm{Exp}(\lambda - t)`$ density, which only makes sense for $`t < \lambda`$. For $`t \ge \lambda`$, $`M(t) = \infty`$ even though $`\lambda/(\lambda - t)`$ is defined for $`t > \lambda`$. **Always state the domain.**

**Example 3.8 (Normal).** For $`Z \sim N(0,1)`$, complete the square:
```math
M_Z(t) = \int_{-\infty}^{\infty} e^{tz}\frac{1}{\sqrt{2\pi}}e^{-z^2/2}\,dz = e^{t^2/2}\int_{-\infty}^{\infty}\frac{1}{\sqrt{2\pi}}e^{-(z-t)^2/2}\,dz = e^{t^2/2}.
```
For $`X = \sigma Z + \mu \sim N(\mu, \sigma^2)`$, Theorem 3.3(a) gives $`M_X(t) = e^{\mu t}M_Z(\sigma t) = e^{\mu t + \sigma^2 t^2/2}`$. If $`Y_1 \sim N(\mu_1, \sigma_1^2)`$ and $`Y_2 \sim N(\mu_2, \sigma_2^2)`$ are independent:
```math
M_{Y_1+Y_2}(t) = e^{\mu_1 t + \sigma_1^2t^2/2}\,e^{\mu_2 t + \sigma_2^2t^2/2} = e^{(\mu_1+\mu_2)t + (\sigma_1^2+\sigma_2^2)t^2/2},
```
the mgf of $`N(\mu_1+\mu_2, \sigma_1^2+\sigma_2^2)`$, finite everywhere. So by uniqueness $`Y_1 + Y_2 \sim N(\mu_1+\mu_2, \sigma_1^2+\sigma_2^2)`$.

### 3.2.2 Proofs of the WLLN and CLT via mgfs
Assume i.i.d. $`X_i`$ with mean $`\mu`$, variance $`\sigma^2`$, and mgf $`M`$ finite near 0.

**WLLN.** By Taylor, $`M(h) = 1 + h\mu + o(h)`$ as $`h \to 0`$. The mgf of $`S_n/n`$ is
```math
\overline{M}_n(t) = \mathbb{E}\big(e^{tX_1/n}\cdots e^{tX_n/n}\big) = \big(M(t/n)\big)^n = \Big(1 + \frac{t\mu}{n} + o(t/n)\Big)^n \to e^{t\mu},
```
which is the mgf of the constant $`\mu`$. Continuity theorem ⇒ $`S_n/n \xrightarrow{d} \mu`$, hence in probability (Theorem 2.6). ∎

**CLT.** Let $`Y_i = X_i - \mu`$, with mgf $`M_Y(h) = 1 + h\,\mathbb{E}Y + \tfrac{h^2}{2}\mathrm{var}(Y) + o(h^2) = 1 + \tfrac{h^2\sigma^2}{2} + o(h^2)`$. The mgf of $`\frac{S_n - n\mu}{\sigma\sqrt n}`$ is
```math
\widetilde{M}_n(t) = \mathbb{E}\prod_{i=1}^n \exp\Big(\frac{t(X_i - \mu)}{\sigma\sqrt n}\Big) = M_Y\Big(\frac{t}{\sigma\sqrt n}\Big)^n = \Big(1 + \frac{t^2}{2n} + o\Big(\frac{t^2}{n}\Big)\Big)^n \to e^{t^2/2},
```
the mgf of $`N(0,1)`$. Continuity theorem ⇒ CLT. ∎

**Key limit used:** $`\big(1 + \frac{a}{n} + o(\frac1n)\big)^n \to e^{a}`$.

### 3.3 Tail bounds from mgfs (Chernoff)
Apply Markov to $`e^{tX}`$ instead of $`(X-\mu)^2`$.

**Simple random walk.** $`X_i = \pm 1`$ with probability $`\tfrac12`$ each, $`S_n = X_1 + \dots + X_n`$. CLT says $`S_n`$ is of order $`\sqrt n`$, so $`\mathbb{P}(\lvert S_n\rvert > na) \to 0`$ for $`a > 0`$. But how fast?
- **Chebyshev:** $`\mathbb{E}S_n = 0`$ and $`\mathrm{var}\,S_n = n`$, so $`\mathbb{P}(\lvert S_n\rvert > na) \le \dfrac{n}{(na)^2} = \dfrac{1}{na^2}`$. Slow.
- **mgf:** $`\mathbb{E}e^{tX_i} = \frac{e^t + e^{-t}}{2} = \cosh t \le e^{t^2/2}`$ (compare power series coefficients). For any $`t > 0`$:
```math
\mathbb{P}(S_n > na) = \mathbb{P}\big(e^{tS_n} > e^{tna}\big) \le \frac{\mathbb{E}e^{tS_n}}{e^{tna}} = \Big(\frac{\mathbb{E}e^{tX_1}}{e^{ta}}\Big)^n \le \big(e^{t^2/2 - ta}\big)^n .
```
Optimise over $`t`$: the minimum is at $`t = a`$, giving $`\mathbb{P}(S_n > na) \le e^{-na^2/2}`$. By symmetry,
```math
\mathbb{P}(\lvert S_n\rvert > na) \le 2e^{-na^2/2}.
```
That's **exponentially** small: a **Chernoff bound** (a "large deviations" bound). It works for any i.i.d. sum whose mgf exists near 0.

**Remark 3.9 (tails and mgfs).** If $`M`$ is finite on $`[-t_0, t_0]`$ then by Markov $`\mathbb{P}(\lvert X\rvert \ge a) \le e^{-t_0 a}\,\mathbb{E}e^{t_0\lvert X\rvert}`$, so the tails decay exponentially. Classification by tail $`a \mapsto \mathbb{P}(\lvert X\rvert \ge a)`$:

| Tail type | Examples | mgf |
|---|---|---|
| Vanishes eventually (bounded) | Bernoulli, binomial, uniform, finite sets | finite for all $`t`$ |
| Super-exponential (faster than any $`e^{-cx}`$; normal $`\sim e^{-cx^2}`$) | Normal, Poisson | finite for all $`t`$ |
| Exponential | Exponential, geometric, Gamma | finite on an interval |
| Polynomial | Pareto $`\mathbb{P}(X > x) = x^{-\alpha-1}`$ ($`x \ge 1`$, moments only for $`k < \alpha`$); Cauchy | **infinite for all $`t \neq 0`$**, so uniqueness/continuity theorems don't apply |

*(Pareto as printed in the notes. Strictly, a tail $`x^{-\alpha-1}`$ gives moments for $`k < \alpha+1`$, and a tail $`x^{-\alpha}`$ gives $`k < \alpha`$.)*

### 3.4 Characteristic functions
$`\phi_X(t) = \mathbb{E}e^{itX}`$ exists for **every** distribution, and $`\phi_{X+Y} = \phi_X\phi_Y`$ for independent $`X, Y`$. Uniqueness and continuity theorems hold as for mgfs, **with no tail assumptions**. Taylor expansion, valid whenever $`\mathbb{E}X^k`$ is finite:
```math
\phi_X(t) = 1 + it\,\mathbb{E}X + i^2t^2\frac{\mathbb{E}X^2}{2} + \dots + i^kt^k\frac{\mathbb{E}X^k}{k!} + o(t^k) \quad (t \to 0).
```
So the same proof gives the CLT assuming **only finite variance**. When the mgf is finite near 0, $`\phi_X(t) = M_X(it)`$ (analytic continuation).

**Example 3.10.**
- (a) $`N(0,1)`$: mgf $`e^{t^2/2}`$, cf $`e^{(it)^2/2} = e^{-t^2/2}`$.
- (b) $`\mathrm{Exp}(1)`$: mgf $`\frac{1}{1-t}`$, cf $`\frac{1}{1-it}`$.
- (c) **Cauchy**, $`f(x) = \frac{1}{\pi(1+x^2)}`$: the mgf is infinite for $`t \ne 0`$, and even $`\mathbb{E}\lvert X\rvert = \infty`$ (no mean). By contour integration, $`\phi_X(t) = e^{-\lvert t\rvert}`$, which is **not differentiable at 0**, matching the missing mean. For i.i.d. Cauchy:
```math
\phi_{S_n/n}(t) = \phi\Big(\frac tn\Big)^n = \big(e^{-\lvert t\rvert/n}\big)^n = e^{-\lvert t\rvert} = \phi_X(t),
```
so $`S_n/n`$ **has the same Cauchy distribution as $`X_1`$**: no LLN, no CLT.

> [!TIP]
> **✅ Key takeaways**
> 1. Generating functions turn **sums of independent variables into products** ("independence means multiply"), and random sums into **compositions** $`G_N(G_X(z))`$.
> 2. Read off moments: pgf at $`s = 1`$ (factorial moments), mgf at $`t = 0`$ ($`M^{(k)}(0) = \mathbb{E}X^k`$).
> 3. **Uniqueness + continuity** theorems are the engine: identify a distribution by its mgf/cf, and prove convergence in distribution by convergence of mgfs/cfs (e.g. the CLT).
> 4. mgfs need **exponential tails** (finite near 0); characteristic functions always exist.
> 5. **Chernoff:** Markov applied to $`e^{tX}`$ and then optimised over $`t`$ gives exponentially small tail bounds, far better than Chebyshev.

> [!WARNING]
> **⚠️ Common misconceptions & errors**
> - ❌ "$`M(t) = \frac{\lambda}{\lambda - t}`$ for all $`t`$." ✅ Only for $`t < \lambda`$; otherwise $`M = \infty`$. **Always give the domain.**
> - ❌ Applying mgf uniqueness/continuity when the mgf is infinite near 0 (Cauchy, Pareto). ✅ Use characteristic functions.
> - ❌ "$`G'(1)`$ is the variance" or "$`G''(1) = \mathbb{E}X^2`$." ✅ $`G''(1) = \mathbb{E}[X(X-1)]`$, so $`\mathrm{var} = G''(1) + G'(1) - G'(1)^2`$.
> - ❌ "$`M_{X+Y} = M_X M_Y`$ always." ✅ Needs **independence**.
> - ❌ "$`G_S = G_X(G_N)`$" for random sums. ✅ It's $`G_N(G_X(z))`$ (outer = number of terms), and $`N`$ must be independent of the $`X_i`$.
> - ❌ "The average of i.i.d. samples always settles down." ✅ Cauchy: $`S_n/n \overset{d}{=} X_1`$ for all $`n`$.
> - ❌ Forgetting the $`o(\cdot)`$ term or the $`\sigma\sqrt{n}`$ scaling in the CLT proof.

---

## 4. Joint distribution of continuous random variables

> [!NOTE]
> **📖 Key definitions**
> - **Joint cdf:** $`F_{X,Y}(x,y) = \mathbb{P}(X \le x, Y \le y)`$.
> - **Jointly continuous** with **joint pdf** $`f_{X,Y}`$: $`F_{X,Y}(x,y) = \int_{-\infty}^x\int_{-\infty}^y f_{X,Y}(u,v)\,dv\,du`$. Then $`\mathbb{P}((X,Y) \in A) = \iint_A f_{X,Y}\,dx\,dy`$ for nice (Borel) $`A`$, and $`f_{X,Y} = \frac{\partial^2}{\partial x\,\partial y}F_{X,Y}`$ where differentiable.
> - **Marginal density:** $`f_X(x) = \int_{-\infty}^{\infty} f_{X,Y}(x,y)\,dy`$.
> - **Jacobian** of the inverse map $`(u,v) \mapsto (x,y)`$:
> ```math
> J(u,v) = \det\begin{pmatrix} \partial x/\partial u & \partial x/\partial v \\ \partial y/\partial u & \partial y/\partial v\end{pmatrix} = \frac{\partial x}{\partial u}\frac{\partial y}{\partial v} - \frac{\partial x}{\partial v}\frac{\partial y}{\partial u}.
> ```
> - **Correlation coefficient:** $`\rho = \dfrac{\mathrm{cov}(X,Y)}{\sigma_X\sigma_Y}`$.
> - **Conditional probability:** $`\mathbb{P}(B \mid A) = \mathbb{P}(A \cap B)/\mathbb{P}(A)`$, defined only for $`\mathbb{P}(A) > 0`$.
> - **Conditional density** of $`Y`$ given $`X = x`$ (for $`f_X(x) > 0`$):
> ```math
> f_{Y\mid X=x}(y) = \frac{f_{X,Y}(x,y)}{f_X(x)} .
> ```

### 4.1 Review
```math
\mathbb{E}\,h(X,Y) = \int_{-\infty}^{\infty}\int_{-\infty}^{\infty} h(x,y)\,f_{X,Y}(x,y)\,dx\,dy .
```
Similarly $`f_Y(y) = \int_{-\infty}^{\infty} f_{X,Y}(x,y)\,dx`$. Strictly it's *a* joint pdf: changing $`f`$ at finitely many points changes no integral.

$`X, Y`$ are **independent** iff $`F_{X,Y}(x,y) = F_X(x)F_Y(y)`$ for all $`x,y`$, iff $`f_{X,Y}(x,y) = f_X(x)f_Y(y)`$. Everything generalises to $`n`$ variables. (To identify a joint pdf, it suffices to check $`\mathbb{P}((X,Y) \in A) = \iint_A f`$ for rectangles, or just for sets $`(-\infty,u] \times (-\infty, v]`$.)

### 4.2 The transformation (change of variables) formula

> **Theorem 4.1.** Let $`T: (x,y) \mapsto (u,v)`$ be **one-to-one** from $`D \subseteq \mathbb{R}^2`$ **onto** $`R \subseteq \mathbb{R}^2`$, with continuous partial derivatives of the inverse, and Jacobian $`J(u,v)`$ as above. If $`(X,Y)`$ has joint pdf $`f_{X,Y}`$ (living on $`D`$) and $`(U,V) = T(X,Y)`$, then
> ```math
> f_{U,V}(u,v) = \begin{cases} f_{X,Y}\big(x(u,v),\,y(u,v)\big)\,\lvert J(u,v)\rvert & (u,v) \in R, \\ 0 & \text{otherwise.}\end{cases}
> ```

*Proof.* For $`A \subseteq D`$ with $`T(A) = B`$ (so $`\{(U,V) \in B\} = \{(X,Y) \in A\}`$ as $`T`$ is one-to-one):
```math
\mathbb{P}((U,V) \in B) = \mathbb{P}((X,Y) \in A) = \iint_A f_{X,Y}(x,y)\,dx\,dy = \iint_B f_{X,Y}\big(x(u,v), y(u,v)\big)\,\lvert J(u,v)\rvert\,du\,dv,
```
by the change of variables formula for integrals, so the integrand is the joint pdf of $`(U,V)`$. ∎

**Intuition:** locally $`T`$ is linear, and $`\lvert J\rvert`$ is the ratio of the areas of a small rectangle and its image. Since probability = density × area and must be preserved, the density is rescaled by $`\lvert J\rvert`$.

**Recipe:** (1) find the range $`R`$; (2) invert to $`x(u,v), y(u,v)`$; (3) compute $`J`$ (of the **inverse** map) and take $`\lvert J\rvert`$; (4) substitute; (5) state the support; (6) integrate out for marginals, or factorise to show independence.

**Example 4.2.** $`X, Y`$ i.i.d. $`\mathrm{Exp}(\lambda)`$, $`U = \frac{X}{X+Y}`$, $`V = X + Y`$. Here $`f_{X,Y} = \lambda^2 e^{-\lambda(x+y)}`$ on $`(0,\infty)^2`$, mapped onto $`(0,1) \times (0,\infty)`$. The inverse is $`x = uv`$, $`y = v(1-u)`$:
```math
J = \det\begin{pmatrix} v & u \\ -v & 1-u\end{pmatrix} = v(1-u) + uv = v, \qquad f_{U,V}(u,v) = \lambda^2 e^{-\lambda v}\cdot v = \underbrace{1}_{f_U(u)}\cdot\underbrace{\lambda^2 v e^{-\lambda v}}_{f_V(v)} .
```
It **factorises** on a **product** set, so $`U \sim U[0,1]`$ and $`V \sim \mathrm{Gamma}(2,\lambda)`$ are **independent**.

**Example 4.3.** Same $`X, Y`$, $`V = X + Y`$, $`W = X - Y`$. This maps $`(0,\infty)^2`$ onto $`\{(v,w) : \lvert w\rvert < v\}`$, with inverse $`x = \frac{v+w}{2}`$, $`y = \frac{v-w}{2}`$ and $`J = \det\begin{pmatrix}1/2 & 1/2\\ 1/2 & -1/2\end{pmatrix} = -\tfrac12`$ (a linear map has constant Jacobian). So
```math
f_{V,W}(v,w) = \tfrac12\lambda^2 e^{-\lambda v} \quad \text{for } \lvert w\rvert < v, \text{ else } 0 .
```
This *looks* like a product, but **the support is not a product set**, so $`V, W`$ are **not** independent (otherwise $`\mathbb{P}(\lvert W\rvert < V) = 1`$ would be impossible). Marginal:
```math
f_W(w) = \int_{\lvert w\rvert}^\infty \tfrac12\lambda^2 e^{-\lambda v}\,dv = \tfrac12\lambda e^{-\lambda\lvert w\rvert},
```
symmetric about 0 (Laplace). Adding the density at $`w`$ and $`-w`$: $`f_{\lvert W\rvert}(w) = \lambda e^{-\lambda w}`$ for $`w > 0`$, so $`\lvert W\rvert \sim \mathrm{Exp}(\lambda)`$. ($`V \sim \mathrm{Gamma}(2,\lambda)`$ was already found in Example 4.2.)

**Example 4.4 (sums, convolution).** For $`U = X + Y`$, $`V = X`$ (Jacobian 1): $`f_{U,V}(u,v) = f_{X,Y}(v, u-v)`$. Integrating out $`v`$:
```math
f_{X+Y}(u) = \int_{-\infty}^{\infty} f_{X,Y}(v, u-v)\,dv \;\overset{\text{indep.}}{=}\; \int_{-\infty}^{\infty} f_X(v)\,f_Y(u-v)\,dv \quad \text{(convolution formula)}.
```

### 4.3 Multivariate normal
Let $`Z_1, \dots, Z_n`$ be i.i.d. $`N(0,1)`$:
```math
f_{\mathbf{Z}}(\mathbf{z}) = \prod_{i=1}^n \frac{1}{\sqrt{2\pi}}e^{-z_i^2/2} = \frac{1}{(2\pi)^{n/2}}\exp\Big(-\tfrac12\mathbf{z}^T\mathbf{z}\Big).
```
Set $`\mathbf{W} = A\mathbf{Z} + \boldsymbol{\mu}`$ with $`A`$ an invertible $`n \times n`$ matrix. The $`n`$-dimensional transformation formula (constant Jacobian) gives
```math
f_{\mathbf{W}}(\mathbf{w}) = \frac{1}{(2\pi)^{n/2}\lvert\det A\rvert}\exp\Big(-\tfrac12(\mathbf{w} - \boldsymbol\mu)^T(AA^T)^{-1}(\mathbf{w}-\boldsymbol\mu)\Big).
```
$`\Sigma = AA^T`$ is the **covariance matrix**: $`\mathrm{cov}(W_i, W_j) = \Sigma_{ij}`$. This is the **multivariate normal** with mean $`\boldsymbol\mu`$ and covariance $`\Sigma`$.

**Bivariate case** ($`X = W_1`$, $`Y = W_2`$), with $`\rho \in (-1,1)`$:
```math
f_{X,Y}(x,y) = \frac{1}{2\pi\sigma_X\sigma_Y\sqrt{1-\rho^2}}\exp\left(-\frac{1}{2(1-\rho^2)}\left[\frac{(x-\mu_X)^2}{\sigma_X^2} - \frac{2\rho(x-\mu_X)(y-\mu_Y)}{\sigma_X\sigma_Y} + \frac{(y-\mu_Y)^2}{\sigma_Y^2}\right]\right).
```
1. It depends only on $`\mu_X, \mu_Y, \sigma_X, \sigma_Y, \rho`$.
2. **$`X, Y`$ independent ⇔ $`\rho = 0`$.** (⇒ holds for any distribution; ⇐ is special to the joint normal: set $`\rho = 0`$ and the density factorises.)

**Standard bivariate normal** ($`\mu = 0`$, $`\sigma = 1`$): $`f(x,y) = \dfrac{1}{2\pi\sqrt{1-\rho^2}}\exp\Big(-\dfrac{x^2 - 2\rho xy + y^2}{2(1-\rho^2)}\Big)`$, with $`\Sigma = \begin{pmatrix}1 & \rho\\ \rho & 1\end{pmatrix}`$.

### 4.4 Conditional densities

**Conditioning on an event $`A`$ with $`\mathbb{P}(A) > 0`$.** The conditional cdf is $`F_{X\mid A}(x) = \mathbb{P}(\{X \le x\} \cap A)/\mathbb{P}(A)`$, with conditional pmf $`p_{X\mid A}(x) = \mathbb{P}(X = x \mid A)`$ or conditional density $`f_{X\mid A}`$ (differentiate). Then $`\mathbb{P}(X \in C \mid A) = \int_C f_{X\mid A}`$ and
```math
\mathbb{E}(X \mid A) = \sum_x x\,p_{X\mid A}(x) \quad\text{or}\quad \int x\,f_{X\mid A}(x)\,dx .
```

**Example 4.5.** $`X, Y`$ i.i.d. $`U[0,1]`$; find the law of $`Y`$ given $`X + Y > 1`$. On the unit square, probability = area. $`\{x + y > 1\}`$ has area $`\tfrac12`$, and $`\{v < y,\ x + v > 1\}`$ has area $`y^2/2`$. So $`F(y) = \frac{y^2/2}{1/2} = y^2`$, the conditional density is $`2y`$ on $`[0,1]`$, and $`\mathbb{E}(Y \mid X+Y > 1) = \int_0^1 2y^2\,dy = \tfrac23`$.

**Conditioning on $`X = x`$ when $`X`$ is continuous.** $`\mathbb{P}(X = x) = 0`$, so the definition above fails. Instead condition on $`\{x \le X \le x + \epsilon\}`$ and let $`\epsilon \downarrow 0`$:
```math
\mathbb{P}\big(Y \le y \mid x \le X \le x+\epsilon\big) = \frac{\int_{-\infty}^y\int_x^{x+\epsilon} f_{X,Y}(u,v)\,du\,dv}{\int_x^{x+\epsilon} f_X(u)\,du} \approx \frac{\epsilon\int_{-\infty}^y f_{X,Y}(x,v)\,dv}{\epsilon f_X(x)} = \int_{-\infty}^y \frac{f_{X,Y}(x,v)}{f_X(x)}\,dv .
```
This defines $`F_{Y\mid X=x}(y)`$. Differentiating gives $`f_{Y\mid X=x}(y) = f_{X,Y}(x,y)/f_X(x)`$, defined whenever $`f_X(x) > 0`$ (and $`f_{X,Y}`$ is smooth enough for the limit to exist). This integrates to 1 in $`y`$ precisely because $`f_X(x) = \int f_{X,Y}(x,y)\,dy`$. The denominator is just a **normalising constant**, not depending on $`y`$.

**Interpretation:** generating $`(X,Y)`$ from $`f_{X,Y}`$ is equivalent to generating $`X \sim f_X`$ and then $`Y \sim f_{Y\mid X=x}`$.

**Example 4.6.** $`(X,Y)`$ uniform on the triangle $`\{0 < y < x < 1\}`$, so $`f_{X,Y} = 2`$ there. Then $`f_{Y\mid X=x}(y) = 2/f_X(x)`$ for $`0 < y < x`$. It's constant in $`y`$, so $`Y \mid X = x \sim U[0, x]`$ (no need to compute $`f_X`$), and $`\mathbb{E}(Y \mid X = x) = x/2`$.

**Example 4.7 (bivariate normal conditional).** Build from independent $`Z_1, Z_2 \sim N(0,1)`$:
```math
X = \sigma_1 Z_1 + \mu_1, \qquad Y = \rho\sigma_2 Z_1 + \sqrt{1-\rho^2}\,\sigma_2 Z_2 + \mu_2 .
```
These have the right means, variances and covariance. Then $`Y = \rho\frac{\sigma_2}{\sigma_1}(X - \mu_1) + \sqrt{1-\rho^2}\,\sigma_2 Z_2 + \mu_2`$, a function of $`X`$ plus a part **independent** of $`X`$. Hence
```math
Y \mid X = x \;\sim\; N\Big(\mu_2 + \rho\frac{\sigma_2}{\sigma_1}(x - \mu_1),\; (1-\rho^2)\sigma_2^2\Big).
```
$`\rho^2`$ is the **fraction of the variance of $`Y`$ explained by $`X`$**. At $`\rho = \pm 1`$, $`Y`$ is a function of $`X`$ (zero conditional variance); at $`\rho = 0`$, observing $`X`$ tells you nothing.

### 4.5 Cautionary tale: Borel's paradox
Uniform on the half-disc $`C = \{y \ge 0, x^2 + y^2 \le 1\}`$, with $`f = 2/\pi`$ on $`C`$.
- $`f_{Y\mid X=0}(y) = \frac{2/\pi}{f_X(0)}`$ on $`[0,1]`$ is constant, so $`Y \mid X = 0`$ is **uniform** on $`[0,1]`$.
- In polar coordinates, $`R`$ and $`\Theta`$ are independent, with $`f_R(r) = 2r`$ on $`[0,1]`$ and $`\Theta \sim U[0,\pi]`$. So $`R \mid \Theta = \pi/2`$ has density $`2r`$, which is **not uniform**.
- But $`\{X = 0\} = \{\Theta = \pi/2\}`$, and $`R = Y`$ there!

**Resolution:** conditioning on a probability-zero event is a **limit**, and the answer depends on **how** you take it. $`\{\lvert X\rvert < \epsilon\}`$ is a thin rectangle, while $`\{\lvert\Theta - \pi/2\rvert < \epsilon\}`$ is a thin sector (a triangle) that favours larger $`r`$. Conditional densities are **not invariant under change of variables**.

> [!TIP]
> **✅ Key takeaways**
> 1. **Transformation formula:** $`f_{U,V} = f_{X,Y}(x(u,v), y(u,v))\,\lvert J\rvert`$, with $`J`$ from the **inverse** map, and always state the **range**.
> 2. **Independence test:** the joint density factorises **and** the support is a **product set**.
> 3. Sums: change variables to $`(X+Y, X)`$ and integrate out, giving the convolution formula when independent.
> 4. Multivariate normal = linear image of i.i.d. normals; $`\Sigma = AA^T`$; for jointly normal variables, **uncorrelated ⇔ independent**.
> 5. $`f_{Y\mid X=x} = f_{X,Y}/f_X`$: treat $`f_X(x)`$ as a normalising constant, so the *shape* in $`y`$ often identifies the law immediately.
> 6. Bivariate normal: $`Y \mid X=x`$ is normal with mean linear in $`x`$ and variance $`(1-\rho^2)\sigma_2^2`$. Trick: write $`Y = (\text{function of } X) + (\text{independent noise})`$.

> [!WARNING]
> **⚠️ Common misconceptions & errors**
> - ❌ Using the Jacobian of the **forward** map $`(x,y) \mapsto (u,v)`$. ✅ Use $`\partial(x,y)/\partial(u,v)`$ (or $`1/\lvert\partial(u,v)/\partial(x,y)\rvert`$), and take the **absolute value**.
> - ❌ Forgetting the new support (e.g. $`\lvert w\rvert < v`$) or that $`T`$ must be **one-to-one**.
> - ❌ "The density factorises, so they're independent." ✅ Also need a product support (Example 4.3).
> - ❌ "$`\rho = 0`$ ⇒ independent" for arbitrary variables. ✅ Not in general. It does hold for **jointly** normal $`(X,Y)`$; each being normal on its own isn't enough.
> - ❌ "$`\mathbb{P}(Y \le y \mid X = x) = \mathbb{P}(Y \le y, X = x)/\mathbb{P}(X = x)`$" for continuous $`X`$. ✅ That's $`0/0`$; use $`f_{X,Y}/f_X`$.
> - ❌ "Conditional densities transform like ordinary densities." ✅ Borel's paradox: they depend on the coordinates/limit used.
> - ❌ Bivariate normal conditional variance $`= \sigma_2^2`$ or $`\rho\sigma_2^2`$. ✅ It's $`(1-\rho^2)\sigma_2^2`$, and it doesn't depend on $`x`$.
