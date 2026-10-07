# A8 Probability · Part 3a: Markov Chains — Basics

*Chapter 5 of the Part A Probability notes (James Martin, MT 2025). [← Part 2](A8-2-Generating-Functions-and-Joint-Densities.md) · [Overview](A8-Probability-Summary.md) · [Part 3b →](A8-3b-Markov-Chains-Long-Run.md) · [Formula sheet](A8-4-Formula-Sheet.md)*

---

## Contents
5. [Markov chains: introduction](#5-markov-chains-introduction) (definition, $`n`$-step probabilities, classes, periodicity, hitting probabilities, recurrence, random walks, mean hitting times)
6. Stationary distributions, convergence, time-reversal → [Part 3b](A8-3b-Markov-Chains-Long-Run.md)

---

## 5. Markov chains: introduction

> [!NOTE]
> **📖 Key definitions** (state space $`I`$ finite or countable)
> - **Distribution on $`I`$:** $`\lambda = (\lambda_i)_{i \in I}`$ with $`\lambda_i \ge 0`$ and $`\sum_i \lambda_i = 1`$, written as a **row vector**.
> - **Markov chain:** $`(X_n)_{n \ge 0}`$ such that for all $`n`$ and states (whenever the conditioning event has positive probability):
> ```math
> \mathbb{P}(X_{n+1} = i_{n+1} \mid X_n = i_n, \dots, X_0 = i_0) = \mathbb{P}(X_{n+1} = i_{n+1} \mid X_n = i_n).
> ```
> "The future is independent of the past, given the present" (**Markov property**).
> - **Homogeneous:** $`p_{ij} = \mathbb{P}(X_{n+1} = j \mid X_n = i)`$ doesn't depend on $`n`$. The **transition matrix** $`P = (p_{ij})`$ is **stochastic**: entries $`\ge 0`$, **rows** sum to 1. We write $`X \sim \mathrm{Markov}(\lambda, P)`$; $`\mathbb{P}_i, \mathbb{E}_i`$ mean "given $`X_0 = i`$".
> - **$`n`$-step transition probability:** $`p_{ij}^{(n)} = \mathbb{P}(X_{r+n} = j \mid X_r = i)`$.
> - **$`i`$ leads to $`j`$** ($`i \to j`$): $`p_{ij}^{(n)} > 0`$ for some $`n \ge 0`$. **Communicate** ($`i \leftrightarrow j`$): $`i \to j`$ and $`j \to i`$. This is an equivalence relation whose classes are the **communicating classes**.
> - **Closed class** $`C`$: $`p_{ij} = 0`$ for $`i \in C, j \notin C`$ (you can't leave). Otherwise **open**. **Absorbing state:** $`\{i\}`$ closed, i.e. $`p_{ii} = 1`$.
> - **Irreducible:** a single communicating class ($`i \to j`$ for all $`i,j`$).
> - **Period** of $`i`$: $`\gcd\{n \ge 1 : p_{ii}^{(n)} > 0\}`$. **Aperiodic** if the period is 1, equivalently $`p_{ii}^{(n)} > 0`$ for all sufficiently large $`n`$.
> - **Hitting probability:** $`h_i^A = \mathbb{P}_i(X_n \in A \text{ for some } n \ge 0)`$ (an **absorption probability** if $`A`$ is a closed class).
> - **Hitting time:** $`H^A = \min\{n \ge 0 : X_n \in A\}`$ (may be $`\infty`$); $`h_i^A = \mathbb{P}_i(H^A < \infty)`$. **Mean hitting time:** $`k_i^A = \mathbb{E}_i(H^A)`$.
> - **Recurrent:** $`\mathbb{P}_i(X_n = i \text{ for some } n \ge 1) = 1`$. **Transient:** that probability is $`< 1`$.
> - **Mean return time:** $`m_i = \mathbb{E}_i\big(\min\{n \ge 1 : X_n = i\}\big)`$. **Positive recurrent:** recurrent with $`m_i < \infty`$. **Null recurrent:** recurrent with $`m_i = \infty`$.

### 5.1 Markov chains and path probabilities
Examples of state spaces: $`\mathbb{Z}^2`$ (lattice walk), characters in text, DNA bases $`\{C,G,A,T\}`$. A homogeneous chain is specified by the **initial distribution** $`\lambda`$ and the **transition matrix** $`P`$; row $`i`$ of $`P`$ is the distribution of $`X_{n+1}`$ given $`X_n = i`$.

> **Theorem 5.1.** $`\mathbb{P}(X_0 = i_0, X_1 = i_1, \dots, X_n = i_n) = \lambda_{i_0}\,p_{i_0 i_1}\,p_{i_1 i_2}\cdots p_{i_{n-1}i_n}`$.

*Proof.* Chain rule for conditional probability, then the Markov property on each factor:
```math
\mathbb{P}(X_0 = i_0)\,\mathbb{P}(X_1 = i_1 \mid X_0 = i_0)\cdots\mathbb{P}(X_n = i_n \mid X_{n-1} = i_{n-1}, \dots, X_0 = i_0) = \lambda_{i_0}p_{i_0i_1}\cdots p_{i_{n-1}i_n}. \quad ∎
```

### 5.2 $`n`$-step transition probabilities

> **Theorem 5.2 (Chapman–Kolmogorov).** (i) $`p_{ij}^{(n+m)} = \sum_{k \in I} p_{ik}^{(n)}p_{kj}^{(m)}`$. (ii) $`p_{ij}^{(n)} = (P^n)_{ij}`$.

*Proof.* (i) Partition on the intermediate state $`X_n`$ and use the Markov property:
```math
\mathbb{P}(X_{n+m} = j \mid X_0 = i) = \sum_k \mathbb{P}(X_n = k \mid X_0 = i)\,\mathbb{P}(X_{n+m} = j \mid X_n = k, X_0 = i) = \sum_k p_{ik}^{(n)}p_{kj}^{(m)} .
```
(ii) Induction: $`p^{(2)}_{ij} = \sum_k p_{ik}p_{kj} = (P^2)_{ij}`$, then $`p^{(3)}_{ij} = \sum_k (P^2)_{ik}P_{kj} = (P^3)_{ij}`$, and so on. ∎

> **Theorem 5.3.** If $`X_0 \sim \lambda`$ then $`X_1 \sim \lambda P`$ and $`X_n \sim \lambda P^n`$ (row vector times matrix).

*Proof.* $`\mathbb{P}(X_1 = j) = \sum_i \mathbb{P}(X_0 = i)\,p_{ij} = (\lambda P)_j`$, and similarly with $`P^n`$. ∎ Consequence: $`(X_0, X_k, X_{2k}, \dots)`$ is $`\mathrm{Markov}(\lambda, P^k)`$.

**Example 5.4 (general two-state chain).** $`I = \{1,2\}`$, $`P = \begin{pmatrix}1-\alpha & \alpha\\ \beta & 1-\beta\end{pmatrix}`$.
1. **Diagonalise.** The eigenvalues are $`1`$ and $`1-\alpha-\beta`$ (every stochastic matrix has eigenvalue 1, with right eigenvector $`(1,\dots,1)^T`$). So $`(P^n)_{11} = A + B(1-\alpha-\beta)^n`$. Using $`p^{(0)}_{11} = 1`$ and $`p^{(1)}_{11} = 1-\alpha`$:
```math
p_{11}^{(n)} = \frac{\beta}{\alpha+\beta} + \frac{\alpha}{\alpha+\beta}(1-\alpha-\beta)^n .
```
2. **Condition on step $`n-1`$:** $`p_{11}^{(n)} = p_{11}^{(n-1)}(1-\alpha) + \big(1 - p_{11}^{(n-1)}\big)\beta = (1-\alpha-\beta)p_{11}^{(n-1)} + \beta`$. Solve this linear recurrence; same answer.

### 5.3 Examples
- **Random walk on a cycle** $`\{0, \dots, M-1\}`$: $`p_{i,i+1} = p`$, $`p_{i,i-1} = 1-p`$ (mod $`M`$).
- **Simple symmetric random walk on $`\mathbb{Z}^d`$:** $`p_{ij} = \frac{1}{2d}`$ if $`\lvert i-j\rvert = 1`$, else 0.
- **Card shuffling** on $`S_{52}`$: pick $`a, b`$ uniformly and independently, and swap cards $`a, b`$. So $`p_{\alpha\beta} = \frac{2}{52^2}`$ if $`\alpha = \beta\tau`$ for a transposition $`\tau`$, $`\frac{1}{52}`$ if $`\alpha = \beta`$, else 0.

### 5.4 Exploring the Markov property
With $`X_i`$ i.i.d. $`\pm1`$ ($`\mathbb{P}(X_i = 1) = p`$) and $`S_n = \sum_{i \le n} X_i`$:
1. $`(X_n)`$ is a Markov chain (i.i.d. is stronger); both rows of $`P`$ equal $`(1-p, p)`$ on $`\{-1, 1\}`$.
2. $`(S_n)`$ is a Markov chain with $`p_{i,i+1} = p`$ and $`p_{i,i-1} = 1-p`$. *(The notes print "$`p_{i,i-1} = p`$", a typo.)*
3. The running maximum $`M_n = \max_{k \le n} S_k`$ is **not** Markov. Given $`(M_0, \dots, M_4) = (0,0,0,1,2)`$ we must have $`S_4 = 2`$, so $`\mathbb{P}(M_5 = 3 \mid \cdot) = p`$. Given $`(0,1,2,2,2)`$, both $`S_4 = 2`$ and $`S_4 = 0`$ are possible, so $`\mathbb{P}(M_5 = 3 \mid \cdot) < p`$. **The path to the present state matters.**

> **Proposition 5.8 (criterion for the Markov property).** If $`Y_{n+1} = f(Y_n, X_{n+1})`$ where $`X_{n+1}`$ is independent of $`Y_0, \dots, Y_n`$, then $`(Y_n)`$ is a Markov chain.

*Proof.*
```math
\begin{aligned}
\mathbb{P}(Y_{n+1} = i_{n+1} \mid Y_n = i_n, \dots, Y_0 = i_0) &= \mathbb{P}\big(f(i_n, X_{n+1}) = i_{n+1} \mid Y_n = i_n, \dots, Y_0 = i_0\big)\\
&= \mathbb{P}\big(f(i_n, X_{n+1}) = i_{n+1}\big) \quad (X_{n+1} \perp Y_0,\dots,Y_n)\\
&= \mathbb{P}\big(f(i_n, X_{n+1}) = i_{n+1} \mid Y_n = i_n\big) = \mathbb{P}(Y_{n+1} = i_{n+1} \mid Y_n = i_n). \quad ∎
\end{aligned}
```
*Examples:* random walk $`S_{n+1} = S_n + X_{n+1}`$; shuffling $`Y_{n+1} = Y_n\tau_{n+1}`$ with i.i.d. uniform transpositions $`\tau_n`$. **Idea:** update using only the current state + fresh independent randomness.

### 5.5 Class structure
**Example 5.9.** For the 7-state chain in the notes (from the matrix: 1→2, 1→7; 2→3; 3→1, 3→4; 4→4, 4→5; 5→6; 6→5; 7 absorbing. The notes' diagram differs slightly, but in both 4 does not lead back to 3, so the classes are the same), the classes are $`\{1,2,3\}`$, $`\{4\}`$, $`\{5,6\}`$, $`\{7\}`$. Only $`\{5,6\}`$ and $`\{7\}`$ are **closed**; 7 is absorbing. Once in a closed class, the chain never leaves.

### 5.6 Periodicity
If every transition flips parity (e.g. between $`\{2,4\}`$ and $`\{1,3,5\}`$), then $`p_{ii}^{(n)} = 0`$ for odd $`n`$, so the period is 2. (If $`p_{ii}^{(n)} = 0`$ for all $`n > 0`$, the period is undefined.)

> **Fact.** All states in a communicating class have the **same period**.

*Proof.* Let $`i \leftrightarrow j`$ and suppose $`d \mid n`$ whenever $`p_{ii}^{(n)} > 0`$. Pick $`a, b`$ with $`p_{ij}^{(a)} > 0`$ and $`p_{ji}^{(b)} > 0`$; then $`p_{ii}^{(a+b)} > 0`$, so $`d \mid a+b`$. If $`p_{jj}^{(m)} > 0`$ then $`p_{ii}^{(a+m+b)} > 0`$, so $`d \mid a+m+b`$, hence $`d \mid m`$. So the sets $`\{n : p_{ii}^{(n)} > 0\}`$ and $`\{m : p_{jj}^{(m)} > 0\}`$ have the same common divisors, hence the same gcd. ∎

So an irreducible chain has a well-defined period. Irreducibility and periodicity are **structural**: they depend only on *which* $`p_{ij}`$ are positive.

*Examples:* the cycle walk is irreducible, with period 2 if $`M`$ is even and aperiodic if $`M`$ is odd. The walk on $`\mathbb{Z}^d`$ is irreducible with period 2. Card shuffling is irreducible (transpositions generate $`S_{52}`$) and aperiodic ($`p_{\alpha\alpha} > 0`$).

**Why we'll need irreducible + aperiodic for convergence:** a period-2 chain remembers the parity of its start forever, and a chain with several closed classes remembers which one it's in.

### 5.7 Hitting probabilities

**Example.** $`I = \{1,2,3,4\}`$, with 1 and 4 absorbing, and from 2 or 3 move to each neighbour with probability $`\tfrac12`$. Let $`h_i = \mathbb{P}_i(\text{hit } 4)`$. Then $`h_4 = 1`$, $`h_1 = 0`$, and by **first-step analysis** $`h_2 = \tfrac12 h_1 + \tfrac12 h_3`$ and $`h_3 = \tfrac12 h_2 + \tfrac12 h_4`$. So $`h_2 = \tfrac13`$ and $`h_3 = \tfrac23`$.

> **Theorem 5.10.** $`(h_i^A)_{i \in I}`$ is the **minimal non-negative solution** of
> ```math
> h_i^A = \begin{cases} 1 & i \in A,\\ \sum_j p_{ij}h_j^A & i \notin A.\end{cases}
> ```
> "Minimal": if $`(x_i)`$ is any non-negative solution then $`h_i^A \le x_i`$ for all $`i`$.

*Proof.* **$`h^A`$ solves it** (condition on the first jump): for $`i \notin A`$,
```math
h_i^A = \mathbb{P}_i(X_n \in A \text{ for some } n \ge 1) = \sum_j \mathbb{P}_i(X_1 = j)\,\mathbb{P}(X_n \in A \text{ some } n \ge 1 \mid X_1 = j) = \sum_j p_{ij}h_j^A,
```
using the Markov property. For $`i \in A`$, $`h_i^A = 1`$ by definition.

**Minimality.** Let $`(x_i)`$ be a non-negative solution. Claim: for all $`M`$ and $`i`$,
```math
x_i \ge \mathbb{P}_i(X_n \in A \text{ for some } n \le M).
```
Induction on $`M`$. For $`M = 0`$: the RHS is 1 if $`i \in A`$ (and $`x_i = 1`$), else 0. Step: if $`i \in A`$, it's clear. If $`i \notin A`$:
```math
\mathbb{P}_i(X_n \in A,\ \text{some } n \le M) = \sum_j p_{ij}\,\mathbb{P}_j(X_n \in A,\ \text{some } n \le M-1) \le \sum_j p_{ij}x_j = x_i .
```
These events increase in $`M`$ with union $`\{X_n \in A \text{ for some } n\}`$, so by continuity of probability $`x_i \ge \lim_M \mathbb{P}_i(\cdots \le M) = h_i^A`$. ∎

**Important example: gambler's ruin.** $`I = \{0,1,2,\dots\}`$, $`p_{00} = 1`$, $`p_{i,i+1} = p`$, $`p_{i,i-1} = q = 1-p`$ for $`i \ge 1`$ (capital vs an infinitely rich bank; a **birth-and-death chain**). For $`h_i = \mathbb{P}_i(\text{hit } 0)`$: $`h_0 = 1`$ and $`h_i = p\,h_{i+1} + q\,h_{i-1}`$ for $`i \ge 1`$. For $`p \ne q`$ the general solution is $`h_i = A + B(q/p)^i`$, with $`A + B = 1`$.

| Case | Minimal non-negative solution | Meaning |
|---|---|---|
| $`p < q`$ | $`(q/p)^i \ge 1`$ grows, so take $`A = 1, B = 0`$: $`h_i = 1`$ | ruin is certain |
| $`p > q`$ | need $`A \ge 0`$; minimal is $`A = 0, B = 1`$: $`h_i = (q/p)^i`$ | positive chance of escaping to $`\infty`$ |
| $`p = q`$ | general solution $`A + Bi`$; $`A = 1`$, $`B \ge 0`$, minimal $`B = 0`$: $`h_i = 1`$ | ruin certain, despite no drift |

*Shortcut:* the chain moves down one step at a time, so $`\mathbb{P}_i(\text{hit }0) = \mathbb{P}_i(\text{hit } i-1)\cdots\mathbb{P}_1(\text{hit } 0) = \alpha^i`$ by spatial homogeneity.

### 5.8 Recurrence and transience
If $`\mathbb{P}_i(\text{return}) = p < 1`$, the number of visits to $`i`$ is **geometric** (each return independently has a $`1-p`$ chance of being the last), so $`\mathbb{P}_i(\text{hit } i \text{ infinitely often}) = 0`$: **transient**. If $`p = 1`$, then $`\mathbb{P}_i(\text{hit } i \text{ i.o.}) = 1`$: **recurrent**.

> **Theorem 5.11.** $`i`$ is recurrent $`\iff \sum_{n=0}^{\infty} p_{ii}^{(n)} = \infty`$.

*Proof.* The number of visits to $`i`$ is $`V = \sum_n \mathbf{1}(X_n = i)`$, so $`\mathbb{E}_iV = \sum_n \mathbb{P}_i(X_n = i) = \sum_n p_{ii}^{(n)}`$. If transient, $`V`$ is geometric with mean $`\frac{1}{1-p} < \infty`$. If recurrent, $`V = \infty`$ a.s., so $`\mathbb{E}_iV = \infty`$. ∎

> **Theorem 5.12.** (a) Recurrence/transience is a **class property**. (b) Every **recurrent class is closed**. (c) Every **finite closed class is recurrent**.

*Proof sketch* (sheet 3). (a) If $`i`$ is recurrent and $`p_{ij}^{(a)}, p_{ji}^{(b)} > 0`$, then $`p_{jj}^{(a+n+b)} \ge p_{ji}^{(b)}p_{ii}^{(n)}p_{ij}^{(a)}`$, so $`\sum_n p_{jj}^{(n)} = \infty`$. (b) If you can leave $`C`$ to a state that can't lead back, there's a positive chance of never returning. (c) A finite closed class is visited forever, so some state is visited infinitely often with positive probability, so it is recurrent, and by (a) the whole class is.

**So for finite chains:** recurrent ⇔ the class is closed. Infinite closed classes can be either.

### 5.9 Random walk on $`\mathbb{Z}^d`$: recurrent iff $`d \le 2`$
Simple symmetric random walk (irreducible, period 2). **Stirling's formula:** $`n! \sim \sqrt{2\pi}\,n^{n+1/2}e^{-n}`$.

**$`d = 1`$.** (Also follows from gambler's ruin with $`p = q`$.) $`p_{00}^{(n)} = 0`$ for odd $`n`$, and returning at time $`2m`$ needs $`m`$ ups and $`m`$ downs:
```math
p_{00}^{(2m)} = \binom{2m}{m}\Big(\frac12\Big)^{2m} = \frac{(2m)!}{m!\,m!}\Big(\frac12\Big)^{2m} \sim \frac{1}{\sqrt\pi}\,\frac{1}{m^{1/2}} .
```
$`\sum m^{-1/2} = \infty`$, so it's **recurrent**. *(Exercise: for $`p \ne q`$, Stirling gives transience.)*

**Intuition for general $`d`$:** a $`d`$-dimensional walk is roughly $`d`$ independent 1-d walks, so $`p_{00}^{(2m)} \approx (m^{-1/2})^d = m^{-d/2}`$. This is summable iff $`d \ge 3`$.

**$`d = 2`$ (exact).** Project onto the diagonals $`x = y`$ and $`x = -y`$: each step moves each projection by $`\pm 1/\sqrt2`$, all four combinations equally likely. So $`W^+_n, W^-_n`$ are **independent** simple symmetric walks (on $`2^{-1/2}\mathbb{Z}`$), and
```math
p_{00}^{(2m)} = \mathbb{P}(W^+_{2m} = 0)\,\mathbb{P}(W^-_{2m} = 0) \sim \Big(\frac{1}{\sqrt\pi}\frac{1}{m^{1/2}}\Big)^2 = \frac{1}{\pi m}, \qquad \sum_m \frac{1}{\pi m} = \infty \Rightarrow \text{recurrent.}
```

**$`d = 3`$.** Return at time $`2m`$ needs $`i, j, k`$ steps in each of $`\pm e_1, \pm e_2, \pm e_3`$ with $`i+j+k = m`$:
```math
p_{00}^{(2m)} = \sum_{\substack{i,j,k \ge 0\\ i+j+k = m}}\frac{(2m)!}{i!^2j!^2k!^2}\Big(\frac16\Big)^{2m} = \binom{2m}{m}\Big(\frac12\Big)^{2m}\sum_{i+j+k=m}\binom{m}{i,j,k}^2\Big(\frac13\Big)^{2m}
\le \binom{2m}{m}\Big(\frac12\Big)^{2m}\max_{i+j+k=m}\binom{m}{i,j,k}\Big(\frac13\Big)^{m},
```
using $`\sum_{i+j+k=m}\binom{m}{i,j,k}(\frac13)^m = 1`$ (a trinomial pmf). For $`m = 3r`$ the max is at $`i = j = k = r`$, and Stirling gives
```math
p_{00}^{(6r)} \lesssim \frac{1}{\sqrt{\pi}}\frac{1}{m^{1/2}}\cdot\frac{3\sqrt3}{2\pi}\frac{1}{m} \asymp m^{-3/2},
```
so $`\sum_r p_{00}^{(6r)} < \infty`$. *(The notes print the constants as $`1/\sqrt{2\pi}`$ and $`1/(2\pi)`$; the correct ones are $`1/\sqrt\pi`$ and $`3\sqrt3/(2\pi)`$. Only the power $`m^{-3/2}`$ matters.)* Since $`p_{00}^{(6r)} \ge (\frac16)^2 p_{00}^{(6r-2)}`$ and $`p_{00}^{(6r)} \ge (\frac16)^4 p_{00}^{(6r-4)}`$, the full sum $`\sum_n p_{00}^{(n)} < \infty`$: **transient**.

**$`d \ge 4`$.** Look only at the first 3 coordinates (ignoring steps that don't change them): that's a 3-d walk, which visits the origin finitely often, so the $`d`$-dim walk does too. **Transient.**

### 5.9.5 Mean hitting times

> **Theorem 5.13.** $`k^A = (k_i^A)`$ is the **minimal non-negative solution** of
> ```math
> k_i^A = \begin{cases} 0 & i \in A,\\ 1 + \sum_j p_{ij}k_j^A & i \notin A.\end{cases}
> ```

*Proof.* First-step analysis for $`i \notin A`$: $`k_i^A = \sum_j \mathbb{E}_i(H^A \mid X_1 = j)\,p_{ij} = \sum_j p_{ij}(1 + k_j^A) = 1 + \sum_j p_{ij}k_j^A`$. Minimality as in 5.10: by induction any non-negative solution satisfies $`y_i \ge \mathbb{E}_i\min(H^A, m)`$ for all $`m`$. ∎

Note: $`h_i^A < 1 \Rightarrow k_i^A = \infty`$, and $`k_i^A`$ can be $`\infty`$ even when $`h_i^A = 1`$.

**Gambler's ruin continued.** If $`k_i < \infty`$ then $`k_i = \beta i`$, since $`\mathbb{E}_i(\text{time to } 0) = \mathbb{E}_i(\text{to } i-1) + \dots + \mathbb{E}_1(\text{to } 0)`$. Plugging into $`k_i = 1 + q\,k_{i-1} + p\,k_{i+1}`$ gives $`(q-p)\beta = 1`$.

| Case | $`k_i`$ |
|---|---|
| $`p < q`$ | $`\dfrac{i}{q-p}`$ (finite) |
| $`p > q`$ | $`\infty`$ (since $`h_i < 1`$) |
| $`p = q`$ | $`\infty`$: no valid $`\beta`$; ruin is **certain but takes infinite expected time** |

### 5.10 Null vs positive recurrence
```math
m_i = \mathbb{E}_i\big(\min\{n \ge 1 : X_n = i\}\big) = 1 + \sum_j p_{ij}\,k_j^{\{i\}} .
```
Transient ⇒ $`m_i = \infty`$. Recurrent ⇒ the return time is finite, but its mean may be infinite (**null recurrent**, e.g. the symmetric walk on $`\mathbb{Z}`$) or finite (**positive recurrent**). Both are **class properties**, so an irreducible chain is exactly one of transient / null recurrent / positive recurrent. Heuristically, the long-run fraction of time spent at $`i`$ is $`1/m_i`$.

> [!TIP]
> **✅ Key takeaways (Chapter 5)**
> 1. A chain = **initial distribution** $`\lambda`$ + **stochastic matrix** $`P`$; path probabilities multiply (Thm 5.1); $`X_n \sim \lambda P^n`$; $`p^{(n)}_{ij} = (P^n)_{ij}`$.
> 2. **Proving Markov:** write $`Y_{n+1} = f(Y_n, \text{fresh independent noise})`$. **Disproving:** find two histories ending in the same state with different next-step probabilities.
> 3. **First-step analysis** gives linear equations for hitting probabilities ($`h_i = \sum_j p_{ij}h_j`$) and mean hitting times ($`k_i = 1 + \sum_j p_{ij}k_j`$). Pick the **minimal non-negative** solution.
> 4. Recurrence ⇔ $`\sum_n p_{ii}^{(n)} = \infty`$ (expected number of visits). It's a class property; finite chains: recurrent ⇔ closed.
> 5. **Pólya:** the simple random walk is recurrent in $`d = 1, 2`$ and transient in $`d \ge 3`$, because $`p_{00}^{(2m)} \asymp m^{-d/2}`$.
> 6. Symmetric gambler's ruin: hits 0 with probability 1, but the mean time is $`\infty`$ (null recurrence flavour).

> [!WARNING]
> **⚠️ Common misconceptions & errors**
> - ❌ "Columns of $`P`$ sum to 1." ✅ **Rows** do (row $`i`$ = distribution of the next state from $`i`$).
> - ❌ "$`X_n \sim P^n\lambda`$." ✅ $`\lambda`$ is a **row** vector: $`X_n \sim \lambda P^n`$.
> - ❌ "Any function of a Markov chain is Markov." ✅ The running maximum $`M_n`$ is not.
> - ❌ Taking *any* solution of the hitting equations. ✅ The equations usually have many solutions; $`h`$ is the **minimal non-negative** one (e.g. $`h_i = 1`$ solves gambler's ruin for $`p > q`$, but the answer is $`(q/p)^i`$).
> - ❌ Forgetting the boundary conditions ($`h_i = 1`$ / $`k_i = 0`$ on $`A`$).
> - ❌ "Recurrent ⇒ finite mean return time." ✅ Null recurrence: $`m_i = \infty`$.
> - ❌ "$`p_{ii}^{(n)} \to 0`$ means transient." ✅ The test is whether $`\sum_n p_{ii}^{(n)}`$ **diverges** ($`d = 1, 2`$ walks have $`p_{00}^{(n)} \to 0`$ yet are recurrent).
> - ❌ "Period = smallest $`n`$ with $`p_{ii}^{(n)} > 0`$." ✅ It's the **gcd** of all such $`n`$.
> - ❌ "Closed ⇒ recurrent" for infinite classes. ✅ Only for **finite** closed classes ($`\mathbb{Z}^3`$ is closed but transient).

---

**Next:** [Part 3b — stationary distributions, convergence to equilibrium, ergodic theorem, time-reversal →](A8-3b-Markov-Chains-Long-Run.md)
