# A8 Probability · Part 1: Foundations and Limit Theorems

*Chapters 1–2 of the Part A Probability notes (James Martin, MT 2025). [← Overview](A8-Probability-Summary.md) · [Part 2 →](A8-2-Generating-Functions-and-Joint-Densities.md) · [Formula sheet](A8-4-Formula-Sheet.md)*

---

## Contents
1. [Review: probability spaces, random variables, distributions, independence](#1-review-probability-spaces-random-variables-distributions-independence)
2. [Convergence of random variables and limit theorems](#2-convergence-of-random-variables-and-limit-theorems)

---

## 1. Review: probability spaces, random variables, distributions, independence

> [!NOTE]
> **📖 Key definitions**
> - **Probability space** $`(\Omega, \mathcal{F}, \mathbb{P})`$: a **sample space** $`\Omega`$ (all outcomes), a collection $`\mathcal{F}`$ of subsets called **events**, and a **probability measure** $`\mathbb{P}: \mathcal{F} \to [0,1]`$.
> - **Conditions on $`\mathcal{F}`$:** (1) $`\Omega \in \mathcal{F}`$; (2) $`A \in \mathcal{F} \Rightarrow A^c \in \mathcal{F}`$; (3) closed under **finite or countable unions**. (So also closed under countable intersections.)
> - **Probability axioms:** $`\mathbb{P}(\Omega) = 1`$, and **countable additivity**: for finitely or countably many **disjoint** events, $`\mathbb{P}\big(\bigcup_i A_i\big) = \sum_i \mathbb{P}(A_i)`$.
> - **Random variable:** a function $`X: \Omega \to \mathbb{R}`$ such that $`\{\omega : X(\omega) \le x\} \in \mathcal{F}`$ for every $`x \in \mathbb{R}`$. It models an *observable*; one experiment has many random variables.
> - **Distribution function (cdf):** $`F(x) = \mathbb{P}(X \le x)`$. The **distribution** of $`X`$ is the map $`B \mapsto \mathbb{P}(X \in B)`$. We write $`X \overset{d}{=} Y`$ if they have the same distribution.
> - **Discrete:** $`\mathbb{P}(X \in B) = 1`$ for some countable $`B`$; described by the **pmf** $`p_X(x) = \mathbb{P}(X = x)`$. Points with $`p_X(x) > 0`$ are **atoms**.
> - **(Absolutely) continuous:** $`F(x) = \int_{-\infty}^x f(u)\,du`$ for some **density** $`f`$.
> - **Independent events:** $`(A_i)_{i \in \mathcal{I}}`$ are independent if $`\mathbb{P}\big(\bigcap_{i \in J} A_i\big) = \prod_{i \in J}\mathbb{P}(A_i)`$ for **every finite** $`J \subseteq \mathcal{I}`$.
> - **Independent random variables:** $`\{X_1 \in B_1\}, \dots, \{X_n \in B_n\}`$ independent for all $`B_1, \dots, B_n`$. A sequence is independent if every finite initial segment is.

### 1.1 Probability spaces and random variables
- Think of $`(\Omega, \mathcal{F}, \mathbb{P})`$ as modelling an **experiment**. In practice we rarely look at $`\Omega`$ directly; we work with events and random variables. In this course, assume $`\mathcal{F}`$ contains every event we want.
- We write $`\mathbb{P}(X \in B)`$ for $`\mathbb{P}(\{\omega \in \Omega : X(\omega) \in B\})`$.
- *Examples of models:* two dice (scores, sum, max, indicator "blue > red"); Geiger counter (time of $`k`$th particle, count in $`[0,t]`$); markets; bacteria; call-centres.

### 1.2 Distribution functions
Every cdf $`F`$ satisfies:
1. $`F`$ is **non-decreasing**;
2. $`F`$ is **right-continuous**;
3. $`F(x) \to 0`$ as $`x \to -\infty`$;
4. $`F(x) \to 1`$ as $`x \to +\infty`$.

**Discrete:** $`\sum_x p_X(x) = 1`$ and $`\mathbb{P}(X \in A) = \sum_{x \in A} p_X(x)`$. The cdf **jumps** at atoms and is flat elsewhere.

**Continuous:** (absolute continuity is stronger than continuity of $`F`$: not every continuous $`F`$ can be written as $`\int_{-\infty}^x f`$.) $`F`$ is continuous, so $`\mathbb{P}(X = x) = F(x) - \lim_{y \uparrow x}F(y) = 0`$ for every $`x`$. The density isn't unique (you can change it at single points); where $`F`$ is differentiable take $`f = F'`$. And
```math
\mathbb{P}(a \le X \le b) = \int_a^b f(u)\,du .
```

### 1.3 Expectation, variance, covariance
```math
\mathbb{E}(X) = \sum_x x\,p_X(x) \quad\text{(discrete)}, \qquad \mathbb{E}(X) = \int_{-\infty}^{\infty} x f(x)\,dx \quad\text{(continuous)},
```
when the sum/integral converges. For a function $`g`$:
```math
\mathbb{E}\,g(X) = \sum_x g(x)\,p_X(x), \qquad \mathbb{E}\,g(X) = \int_{-\infty}^{\infty} g(x) f(x)\,dx .
```
(Expectation can be defined for *any* random variable by approximating with discrete ones, like building an integral from step functions.)

**Basic properties** (used constantly):
1. $`\mathbb{E}\,\mathbf{1}_A = \mathbb{P}(A)`$ for any event $`A`$.
2. If $`\mathbb{P}(X \ge 0) = 1`$ then $`\mathbb{E}X \ge 0`$.
3. $`\mathbb{E}(aX) = a\,\mathbb{E}X`$.
4. $`\mathbb{E}(X+Y) = \mathbb{E}X + \mathbb{E}Y`$ (**always**, no independence needed).

**Variance and covariance:**
```math
\mathrm{var}(X) = \mathbb{E}\big[(X - \mathbb{E}X)^2\big] = \mathbb{E}(X^2) - (\mathbb{E}X)^2,
```
```math
\mathrm{cov}(X,Y) = \mathbb{E}\big[(X-\mathbb{E}X)(Y-\mathbb{E}Y)\big] = \mathbb{E}(XY) - \mathbb{E}X\,\mathbb{E}Y .
```
- $`\mathrm{var}(X) = \mathrm{cov}(X,X)`$; bilinearity: $`\mathrm{cov}(aX+b, cY+d) = ac\,\mathrm{cov}(X,Y)`$; so $`\mathrm{var}(aX+b) = a^2\mathrm{var}(X)`$.
- Sums:
```math
\mathrm{var}(X_1 + \dots + X_n) = \sum_{i=1}^n \mathrm{var}(X_i) + 2\sum_{1 \le i < j \le n}\mathrm{cov}(X_i, X_j).
```

### 1.4 Independence
- **Mutual ≠ pairwise.** It's possible that $`A_1, A_2`$ are independent, $`A_2, A_3`$ are independent and $`A_1, A_3`$ are independent, but $`A_1, A_2, A_3`$ are **not**.
- For random variables it's enough to check the cdfs factorise:
```math
\mathbb{P}(X_1 \le x_1, \dots, X_n \le x_n) = F_{X_1}(x_1)\cdots F_{X_n}(x_n) \quad \forall x_1,\dots,x_n ,
```
or, for discrete random variables, $`\mathbb{P}(X_1 = x_1, \dots, X_n = x_n) = \prod_i \mathbb{P}(X_i = x_i)`$.
- If $`X, Y`$ are independent (with finite means) then $`\mathbb{E}(XY) = \mathbb{E}X\,\mathbb{E}Y`$, i.e. $`\mathrm{cov}(X,Y) = 0`$ (**uncorrelated**). The converse is **false**.
- Slogan: **"independence means multiply"** (probabilities, expectations of products, generating functions, densities).

### 1.5 Standard distributions (summary; full table in the [formula sheet](A8-4-Formula-Sheet.md))
| Distribution | pmf / pdf | Mean | Variance | Key facts |
|---|---|---|---|---|
| $`U[a,b]`$ | $`\frac{1}{b-a}`$ on $`[a,b]`$ | $`\frac{a+b}{2}`$ | $`\frac{(b-a)^2}{12}`$ | |
| $`\mathrm{Exp}(\lambda)`$ | $`\lambda e^{-\lambda x}`$, $`x \ge 0`$ | $`1/\lambda`$ | $`1/\lambda^2`$ | $`F(x) = 1-e^{-\lambda x}`$; **memoryless**; $`aX \sim \mathrm{Exp}(\lambda/a)`$ for $`a > 0`$ |
| $`N(\mu,\sigma^2)`$ | $`\frac{1}{\sqrt{2\pi\sigma^2}}e^{-(x-\mu)^2/2\sigma^2}`$ | $`\mu`$ | $`\sigma^2`$ | $`aX+b \sim N(a\mu+b, a^2\sigma^2)`$; independent sums add means and variances |
| $`\mathrm{Gamma}(r,\lambda)`$ | $`\frac{\lambda^r}{\Gamma(r)}x^{r-1}e^{-\lambda x}`$, $`x \ge 0`$ | $`r/\lambda`$ | $`r/\lambda^2`$ | shape $`r`$, rate $`\lambda`$; $`r=1`$ is $`\mathrm{Exp}(\lambda)`$; sum of $`n`$ i.i.d. $`\mathrm{Exp}(\lambda)`$ is $`\mathrm{Gamma}(n,\lambda)`$ |
| $`\mathrm{Bernoulli}(p)`$ | $`p_X(1) = p,\ p_X(0) = 1-p`$ | $`p`$ | $`p(1-p)`$ | indicator of an event with probability $`p`$ |
| $`\mathrm{Bin}(n,p)`$ | $`\binom{n}{k}p^k(1-p)^{n-k}`$ | $`np`$ | $`np(1-p)`$ | sum of $`n`$ i.i.d. Bernoulli |
| $`\mathrm{Geom}(p)`$ on $`\{1,2,\dots\}`$ | $`(1-p)^{k-1}p`$ | $`1/p`$ | $`(1-p)/p^2`$ | # trials to first success; $`\mathbb{P}(X > k) = (1-p)^k`$; memoryless |
| $`\mathrm{Poisson}(\lambda)`$ | $`e^{-\lambda}\lambda^r/r!`$ | $`\lambda`$ | $`\lambda`$ | independent sums: $`\mathrm{Po}(\lambda)+\mathrm{Po}(\mu) = \mathrm{Po}(\lambda+\mu)`$ |

$`\Gamma(r) = \int_0^\infty z^{r-1}e^{-z}\,dz`$, with $`\Gamma(r) = (r-1)!`$ for integer $`r`$. $`\Phi(x) = \int_{-\infty}^x \frac{1}{\sqrt{2\pi}}e^{-z^2/2}\,dz`$ is the standard normal cdf. Geometric conventions vary: $`Y = X - 1`$ (failures before first success) has $`p_Y(k) = (1-p)^k p`$ on $`\{0,1,\dots\}`$.

> [!TIP]
> **✅ Key takeaways**
> 1. A random variable is a **function on $`\Omega`$**; its **distribution** is what probabilities it gives. Different random variables can share a distribution (two fair dice).
> 2. Linearity of expectation **always** holds; "expectation of a product = product of expectations" needs (at least) uncorrelatedness, typically independence.
> 3. Variance of a sum = sum of variances **+ 2 × covariances**.
> 4. Independence of a family needs **every finite sub-family** to multiply, not just pairs.

> [!WARNING]
> **⚠️ Common misconceptions & errors**
> - ❌ "$`X \overset{d}{=} Y`$ means $`X = Y`$." ✅ Same *distribution*, possibly different values (red die vs blue die).
> - ❌ "Uncorrelated ⇒ independent." ✅ Not in general. It does hold for **jointly normal** variables (§4.3 in Part 2); each being normal on its own isn't enough.
> - ❌ "Pairwise independent ⇒ independent."
> - ❌ "$`\mathrm{var}(X+Y) = \mathrm{var}\,X + \mathrm{var}\,Y`$ always." ✅ Only when $`\mathrm{cov}(X,Y) = 0`$.
> - ❌ "$`\mathrm{var}(aX+b) = a\mathrm{var}\,X + b`$." ✅ $`a^2\mathrm{var}\,X`$.
> - ❌ "A density value $`f(x)`$ is a probability (so $`\le 1`$)." ✅ It's a density; $`\mathbb{P}(X = x) = 0`$ for continuous $`X`$, and $`f`$ can exceed 1.
> - ❌ Mixing up geometric conventions (support $`\{1,2,\dots\}`$ vs $`\{0,1,\dots\}`$). State which one you use.

---

## 2. Convergence of random variables and limit theorems

> [!NOTE]
> **📖 Key definitions** (let $`X_1, X_2, \dots`$ and $`X`$ be random variables, $`F_n, F`$ their cdfs)
> - **Almost surely (a.s.):** $`X_n \to X`$ a.s. if
> ```math
> \mathbb{P}\big(X_n \to X \text{ as } n \to \infty\big) = 1 .
> ```
> Here $`\{X_n \to X\} = \{\omega : X_n(\omega) \to X(\omega)\}`$ is an **event**.
> - **In probability:** $`X_n \xrightarrow{P} X`$ if for every $`\epsilon > 0`$,
> ```math
> \mathbb{P}\big(\lvert X_n - X\rvert < \epsilon\big) \to 1 \quad (n \to \infty).
> ```
> - **In distribution (weakly):** $`X_n \xrightarrow{d} X`$ if
> ```math
> F_n(x) \to F(x) \quad \text{for every } x \text{ at which } F \text{ is continuous.}
> ```
> - **i.i.d.:** independent and identically distributed.
> - **Little-o:** $`f(h) = o(g(h))`$ as $`h \to 0`$ means $`f(h)/g(h) \to 0`$; $`a_n = o(b_n)`$ means $`a_n/b_n \to 0`$.

### 2.1 Three ways of being "close"
1. A particular realisation: we observed $`\lvert X - Y\rvert < \epsilon`$.
2. A statement about the **joint** distribution: e.g. $`\mathbb{P}(\lvert X - Y\rvert < \epsilon) > 1 - \epsilon`$ or $`\mathbb{E}\lvert X-Y\rvert < \epsilon`$.
3. A comparison of **distributions**: e.g. $`\lvert F_X(x) - F_Y(x)\rvert < \epsilon`$ for all $`x`$.

These correspond to a.s., in probability, and in distribution.

### 2.2 Convergence in distribution
- Only the **distributions** of the $`X_n`$ and $`X`$ matter: they need not even live on the same probability space. It's really convergence of distributions. So we write things like $`X_n \xrightarrow{d} N(0,1)`$, or $`X_n \xrightarrow{d} 0`$ when the limit is the constant 0.
- **Why only continuity points?**

**Example 2.1.** $`X_n \sim U[-1/n, 1/n]`$. Then $`F_n(x) \to 0`$ for $`x < 0`$ and $`\to 1`$ for $`x > 0`$, so $`X_n \xrightarrow{d} 0`$. But $`F_n(0) = 1/2`$ for all $`n`$ while $`F(0) = 1`$. That's fine, because 0 isn't a continuity point of $`F`$.

**Example 2.2.** $`X_n \equiv 1/n`$ and $`X \equiv 0`$. Then $`X_n \xrightarrow{d} X`$, even though $`\mathbb{P}(X_n \le 0) = 0`$ for all $`n`$ while $`\mathbb{P}(X \le 0) = 1`$.

**Example 2.3 (geometric → exponential).** $`X_n \sim \mathrm{Geom}(p_n)`$ on $`\{1,2,\dots\}`$ with $`p_n \to 0`$. Then $`p_n X_n \xrightarrow{d} \mathrm{Exp}(1)`$.

*Proof.* $`\mathbb{P}(X_n > k) = (1-p_n)^k`$. For $`x \ge 0`$:
```math
\mathbb{P}(p_n X_n > x) = \mathbb{P}\Big(X_n > \big\lfloor x/p_n \big\rfloor\Big) = (1-p_n)^{\lfloor x/p_n\rfloor} \to e^{-x},
```
using $`(1-\epsilon)^{x/\epsilon} \to e^{-x}`$ as $`\epsilon \to 0`$ and that $`\lfloor x/p_n\rfloor - x/p_n`$ is bounded. So $`F_n(x) \to 1 - e^{-x}`$ for $`x > 0`$ and $`F_n(x) = 0`$ for $`x \le 0`$. That is the $`\mathrm{Exp}(1)`$ cdf. ∎

### 2.3 Comparing the modes

> **Theorem 2.4.**
> ```math
> X_n \to X \text{ a.s.} \;\Longrightarrow\; X_n \xrightarrow{P} X \;\Longrightarrow\; X_n \xrightarrow{d} X,
> ```
> and the reverse implications do **not** hold in general.

**Lemma 2.5 (continuity of probability).** If $`A_1 \subseteq A_2 \subseteq \cdots`$ are increasing events, then
```math
\lim_{n \to \infty}\mathbb{P}(A_n) = \mathbb{P}\Big(\bigcup_{n \ge 1} A_n\Big).
```
*Proof.* Write the union as a **disjoint** union and use countable additivity:
```math
\begin{aligned}
\mathbb{P}\Big(\bigcup_{n} A_n\Big) &= \mathbb{P}\Big(A_1 \cup \bigcup_{i \ge 1}(A_{i+1}\setminus A_i)\Big) = \mathbb{P}(A_1) + \sum_{i=1}^{\infty}\mathbb{P}(A_{i+1}\setminus A_i) \\
&= \lim_{n\to\infty}\Big(\mathbb{P}(A_1) + \sum_{i=1}^{n-1}\mathbb{P}(A_{i+1}\setminus A_i)\Big) = \lim_{n\to\infty}\mathbb{P}(A_n). \qquad ∎
\end{aligned}
```

**Proof of Theorem 2.4.**

**(1) In probability ⇒ in distribution.** Fix a continuity point $`x`$ of $`F`$ and $`\epsilon > 0`$. If $`X_n \le x`$ then either $`X \le x + \epsilon`$ or $`\lvert X_n - X\rvert > \epsilon`$. So
```math
F_n(x) \le \mathbb{P}(X \le x+\epsilon) + \mathbb{P}(\lvert X_n - X\rvert > \epsilon) \to F(x+\epsilon),
```
giving $`F_n(x) < F(x+\epsilon) + \epsilon`$ for large $`n`$. Similarly (look at $`1 - F_n(x) = \mathbb{P}(X_n > x)`$), $`F_n(x) > F(x-\epsilon) - \epsilon`$ for large $`n`$. Since $`\epsilon`$ is arbitrary and $`F`$ is continuous at $`x`$, $`F_n(x) \to F(x)`$.

**(2) In distribution ⇏ in probability.** Take $`X \overset{d}{=} Y`$ with $`\mathbb{P}(X = Y) < 1`$. The constant sequence $`X, X, X, \dots`$ converges to $`Y`$ in distribution but not in probability.

**(3) a.s. ⇒ in probability.** Fix $`\epsilon > 0`$. Let $`A_N = \{\lvert X_n - X\rvert < \epsilon \text{ for all } n \ge N\}`$. If $`X_n \to X`$ then some $`A_N`$ occurs, so $`\mathbb{P}(\bigcup_N A_N) = 1`$. The $`A_N`$ increase, so by Lemma 2.5 $`\mathbb{P}(A_N) \to 1`$. And $`A_N \subseteq \{\lvert X_N - X\rvert < \epsilon\}`$, so $`\mathbb{P}(\lvert X_N - X\rvert < \epsilon) \to 1`$.

**(4) In probability ⇏ a.s.** Take **independent** $`X_n`$ with $`\mathbb{P}(X_n = 1) = 1/n`$ and $`\mathbb{P}(X_n = 0) = (n-1)/n`$.
- In probability: $`\mathbb{P}(\lvert X_n\rvert < \epsilon) \ge \mathbb{P}(X_n = 0) \to 1`$.
- Not a.s.: $`\{X_n \to 0\} = \{X_n = 0 \text{ eventually}\} = \bigcup_N B_N`$ with $`B_N = \{X_n = 0\ \forall n \ge N\}`$. For any $`K`$,
```math
\mathbb{P}(B_N) \le \prod_{n=N}^{N+K}\frac{n-1}{n} = \frac{N-1}{N}\cdot\frac{N}{N+1}\cdots\frac{N+K-1}{N+K} = \frac{N-1}{N+K} \xrightarrow{K\to\infty} 0 .
```
So $`\mathbb{P}(B_N) = 0`$, hence (Lemma 2.5) $`\mathbb{P}(\bigcup_N B_N) = 0`$, i.e. $`\mathbb{P}(X_n \to 0) = 0`$. ∎

**Theorem 2.6 (partial converse).** If $`X_n \xrightarrow{d} c`$ for a **constant** $`c`$ (all $`X_n`$ on one space), then $`X_n \xrightarrow{P} c`$. *(Proof on problem sheet: $`\mathbb{P}(\lvert X_n - c\rvert \ge \epsilon) \le F_n(c-\epsilon) + 1 - F_n(c+\epsilon/2) \to 0 + 1 - 1`$, since $`c-\epsilon`$ and $`c+\epsilon/2`$ are continuity points of the limit cdf.)*

### 2.4 Markov, Chebyshev and the weak law

**Markov's inequality.** If $`X \ge 0`$ (i.e. $`\mathbb{P}(X \ge 0) = 1`$) then for any $`z > 0`$:
```math
\mathbb{P}(X \ge z) \le \frac{\mathbb{E}X}{z}.
```
*Proof.* Let $`X_z = z\,\mathbf{1}\{X \ge z\}`$. Since $`X \ge 0`$, $`X \ge X_z`$ always. So $`\mathbb{E}X \ge \mathbb{E}X_z = z\,\mathbb{P}(X \ge z)`$. ∎

**Chebyshev's inequality.** If $`Y`$ has finite mean and variance, then for any $`\epsilon > 0`$:
```math
\mathbb{P}\big(\lvert Y - \mathbb{E}Y\rvert \ge \epsilon\big) \le \frac{\mathrm{var}(Y)}{\epsilon^2}.
```
*Proof.* $`\mathbb{P}(\lvert Y - \mathbb{E}Y\rvert \ge \epsilon) = \mathbb{P}\big((Y-\mathbb{E}Y)^2 \ge \epsilon^2\big) \le \mathbb{E}\big[(Y-\mathbb{E}Y)^2\big]/\epsilon^2`$, by Markov with $`X = (Y-\mathbb{E}Y)^2`$ and $`z = \epsilon^2`$. ∎

> **Weak Law of Large Numbers (WLLN).** If $`X_1, X_2, \dots`$ are i.i.d. with finite mean $`\mu`$ and $`S_n = X_1 + \dots + X_n`$, then $`S_n/n \xrightarrow{P} \mu`$:
> ```math
> \forall \epsilon > 0: \quad \mathbb{P}\Big(\Big\lvert \frac{S_n}{n} - \mu\Big\rvert < \epsilon\Big) \to 1 \quad (n \to \infty).
> ```
> (By Theorem 2.6 we could equally write $`S_n/n \xrightarrow{d} \mu`$.)

*Proof (assuming finite variance $`\sigma^2`$).* $`\mathbb{E}(S_n/n) = \mu`$ and, by independence,
```math
\mathrm{var}\Big(\frac{S_n}{n}\Big) = \frac{\mathrm{var}\,X_1 + \dots + \mathrm{var}\,X_n}{n^2} = \frac{n\sigma^2}{n^2} = \frac{\sigma^2}{n}.
```
Chebyshev: $`\mathbb{P}(\lvert S_n/n - \mu\rvert \ge \epsilon) \le \sigma^2/(n\epsilon^2) \to 0`$. ∎ *(An alternative proof via mgfs is in §3.2.2 of [Part 2](A8-2-Generating-Functions-and-Joint-Densities.md). It assumes the mgf is finite near 0, which is **stronger** than finite variance. The finite-mean-only case is stated in the notes but not proved.)*

**Remark.** The same proof works much more generally: identical distributions are never used, and independence can be dropped as long as $`\mathrm{var}(S_n)`$ grows slower than $`n^2`$. E.g. with bounded variances and $`\sum_{i<j}\mathrm{cov}(X_i,X_j) = o(n^2)`$, we get $`(S_n - \mathbb{E}S_n)/n \xrightarrow{P} 0`$.

### 2.5 Strong law of large numbers

> **Strong Law of Large Numbers (SLLN).** If $`X_1, X_2, \dots`$ are i.i.d. with mean $`\mu`$, then
> ```math
> \frac{S_n}{n} \to \mu \quad \text{almost surely}.
> ```

**Proof assuming $`\mathbb{E}X_n^4 < \infty`$** *(non-examinable, but a nice technique)*. Centre: $`W_n = X_n - \mu`$, so $`\mathbb{E}W_n = 0`$ and $`\mathbb{E}W_n^4 < \infty`$ (exercise: $`\mathbb{E}X^4 < \infty \Rightarrow \mathbb{E}W^4 < \infty`$), and $`(\mathbb{E}W_n^2)^2 = \mathbb{E}W_n^4 - \mathrm{var}(W_n^2) \le \mathbb{E}W_n^4`$. Expand $`\mathbb{E}[(S_n - n\mu)^4] = \mathbb{E}[(W_1 + \dots + W_n)^4]`$:
```math
\sum_{i}\mathbb{E}W_i^4 + 4\sum_{i \ne j}\mathbb{E}W_i^3W_j + 3\sum_{i \ne j}\mathbb{E}W_i^2W_j^2 + 6\!\!\sum_{i,j,k \text{ distinct}}\!\!\mathbb{E}W_i^2W_jW_k + \!\!\sum_{i,j,k,l \text{ distinct}}\!\!\mathbb{E}W_iW_jW_kW_l
```
(the exact constants don't matter). By independence and $`\mathbb{E}W_i = 0`$, every term containing some $`W_i`$ to the first power vanishes (e.g. $`\mathbb{E}W_i^3W_j = \mathbb{E}W_i^3\,\mathbb{E}W_j = 0`$). Only $`\mathbb{E}W_i^4`$ and $`\mathbb{E}W_i^2W_j^2`$ terms survive:
```math
\mathbb{E}\big[(S_n - n\mu)^4\big] = n\,\mathbb{E}W_1^4 + 3n(n-1)\big(\mathbb{E}W_1^2\big)^2 \le 3n^2\,\mathbb{E}W_1^4 .
```
Hence
```math
\mathbb{E}\Big[\sum_{n=1}^{\infty}\Big(\frac{S_n}{n} - \mu\Big)^4\Big] = \sum_{n=1}^\infty \frac{\mathbb{E}\big[(S_n - n\mu)^4\big]}{n^4} \le \sum_{n=1}^\infty \frac{3\,\mathbb{E}W_1^4}{n^2} < \infty .
```
A random variable with finite mean is finite a.s., so $`\sum_n (S_n/n - \mu)^4 < \infty`$ a.s. The terms of a convergent series tend to 0, so $`S_n/n \to \mu`$ a.s. ∎

### 2.6 Central limit theorem

> **Central Limit Theorem (CLT).** If $`X_1, X_2, \dots`$ are i.i.d. with mean $`\mu`$ and variance $`\sigma^2 \in (0, \infty)`$, and $`S_n = X_1 + \dots + X_n`$, then
> ```math
> \frac{S_n - n\mu}{\sigma\sqrt{n}} \xrightarrow{d} N(0,1) \quad (n \to \infty).
> ```
> *(Proved in Part 2 §3.2.2 using mgfs, under the extra assumption that the mgf is finite near 0. The characteristic-function version (§3.4) needs only finite variance.)*

**Read it in three stages (Remark 2.7):**
1. $`S_n`$ concentrates around $`n\mu`$ (LLN);
2. its fluctuations around $`n\mu`$ are of order $`\sqrt{n}`$;
3. the shape of those fluctuations is **normal**, whatever the distribution of $`X_i`$ (universality).

Remembering only "it's normal" without the centring $`n\mu`$ and scale $`\sigma\sqrt n`$ is useless!

**Equivalent forms (Remark 2.8):** for $`Z \sim N(0,1)`$ and all $`a \le b`$,
```math
\mathbb{P}\Big(a \le \frac{S_n - n\mu}{\sigma\sqrt{n}} \le b\Big) \to \mathbb{P}(a \le Z \le b) = \Phi(b) - \Phi(a), \qquad \frac{S_n - n\mu}{\sqrt{n}} \xrightarrow{d} N(0, \sigma^2).
```
First form: apply the definition at $`b`$ and $`a`$ (continuity points, since $`\Phi`$ is continuous) and subtract. Second form: $`\sigma Z \sim N(0, \sigma^2)`$.

**Example 2.9 (insurance reserves, risk pooling).** $`n = 10{,}000`$ policies, each claim with mean £240 and sd £800. What reserve is exceeded by total claims $`S_n`$ with probability only 1%? With $`\Phi^{-1}(0.99) = 2.326`$:
```math
\mathbb{P}\Big(\frac{S_n - n\mu}{\sigma\sqrt n} < \Phi^{-1}(0.99)\Big) \approx 0.99 \;\Longrightarrow\; \text{reserve} \approx n\mu + \Phi^{-1}(0.99)\,\sigma\sqrt n = £2{,}586{,}080 .
```
That's £258.61 per customer, only £18.61 above $`\mu`$. The surplus per customer scales like $`n^{-1/2}`$: 10× bigger for 100 customers, 10× smaller for $`10^6`$. This is **risk pooling**. It relies on **independence**, which fails for e.g. flood risk (many properties hit at once).

**Example 2.10 (Binomial: CLT vs Poisson limit).**
- **Fixed $`p`$:** $`Y_n \sim \mathrm{Bin}(n,p)`$ is a sum of $`n`$ i.i.d. Bernoulli($`p`$), each with mean $`p`$ and variance $`p(1-p)`$, so $`\dfrac{Y_n - np}{\sqrt n} \xrightarrow{d} N\big(0, p(1-p)\big)`$.
- **Rare events:** $`W_n \sim \mathrm{Bin}(n, p_n)`$ with $`\lambda_n = np_n \to \lambda`$. Then $`W_n \xrightarrow{d} \mathrm{Poisson}(\lambda)`$. It suffices to show $`\mathbb{P}(W_n = k) \to \frac{\lambda^k}{k!}e^{-\lambda}`$ for each fixed $`k`$:
```math
\mathbb{P}(W_n = k) = \binom{n}{k}\Big(\frac{\lambda_n}{n}\Big)^k\Big(1 - \frac{\lambda_n}{n}\Big)^{n-k} = \underbrace{\frac{n(n-1)\cdots(n-k+1)}{n^k}}_{\to 1}\,\frac{\lambda_n^k}{k!}\,\underbrace{\Big(1-\frac{\lambda_n}{n}\Big)^{n}}_{\to e^{-\lambda}}\,\underbrace{\Big(1-\frac{\lambda_n}{n}\Big)^{-k}}_{\to 1} \to \frac{\lambda^k}{k!}e^{-\lambda}.
```

> [!TIP]
> **✅ Key takeaways**
> 1. **a.s. ⇒ in probability ⇒ in distribution**, and neither reverses (except: in distribution to a **constant** ⇒ in probability).
> 2. Convergence in distribution is about **distributions only** and ignores discontinuity points of the limit cdf.
> 3. **Markov** ($`\mathbb{P}(X \ge z) \le \mathbb{E}X/z`$ for $`X \ge 0`$) is the mother inequality. Chebyshev is Markov applied to $`(Y - \mathbb{E}Y)^2`$, and Chernoff (Part 2) is Markov applied to $`e^{tX}`$.
> 4. WLLN: $`S_n/n \to \mu`$ in probability. SLLN: almost surely. CLT: fluctuations of size $`\sigma\sqrt n`$, normally distributed.
> 5. Binomial limits: fixed $`p`$ ⇒ normal (CLT); $`np_n \to \lambda`$ ⇒ Poisson.
> 6. Continuity of probability (Lemma 2.5) turns "for some $`N`$" events into limits; it's used again for hitting probabilities in Markov chains.

> [!WARNING]
> **⚠️ Common misconceptions & errors**
> - ❌ "$`X_n \xrightarrow{d} X`$ means $`X_n`$ and $`X`$ are close." ✅ Only their *distributions* are close; they need not be on the same space.
> - ❌ "Need $`F_n(x) \to F(x)`$ for **all** $`x`$." ✅ Only at continuity points of $`F`$ (Example 2.2 fails at $`x = 0`$).
> - ❌ "In probability ⇒ a.s." ✅ Counterexample: independent $`\mathrm{Bernoulli}(1/n)`$.
> - ❌ "Markov's inequality works for any $`X`$." ✅ Needs $`X \ge 0`$.
> - ❌ "CLT: $`S_n \approx N(\mu, \sigma^2)`$." ✅ $`S_n \approx N(n\mu, n\sigma^2)`$, i.e. $`\bar X_n \approx N(\mu, \sigma^2/n)`$.
> - ❌ "CLT applies to anything." ✅ Needs i.i.d. and **finite variance** $`\sigma^2 > 0`$. Cauchy samples don't even satisfy the LLN (Part 2).
> - ❌ Forgetting **independence** when applying CLT/risk pooling in models (floods!).
> - ❌ In proofs of a.s. statements, confusing "$`\mathbb{P}(A_N) \to 1`$" with "$`\mathbb{P}(\bigcup A_N) = 1`$". You need monotonicity (Lemma 2.5) to pass between them.
