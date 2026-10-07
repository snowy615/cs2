# A8 Probability · Formula Sheet and Misconceptions

*Quick-reference companion to the [Overview](A8-Probability-Summary.md) · [Part 1](A8-1-Foundations-and-Limit-Theorems.md) · [Part 2](A8-2-Generating-Functions-and-Joint-Densities.md) · [Part 3a](A8-3a-Markov-Chains.md) · [Part 3b](A8-3b-Markov-Chains-Long-Run.md). Entries marked † are standard facts not derived explicitly in the notes.*

---

## 1. Distributions

### Discrete
| Distribution | pmf | Mean | Variance | pgf $`G(s)`$ | mgf $`M(t)`$ |
|---|---|---|---|---|---|
| Bernoulli($`p`$) | $`p_X(1) = p,\ p_X(0) = 1-p`$ | $`p`$ | $`p(1-p)`$ | $`1-p+ps`$ † | $`1-p+pe^t`$ † |
| Binomial($`n,p`$) | $`\binom nk p^k(1-p)^{n-k}`$ | $`np`$ | $`np(1-p)`$ | $`(1-p+ps)^n`$ † | $`(1-p+pe^t)^n`$ † |
| Geometric($`p`$) on $`\{1,2,\dots\}`$ | $`(1-p)^{k-1}p`$ | $`1/p`$ † | $`(1-p)/p^2`$ † | $`\dfrac{ps}{1-(1-p)s}`$ † | $`\dfrac{pe^t}{1-(1-p)e^t}`$, $`t < -\ln(1-p)`$ † |
| Geometric($`p`$) on $`\{0,1,\dots\}`$ | $`(1-p)^{k}p`$ | $`(1-p)/p`$ † | $`(1-p)/p^2`$ † | $`\dfrac{p}{1-(1-p)s}`$ † | $`\dfrac{p}{1-(1-p)e^t}`$, $`t < -\ln(1-p)`$ † |
| Poisson($`\lambda`$) | $`e^{-\lambda}\lambda^r/r!`$ | $`\lambda`$ | $`\lambda`$ | $`e^{\lambda(s-1)}`$ † | $`e^{\lambda(e^t-1)}`$ † |
| Uniform on $`\{1,\dots,n\}`$ | $`1/n`$ | $`\frac{n+1}{2}`$ † | $`\frac{n^2-1}{12}`$ † | $`\frac{s(1-s^n)}{n(1-s)}`$ † | |

Geometric tail: $`\mathbb{P}(X > k) = (1-p)^k`$ (support $`\{1,2,\dots\}`$). Memoryless: $`\mathbb{P}(X > k+m \mid X > k) = \mathbb{P}(X > m)`$.

### Continuous
| Distribution | pdf | Mean | Variance | mgf $`M(t)`$ | cf $`\phi(t)`$ |
|---|---|---|---|---|---|
| $`U[a,b]`$ | $`\frac{1}{b-a}`$ on $`[a,b]`$ | $`\frac{a+b}2`$ † | $`\frac{(b-a)^2}{12}`$ † | $`\frac{e^{tb}-e^{ta}}{t(b-a)}`$ ($`t \ne 0`$) † | |
| $`\mathrm{Exp}(\lambda)`$ | $`\lambda e^{-\lambda x}`$, $`x \ge 0`$ | $`1/\lambda`$ | $`1/\lambda^2`$ | $`\frac{\lambda}{\lambda - t}`$, **$`t < \lambda`$** | $`\frac{\lambda}{\lambda - it}`$ |
| $`\mathrm{Gamma}(r,\lambda)`$ | $`\frac{\lambda^r}{\Gamma(r)}x^{r-1}e^{-\lambda x}`$, $`x \ge 0`$ | $`r/\lambda`$ † | $`r/\lambda^2`$ † | $`\big(\frac{\lambda}{\lambda-t}\big)^r`$, $`t < \lambda`$ † | |
| $`N(\mu,\sigma^2)`$ | $`\frac{1}{\sqrt{2\pi\sigma^2}}e^{-\frac{(x-\mu)^2}{2\sigma^2}}`$ | $`\mu`$ | $`\sigma^2`$ | $`e^{\mu t + \sigma^2t^2/2}`$ | $`e^{i\mu t - \sigma^2t^2/2}`$ |
| Cauchy | $`\frac{1}{\pi(1+x^2)}`$ | **none** | **none** | $`\infty`$ for $`t \ne 0`$ | $`e^{-\lvert t\rvert}`$ |
| Laplace (Ex. 4.3) | $`\frac12\lambda e^{-\lambda\lvert w\rvert}`$ | $`0`$ † | $`2/\lambda^2`$ † | $`\frac{\lambda^2}{\lambda^2 - t^2}`$, $`\lvert t\rvert < \lambda`$ † | |

$`\Gamma(r) = \int_0^\infty z^{r-1}e^{-z}dz`$, with $`\Gamma(n) = (n-1)!`$. Exp cdf $`F(x) = 1 - e^{-\lambda x}`$. Memoryless: $`\mathbb{P}(X > x+t \mid X > t)`$ doesn't depend on $`t`$.

### Closure facts (independent summands)
| Sum of independent... | is... |
|---|---|
| $`\mathrm{Bin}(n,p) + \mathrm{Bin}(m,p)`$ † | $`\mathrm{Bin}(n+m, p)`$ |
| $`\mathrm{Po}(\lambda) + \mathrm{Po}(\mu)`$ | $`\mathrm{Po}(\lambda+\mu)`$ |
| $`N(\mu_1,\sigma_1^2) + N(\mu_2,\sigma_2^2)`$ | $`N(\mu_1+\mu_2, \sigma_1^2+\sigma_2^2)`$ |
| $`\mathrm{Gamma}(r_1,\lambda) + \mathrm{Gamma}(r_2,\lambda)`$ | $`\mathrm{Gamma}(r_1+r_2,\lambda)`$ |
| $`n`$ i.i.d. $`\mathrm{Exp}(\lambda)`$ | $`\mathrm{Gamma}(n,\lambda)`$ |
| $`n`$ i.i.d. Cauchy, divided by $`n`$ | Cauchy (same!) |

Scaling: $`aX + b \sim N(a\mu+b, a^2\sigma^2)`$; $`X \sim \mathrm{Exp}(\lambda) \Rightarrow aX \sim \mathrm{Exp}(\lambda/a)`$ for $`a > 0`$.

---

## 2. Expectation, variance, inequalities
```math
\operatorname{var}X = \mathbb{E}X^2 - (\mathbb{E}X)^2, \qquad \operatorname{cov}(X,Y) = \mathbb{E}XY - \mathbb{E}X\,\mathbb{E}Y, \qquad \operatorname{var}(aX+b) = a^2\operatorname{var}X
```
```math
\operatorname{var}\Big(\sum_i X_i\Big) = \sum_i \operatorname{var}X_i + 2\sum_{i<j}\operatorname{cov}(X_i,X_j), \qquad \rho = \frac{\operatorname{cov}(X,Y)}{\sigma_X\sigma_Y} \in [-1,1]
```
| Inequality | Statement | Condition |
|---|---|---|
| Markov | $`\mathbb{P}(X \ge z) \le \mathbb{E}X/z`$ | $`X \ge 0`$, $`z > 0`$ |
| Chebyshev | $`\mathbb{P}(\lvert Y - \mathbb{E}Y\rvert \ge \epsilon) \le \operatorname{var}Y/\epsilon^2`$ | finite variance |
| Chernoff | $`\mathbb{P}(X \ge a) \le e^{-ta}M_X(t)`$, then optimise over $`t > 0`$ | mgf finite |
| SRW Chernoff | $`\mathbb{P}(\lvert S_n\rvert > na) \le 2e^{-na^2/2}`$ | $`\pm1`$ steps, uses $`\cosh t \le e^{t^2/2}`$ |

---

## 3. Convergence and limit theorems
```math
X_n \to X \text{ a.s.} \;\Longrightarrow\; X_n \xrightarrow{P} X \;\Longrightarrow\; X_n \xrightarrow{d} X, \qquad X_n \xrightarrow{d} c \text{ (constant)} \;\Longrightarrow\; X_n \xrightarrow{P} c
```
| Result | Statement (i.i.d., $`S_n = X_1 + \dots + X_n`$) | Assumption |
|---|---|---|
| WLLN | $`S_n/n \xrightarrow{P} \mu`$ | finite mean (proof: finite variance + Chebyshev, or mgf) |
| SLLN | $`S_n/n \to \mu`$ a.s. | finite mean (proof shown with $`\mathbb{E}X^4 < \infty`$) |
| CLT | $`\dfrac{S_n - n\mu}{\sigma\sqrt n} \xrightarrow{d} N(0,1)`$ | $`0 < \sigma^2 < \infty`$ |
| Poisson limit | $`\mathrm{Bin}(n,p_n) \xrightarrow{d} \mathrm{Po}(\lambda)`$ | $`np_n \to \lambda`$ |
| Geometric → Exp | $`p_nX_n \xrightarrow{d} \mathrm{Exp}(1)`$ | $`X_n \sim \mathrm{Geom}(p_n)`$, $`p_n \to 0`$ |

**Continuity of probability:** $`A_n \uparrow \;\Rightarrow\; \mathbb{P}(A_n) \to \mathbb{P}(\bigcup A_n)`$. **Useful limits:** $`(1 + \frac an + o(\frac1n))^n \to e^a`$; $`(1-\epsilon)^{x/\epsilon} \to e^{-x}`$; Stirling $`n! \sim \sqrt{2\pi}\,n^{n+1/2}e^{-n}`$; $`\binom{2m}{m}4^{-m} \sim \frac{1}{\sqrt{\pi m}}`$.

---

## 4. Generating functions
| | pgf $`G(s) = \mathbb{E}s^X`$ | mgf $`M(t) = \mathbb{E}e^{tX}`$ | cf $`\phi(t) = \mathbb{E}e^{itX}`$ |
|---|---|---|---|
| Defined for | $`X \in \{0,1,\dots\}`$, $`\lvert s\rvert \le 1`$ at least | where finite (need a neighbourhood of 0) | all $`t`$, $`\lvert\phi\rvert \le 1`$ |
| Moments | $`G^{(k)}(1) = \mathbb{E}[X(X-1)\cdots(X-k+1)]`$ | $`M^{(k)}(0) = \mathbb{E}X^k`$ | $`\phi^{(k)}(0) = i^k\mathbb{E}X^k`$ |
| Probabilities | $`p_X(k) = G^{(k)}(0)/k!`$ | | |
| Independent sum | $`G_XG_Y`$ | $`M_XM_Y`$ | $`\phi_X\phi_Y`$ |
| Linear map $`aX+b`$ | | $`e^{bt}M_X(at)`$ | $`e^{ibt}\phi_X(at)`$ † |
| Random sum $`X_1+\dots+X_N`$ | $`G_N(G_X(s))`$ | $`G_N(M_X(t))`$ † | |
| Uniqueness / continuity | yes | yes, if finite on $`[-t_0,t_0]`$ | yes, always |

```math
\mathbb{E}X = G'(1), \qquad \operatorname{var}X = G''(1) + G'(1) - G'(1)^2, \qquad \mathbb{E}S = \mathbb{E}N\,\mathbb{E}X \ \text{(random sum)} †
```

---

## 5. Joint densities
```math
f_X(x) = \int f_{X,Y}(x,y)\,dy, \qquad \mathbb{E}h(X,Y) = \iint h\,f_{X,Y}, \qquad \text{indep.} \iff f_{X,Y} = f_Xf_Y
```
```math
f_{U,V}(u,v) = f_{X,Y}\big(x(u,v),y(u,v)\big)\,\lvert J(u,v)\rvert, \qquad J = \frac{\partial x}{\partial u}\frac{\partial y}{\partial v} - \frac{\partial x}{\partial v}\frac{\partial y}{\partial u}
```
```math
f_{X+Y}(u) = \int f_{X,Y}(v, u-v)\,dv \overset{\text{indep}}{=} \int f_X(v)f_Y(u-v)\,dv, \qquad f_{Y\mid X=x}(y) = \frac{f_{X,Y}(x,y)}{f_X(x)}
```
**Multivariate normal:** $`\mathbf{W} = A\mathbf{Z} + \boldsymbol\mu`$, $`\Sigma = AA^T`$, $`f_{\mathbf W}(\mathbf w) = \dfrac{1}{(2\pi)^{n/2}\lvert\det A\rvert}e^{-\frac12(\mathbf w-\boldsymbol\mu)^T\Sigma^{-1}(\mathbf w-\boldsymbol\mu)}`$.

**Bivariate normal conditional:** $`Y \mid X = x \sim N\big(\mu_2 + \rho\frac{\sigma_2}{\sigma_1}(x-\mu_1),\ (1-\rho^2)\sigma_2^2\big)`$. Jointly normal: independent ⇔ $`\rho = 0`$.

**Classic transforms:** for $`X, Y`$ i.i.d. $`\mathrm{Exp}(\lambda)`$: $`\frac{X}{X+Y} \sim U[0,1]`$ independent of $`X+Y \sim \mathrm{Gamma}(2,\lambda)`$; $`X - Y`$ is Laplace and $`\lvert X - Y\rvert \sim \mathrm{Exp}(\lambda)`$. In polar coordinates, uniform on the unit (half-)disc has $`f_R(r) = 2r`$, with $`\Theta`$ uniform and independent of $`R`$.

---

## 6. Markov chains
```math
\mathbb{P}(X_0 = i_0, \dots, X_n = i_n) = \lambda_{i_0}p_{i_0i_1}\cdots p_{i_{n-1}i_n}, \qquad p^{(n+m)}_{ij} = \sum_k p^{(n)}_{ik}p^{(m)}_{kj}, \qquad p^{(n)}_{ij} = (P^n)_{ij}, \qquad X_n \sim \lambda P^n
```
| Quantity | Equations (minimal non-negative solution) |
|---|---|
| Hitting probability $`h_i^A`$ | $`h_i = 1`$ ($`i \in A`$); $`h_i = \sum_j p_{ij}h_j`$ ($`i \notin A`$) |
| Mean hitting time $`k_i^A`$ | $`k_i = 0`$ ($`i \in A`$); $`k_i = 1 + \sum_j p_{ij}k_j`$ ($`i \notin A`$) |
| Mean return time $`m_i`$ | $`m_i = 1 + \sum_j p_{ij}k_j^{\{i\}}`$; positive recurrent irreducible: $`m_i = 1/\pi_i`$ |
| Stationary $`\pi`$ | $`\pi P = \pi`$, $`\sum_i\pi_i = 1`$ |
| Detailed balance | $`\pi_ip_{ij} = \pi_jp_{ji}`$ (⇒ stationary, reversible) |
| Time-reversal | $`q_{ij} = \pi_jp_{ji}/\pi_i`$ |

**Recurrence:** recurrent $`\iff \sum_n p^{(n)}_{ii} = \infty`$. **Classes:** recurrence, transience, null/positive recurrence and period are class properties; recurrent ⇒ closed; finite & closed ⇒ recurrent.

**Two-state chain** $`\begin{pmatrix}1-\alpha&\alpha\\\beta&1-\beta\end{pmatrix}`$: $`p^{(n)}_{11} = \frac{\beta}{\alpha+\beta} + \frac{\alpha}{\alpha+\beta}(1-\alpha-\beta)^n`$, and $`\pi = \big(\frac{\beta}{\alpha+\beta}, \frac{\alpha}{\alpha+\beta}\big)`$.

**Gambler's ruin** (up $`p`$, down $`q`$, absorbed at 0):
| | $`h_i = \mathbb{P}_i(\text{hit }0)`$ | $`k_i = \mathbb{E}_i(\text{time to }0)`$ |
|---|---|---|
| $`p < q`$ | $`1`$ | $`i/(q-p)`$ |
| $`p = q`$ | $`1`$ | $`\infty`$ |
| $`p > q`$ | $`(q/p)^i`$ | $`\infty`$ |

**Reflected walk on $`\mathbb{N}`$** ($`p < q`$): $`\pi_i = (1-\frac pq)(\frac pq)^i`$. **Random walk on a graph:** $`\pi_i = d_i/\sum_j d_j`$ (reversible). **Cycle:** uniform $`\pi`$; reversible iff $`p = \frac12`$.

**Pólya:** the SRW on $`\mathbb{Z}^d`$ has $`p^{(2m)}_{00} \asymp m^{-d/2}`$. It is **recurrent for $`d = 1,2`$** (null recurrent †) and **transient for $`d \ge 3`$**. In $`d = 1`$, $`p^{(2m)}_{00} \sim (\pi m)^{-1/2}`$; in $`d = 2`$, $`p^{(2m)}_{00} \sim (\pi m)^{-1}`$.

**Main theorems:** irreducible: stationary exists ⇔ positive recurrent, and then $`\pi_i = 1/m_i`$ (unique). + aperiodic: $`p^{(n)}_{ij} \to \pi_j`$. Ergodic (irreducible): $`V_i(n)/n \to 1/m_i`$ a.s.

---

## 7. Concepts you must not confuse
| Pair | Difference |
|---|---|
| a.s. vs in probability vs in distribution | the event $`\{X_n \to X\}`$ has probability 1 / $`\mathbb{P}(\lvert X_n - X\rvert < \epsilon) \to 1`$ for each $`\epsilon`$ / cdfs converge at continuity points (distributions only) |
| WLLN vs SLLN | convergence of $`S_n/n`$ in probability vs almost surely |
| LLN vs CLT | where $`S_n`$ concentrates ($`n\mu`$) vs size ($`\sigma\sqrt n`$) and shape (normal) of fluctuations |
| Uncorrelated vs independent | $`\operatorname{cov} = 0`$ vs everything factorises; not equivalent in general (they are for jointly normal variables) |
| Pairwise vs mutual independence | pairs vs every finite sub-family |
| pgf vs mgf vs cf | non-negative integers vs needs exponential tails vs always exists |
| $`G''(1)`$ vs $`\mathbb{E}X^2`$ | factorial moment $`\mathbb{E}X(X-1)`$ vs second moment |
| Jacobian of $`T`$ vs of $`T^{-1}`$ | the formula uses $`\partial(x,y)/\partial(u,v)`$ (of the inverse), in absolute value |
| Conditioning on $`\mathbb{P}(A) > 0`$ vs on $`X = x`$ | ratio of probabilities vs ratio of densities (a limit; Borel paradox) |
| Closed vs recurrent | equal for finite classes; infinite closed classes may be transient |
| Null vs positive recurrent | $`m_i = \infty`$ vs $`m_i < \infty`$; only positive recurrence gives a stationary distribution |
| Irreducible vs aperiodic | one class vs gcd of return times is 1; convergence of $`p^{(n)}`$ needs both |
| $`\pi P = \pi`$ vs $`P\pi = \pi`$ | left eigenvector (what you want) vs right eigenvector (solved by constants) |
| Stationary vs detailed balance | global balance vs edge-by-edge balance (stronger, ⇒ reversible) |
| Convergence to equilibrium vs ergodic theorem | $`\mathbb{P}(X_n = j) \to \pi_j`$ (needs aperiodic) vs time averages $`V_i(n)/n \to 1/m_i`$ (doesn't) |
| Hitting probability vs mean hitting time | $`h_i = 1`$ can still have $`k_i = \infty`$ (symmetric gambler's ruin) |

## 8. Top 15 mistakes
1. Applying CLT/LLN without independence or without finite variance (Cauchy!).
2. Writing $`S_n \approx N(\mu,\sigma^2)`$ instead of $`N(n\mu, n\sigma^2)`$.
3. Forgetting the **domain** of an mgf ($`\mathrm{Exp}`$: $`t < \lambda`$).
4. Using mgf uniqueness when the mgf isn't finite near 0.
5. Using the forward Jacobian or forgetting $`\lvert\cdot\rvert`$.
6. Forgetting the **support** after a transformation; claiming independence from a factorised formula on a non-product support.
7. Assuming $`\rho = 0`$ ⇒ independent in general. It's fine for **jointly** normal variables; separately normal isn't enough.
8. Conditioning on a probability-zero event as if it were an ordinary event.
9. Columns instead of rows summing to 1; $`P^n\lambda`$ instead of $`\lambda P^n`$.
10. Taking a non-minimal solution of the hitting equations.
11. Deciding transience from $`p^{(n)}_{ii} \to 0`$ instead of from $`\sum_n p^{(n)}_{ii}`$.
12. Claiming convergence to $`\pi`$ for a periodic chain.
13. Claiming a stationary distribution for a null-recurrent chain (symmetric walk on $`\mathbb{Z}`$).
14. Not normalising $`\pi`$; using $`m_i = \pi_i`$ instead of $`1/\pi_i`$.
15. Concluding "not reversible" just because of drift (trees/birth–death chains are reversible).
