# Machine Learning · Part 6: Unsupervised Learning — PCA and Clustering

*Lectures 18–19 (PCA notes with contributions from James Worrell). [← Part 5](ML-5-Neural-Networks.md) · [Overview](ML-Summary.md) · [Formula sheet →](ML-7-Formula-Sheet.md)*

---

## Contents
27. [Singular value decomposition](#27-singular-value-decomposition)
28. [Low-rank approximation and the Eckart–Young theorem](#28-low-rank-approximation-and-the-eckartyoung-theorem)
29. [Principal component analysis](#29-principal-component-analysis)
30. [Computing an SVD: power iteration](#30-computing-an-svd-power-iteration)
31. [Kernel PCA](#31-kernel-pca)
32. [Clustering: k-means](#32-clustering-k-means)
33. [Transforming input formats and multidimensional scaling](#33-transforming-input-formats-and-multidimensional-scaling)
34. [Hierarchical clustering](#34-hierarchical-clustering)
35. [Spectral clustering](#35-spectral-clustering)

---

## 27. Singular value decomposition

> [!NOTE]
> **📖 Key definitions**
> - **PCA (principal components analysis):** dimensionality reduction that gives a compact representation with minimal information loss. It assumes the data lie close to a low-dimensional *linear* manifold because the variables are correlated; the principal components are **uncorrelated** directions. Often a preprocessing step before nearest-neighbour search or clustering; can be kernelised for non-linear manifolds.
> - **PCA goal:** given $`n`$ data vectors in $`\mathbb{R}^m`$ (mean-centred), find $`k \ll m`$ **orthonormal** directions $`\mathbf{u}_1, \dots, \mathbf{u}_k`$ ("principal components") such that each $`\mathbf{x}_i \approx z_{i1}\mathbf{u}_1 + \dots + z_{ik}\mathbf{u}_k`$ as well as possible. $`\mathbf{u}_1`$ = direction of greatest variance, $`\mathbf{u}_2`$ = greatest variance orthogonal to $`\mathbf{u}_1`$, etc. Output: a compact, lower-dimensional, hopefully less noisy representation $`\mathbf{z}_i \in \mathbb{R}^k`$. (Example 1: 1,400 European genomes with $`m \approx 200{,}000`$ project onto 2 components that recover the map of Europe.)
> - **SVD** of a real $`m\times n`$ matrix $`\mathbf{A}`$ of rank $`r`$: $`\mathbf{A} = \mathbf{U}\boldsymbol\Sigma\mathbf{V}^\top`$ with $`\mathbf{U}`$ ($`m\times r`$, $`\mathbf{U}^\top\mathbf{U} = \mathbf{I}_r`$), $`\mathbf{V}`$ ($`n\times r`$, $`\mathbf{V}^\top\mathbf{V} = \mathbf{I}_r`$), $`\boldsymbol\Sigma = \operatorname{diag}(\sigma_1, \dots, \sigma_r)`$ with $`\sigma_1 \ge \dots \ge \sigma_r > 0`$ the **singular values**. Columns of $`\mathbf{U}`$: **left singular vectors**; of $`\mathbf{V}`$: **right singular vectors**. Equivalently
> ```math
> \mathbf{A} = \sum_{i=1}^r\sigma_i\mathbf{u}_i\mathbf{v}_i^\top \quad\text{(sum of } r \text{ rank-one matrices)}, \qquad \mathbf{A}\mathbf{v}_i = \sigma_i\mathbf{u}_i .
> ```
> - **Full SVD:** pad $`\mathbf{U}`$ to $`m\times m`$ orthogonal, $`\mathbf{V}`$ to $`n\times n`$ orthogonal, $`\boldsymbol\Sigma`$ to $`m\times n`$ with zeros: "rotation, scaling, rotation". (The version above is the *reduced* SVD.)
> - **Frobenius norm:** $`\lVert\mathbf{A}\rVert_F = \sqrt{\sum_{i,j}a_{ij}^2} = \sqrt{\operatorname{trace}(\mathbf{A}^\top\mathbf{A})}`$.

### 27.1 Existence (Theorem 1): every matrix has an SVD
*Proof sketch (the construction matters).* Define $`\mathbf{v}_1, \mathbf{v}_2, \dots`$ greedily:
```math
\mathbf{v}_1 = \arg\max_{\lVert\mathbf{v}\rVert = 1}\lVert\mathbf{A}\mathbf{v}\rVert, \qquad \mathbf{v}_2 = \arg\max_{\mathbf{v}\perp\mathbf{v}_1,\ \lVert\mathbf{v}\rVert = 1}\lVert\mathbf{A}\mathbf{v}\rVert, \qquad \mathbf{v}_3 = \arg\max_{\mathbf{v}\perp\mathbf{v}_1,\mathbf{v}_2,\ \lVert\mathbf{v}\rVert=1}\lVert\mathbf{A}\mathbf{v}\rVert, \ \dots
```
stopping at the first $`r`$ such that $`\mathbf{A}`$ is zero on $`\{\mathbf{v}_1, \dots, \mathbf{v}_r\}^\perp`$. Set $`\sigma_i = \lVert\mathbf{A}\mathbf{v}_i\rVert`$ and $`\mathbf{u}_i = \mathbf{A}\mathbf{v}_i/\sigma_i`$. The $`\mathbf{v}_i`$ are orthonormal by construction. With $`\mathbf{B} = \sum_i\sigma_i\mathbf{u}_i\mathbf{v}_i^\top`$, we have $`\mathbf{A}\mathbf{v}_i = \mathbf{B}\mathbf{v}_i`$ for each $`i`$ and $`\mathbf{A}\mathbf{v} = \mathbf{B}\mathbf{v} = 0`$ on the orthogonal complement, so $`\mathbf{A} = \mathbf{B}`$. It remains to show the $`\mathbf{u}_i`$ are orthonormal: for $`i < j`$ consider $`\mathbf{w}(t) = (\mathbf{v}_i + t\mathbf{v}_j)/\sqrt{1+t^2}`$, a unit vector orthogonal to $`\mathbf{v}_1, \dots, \mathbf{v}_{i-1}`$, so $`\lVert\mathbf{A}\mathbf{w}(t)\rVert^2`$ is maximised at $`t = 0`$. But
```math
\lVert\mathbf{A}\mathbf{w}(t)\rVert^2 = \frac{\lVert\sigma_i\mathbf{u}_i + t\sigma_j\mathbf{u}_j\rVert^2}{1+t^2} = \frac{\sigma_i^2 + 2t\sigma_i\sigma_j(\mathbf{u}_i^\top\mathbf{u}_j) + t^2\sigma_j^2}{1+t^2},
```
whose derivative at $`t = 0`$ is $`2\sigma_i\sigma_j\mathbf{u}_i^\top\mathbf{u}_j`$; it must vanish, so $`\mathbf{u}_i^\top\mathbf{u}_j = 0`$. ∎

**Takeaway from the construction:** the right singular vectors are the **length-maximising directions**, found one at a time, each orthogonal to the previous ones.

### 27.2 SVD and eigenvalues (Proposition 2)
> The singular values of $`\mathbf{A}`$ are the **square roots of the non-zero eigenvalues of $`\mathbf{A}^\top\mathbf{A}`$** (equivalently of $`\mathbf{A}\mathbf{A}^\top`$; same non-zero eigenvalues). Right singular vectors are eigenvectors of $`\mathbf{A}^\top\mathbf{A}`$, left singular vectors eigenvectors of $`\mathbf{A}\mathbf{A}^\top`$.

*Proof.* With a full SVD, $`\mathbf{A}^\top\mathbf{A}\mathbf{V} = (\mathbf{U}\boldsymbol\Sigma\mathbf{V}^\top)^\top(\mathbf{U}\boldsymbol\Sigma\mathbf{V}^\top)\mathbf{V} = \mathbf{V}\boldsymbol\Sigma^\top\mathbf{U}^\top\mathbf{U}\boldsymbol\Sigma\mathbf{V}^\top\mathbf{V} = \mathbf{V}\boldsymbol\Sigma^\top\boldsymbol\Sigma = \mathbf{V}\boldsymbol\Sigma^2`$ (using $`\mathbf{U}^\top\mathbf{U} = \mathbf{I}`$, $`\mathbf{V}^\top\mathbf{V} = \mathbf{I}`$). So each $`\mathbf{v}_i`$ is an eigenvector with eigenvalue $`\sigma_i^2`$; the remaining columns of $`\mathbf{V}`$ span the null eigenspace. ∎ Consequence: singular values are **unique** (SVDs aren't, but the $`\sigma_i`$ are). Alternative route to existence: diagonalise the symmetric PSD matrix $`\mathbf{A}^\top\mathbf{A}`$ (footnote 2).

Since $`\mathbf{A}^\top\mathbf{A} = \mathbf{V}\boldsymbol\Sigma^2\mathbf{V}^\top`$ and $`\operatorname{trace}`$ is invariant under this:
```math
\lVert\mathbf{A}\rVert_F^2 = \operatorname{trace}(\mathbf{A}^\top\mathbf{A}) = \operatorname{trace}(\boldsymbol\Sigma^\top\boldsymbol\Sigma) = \sigma_1^2 + \dots + \sigma_r^2 .
```

---

## 28. Low-rank approximation and the Eckart–Young theorem
Truncate the SVD after $`k \le r`$ terms:
```math
\mathbf{A}_k = \sum_{i=1}^k\sigma_i\mathbf{u}_i\mathbf{v}_i^\top \qquad (\text{rank } k,\ \text{singular values } \sigma_1, \dots, \sigma_k).
```
Then $`\mathbf{A} - \mathbf{A}_k = \sum_{i=k+1}^r\sigma_i\mathbf{u}_i\mathbf{v}_i^\top`$ has singular values $`\sigma_{k+1}, \dots, \sigma_r`$, so $`\lVert\mathbf{A} - \mathbf{A}_k\rVert_F = \sqrt{\sigma_{k+1}^2 + \dots + \sigma_r^2}`$ (the notes print this without the squares; the squared form is correct). Useful fact: $`\mathbf{M} - \mathbf{M}_{i-1}`$ has singular values $`\sigma_i, \dots, \sigma_r`$, so $`\sigma_i(\mathbf{M}) = \sigma_1(\mathbf{M} - \mathbf{M}_{i-1})`$; in particular $`\sigma_{k+1}(\mathbf{A})`$ is the top singular value of $`\mathbf{A} - \mathbf{A}_k`$.

> **Theorem 3 (Eckart–Young).** For any $`k`$ and any $`m\times n`$ matrix $`\mathbf{B}`$ of rank at most $`k`$: $`\lVert\mathbf{A} - \mathbf{A}_k\rVert_F \le \lVert\mathbf{A} - \mathbf{B}\rVert_F`$. **The truncated SVD is the best rank-$`k`$ approximation.**

> **Theorem 4 (projection form).** With $`\mathbf{P}_k = \sum_{i=1}^k\mathbf{u}_i\mathbf{u}_i^\top`$ (orthogonal projection onto the top-$`k`$ left singular vectors), for any rank-$`k`$ orthogonal projection $`\mathbf{P}`$: $`\lVert\mathbf{A} - \mathbf{P}_k\mathbf{A}\rVert_F \le \lVert\mathbf{A} - \mathbf{P}\mathbf{A}\rVert_F`$.

Equivalence: $`\mathbf{A}_k = \mathbf{P}_k\mathbf{A}`$, and for any rank-$`k`$ $`\mathbf{B}`$, projecting onto $`\mathbf{B}`$'s column space can only help ($`\mathbf{P}\mathbf{x}`$ is the closest point of that space to $`\mathbf{x}`$). **Intuition:** Pythagoras gives $`\lVert\mathbf{A} - \mathbf{P}\mathbf{A}\rVert_F^2 = \lVert\mathbf{A}\rVert_F^2 - \lVert\mathbf{P}\mathbf{A}\rVert_F^2`$, so minimising the error = **maximising the retained length**, exactly the greedy construction in §27.1.

**Choosing $`k`$ (relative error):**
```math
\frac{\lVert\mathbf{X} - \mathbf{X}_k\rVert_F^2}{\lVert\mathbf{X}\rVert_F^2} = \frac{\sigma_{k+1}^2 + \dots + \sigma_r^2}{\sigma_1^2 + \dots + \sigma_r^2}.
```
Pick the smallest $`k`$ bringing this under a tolerance (say 0.05): read it off the singular values, no trial and error.

### 28.1 Proof of Eckart–Young (appendix; via a lemma)
Notation $`\sigma_i(\mathbf{A})`$ = $`i`$-th singular value ($`0`$ beyond the rank). "Triangle inequality" (exercise): $`\sigma_1(\mathbf{A} + \mathbf{B}) \le \sigma_1(\mathbf{A}) + \sigma_1(\mathbf{B})`$ ($`\sigma_1`$ is the operator 2-norm).

**Lemma 1.** If $`\mathbf{B}`$ has rank $`\le k`$ then $`\sigma_{k+i}(\mathbf{A}) \le \sigma_i(\mathbf{A} - \mathbf{B})`$ for all $`i`$.

*Case $`i = 1`$.* $`\ker\mathbf{B}`$ has dimension $`\ge n - k`$, so it meets $`\operatorname{span}(\mathbf{v}_1, \dots, \mathbf{v}_{k+1})`$ (dimension $`k+1`$) in some unit vector $`\mathbf{w}`$. Then $`\lVert\mathbf{A}\mathbf{w}\rVert = \lVert(\mathbf{A} - \mathbf{B})\mathbf{w}\rVert \le \sigma_1(\mathbf{A} - \mathbf{B})`$. On the other hand, since $`\mathbf{w} \in \operatorname{span}(\mathbf{v}_1, \dots, \mathbf{v}_{k+1})`$,
```math
\lVert\mathbf{A}\mathbf{w}\rVert^2 = \Big\lVert\sum_{i=1}^{k+1}\sigma_i\mathbf{u}_i(\mathbf{v}_i^\top\mathbf{w})\Big\rVert^2 = \sum_{i=1}^{k+1}\sigma_i^2(\mathbf{v}_i^\top\mathbf{w})^2 \ge \sigma_{k+1}^2\sum_{i=1}^{k+1}(\mathbf{v}_i^\top\mathbf{w})^2 = \sigma_{k+1}^2 .
```
Hence $`\sigma_{k+1}(\mathbf{A}) \le \sigma_1(\mathbf{A} - \mathbf{B})`$.

*General $`i`$.* Using $`\mathbf{B} = \mathbf{B}_k`$ and the truncation $`(\mathbf{A}-\mathbf{B})_{i-1}`$:
```math
\sigma_i(\mathbf{A} - \mathbf{B}) = \sigma_1\big(\mathbf{A} - \mathbf{B} - (\mathbf{A}-\mathbf{B})_{i-1}\big) + \sigma_1(\mathbf{B} - \mathbf{B}_k) \ \ge\ \sigma_1\big(\mathbf{A} - (\mathbf{A}-\mathbf{B})_{i-1} - \mathbf{B}_k\big) \ \ge\ \sigma_{i+k}(\mathbf{A}),
```
by the triangle inequality and then the $`i = 1`$ case, since $`\operatorname{rank}\big((\mathbf{A}-\mathbf{B})_{i-1} + \mathbf{B}_k\big) \le i + k - 1`$. (The first equality uses $`\sigma_1(\mathbf{B} - \mathbf{B}_k) = 0`$ because $`\operatorname{rank}\mathbf{B} \le k`$, and the second uses the "useful fact" from §28 with $`\mathbf{M} = \mathbf{A} - \mathbf{B}`$.) ∎

**Proof of Theorem 3.** $`\lVert\mathbf{A} - \mathbf{A}_k\rVert_F^2 = \sum_{i=k+1}^r\sigma_i(\mathbf{A})^2 \le \sum_{i=1}^{r-k}\sigma_i(\mathbf{A}-\mathbf{B})^2 \le \lVert\mathbf{A} - \mathbf{B}\rVert_F^2`$ by Lemma 1. ∎

---

## 29. Principal component analysis
Data $`\mathbf{x}_1, \dots, \mathbf{x}_n \in \mathbb{R}^m`$, mean $`\boldsymbol\mu = \frac1n\sum_i\mathbf{x}_i`$; **centre** by replacing $`\mathbf{x}_i`$ with $`\mathbf{x}_i - \boldsymbol\mu`$. Goal: points $`\widetilde{\mathbf{x}}_i`$ in a $`k`$-dimensional subspace minimising the **reconstruction error** $`\sum_i\lVert\mathbf{x}_i - \widetilde{\mathbf{x}}_i\rVert^2`$. (Best-fit $`k`$-dim subspace for centred data = best-fit affine subspace for the original data.)

Put the centred points as **columns** of the $`m\times n`$ matrix $`\mathbf{X}`$, approximants as columns of $`\widetilde{\mathbf{X}}`$. The reconstruction error is exactly $`\lVert\mathbf{X} - \widetilde{\mathbf{X}}\rVert_F^2`$, so by Eckart–Young the optimum is $`\widetilde{\mathbf{X}} = \mathbf{X}_k`$. With $`\mathbf{U}_k`$ ($`m\times k`$) the top-$`k`$ left singular vectors and $`\mathbf{Z} = \mathbf{U}_k^\top\mathbf{X}`$ ($`k\times n`$):
> ```math
> \mathbf{X}_k = \mathbf{U}_k\mathbf{U}_k^\top\mathbf{X} = \mathbf{U}_k\mathbf{Z}.
> ```
> **PCA output:** $`\mathbf{U}_k`$ (the principal components) and $`\mathbf{Z}`$ (each column $`\mathbf{z}_i \in \mathbb{R}^k`$ = coordinates of $`\mathbf{x}_i`$ in the component basis).

**Applications:**
- **Face recognition (eigenfaces):** faces as pixel vectors ($`m`$ in the tens/hundreds of thousands); a few hundred components suffice. For a new face $`\mathbf{x}`$, compute $`\mathbf{U}_k^\top(\mathbf{x} - \boldsymbol\mu)`$ and find the nearest $`\mathbf{z}_i`$.
- **Latent semantic analysis:** documents as keyword-frequency vectors; raw inner products $`\mathbf{x}_i^\top\mathbf{x}_j`$ ignore correlated keywords (*football* vs *Premier League*); after PCA, $`\mathbf{z}_i^\top\mathbf{z}_j`$ is a better similarity.

**Link to Part 1:** $`\mathbf{X}\mathbf{X}^\top`$ is ($`n\times`$) the empirical covariance, so the principal components are its top eigenvectors: directions of greatest variance.

---

## 30. Computing an SVD: power iteration
To find $`\mathbf{v}_1`$ (top eigenvector of $`\mathbf{A}^\top\mathbf{A}`$, eigenvalues $`\lambda_i = \sigma_i^2`$):
1. Pick a unit vector $`\mathbf{x}_0 \in \mathbb{R}^n`$ uniformly at random (sample $`\mathcal{N}(0,1)`$ coordinates, normalise).
2. Iterate $`\mathbf{x}_k = (\mathbf{A}^\top\mathbf{A})\mathbf{x}_{k-1}`$, $`\mathbf{y}_k = \mathbf{x}_k/\lVert\mathbf{x}_k\rVert`$; stop when $`\lVert\mathbf{y}_k - \mathbf{y}_{k-1}\rVert`$ is small.

**Why it works:** repeated multiplication amplifies the component along the largest eigenvalue fastest. Convergence speed is governed by the **spectral gap** $`\lambda_1/\lambda_2`$.

> **Proposition 5.** With probability $`\ge 9/10`$ over $`\mathbf{x}_0`$, for all $`k \ge 0`$: $`\lvert\mathbf{y}_k^\top\mathbf{v}_1\rvert \ge 1 - 20n(\lambda_2/\lambda_1)^k`$.

*Proof.* Expand $`\mathbf{x}_0 = \sum_i b_i\mathbf{v}_i`$. A random unit vector has $`\lvert b_1\rvert > \frac{1}{20\sqrt n}`$ with probability $`\ge 9/10`$ (the expected $`\lvert b_1\rvert`$ is $`\approx 1/\sqrt n`$ by symmetry; a concentration bound). Then
```math
\mathbf{y}_k^\top\mathbf{v}_1 = \frac{\mathbf{x}_0^\top(\mathbf{A}^\top\mathbf{A})^k\mathbf{v}_1}{\lVert(\mathbf{A}^\top\mathbf{A})^k\mathbf{x}_0\rVert} = \frac{\lambda_1^kb_1}{\sqrt{b_1^2\lambda_1^{2k} + \dots + b_n^2\lambda_n^{2k}}} \ge \frac{\lambda_1^kb_1}{\sqrt{b_1^2\lambda_1^{2k} + n\lambda_2^{2k}}} \ge \frac{\lambda_1^kb_1}{b_1\lambda_1^k + \sqrt n\lambda_2^k} = \frac{1}{1 + \frac{\sqrt n}{b_1}(\lambda_2/\lambda_1)^k} \ge 1 - 20n(\lambda_2/\lambda_1)^k,
```
using $`\sqrt{a+b} \le \sqrt a + \sqrt b`$, $`\sum_{i\ge2}b_i^2 \le 1`$, and $`\frac{1}{1+t} \ge 1 - t`$. ∎ (If $`\lambda_1 = \lambda_2`$ it converges to *some* top-eigenspace vector.)

**Further vectors:** to get $`\mathbf{v}_2`$, project out $`\mathbf{v}_1`$ at each step: $`\mathbf{x}_k \leftarrow \mathbf{x}_k - (\mathbf{x}_k^\top\mathbf{v}_1)\mathbf{v}_1`$; and so on. (In-principle method; real implementations worry about numerical stability.)

---

## 31. Kernel PCA
Feature map $`\phi : \mathbb{R}^m\to\mathbb{R}^d`$ with kernel $`\kappa(\mathbf{x}, \mathbf{x}') = \phi(\mathbf{x})^\top\phi(\mathbf{x}')`$. Centre in feature space: $`\mathbf{x}_i^* = \phi(\mathbf{x}_i) - \boldsymbol\mu`$, $`\boldsymbol\mu = \frac1n\sum_i\phi(\mathbf{x}_i)`$; data matrix $`\mathbf{X}`$ ($`d\times n`$, columns $`\mathbf{x}_i^*`$). Goal: $`\mathbf{Z} = \mathbf{U}_k^\top\mathbf{X}`$ **without ever forming $`\mathbf{X}`$ or $`\mathbf{U}_k`$** ($`d`$ may be huge or infinite).

1. **$`\mathbf{X}^\top\mathbf{X}`$ from the kernel** ($`n\times n`$): expanding $`(\phi(\mathbf{x}_i) - \boldsymbol\mu)^\top(\phi(\mathbf{x}_j) - \boldsymbol\mu)`$,
```math
(\mathbf{X}^\top\mathbf{X})_{ij} = \kappa(\mathbf{x}_i, \mathbf{x}_j) - \frac1n\sum_{l=1}^n\kappa(\mathbf{x}_i, \mathbf{x}_l) - \frac1n\sum_{k=1}^n\kappa(\mathbf{x}_k, \mathbf{x}_j) + \frac{1}{n^2}\sum_{k,l=1}^n\kappa(\mathbf{x}_k, \mathbf{x}_l).
```
2. **Right singular vectors** $`\mathbf{v}_1, \dots, \mathbf{v}_k \in \mathbb{R}^n`$ = top eigenvectors of $`\mathbf{X}^\top\mathbf{X}`$ (power iteration works on this explicit matrix).
3. **Left singular vectors** implicitly: $`\mathbf{u}_i = \mathbf{X}\mathbf{v}_i/\lVert\mathbf{X}\mathbf{v}_i\rVert`$ with $`\lVert\mathbf{X}\mathbf{v}_i\rVert^2 = \mathbf{v}_i^\top(\mathbf{X}^\top\mathbf{X})\mathbf{v}_i`$, so $`\mathbf{u}_i = \sum_j\alpha_{ij}\mathbf{x}_j^*`$ is a known combination of data vectors.
4. **Coordinates:** $`\mathbf{Z} = \mathbf{U}_k^\top\mathbf{X}`$ then needs only $`\mathbf{x}_j^{*\top}\mathbf{x}_l^*`$, i.e. entries of $`\mathbf{X}^\top\mathbf{X}`$.

Effect (Fig. 2): with $`\kappa(\mathbf{x}, \mathbf{y}) = (\mathbf{x}^\top\mathbf{y} + 1)^2`$, i.e. $`\phi(\mathbf{x}) = (1, \sqrt2x_1, \sqrt2x_2, \sqrt2x_1x_2, x_1^2, x_2^2)`$, three concentric rings become separable; since $`\phi`$ contains $`x_1^2 + x_2^2`$ the top component ≈ the radius. **Non-linear dimensionality reduction**.

> [!TIP]
> **✅ Key takeaways (PCA)**
> 1. SVD: $`\mathbf{A} = \sum_i\sigma_i\mathbf{u}_i\mathbf{v}_i^\top`$; $`\sigma_i^2`$ = eigenvalues of $`\mathbf{A}^\top\mathbf{A}`$; $`\lVert\mathbf{A}\rVert_F^2 = \sum\sigma_i^2`$.
> 2. **Eckart–Young:** truncated SVD = best rank-$`k`$ approximation in Frobenius norm; error $`= \sqrt{\sum_{i>k}\sigma_i^2}`$; use the ratio of squared singular values to choose $`k`$.
> 3. **PCA = SVD of the centred data matrix:** components $`\mathbf{U}_k`$, coordinates $`\mathbf{Z} = \mathbf{U}_k^\top\mathbf{X}`$, reconstruction $`\mathbf{U}_k\mathbf{Z}`$. Always **centre first**.
> 4. Power iteration finds the top eigenvector at a rate set by the spectral gap; deflate to get more.
> 5. Kernel PCA: everything needed is in $`\mathbf{X}^\top\mathbf{X}`$, which the kernel gives after a centring correction.

> [!WARNING]
> **⚠️ Common misconceptions & errors**
> - ❌ Forgetting to **mean-centre** before PCA (the first component then points at the mean).
> - ❌ "Singular values = eigenvalues of $`\mathbf{A}`$." ✅ Square roots of eigenvalues of $`\mathbf{A}^\top\mathbf{A}`$; $`\mathbf{A}`$ needn't be square.
> - ❌ "The SVD is unique." ✅ Singular **values** are unique; vectors aren't (signs, repeated values).
> - ❌ $`\lVert\mathbf{A} - \mathbf{A}_k\rVert_F = \sqrt{\sigma_{k+1} + \dots}`$ (as printed in the notes). ✅ Squares inside the root.
> - ❌ Rows vs columns: in the PCA section the data are **columns** of $`\mathbf{X}`$ ($`m\times n`$), unlike regression's $`N\times D`$ design matrix.
> - ❌ "Kernel PCA constructs $`\phi(\mathbf{x})`$." ✅ It never does; only kernel evaluations.

---

## 32. Clustering: k-means

> [!NOTE]
> **📖 Key definitions**
> - **Clustering:** unsupervised task of partitioning data so points in the same cluster are more similar to each other than to points in other clusters. Inherently ambiguous (news by topic vs by region).
> - **Input formats:** $`N`$ points in $`\mathbb{R}^D`$, **or** an $`N\times N`$ (dis)similarity matrix. **Output types:** **flat** (a partition) or **hierarchical** (a tree).
> - **k-means objective** (partition $`C_1, \dots, C_k`$, means $`\boldsymbol\mu_1, \dots, \boldsymbol\mu_k`$):
> ```math
> W(C_1, \dots, C_k; \boldsymbol\mu_1, \dots, \boldsymbol\mu_k) = \sum_{j=1}^k\sum_{i\in C_j}\lVert\mathbf{x}_i - \boldsymbol\mu_j\rVert_2^2 .
> ```
> - **Lloyd's algorithm:** initialise means; repeat { assign each point to its nearest mean; set each mean to the average of its points } until nothing changes.
> - **Variants:** Euclidean (not squared) distance ⇒ **k-median**; any $`\ell_p`$; max instead of sum of dissimilarities to the **centre** ⇒ **k-center**.

**Facts:**
- Minimising $`W`$ is **NP-hard even for $`k = 2`$**, so Lloyd's is a heuristic (k-means++ gives guarantees).
- The objective only rewards **tight** clusters; nothing in $`W`$ explicitly pushes different clusters apart, but with a sensible $`k`$ that happens automatically.
- **Alternating minimisation:** with means fixed, the best partition is nearest-mean; with the partition fixed, the best means are averages. Each step **strictly decreases** $`W`$ (or stops), and there are finitely many partitions, so it **converges** (but to a local optimum, and no polynomial bound on iterations; fast in practice).
- **Initialisation:** pick $`k`$ data points at random (means should be representative of the data for fast convergence); results depend on it; run several times, keep the lowest $`W`$.
- **Choosing $`k`$:** $`W`$ always decreases as $`k`$ grows (splitting a cluster can't increase it), so don't minimise over $`k`$. Plot $`W`$ vs $`k`$ and pick the **elbow** (Fig. 2: 8 true clusters ⇒ elbow at 8). The plotted values are themselves only Lloyd's local optima, not the true minima of $`W`$.
- k-means produces **convex** clusters (nearest-mean regions), so it fails on concentric rings (Fig. 6).

---

## 33. Transforming input formats and multidimensional scaling
**Euclidean → dissimilarity:** $`d(\mathbf{x}, \mathbf{x}') = f\big(\sum_{j=1}^D w_jd_j(x_j, x_j')\big)`$ with per-feature distances $`d_j`$, weights $`w_j`$ and non-decreasing $`f`$. With $`d_j = (x_j - x_j')^2`$, $`w_j = 1`$, $`f = \sqrt{\cdot}`$ it's Euclidean distance; weights let you emphasise features; categorical features can use $`d_j = \mathbb{1}(x_j \ne x_j')`$ (≈ one-hot + Euclidean).

**Dissimilarity → Euclidean (multidimensional scaling, MDS):** given $`D_{ij} = \lVert\mathbf{x}_i - \mathbf{x}_j\rVert_2^2`$, can we recover points? Only up to rotation/translation. With $`M_{ij} = \mathbf{x}_i\cdot\mathbf{x}_j`$ we have $`D_{ij} = M_{ii} + M_{jj} - 2M_{ij}`$; assuming centred data ($`\sum_i\mathbf{x}_i = 0`$) $`M`$ is recoverable from $`D`$ (exercise; answer: centring makes the row and column sums of $`M`$ vanish, so **double-centre** $`D`$: $`M_{ij} = -\tfrac12\big(D_{ij} - \bar D_{i\cdot} - \bar D_{\cdot j} + \bar D_{\cdot\cdot}\big)`$, subtracting row and column means and adding the grand mean; without centring $`M`$ is determined only up to translation). $`M`$ is symmetric PSD, so its SVD is $`M = U\Sigma U^\top`$; set $`\widetilde{\mathbf{X}} = U\Sigma^{1/2}`$ (rows $`\widetilde{\mathbf{x}}_i`$). Then $`\widetilde{\mathbf{X}}\widetilde{\mathbf{X}}^\top = U\Sigma^{1/2}\Sigma^{1/2}U^\top = M`$: an embedding with the right inner products. In practice one often defines a dissimilarity directly (Hamming distance for DNA sequences) or a similarity (cosine kernel for text); when the similarity is a **Mercer kernel**, $`M`$ is PSD and the SVD construction is valid. If $`M`$ isn't PSD, minimise the **stress** $`\sum_{i\ne j}\big(\lVert\widetilde{\mathbf{x}}_i - \widetilde{\mathbf{x}}_j\rVert_2^2 - D_{ij}\big)^2`$ by gradient descent (non-convex, local optimum).

---

## 34. Hierarchical clustering
Flat clustering loses structure: with articles on physics, maths, cricket, football and $`k = 3`$ you must break one natural pair. A **dendrogram** (tree; leaves = data, internal nodes = clusters at different granularities) keeps it. **Divisive** algorithms split top-down (e.g. k-means with $`k = 2`$ recursively); **agglomerative** algorithms merge bottom-up: the **linkage algorithms**.

**Linkage algorithm:** start with $`N`$ singleton clusters; repeat $`N-1`$ times: merge the two **closest** clusters, then update the dissimilarity of the new cluster to all others using a linkage rule:
```math
\text{single: } d(C, C') = \min_{\mathbf{x}\in C,\mathbf{x}'\in C'}d(\mathbf{x}, \mathbf{x}'), \qquad
\text{average: } d(C, C') = \frac{1}{\lvert C\rvert\lvert C'\rvert}\sum_{\mathbf{x}\in C,\mathbf{x}'\in C'}d(\mathbf{x}, \mathbf{x}'), \qquad
\text{complete: } d(C, C') = \max_{\mathbf{x}\in C,\mathbf{x}'\in C'}d(\mathbf{x}, \mathbf{x}').
```
Single = least dissimilar pair, complete = most dissimilar pair, average usually works well in practice. Input is just the $`N\times N`$ dissimilarity matrix. **Reading a dendrogram:** the height of a merge is the dissimilarity at which the clusters were joined (Fig. 3: physics–maths at 0.1, cricket–football at 0.2, all four at 0.5); cutting the tree at a height gives a flat clustering.

---

## 35. Spectral clustering

> [!NOTE]
> **📖 Key definitions**
> - **Idea:** if we *knew* the data were rings we'd switch to polar coordinates; spectral clustering finds such a change of coordinates implicitly (kernel PCA + k-means is a related but different route). For well-clustered data most of a point's $`k`$ nearest neighbours lie in its own cluster and get higher weights, so clusters in the data become clusters in the graph.
> - **Similarity graph:** one node per point; connect each node to its $`k`$ nearest neighbours (undirected, so degrees may exceed $`k`$); edge weight decreasing in distance, e.g. $`w_{ij} = \exp(-\lVert\mathbf{x}_i - \mathbf{x}_j\rVert_2^2/\sigma)`$.
> - **Weighted adjacency matrix** $`W`$ (symmetric, $`W_{ii} = 0`$); **degree matrix** $`D = \operatorname{diag}(\sum_jW_{ij})`$; **Laplacian** $`L = D - W`$. Always PSD; smallest eigenvalue 0 with eigenvector $`\mathbf{1}`$. **Normalised Laplacian** $`\widetilde L = I - D^{-1}W`$ (better for unbalanced degrees).
> - **Cluster in a graph:** a node subset with most heavy edges inside and few leaving. **Min-cut** (partition minimising the weight of crossing edges) tends to cut off a single node; balanced/sparsest cuts are better but NP-hard. Spectral clustering is a **relaxation** of sparsest cut.

**Worked example (Fig. 7a):** two unit-weight triangles $`\{1,2,3\}`$ and $`\{4,5,6\}`$ give $`D = 2I`$ and $`L`$ block-diagonal with two copies of $`\begin{bmatrix}2&-1&-1\\-1&2&-1\\-1&-1&2\end{bmatrix}`$.

**Key fact:** a graph with $`k`$ connected components has exactly $`k`$ orthogonal eigenvectors with eigenvalue 0 (exercise); e.g. for two components, $`[1,1,1,-1,-1,-1]^\top`$ is also a 0-eigenvector (Fig. 7a). Eigenvectors are orthogonal because $`L`$ is symmetric; an eigensolver returns *some* orthonormal basis of the 0-eigenspace, not necessarily these two vectors. Nodes with identical connectivity get identical embeddings (Fig. 8a: nodes 1–3 at one point, 4–6 at another). For a graph with two *nearly* separate clusters (Fig. 7c), the **second-smallest** eigenvalue is small and its eigenvector still separates them (Fig. 8b: nodes 1–3 near one value, 4–6 near another).

> **Algorithm (spectral clustering).** Input $`W`$, $`k`$.
> 1. $`D`$, $`L = D - W`$.
> 2. Eigenvectors $`\mathbf{v}_1, \dots, \mathbf{v}_k`$ of the $`k`$ **smallest** eigenvalues.
> 3. Embedding matrix $`\mathbf{V}_k = [\mathbf{v}_2, \dots, \mathbf{v}_k]`$ ($`N\times(k-1)`$); row $`i`$ is the embedding of point $`i`$. ($`\mathbf{v}_1 \propto\mathbf{1}`$ is dropped, assuming a connected graph; otherwise run per component.)
> 4. Cluster the rows, e.g. with k-means.

Like kernel PCA, the first half is **non-linear dimensionality reduction**; on the concentric rings it produces three clean groups (Fig. 9).

> [!TIP]
> **✅ Key takeaways (clustering)**
> 1. k-means: NP-hard objective, Lloyd's alternating heuristic converges to a local optimum; restart several times; choose $`k`$ by the **elbow**; convex clusters only.
> 2. Convert formats: features → dissimilarities via weighted per-feature distances; dissimilarities → coordinates via MDS (SVD of the Gram matrix, or minimise stress).
> 3. Hierarchical (agglomerative) clustering builds a dendrogram with single/average/complete linkage.
> 4. Spectral clustering: build a $`k`$-NN similarity graph, take the bottom eigenvectors of the Laplacian as coordinates, then k-means. Handles non-convex clusters.

> [!WARNING]
> **⚠️ Common misconceptions & errors**
> - ❌ "Lloyd's algorithm finds the optimal k-means clustering." ✅ Local optimum only.
> - ❌ "Pick $`k`$ minimising $`W`$." ✅ That gives $`k = N`$; use the elbow.
> - ❌ "The Laplacian's largest eigenvectors are used." ✅ The **smallest** (and the very smallest is the trivial $`\mathbf{1}`$).
> - ❌ "Single linkage uses the cluster means." ✅ It uses the closest pair of points; none of the three linkages uses means.
> - ❌ "MDS recovers the original coordinates." ✅ Only up to rotation/translation (any isometry).
