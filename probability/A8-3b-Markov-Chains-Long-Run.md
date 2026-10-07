# A8 Probability · Part 3b: Markov Chains — Long-Run Behaviour

*Chapter 6 of the Part A Probability notes (James Martin, MT 2025). [← Part 3a](A8-3a-Markov-Chains.md) · [Overview](A8-Probability-Summary.md) · [Formula sheet →](A8-4-Formula-Sheet.md)*

*Uses the definitions from [Part 3a](A8-3a-Markov-Chains.md): irreducible, aperiodic, recurrent, mean return time $`m_i`$, hitting times.*

---

## 6. Stationary distributions, convergence to equilibrium, time-reversal

> [!NOTE]
> **📖 Key definitions**
> - **Stationary (invariant, equilibrium) distribution:** a distribution $`\pi`$ with
> ```math
> \pi P = \pi, \qquad\text{i.e.}\qquad \pi_j = \sum_i \pi_i\,p_{ij}\ \ \forall j .
> ```
> It is a **left** eigenvector with eigenvalue 1, normalised so that $`\sum_i \pi_i = 1`$. If $`X_0 \sim \pi`$ then $`X_n \sim \pi`$ for all $`n`$, and $`(X_n, X_{n+1}, \dots) \overset{d}{=} (X_0, X_1, \dots)`$.
> - **Visits before time $`n`$:** $`V_i(n) = \sum_{r=0}^{n-1}\mathbf{1}(X_r = i)`$.
> - **Time-reversal:** for $`(X_0, \dots, X_N) \sim \mathrm{Markov}(\pi, P)`$, set $`Y_n = X_{N-n}`$.
> - **Reversible** (time-reversible in equilibrium): the time-reversal has the same transition matrix $`P`$.
> - **Detailed balance:** $`\pi_i\,p_{ij} = \pi_j\,p_{ji}`$ for all $`i, j`$.

### 6.1 The main theorems (statements examinable; proofs not)

> **Theorem 6.1 (existence & uniqueness).** Let $`P`$ be **irreducible**. (a) $`P`$ has a stationary distribution **iff** $`P`$ is **positive recurrent**. (b) It is then unique, with
> ```math
> \pi_i = \frac{1}{m_i}.
> ```

> **Theorem 6.2 (convergence to equilibrium).** Let $`P`$ be **irreducible and aperiodic** with stationary distribution $`\pi`$. Then for **any** initial distribution, $`\mathbb{P}(X_n = j) \to \pi_j`$ as $`n \to \infty`$. In particular $`p_{ij}^{(n)} \to \pi_j`$ for all $`i, j`$.

> **Theorem 6.3 (ergodic theorem).** Let $`P`$ be **irreducible**. For any initial distribution and all $`i`$,
> ```math
> \mathbb{P}\Big(\frac{V_i(n)}{n} \to \frac{1}{m_i} \text{ as } n \to \infty\Big) = 1 .
> ```
> Long-run fraction of time at $`i`$ = $`1/m_i`$: this equals $`\pi_i`$ if positive recurrent and $`0`$ if null recurrent or transient. It generalises the SLLN (i.i.d. ⇒ fraction of entries equal to $`i`$ → $`\mathbb{P}(X = i)`$). **No aperiodicity needed.**

### 6.2 Examples

**Example 6.4.** $`P = \begin{pmatrix}0 & 1 & 0\\ 0 & \frac12 & \frac12\\ \frac12 & 0 & \frac12\end{pmatrix}`$. Solving $`\pi_1 = \frac12\pi_3`$, $`\pi_2 = \pi_1 + \frac12\pi_2`$, $`\pi_3 = \frac12\pi_2 + \frac12\pi_3`$ (one equation is redundant, so add $`\sum\pi_i = 1`$) gives $`\pi = (\frac15, \frac25, \frac25)`$ and $`m = (5, \frac52, \frac52)`$. From any start, $`p_{11}^{(n)} \to \frac15`$.

⚠️ Solve $`\pi P = \pi`$, **not** $`P\pi = \pi`$: any constant vector solves the latter (rows sum to 1), which would trick you into "uniform".

**Example 6.5 (cycle).** $`\pi_i = 1/M`$ solves $`\pi_i = (1-p)\pi_{i+1} + p\pi_{i-1}`$. If $`M`$ is odd (aperiodic), $`p_{00}^{(n)} \to 1/M`$. If $`M`$ is even (period 2), $`p_{00}^{(n)} = 0`$ for odd $`n`$ and $`p_{00}^{(2m)} \to 2/M`$ (consider the 2-step chain on even sites).

**Example 6.6 (random walk on a graph).** From vertex $`i`$ with degree $`d_i > 0`$, move to each neighbour with probability $`1/d_i`$. If the graph is connected (so the chain is irreducible), then **$`\pi_i \propto d_i`$**:
```math
\sum_i d_i\,p_{ij} = \sum_i d_i\frac{1}{d_i}\mathbf{1}(i \sim j) = d_j \quad\Longrightarrow\quad \pi_i = \frac{d_i}{\sum_j d_j}.
```
For the 6-vertex, 7-edge graph in the notes, $`d = (3,2,2,4,2,1)`$ and $`\sum d = 14`$, so $`\pi = (\frac{3}{14}, \frac17, \frac17, \frac27, \frac17, \frac1{14})`$ and $`m_1 = \frac{14}{3}`$. The chain is aperiodic, so $`\mathbb{P}(X_n = 1) \to \frac{3}{14}`$.

**Example 6.7 (reflected walk on $`\{0,1,2,\dots\}`$).** $`p_{i,i+1} = p`$, $`p_{i,i-1} = q`$ for $`i \ge 1`$, $`p_{00} = q`$.
- $`p > q`$: transient, so there is **no** stationary distribution.
- $`p = q`$: null recurrent, so there is **no** stationary distribution.
- $`p < q`$: positive recurrent. $`\pi_i = p\,\pi_{i-1} + q\,\pi_{i+1}`$ gives $`\pi_i = A + B(p/q)^i`$; normalising forces $`A = 0`$, so
```math
\pi_i = \Big(1 - \frac pq\Big)\Big(\frac pq\Big)^i \qquad \text{(geometric with parameter } 1 - p/q\text{)}.
```

**Example 6.8 (two-state; non-irreducible).** For $`P = \begin{pmatrix}1-\alpha & \alpha\\ \beta & 1-\beta\end{pmatrix}`$, $`\pi = \big(\frac{\beta}{\alpha+\beta}, \frac{\alpha}{\alpha+\beta}\big)`$, consistent with $`p_{11}^{(n)} \to \frac{\beta}{\alpha+\beta}`$ from Example 5.4.

For a **block-diagonal** chain on $`\{1,2\} \cup \{3,4\}`$ (two separate two-state chains with parameters $`\alpha,\beta`$ and $`\gamma,\delta`$), **every** mixture is stationary:
```math
\Big(x\frac{\beta}{\alpha+\beta},\; x\frac{\alpha}{\alpha+\beta},\; (1-x)\frac{\delta}{\gamma+\delta},\; (1-x)\frac{\gamma}{\gamma+\delta}\Big), \quad x \in [0,1].
```
These are **all** the stationary distributions (exercise). Uniqueness fails because the chain is **not irreducible**.

**Example 6.9 (random-to-front on permutations of $`m`$ books).** Each day book $`i`$ is chosen with probability $`q_i`$ and moved to the front. Think about **long-run behaviour** instead of solving $`\pi P = \pi`$: the front book is the most recently chosen; the second is the most recent among the rest; and so on. Precisely, $`\mathbb{P}(\sigma_1 = i) = q_i`$, $`\mathbb{P}(\sigma_2 = j \mid \sigma_1 = i) = \frac{q_j}{1-q_i}`$, and in general $`\mathbb{P}(\sigma_r = i_r \mid \sigma_1, \dots, \sigma_{r-1}) = \frac{q_{i_r}}{\sum_{j \notin \{i_1,\dots,i_{r-1}\}} q_j}`$ (the state space $`S_m`$ has $`m!`$ states). So
```math
\pi(i_1, i_2, \dots, i_m) = \prod_{r=1}^m \frac{q_{i_r}}{\sum_{j \ge r} q_{i_j}}, \qquad\text{e.g. for } m = 3:\ \pi(1,2,3) = q_1\frac{q_2}{q_2 + q_3}.
```

### 6.3 Proof ideas (non-examinable, but build intuition)
- **Ergodic theorem (6.3).** If transient, there are finitely many visits, so $`V_i(n)/n \to 0 = 1/m_i`$. If recurrent, the inter-visit times $`X_1, X_2, \dots`$ are i.i.d. with mean $`m_i`$, so by the SLLN $`(X_1 + \dots + X_K)/K \to m_i`$ a.s. Hence the time of the $`K`$th visit satisfies $`T_K/K \to m_i`$, which is equivalent to $`V_i(n)/n \to 1/m_i`$.
- **Lemma 6.10.** If $`P`$ is positive recurrent, it has a stationary $`\pi \propto 1/m_i`$. The long-run rate of jumps $`i \to j`$ is $`\frac{1}{m_i}p_{ij}`$; the rate of jumps into $`j`$ = the rate of visits to $`j`$, so $`\frac{1}{m_j} = \sum_i \frac{1}{m_i}p_{ij}`$. (For infinite $`I`$ one first gets $`\ge`$, then equality by summing over $`j`$.) Each $`m_i < \infty`$, so $`0 < \sum_i 1/m_i \le 1`$ (long-run proportions), and $`\pi_i = (1/m_i)/\sum_k 1/m_k`$ is a genuine distribution.
- **Lemma 6.11.** If $`\pi`$ is stationary then $`\pi_i = 1/m_i`$. Start in $`\pi`$: $`\mathbb{E}V_i(n)/n = \frac1n\sum_r \mathbb{P}(X_r = i) = \pi_i`$. Bounded convergence with the ergodic theorem gives $`\mathbb{E}V_i(n)/n \to 1/m_i`$. This gives uniqueness, and shows no stationary distribution exists in the null recurrent/transient case.
- **Convergence (6.2) by coupling.** Run $`X \sim \mathrm{Markov}(\lambda, P)`$ and an independent $`Y \sim \mathrm{Markov}(\pi, P)`$. Let $`T = \inf\{n : X_n = Y_n\}`$, and let $`Z`$ follow $`X`$ before $`T`$ and $`Y`$ after. Then $`Z \sim \mathrm{Markov}(\lambda, P)`$ and
```math
\lvert \mathbb{P}(Z_n = j) - \pi_j\rvert = \lvert \mathbb{P}(Z_n = j) - \mathbb{P}(Y_n = j)\rvert \le \mathbb{P}(Z_n \ne Y_n) = \mathbb{P}(T > n) \to 0 .
```
$`T < \infty`$ a.s. because the product chain $`W_n = (X_n, Y_n)`$, with $`\tilde p_{(i,k)(j,l)} = p_{ij}p_{kl}`$, is **irreducible** (this uses **aperiodicity**: $`p^{(n)}_{ij}p^{(n)}_{kl} > 0`$ for all large $`n`$) and has stationary distribution $`\tilde\pi_{(i,k)} = \pi_i\pi_k`$, so it is positive recurrent (Thm 6.1). It starts from $`\mu_{(i,k)} = \lambda_i\pi_k`$. Fix $`b \in I`$; then $`T \le T_b = \inf\{n : W_n = (b,b)\}`$, and $`\mathbb{P}(T_b < \infty) = 1`$ since an irreducible recurrent chain visits every state. **With period 2 this fails:** chains started with opposite parity never meet.

### 6.4 Time-reversal
*Motivation:* given the present, past and future are independent. That statement is symmetric in time, so the reversed process (started in equilibrium) should also be Markov.


> **Theorem 6.12.** Let $`P`$ be irreducible with stationary $`\pi`$, $`(X_0, \dots, X_N) \sim \mathrm{Markov}(\pi, P)`$, and $`Y_n = X_{N-n}`$. Then $`(Y_n) \sim \mathrm{Markov}(\pi, Q)`$ with
> ```math
> q_{ij} = \frac{\pi_j}{\pi_i}\,p_{ji},
> ```
> and $`\pi`$ is also stationary for $`Q`$.

*Proof.* **$`Q`$ is stochastic:** $`q_{ij} \ge 0`$, and $`\sum_j q_{ij} = \frac{1}{\pi_i}\sum_j \pi_j p_{ji} = \frac{\pi_i}{\pi_i} = 1`$ (stationarity of $`\pi`$). **$`\pi`$ is stationary for $`Q`$:** $`\sum_i \pi_i q_{ij} = \pi_j\sum_i p_{ji} = \pi_j`$. **Path probabilities:**
```math
\begin{aligned}
\mathbb{P}(Y_0 = i_0, \dots, Y_N = i_N) &= \mathbb{P}(X_0 = i_N, \dots, X_N = i_0) = \pi_{i_N}p_{i_Ni_{N-1}}\cdots p_{i_1i_0}\\
&= \pi_{i_N}\Big(q_{i_{N-1}i_N}\frac{\pi_{i_{N-1}}}{\pi_{i_N}}\Big)\cdots\Big(q_{i_0i_1}\frac{\pi_{i_0}}{\pi_{i_1}}\Big) = \pi_{i_0}q_{i_0i_1}\cdots q_{i_{N-1}i_N}. \quad ∎
\end{aligned}
```
**Interpretation:** in equilibrium the long-run rate of jumps $`j \to i`$ in $`X`$ is $`\pi_jp_{ji}`$; this must equal the rate of jumps $`i \to j`$ in the reversal, $`\pi_iq_{ij}`$.

> **Theorem 6.13.** An irreducible $`P`$ with stationary $`\pi`$ is reversible **iff** detailed balance holds: $`\pi_ip_{ij} = \pi_jp_{ji}`$ for all $`i,j`$.

> **Theorem 6.14.** If $`P`$ and a distribution $`\pi`$ satisfy **detailed balance**, then $`\pi`$ is **stationary**.

*Proof.* $`\pi_j = \pi_j\sum_i p_{ji} = \sum_i \pi_jp_{ji} = \sum_i \pi_ip_{ij}`$. ∎

**Remark 6.15 (practical method).** To find $`\pi`$, **try detailed balance first**: it's usually far easier than $`\pi P = \pi`$. If it has a (normalisable) solution, that's $`\pi`$ and the chain is reversible. If it has no solution, the chain is **not** reversible (and you must solve $`\pi P = \pi`$).

**Example 6.16 (cycle).** $`\pi`$ is uniform. If $`p \ne \frac12`$ the walk drifts, so it is **not reversible**: its reversal is the same walk with the opposite drift, $`q_{i,i+1} = 1-p`$, $`q_{i,i-1} = p`$. If $`p = \frac12`$, detailed balance holds ($`\pi_ip_{ij} = \frac1M\cdot\frac12`$ for neighbours), so it is reversible.

**Example 6.17 (reflected walk, $`p < \frac12`$).** $`\pi_i = \big(1 - \frac{p}{1-p}\big)\big(\frac{p}{1-p}\big)^i`$. Despite the drift, it **is reversible**: there are no cycles in the transition graph, and up- and down-crossings of each edge $`(i, i+1)`$ must alternate, so their long-run rates are equal. In general, any chain whose transition graph is a **tree** (e.g. nearest-neighbour chains on $`\mathbb{Z}`$) is reversible.

**Example 6.18 (random walk on a graph).** Always reversible: $`\pi_i \propto d_i`$, so $`\pi_ip_{ij} = \frac{d_i}{\sum d}\cdot\frac{1}{d_i} = \frac{1}{\sum d} = \pi_jp_{ji}`$ for neighbours $`i \sim j`$.

> [!TIP]
> **✅ Key takeaways (Chapter 6)**
> 1. **Stationary:** $`\pi P = \pi`$, a **left** eigenvector, normalised.
> 2. **Irreducible:** a stationary distribution exists ⇔ positive recurrent, and then $`\pi_i = 1/m_i`$ (unique). So $`m_i = 1/\pi_i`$ is a quick way to get mean return times.
> 3. **Irreducible + aperiodic + positive recurrent** ⇒ $`p^{(n)}_{ij} \to \pi_j`$ from any start (coupling).
> 4. **Ergodic theorem** (irreducible only): the fraction of time at $`i`$ → $`1/m_i`$ a.s. It is the SLLN for Markov chains.
> 5. **Detailed balance** ⇒ stationary (and reversible). Try it first. It always works for birth–death chains, trees, and random walks on graphs ($`\pi \propto`$ degree).
> 6. Time-reversal: $`q_{ij} = \pi_jp_{ji}/\pi_i`$; reversible ⇔ $`Q = P`$ ⇔ detailed balance.

> [!WARNING]
> **⚠️ Common misconceptions & errors**
> - ❌ Solving $`P\pi = \pi`$. ✅ That's solved by constant vectors; you need $`\pi P = \pi`$.
> - ❌ "Every chain has a unique stationary distribution." ✅ Needs irreducible **and positive recurrent** (null-recurrent/transient chains have none; reducible chains can have many).
> - ❌ "A stationary distribution exists ⇒ $`p^{(n)}_{ij} \to \pi_j`$." ✅ Also needs **aperiodicity** (period-2 cycle: $`p_{00}^{(n)}`$ alternates).
> - ❌ "The ergodic theorem needs aperiodicity." ✅ It doesn't: time-averages converge even when $`p^{(n)}`$ oscillates.
> - ❌ "Detailed balance ⇔ stationary." ✅ Detailed balance ⇒ stationary, but **not conversely**: the drifting cycle is stationary-uniform but not reversible.
> - ❌ "A drift means not reversible." ✅ The reflected walk on $`\mathbb{N}`$ has a drift but is reversible (tree graph).
> - ❌ Forgetting to **normalise** $`\pi`$, or forgetting that one balance equation is redundant.
> - ❌ "$`m_i = \pi_i`$." ✅ $`m_i = 1/\pi_i`$.
