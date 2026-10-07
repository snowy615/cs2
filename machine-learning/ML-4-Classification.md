# Machine Learning · Part 4: Classification

*Lectures 10–13. [← Part 3](ML-3-Optimisation.md) · [Overview](ML-Summary.md) · [Part 5 →](ML-5-Neural-Networks.md) · [Formula sheet](ML-7-Formula-Sheet.md)*

Classification = supervised learning with a **categorical** output $`y \in \{1, \dots, C\}`$. Two families: **generative** models learn the joint $`p(\mathbf{x}, y)`$ and use Bayes' rule to classify; **discriminative** models learn $`p(y\mid\mathbf{x})`$ (or just a decision rule) directly. There is no principled way to decide which family is better in general; each has advantages (Murphy §8.6).

---

## Contents
16. [Generative models: Naïve Bayes and Gaussian discriminant analysis](#16-generative-models-naïve-bayes-and-gaussian-discriminant-analysis)
17. [Logistic regression](#17-logistic-regression)
18. [Support vector machines: primal](#18-support-vector-machines-primal)
19. [SVM dual, KKT and support vectors](#19-svm-dual-kkt-and-support-vectors)
20. [Kernels](#20-kernels)
21. [Multiclass strategies and measuring performance](#21-multiclass-strategies-and-measuring-performance)

---

## 16. Generative models: Naïve Bayes and Gaussian discriminant analysis

> [!NOTE]
> **📖 Key definitions**
> - **Generative model:** $`p(\mathbf{x}, y\mid\boldsymbol\theta, \boldsymbol\pi) = p(y\mid\boldsymbol\pi)\cdot p(\mathbf{x}\mid y, \boldsymbol\theta)`$: a **class prior** $`p(y = c\mid\boldsymbol\pi) = \pi_c`$ (with $`\sum_c\pi_c = 1`$) times **class-conditional** distributions.
> - **Prediction via Bayes' rule:**
> ```math
> p(y = c\mid\mathbf{x}_{\text{new}}, \boldsymbol\theta) = \frac{p(y=c\mid\boldsymbol\theta)\,p(\mathbf{x}_{\text{new}}\mid y=c, \boldsymbol\theta)}{\sum_{c'=1}^C p(y=c'\mid\boldsymbol\theta)\,p(\mathbf{x}_{\text{new}}\mid y=c', \boldsymbol\theta)}, \qquad \widehat{y} = \arg\max_c p(y=c\mid\mathbf{x}_{\text{new}}, \boldsymbol\theta).
> ```
> (numerator = joint, denominator = marginal $`p(\mathbf{x}_{\text{new}})`$.)
> - **Naïve Bayes assumption:** features are **conditionally independent given the class**: $`p(\mathbf{x}\mid y=c, \boldsymbol\theta_c) = \prod_{j=1}^D p(x_j\mid y=c, \boldsymbol\theta_{jc})`$. (Not "features are independent": they're dependent, but *only through the label*.)
> - **Gaussian discriminant analysis (GDA):** $`p(\mathbf{x}\mid y=c) = \mathcal{N}(\mathbf{x}\mid\boldsymbol\mu_c, \boldsymbol\Sigma_c)`$. **QDA:** separate $`\boldsymbol\Sigma_c`$ per class (quadratic boundaries). **LDA:** shared $`\boldsymbol\Sigma`$ (**parameter tying**) ⇒ linear boundaries.
> - **Softmax:** $`\operatorname{softmax}(\boldsymbol\eta)_c = \dfrac{\exp(\eta_c)}{\sum_{c'}\exp(\eta_{c'})}`$: maps any real vector to a probability distribution with its mode at the max. Invariant to adding a constant to all entries, **not** to scaling: $`\operatorname{softmax}([1,2,3]) \approx [0.09, 0.245, 0.665]`$ but $`\operatorname{softmax}([10,20,30]) \approx [2\times10^{-9}, 4\times10^{-5}, 1]`$.
> - **Sigmoid:** $`\sigma(t) = \dfrac{1}{1 + e^{-t}} \in (0,1)`$, monotone, $`\sigma(t) \to 0`$ as $`t \to -\infty`$ and $`\to 1`$ as $`t \to \infty`$. The 2-class softmax.

Running example: predict a voter's candidate from (voted in 2012? yes/no, income, state).

### 16.1 Estimating the class prior $`\boldsymbol\pi`$ (Lagrange multipliers)
Likelihood of i.i.d. data: $`p(\mathcal{D}\mid\boldsymbol\theta,\boldsymbol\pi) = \prod_{i=1}^N\Big(\prod_{c=1}^C\pi_c^{\mathbb{1}(y_i = c)}\Big)p(\mathbf{x}_i\mid y_i, \boldsymbol\theta)`$. With $`N_c`$ = number of points in class $`c`$:
```math
\log p(\mathcal{D}\mid\boldsymbol\theta,\boldsymbol\pi) = \sum_{c=1}^C N_c\log\pi_c + \sum_{i=1}^N\log p(\mathbf{x}_i\mid y_i, \boldsymbol\theta).
```
The two terms **separate**: $`\boldsymbol\pi`$ can be estimated without knowing the class-conditional model. Maximise $`\sum_c N_c\log\pi_c`$ subject to $`\sum_c\pi_c = 1`$.

**Method of Lagrange multipliers.** For $`\arg\max_{\mathbf{z}} f(\mathbf{z})`$ subject to $`g(\mathbf{z}) = 0`$, form $`\Lambda(\mathbf{z}, \lambda) = f(\mathbf{z}) + \lambda g(\mathbf{z})`$. A solution must be a stationary point of $`\Lambda`$: $`\partial\Lambda/\partial\lambda = 0`$ recovers the constraint, and $`\nabla_{\mathbf{z}}\Lambda = 0`$ gives $`\nabla f = -\lambda\nabla g`$ (gradients parallel; otherwise you could increase $`f`$ while staying on $`g = 0`$).

Here $`\Lambda(\boldsymbol\pi, \lambda) = \sum_c N_c\log\pi_c + \lambda(\sum_c\pi_c - 1)`$:
```math
\frac{\partial\Lambda}{\partial\pi_c} = \frac{N_c}{\pi_c} + \lambda = 0 \ \Rightarrow\ \pi_c = -\frac{N_c}{\lambda}; \qquad \sum_c\pi_c = 1 \ \Rightarrow\ -\frac{\sum_c N_c}{\lambda} = 1 \ \Rightarrow\ \lambda = -N.
```
> ```math
> \pi_c = \frac{N_c}{N} \qquad \text{(empirical class frequencies).}
> ```

### 16.2 Naïve Bayes classifier (NBC)
Without the assumption, a distribution over $`D`$ binary features needs $`2^D`$ parameters per class ($`C\cdot2^D`$ total): it would fit anything and overfit, and even with $`\gg 2^D`$ data it would be computationally prohibitive (too many parameters hurts both statistically and computationally). With it, the log-likelihood factorises per feature:
```math
\log p(\mathcal{D}\mid\boldsymbol\theta,\boldsymbol\pi) = \sum_{c=1}^C N_c\log\pi_c + \sum_{c=1}^C\sum_{j=1}^D\sum_{i: y_i = c}\log p(x_{ij}\mid\boldsymbol\theta_{jc}),
```
so each $`\boldsymbol\theta_{jc}`$ is fitted using **only feature $`j`$ of the class-$`c`$ points**: $`O(C\cdot D)`$ parameters for binary features ($`O(C\cdot D\cdot K)`$ with $`K`$-valued categoricals), linear rather than exponential in $`D`$; one-dimensional problems, easy MLEs:
| Feature type | Model for $`p(x_j\mid y = c)`$ | MLE |
|---|---|---|
| Real ($`x_j \in \mathbb{R}`$) | Gaussian $`\mathcal{N}(\mu_{jc}, \sigma^2_{jc})`$ (or Laplace, etc.) | empirical mean and variance within class $`c`$ |
| Categorical ($`K`$ values) | "multinoulli" $`p(x_j = l\mid y=c) = \mu_{jc,l}`$, $`\sum_l\mu_{jc,l} = 1`$ | $`\mu_{jc,l}`$ = fraction of class-$`c`$ points with $`x_{ij} = l`$ (same Lagrange argument) |
| Binary | Bernoulli($`\theta_{jc}`$) | fraction of class-$`c`$ points with $`x_{ij} = 1`$ |

You can **mix feature types** freely; no need to one-hot encode. The assumption is "far too simplistic" yet NBC works surprisingly well, because the goal is **classification, not modelling the data** (building a generative model can be a goal in itself, in unsupervised settings where you want to generate new data; here it's only a tool); simpler models often predict better with limited data.

### 16.3 Gaussian discriminant analysis
MLEs (Murphy §4.1):
```math
\widehat{\boldsymbol\mu}_c = \frac{1}{N_c}\sum_{i: y_i = c}\mathbf{x}_i, \qquad \widehat{\boldsymbol\Sigma}_c = \frac{1}{N_c}\sum_{i: y_i=c}(\mathbf{x}_i - \widehat{\boldsymbol\mu}_c)(\mathbf{x}_i - \widehat{\boldsymbol\mu}_c)^\top .
```
**QDA posterior:**
```math
p(y = c\mid\mathbf{x}, \boldsymbol\theta) = \frac{\pi_c\lvert 2\pi\boldsymbol\Sigma_c\rvert^{-1/2}\exp\big(-\tfrac12(\mathbf{x}-\boldsymbol\mu_c)^\top\boldsymbol\Sigma_c^{-1}(\mathbf{x}-\boldsymbol\mu_c)\big)}{\sum_{c'}\pi_{c'}\lvert 2\pi\boldsymbol\Sigma_{c'}\rvert^{-1/2}\exp\big(-\tfrac12(\mathbf{x}-\boldsymbol\mu_{c'})^\top\boldsymbol\Sigma_{c'}^{-1}(\mathbf{x}-\boldsymbol\mu_{c'})\big)}.
```
**Decision boundary** between $`c`$ and $`c'`$: where the posteriors are equal (ratio 1). Taking logs:
```math
\tfrac12\Big((\mathbf{x}-\boldsymbol\mu_{c'})^\top\boldsymbol\Sigma_{c'}^{-1}(\mathbf{x}-\boldsymbol\mu_{c'}) - (\mathbf{x}-\boldsymbol\mu_c)^\top\boldsymbol\Sigma_c^{-1}(\mathbf{x}-\boldsymbol\mu_c)\Big) = \log\frac{\pi_{c'}\lvert2\pi\boldsymbol\Sigma_{c'}\rvert^{-1/2}}{\pi_c\lvert2\pi\boldsymbol\Sigma_c\rvert^{-1/2}},
```
a **quadratic curve** in $`\mathbf{x}`$ (hence "quadratic discriminant"). With more than two classes not every $`\mathbf{x}`$ on this curve is on the $`c`$/$`c'`$ boundary (a third class $`\tilde c`$ may have a larger posterior there), so the actual boundaries are **pieces** of these quadratic curves.

**LDA (shared $`\boldsymbol\Sigma`$):** the $`\lvert2\pi\boldsymbol\Sigma\rvert^{-1/2}`$ factors cancel and expanding the quadratic,
```math
p(y=c\mid\mathbf{x}) \propto \pi_c\exp\big(-\tfrac12(\mathbf{x}-\boldsymbol\mu_c)^\top\boldsymbol\Sigma^{-1}(\mathbf{x}-\boldsymbol\mu_c)\big) = \exp\Big(\underbrace{\boldsymbol\mu_c^\top\boldsymbol\Sigma^{-1}\mathbf{x} - \tfrac12\boldsymbol\mu_c^\top\boldsymbol\Sigma^{-1}\boldsymbol\mu_c + \log\pi_c}_{\text{linear in }\mathbf{x}}\Big)\cdot\underbrace{\exp\big(-\tfrac12\mathbf{x}^\top\boldsymbol\Sigma^{-1}\mathbf{x}\big)}_{\text{same for every class}}.
```
With $`\boldsymbol\beta_c = \boldsymbol\Sigma^{-1}\boldsymbol\mu_c`$ and $`\gamma_c = -\tfrac12\boldsymbol\mu_c^\top\boldsymbol\Sigma^{-1}\boldsymbol\mu_c + \log\pi_c`$:
> ```math
> p(y = c\mid\mathbf{x}, \boldsymbol\theta) = \frac{\exp(\boldsymbol\beta_c^\top\mathbf{x} + \gamma_c)}{\sum_{c'}\exp(\boldsymbol\beta_{c'}^\top\mathbf{x} + \gamma_{c'})} = \operatorname{softmax}(\boldsymbol\eta)_c, \quad \boldsymbol\eta = [\boldsymbol\beta_1^\top\mathbf{x} + \gamma_1, \dots, \boldsymbol\beta_C^\top\mathbf{x} + \gamma_C].
> ```
The $`\mathbf{x}^\top\boldsymbol\Sigma^{-1}\mathbf{x}`$ term cancels in the ratio, so boundaries are **linear** (Fig. 1b).

**Two-class LDA ⇒ sigmoid:**
```math
p(y = 1\mid\mathbf{x}) = \frac{e^{\boldsymbol\beta_1^\top\mathbf{x} + \gamma_1}}{e^{\boldsymbol\beta_1^\top\mathbf{x} + \gamma_1} + e^{\boldsymbol\beta_0^\top\mathbf{x} + \gamma_0}} = \frac{1}{1 + \exp\big(-((\boldsymbol\beta_1 - \boldsymbol\beta_0)^\top\mathbf{x} + (\gamma_1 - \gamma_0))\big)} = \operatorname{sigmoid}\big((\boldsymbol\beta_1-\boldsymbol\beta_0)^\top\mathbf{x} + (\gamma_1 - \gamma_0)\big).
```
Same functional form as logistic regression (next), but the parameters are obtained completely differently.

**When to prefer GDA over Naïve Bayes:** when features are correlated *within* a class (height and weight of zebras vs giraffes), a full-covariance Gaussian captures this and NB cannot.

**Overfitting:** GDA has $`\approx C\cdot D^2`$ parameters. Remedies: diagonal $`\boldsymbol\Sigma_c`$ (**= Naïve Bayes with Gaussian features**), shared $`\boldsymbol\Sigma`$ (LDA), or priors. Naming warning: "discriminant analysis" refers to the shape of the boundary; these are **generative** models.

> [!TIP]
> **✅ Key takeaways**
> 1. Generative classification = model $`p(y)`$ and $`p(\mathbf{x}\mid y)`$, classify with Bayes' rule. The class prior MLE is always $`N_c/N`$.
> 2. Naïve Bayes: conditional independence ⇒ one 1-D estimation per (feature, class); parameters linear in $`D`$ ($`O(CD)`$ for binary features); mixes feature types; surprisingly good.
> 3. QDA: per-class covariance ⇒ quadratic boundaries. LDA: tied covariance ⇒ linear boundaries ⇒ **softmax** (sigmoid for two classes).
> 4. The Lagrange-multiplier derivation (gradients parallel at the constrained optimum) is reused for SVMs.

> [!WARNING]
> **⚠️ Common misconceptions & errors**
> - ❌ "Naïve Bayes assumes features are independent." ✅ **Conditionally** independent given the class.
> - ❌ "LDA is a discriminative model." ✅ Generative; the name refers to the linear boundary.
> - ❌ "Softmax is scale-invariant." ✅ Shift-invariant only; scaling sharpens it.
> - ❌ Using $`N`$ instead of $`N_c`$ in the class-conditional MLEs.
> - ❌ Forgetting the $`\log\pi_c`$ and $`\log\lvert\boldsymbol\Sigma_c\rvert`$ terms in the QDA boundary.

---

## 17. Logistic regression

> [!NOTE]
> **📖 Key definitions**
> - **Logistic regression model** (binary, labels $`\{0,1\}`$, bias folded into $`\mathbf{x}`$): $`y \sim \operatorname{Bernoulli}(\sigma(\mathbf{w}\cdot\mathbf{x}))`$, i.e. $`p(y = 1\mid\mathbf{x}, \mathbf{w}) = \sigma(\mathbf{w}\cdot\mathbf{x}) = \dfrac{1}{1 + \exp(-\mathbf{w}\cdot\mathbf{x})}`$. A **discriminative** model; inputs treated as fixed.
> - **Prediction:** $`\widehat{y}_{\text{new}} = \mathbb{1}\big(\sigma(\mathbf{w}\cdot\mathbf{x}_{\text{new}}) \ge \tfrac12\big) = \mathbb{1}(\mathbf{w}\cdot\mathbf{x}_{\text{new}} \ge 0)`$: a **linear** boundary (hyperplane). Since $`\sigma(\mathbf{w}\cdot\mathbf{x})`$ depends on $`\mathbf{x}`$ only through $`\mathbf{w}\cdot\mathbf{x}`$, the contours of the predicted probability are hyperplanes parallel to the boundary (Fig. 2; the ★ points are misclassified training points).
> - **Generalised linear model:** $`\mathbb{E}[y\mid\mathbf{x}] = f(\mathbf{w}\cdot\mathbf{x})`$ for a fixed univariate $`f`$; logistic regression has $`f = \sigma`$. GLMs capture (limited) non-linearity without basis expansion and are also used for regression; don't confuse them with "general linear models". ("Regression" in the name is historical; logistic regression is almost always used for classification.)
> - **Cross-entropy loss** for a single point: $`-\big(y\log\mu + (1-y)\log(1-\mu)\big)`$ with $`\mu`$ = predicted probability of class 1.

### 17.1 Likelihood and NLL
With $`\mu_i = \sigma(\mathbf{w}^\top\mathbf{x}_i)`$ and independent labels,
```math
p(\mathbf{y}\mid\mathbf{X},\mathbf{w}) = \prod_{i=1}^N\mu_i^{y_i}(1 - \mu_i)^{1-y_i}, \qquad \operatorname{NLL}(\mathbf{y}\mid\mathbf{X},\mathbf{w}) = -\sum_{i=1}^N\big(y_i\log\mu_i + (1-y_i)\log(1-\mu_i)\big).
```
(The exponent trick picks $`\mu_i`$ when $`y_i = 1`$ and $`1-\mu_i`$ when $`y_i = 0`$.) Per point this is the **cross-entropy**. If $`y_i = 1`$ the loss is $`-\log\mu_i`$: $`\to 0`$ as $`\mu_i \to 1`$ and $`\to\infty`$ as $`\mu_i \to 0`$: a **huge penalty for being confidently wrong**.

### 17.2 Gradient, Hessian, convexity
(Sheet 3 derives these; use $`\sigma'(t) = \sigma(t)(1-\sigma(t))`$.)
> ```math
> \nabla_{\mathbf{w}}\operatorname{NLL} = \sum_{i=1}^N\mathbf{x}_i(\mu_i - y_i) = \mathbf{X}^\top(\boldsymbol\mu - \mathbf{y}), \qquad \mathbf{H}_{\mathbf{w}} = \mathbf{X}^\top\mathbf{S}\mathbf{X}, \quad \mathbf{S} = \operatorname{diag}\big(\mu_i(1-\mu_i)\big).
> ```
**Convexity proof:** for any $`\mathbf{z}`$, let $`\mathbf{z}' = \mathbf{X}\mathbf{z}`$; then $`\mathbf{z}^\top\mathbf{H}\mathbf{z} = \sum_i S_{ii}(z_i')^2 \ge 0`$ since $`S_{ii} = \mu_i(1-\mu_i) > 0`$. So $`\mathbf{H}`$ is PSD everywhere ⇒ NLL is **convex** ⇒ any convex optimiser finds the global minimum. (If $`N > D`$ and $`\mathbf{X}`$ has rank $`D`$ it's positive definite: $`\mathbf{z}^\top\mathbf{H}\mathbf{z} = 0 \Rightarrow \mathbf{X}\mathbf{z} = 0 \Rightarrow \mathbf{z} = 0`$. If $`\mathbf{X}`$ doesn't have rank $`D`$ we may be in a degenerate case: the Hessian is only PSD and the minimiser needn't be unique.)

**No closed form:** setting the gradient to zero gives non-linear equations in $`\mathbf{w}`$. Use GD, SGD, or Newton.

### 17.3 Newton's method = iteratively reweighted least squares (IRLS)
With $`\mathbf{g}_t = \mathbf{X}^\top(\boldsymbol\mu_t - \mathbf{y})`$ and $`\mathbf{H}_t = \mathbf{X}^\top\mathbf{S}_t\mathbf{X}`$:
```math
\mathbf{w}_{t+1} = \mathbf{w}_t - \mathbf{H}_t^{-1}\mathbf{g}_t = \mathbf{w}_t + (\mathbf{X}^\top\mathbf{S}_t\mathbf{X})^{-1}\mathbf{X}^\top(\mathbf{y} - \boldsymbol\mu_t) = (\mathbf{X}^\top\mathbf{S}_t\mathbf{X})^{-1}\mathbf{X}^\top\mathbf{S}_t\underbrace{\big(\mathbf{X}\mathbf{w}_t + \mathbf{S}_t^{-1}(\mathbf{y} - \boldsymbol\mu_t)\big)}_{\mathbf{z}_t}.
```
(Middle step: write $`\mathbf{w}_t = (\mathbf{X}^\top\mathbf{S}_t\mathbf{X})^{-1}\mathbf{X}^\top\mathbf{S}_t\mathbf{X}\mathbf{w}_t`$ and factor.) So $`\mathbf{w}_{t+1}`$ solves the **weighted least squares** problem $`\min_{\mathbf{w}}\sum_i S_{t,ii}(z_{t,i} - \mathbf{w}^\top\mathbf{x}_i)^2`$ with targets $`\mathbf{z}_t`$ and weights $`\mathbf{S}_t`$ that change every iteration: hence the name.

### 17.4 Multiclass logistic regression
One weight vector $`\mathbf{w}_c`$ per class (columns of a $`D\times C`$ matrix $`\mathbf{W}`$):
```math
p(y = c\mid\mathbf{x}, \mathbf{W}) = \frac{\exp(\mathbf{w}_c\cdot\mathbf{x})}{\sum_{c'=1}^C\exp(\mathbf{w}_{c'}\cdot\mathbf{x})} = \operatorname{softmax}\big([\mathbf{w}_1\cdot\mathbf{x}, \dots, \mathbf{w}_C\cdot\mathbf{x}]^\top\big)_c .
```
Boundaries are still linear (Fig. 3). NLL is convex (Murphy 8.3.7); optimise as before.

**Extensions:** basis expansion and $`\ell_1`$/$`\ell_2`$ regularisation apply exactly as in regression. In fact regularisation is **necessary if the data are linearly separable**: the NLL can be driven to 0 only as $`\lVert\mathbf{w}\rVert \to \infty`$ (scaling $`\mathbf{w}`$ up pushes every $`\mu_i`$ to 0/1), so the MLE doesn't exist.

> [!TIP]
> **✅ Key takeaways**
> 1. Logistic regression = Bernoulli output with sigmoid of a linear function; NLL = cross-entropy; **convex** (PSD Hessian), no closed form.
> 2. Gradient $`\mathbf{X}^\top(\boldsymbol\mu - \mathbf{y})`$ has the same shape as least squares' $`\mathbf{X}^\top(\mathbf{X}\mathbf{w} - \mathbf{y})`$: "prediction minus target".
> 3. Newton on this problem is IRLS: a weighted least-squares solve per step.
> 4. Softmax generalises to $`C`$ classes; LDA and logistic regression share the functional form but differ in how the parameters are fitted (generative vs discriminative).
> 5. Separable data ⇒ regularise.

> [!WARNING]
> **⚠️ Common misconceptions & errors**
> - ❌ "Logistic regression is a regression method." ✅ It's a classifier (GLM with Bernoulli output).
> - ❌ "Threshold at 0.5 ⇔ $`\mathbf{w}\cdot\mathbf{x} \ge 0.5`$." ✅ $`\sigma(t) \ge 0.5 \Leftrightarrow t \ge 0`$.
> - ❌ Writing the NLL with $`\sum`$ but no minus sign, or dropping the $`(1-y_i)\log(1-\mu_i)`$ term.
> - ❌ "Convex ⇒ closed form." ✅ Convex guarantees GD/Newton find the optimum; there's still no formula.
> - ❌ Sign error in Newton: $`\mathbf{w}_{t+1} = \mathbf{w}_t - \mathbf{H}^{-1}\mathbf{g}`$ with $`\mathbf{g} = \mathbf{X}^\top(\boldsymbol\mu - \mathbf{y})`$.

---

## 18. Support vector machines: primal

> [!NOTE]
> **📖 Key definitions** (labels now $`y \in \{-1, +1\}`$)
> - **Hyperplane:** $`\{\mathbf{x} : \mathbf{w}\cdot\mathbf{x} + w_0 = 0\}`$; $`\mathbf{w}`$ is **orthogonal** to it; the two sides are the half-spaces $`\mathbf{w}\cdot\mathbf{x} + w_0 > 0`$ and $`< 0`$.
> - **Distance from $`\mathbf{x}`$ to the hyperplane:** $`\dfrac{\lvert\mathbf{w}\cdot\mathbf{x} + w_0\rvert}{\lVert\mathbf{w}\rVert_2}`$.
> - **Linearly separable data:** some hyperplane puts all positives on one side and all negatives on the other.
> - **Margin** of a data point: its distance from the separating hyperplane. **Maximum-margin principle:** choose the separator that **maximises the smallest margin**.
> - **Support vectors:** the training points closest to the separator (smallest margin); they alone determine the solution.
> - **Signed distance:** $`(\mathbf{w}\cdot\mathbf{x} + w_0)/\lVert\mathbf{w}\rVert_2`$ (positive on the $`\mathbf{w}`$ side); $`\mathbf{w}\cdot\mathbf{x}/\lVert\mathbf{w}\rVert_2`$ is the projection of $`\mathbf{x}`$ onto the normal direction and $`\lvert w_0\rvert/\lVert\mathbf{w}\rVert_2`$ is the hyperplane's distance from the origin (Fig. 2).
> - **Slack variables** $`\zeta_i \ge 0`$: how much point $`i`$ is allowed to violate the margin.
> - **Hinge loss:** $`\ell_{\text{hinge}}(\mathbf{w}, w_0; \mathbf{x}_i, y_i) = \max\{0, 1 - y_i(\mathbf{w}\cdot\mathbf{x}_i + w_0)\}`$.

Unlike logistic regression, the SVM has **no natural probabilistic interpretation** (one can be bolted on; Murphy 14.5.5). This is why its decision threshold is tuned by re-weighting the objective rather than by moving a probability cut-off (§21.2).

### 18.1 Distance to a hyperplane (derivation via Lagrange)
Find the point $`\mathbf{x}`$ on the hyperplane closest to $`\mathbf{x}^*`$: minimise $`\lVert\mathbf{x} - \mathbf{x}^*\rVert_2^2`$ subject to $`\mathbf{w}\cdot\mathbf{x} + w_0 = 0`$. Lagrangian $`\Lambda(\mathbf{x}, \lambda) = \lVert\mathbf{x}-\mathbf{x}^*\rVert_2^2 - 2\lambda(\mathbf{w}\cdot\mathbf{x} + w_0)`$; $`\nabla_{\mathbf{x}}\Lambda = 2\mathbf{x} - 2\mathbf{x}^* - 2\lambda\mathbf{w} = 0`$ gives $`\mathbf{x} = \mathbf{x}^* + \lambda\mathbf{w}`$. Substituting into the constraint: $`\lambda = -\dfrac{\mathbf{w}\cdot\mathbf{x}^* + w_0}{\lVert\mathbf{w}\rVert_2^2}`$. The distance is $`\lVert\lambda\mathbf{w}\rVert_2 = \lvert\lambda\rvert\lVert\mathbf{w}\rVert_2 = \dfrac{\lvert\mathbf{w}\cdot\mathbf{x}^* + w_0\rvert}{\lVert\mathbf{w}\rVert_2}`$. ∎

### 18.2 Separable case: the hard-margin SVM
A separating hyperplane satisfies $`y_i(\mathbf{w}\cdot\mathbf{x}_i + w_0) > 0`$ for all $`i`$. Because the dataset is finite we can rescale $`(\mathbf{w}, w_0)`$ so that
```math
y_i(\mathbf{w}\cdot\mathbf{x}_i + w_0) \ge 1 \qquad (i = 1, \dots, N)
```
with no loss of generality (if the inequality holds with some $`\epsilon > 0`$ on the right, divide $`\mathbf{w}, w_0`$ by $`\epsilon`$). (With *infinitely* many points this can fail: in 1-D, $`\{(x,+1): x > 0\}\cup\{(x,-1): x < 0\}`$ is separable with strict inequalities but no $`(w, w_0)`$ achieves $`y(wx + w_0) \ge 1`$ everywhere.) Then every point's margin is $`\dfrac{y_i(\mathbf{w}\cdot\mathbf{x}_i + w_0)}{\lVert\mathbf{w}\rVert_2} \ge \dfrac{1}{\lVert\mathbf{w}\rVert_2}`$ (the signed version equals the absolute one because $`y_i`$ is the sign). So maximising the minimum margin $`1/\lVert\mathbf{w}\rVert_2`$ is equivalent to:
> ```math
> \min_{\mathbf{w}, w_0}\ \tfrac12\lVert\mathbf{w}\rVert_2^2 \qquad\text{subject to}\qquad y_i(\mathbf{w}\cdot\mathbf{x}_i + w_0) \ge 1 \quad (i = 1, \dots, N).
> ```
A **quadratic program**: the objective $`\tfrac12\lVert\mathbf{w}\rVert^2`$ is a PSD quadratic (convex), and each constraint is a half-space in $`\mathbb{R}^{D+1}`$ (variables $`\mathbf{w}, w_0`$), whose intersection is convex. Generic convex solvers work but can be slow; specialised QP methods (via the dual) are faster. The constraint is **tight** exactly for the support vectors (Fig. 3: the dotted lines through them). *Sheet 3 exercise:* show that the "natural" formulation $`\max_{\mathbf{w}, w_0}\min_i y_i(\mathbf{w}\cdot\mathbf{x}_i + w_0)/\lVert\mathbf{w}\rVert_2`$ is equivalent to this program.

### 18.3 Non-separable case: the soft-margin SVM
If no separator exists the program above is infeasible, and we now expect non-zero training *and* test error. Minimising the *number* of misclassified points (≈ satisfying as many constraints as possible) is **NP-hard** (unless P = NP). Instead allow each constraint to be violated by a slack $`\zeta_i`$ and pay for it:
> ```math
> \min_{\mathbf{w}, w_0, \boldsymbol\zeta}\ \tfrac12\lVert\mathbf{w}\rVert_2^2 + C\sum_{i=1}^N\zeta_i \quad\text{s.t.}\quad y_i(\mathbf{w}\cdot\mathbf{x}_i + w_0) \ge 1 - \zeta_i,\quad \zeta_i \ge 0 \quad (i = 1,\dots,N).
> ```
With slack the program is **always feasible** (take $`\zeta_i`$ large enough). $`\zeta_i \ge 0`$ matters: negative slack would let a point far on the correct side "pay for" misclassified ones. There is **no bonus** for a huge margin, only a penalty for a small or negative one. $`C`$ trades margin against violations (a hyperparameter; cf. $`1/\lambda`$).

### 18.4 Loss-function view
At the optimum, $`\zeta_i = \max\{0, 1 - y_i(\mathbf{w}\cdot\mathbf{x}_i + w_0)\}`$ (either zero or exactly the violation). So the SVM is unconstrained minimisation of
```math
\mathcal{L}_{\text{SVM}}(\mathbf{w}, w_0\mid\mathbf{X},\mathbf{y}) = \underbrace{\tfrac12\lVert\mathbf{w}\rVert_2^2}_{\text{regulariser}} + C\sum_{i=1}^N\underbrace{\max\{0, 1 - y_i(\mathbf{w}\cdot\mathbf{x}_i + w_0)\}}_{\text{hinge loss}}.
```
**Compare logistic regression:** mapping $`y_i \in \{0,1\}`$ to $`z_i = 2y_i - 1 \in \{-1, 1\}`$, the per-point NLL is $`\log\big(1 + e^{-z_i(\mathbf{w}\cdot\mathbf{x}_i)}\big)`$, the **logistic loss**. Both are decreasing functions of the margin $`z(\mathbf{w}\cdot\mathbf{x})`$ (Fig. 5); hinge is exactly 0 beyond margin 1, logistic only approaches 0. Same "regulariser + sum of losses" template as ridge.

---

## 19. SVM dual, KKT and support vectors

> [!NOTE]
> **📖 Key definitions**
> - **Primal:** minimise $`F(\mathbf{z})`$ s.t. $`g_i(\mathbf{z}) \ge 0`$ ($`i = 1..m`$), $`h_j(\mathbf{z}) = 0`$ ($`j = 1..l`$).
> - **Lagrangian** with dual variables $`\alpha_i`$ (inequalities) and $`\mu_j`$ (equalities): $`\Lambda(\mathbf{z}; \boldsymbol\alpha, \boldsymbol\mu) = F(\mathbf{z}) - \sum_i\alpha_ig_i(\mathbf{z}) - \sum_j\mu_jh_j(\mathbf{z})`$.
> - **KKT conditions** (necessary for a minimum; **necessary and sufficient** when $`F, g_i`$ convex and $`h_j`$ affine, as for the SVM):
>   0. **Stationarity:** $`(\mathbf{z}^*, \boldsymbol\alpha^*, \boldsymbol\mu^*)`$ is a critical point of $`\Lambda`$, i.e. $`\nabla_{\mathbf{z}}\Lambda = \mathbf{0}`$ (this is what "set the primal derivatives to zero" uses below);
>   1. **Primal feasibility:** $`g_i(\mathbf{z}^*) \ge 0`$, $`h_j(\mathbf{z}^*) = 0`$;
>   2. **Dual feasibility:** $`\alpha_i^* \ge 0`$;
>   3. **Complementary slackness:** $`\alpha_i^*\,g_i(\mathbf{z}^*) = 0`$ for every $`i`$ (either the multiplier is zero or the constraint is tight).
> - **Dual problem:** maximise, over the dual variables, the Lagrangian minimised over the primal variables.

### 19.1 Deriving the SVM dual
Lagrangian of the soft-margin program (multipliers $`\alpha_i`$ for the margin constraints, $`\mu_i`$ for $`\zeta_i \ge 0`$):
```math
\Lambda(\mathbf{w}, w_0, \boldsymbol\zeta; \boldsymbol\alpha, \boldsymbol\mu) = \tfrac12\mathbf{w}^\top\mathbf{w} + C\sum_i\zeta_i - \sum_i\alpha_i\big(y_i(\mathbf{w}\cdot\mathbf{x}_i + w_0) - 1 + \zeta_i\big) - \sum_i\mu_i\zeta_i .
```
Set the derivatives with respect to the **primal** variables to zero:
```math
\frac{\partial\Lambda}{\partial w_0} = -\sum_i\alpha_iy_i = 0, \qquad \frac{\partial\Lambda}{\partial\zeta_i} = C - \alpha_i - \mu_i = 0, \qquad \nabla_{\mathbf{w}}\Lambda = \mathbf{w} - \sum_i\alpha_iy_i\mathbf{x}_i = \mathbf{0}.
```
So $`\mathbf{w} = \sum_i\alpha_iy_i\mathbf{x}_i`$, $`\sum_i\alpha_iy_i = 0`$, and $`\mu_i = C - \alpha_i \ge 0`$ combined with $`\alpha_i \ge 0`$ gives $`0 \le \alpha_i \le C`$ (the $`\mu_i`$ disappear). Substituting $`\mathbf{w}`$ back:
```math
g(\boldsymbol\alpha) = \tfrac12\sum_i\sum_j\alpha_i\alpha_jy_iy_j(\mathbf{x}_i\cdot\mathbf{x}_j) + \sum_i\alpha_i - \sum_i\alpha_iy_i\mathbf{x}_i\cdot\Big(\sum_j\alpha_jy_j\mathbf{x}_j\Big) = \sum_{i=1}^N\alpha_i - \tfrac12\sum_{i=1}^N\sum_{j=1}^N\alpha_i\alpha_jy_iy_j(\mathbf{x}_i\cdot\mathbf{x}_j).
```
(The $`C\sum\zeta_i`$, $`-\sum\alpha_i\zeta_i`$ and $`-\sum\mu_i\zeta_i`$ terms cancel because $`C - \alpha_i - \mu_i = 0`$; the $`w_0`$ term vanishes because $`\sum\alpha_iy_i = 0`$.) $`g`$ is a **concave quadratic**, so it has a unique maximum value (and a unique maximiser when the Gram matrix is positive definite, §20.3):
> **SVM dual:**
> ```math
> \max_{\boldsymbol\alpha}\ \sum_{i=1}^N\alpha_i - \tfrac12\sum_{i=1}^N\sum_{j=1}^N\alpha_i\alpha_jy_iy_j(\mathbf{x}_i\cdot\mathbf{x}_j) \quad\text{s.t.}\quad \sum_{i=1}^N\alpha_iy_i = 0,\quad 0 \le \alpha_i \le C .
> ```

### 19.2 Primal vs dual
| | Primal | Dual |
|---|---|---|
| Variables | $`N + D + 1`$ | $`N`$ |
| Constraints | complex polytope | **box** $`0 \le \alpha_i \le C`$ + one equality |
| Objective size | $`N + D`$ terms | $`\approx N^2`$ inner products |
| Best when | $`D \ll N`$ | $`D`$ huge (e.g. after basis expansion) |

### 19.3 Support vectors and complementary slackness
Complementary slackness: $`\alpha_i\big(y_i(\mathbf{w}\cdot\mathbf{x}_i + w_0) - 1 + \zeta_i\big) = 0`$. So either $`\alpha_i = 0`$ or the margin constraint is **tight**, i.e. $`\mathbf{x}_i`$ is a **support vector**. Since
```math
\mathbf{w} = \sum_{i=1}^N\alpha_iy_i\mathbf{x}_i,
```
only points with $`\alpha_i > 0`$ contribute: the support vectors literally **support** the solution. Complementary slackness also holds for the $`\zeta_i \ge 0`$ constraints: $`\mu_i\zeta_i = (C - \alpha_i)\zeta_i = 0`$. Hence three cases: $`\alpha_i = 0`$ ⇒ $`\zeta_i = 0`$, point beyond the margin; $`0 < \alpha_i < C`$ ⇒ $`\zeta_i = 0`$ and $`y_i(\mathbf{w}\cdot\mathbf{x}_i + w_0) = 1`$, point **exactly on** the margin; $`\alpha_i = C`$ ⇒ $`\zeta_i`$ may be positive, point inside the margin or misclassified. (Exercise: recover $`w_0`$ from any support vector with $`0 < \alpha_i < C`$ using $`y_i(\mathbf{w}\cdot\mathbf{x}_i + w_0) = 1`$; once $`\mathbf{w}`$ is known this is a one-dimensional problem, linear time given the pairs $`(\mathbf{w}\cdot\mathbf{x}_i, y_i)`$.)

---

## 20. Kernels

> [!NOTE]
> **📖 Key definitions**
> - **Kernel:** a function $`\kappa : \mathcal{X}\times\mathcal{X} \to \mathbb{R}`$ (we focus on symmetric, non-negative ones). $`\mathcal{X}`$ need not be $`\mathbb{R}^D`$; it can be strings, graphs, …
> - **Gram matrix:** $`\mathbf{K}_{ij} = \kappa(\mathbf{x}_i, \mathbf{x}_j)`$ ($`N\times N`$).
> - **Mercer (positive definite) kernel:** the Gram matrix is PSD for **every** finite set of inputs. Any kernel that is an inner product in some (possibly implicit, possibly infinite-dimensional) feature space is Mercer; **conversely every Mercer kernel is such an inner product**.
> - **Kernel trick:** compute $`\phi(\mathbf{x})\cdot\phi(\mathbf{x}')`$ *without* computing $`\phi`$.
> - **Polynomial kernel** of degree $`d`$: $`\kappa_d(\mathbf{x}, \mathbf{x}') = (1 + \mathbf{x}\cdot\mathbf{x}')^d`$, computable in $`O(D\log d)`$.
> - **RBF / spherical Gaussian kernel:** $`\kappa(\mathbf{x}, \mathbf{x}') = \exp\Big(-\dfrac{\lVert\mathbf{x}-\mathbf{x}'\rVert_2^2}{2\sigma^2}\Big)`$, **bandwidth** $`\sigma`$ ($`\gamma = 1/2\sigma^2`$ in Part 2). "RBF" = depends only on $`\lVert\mathbf{x}-\mathbf{x}'\rVert`$. (A general Gaussian with covariance $`\boldsymbol\Sigma`$ is Mercer but not RBF.)
> - **String kernel:** $`\kappa(\mathbf{x}, \mathbf{x}') = \sum_{s\in\mathcal{A}^*}w_s\phi_s(\mathbf{x})\phi_s(\mathbf{x}')`$ with $`\phi_s(\mathbf{x})`$ = number of occurrences of substring $`s`$; Mercer; computable in $`O(\lvert\mathbf{x}\rvert + \lvert\mathbf{x}'\rvert)`$. Example: proteins as strings over the 20-letter amino-acid alphabet; the two proteins in the notes (lengths 107 and 150) share the substring `LQE`, which $`\phi_{\texttt{LQE}}`$ counts.

### 20.1 Why kernels appear
In the dual objective and in the solution $`\mathbf{w} = \sum_i\alpha_iy_i\mathbf{x}_i`$, inputs appear **only through inner products**: predicting a new point uses
```math
\mathbf{w}\cdot\mathbf{x}_{\text{new}} = \sum_{i=1}^N\alpha_iy_i(\mathbf{x}_i\cdot\mathbf{x}_{\text{new}}) \quad (\text{only support vectors contribute}).
```
So the SVM only needs black-box access to $`\kappa(\mathbf{x}, \mathbf{x}') = \mathbf{x}\cdot\mathbf{x}'`$. Replace it by $`\phi(\mathbf{x})\cdot\phi(\mathbf{x}')`$ for any feature map and you get a non-linear classifier with the **same $`N`$ dual variables**: increasingly complex classifiers at no increase in the number of optimisation variables (but still a risk of overfitting; regularise via $`C`$).

### 20.2 The kernel trick (polynomial example)
The plain degree-2 expansion for $`D = 2`$ is $`\psi_2([x_1, x_2]^\top) = [1, x_1, x_2, x_1^2, x_2^2, x_1x_2]^\top`$. The primal with a degree-$`d`$ expansion of $`D`$ inputs has $`\approx D^d`$ variables; the dual still has $`N`$ (only the inner products change), but each inner product would cost $`O(D^d)`$ and the dual needs $`\approx N^2/2`$ of them. The trick cuts each one to $`O(D\log d)`$. For $`D = 2`$, $`d = 2`$, use the $`\sqrt2`$-scaled expansion $`\phi_2([x_1, x_2]^\top) = [1, \sqrt2x_1, \sqrt2x_2, x_1^2, x_2^2, \sqrt2x_1x_2]^\top`$. Then
```math
\phi_2(\mathbf{x})\cdot\phi_2(\mathbf{x}') = 1 + 2x_1x_1' + 2x_2x_2' + x_1^2x_1'^2 + x_2^2x_2'^2 + 2x_1x_1'x_2x_2' = (1 + \mathbf{x}\cdot\mathbf{x}')^2 .
```
In general $`\phi_d(\mathbf{x})\cdot\phi_d(\mathbf{x}') = (1 + \mathbf{x}\cdot\mathbf{x}')^d`$: all monomials up to degree $`d`$ (so arbitrary degree-$`d`$ separating surfaces) at cost $`O(D\log d)`$ instead of $`O(D^d)`$.

### 20.3 Building and using Mercer kernels
- **Why closure rules?** Defining a similarity directly on discrete objects is often easier than designing a feature map, but *proving* a kernel is Mercer is hard; the rules let you compose known Mercer kernels. If $`\kappa_1, \kappa_2`$ are Mercer and $`\alpha > 0`$, then $`\kappa_1 + \kappa_2`$, $`\alpha\kappa_1`$ and $`\kappa_1\cdot\kappa_2`$ are Mercer. For $`\mathcal{X} \subseteq\mathbb{R}^D`$ and PD $`\mathbf{B}`$, $`\kappa(\mathbf{x}, \mathbf{x}') = \mathbf{x}^\top\mathbf{B}\mathbf{x}'`$ is Mercer.
- **Why insist on Mercer?** With $`\mathbf{K}`$ the Gram matrix, $`\odot`$ the element-wise (Hadamard) product and $`\mathbf{1}`$ the all-ones vector, the dual objective is
```math
g(\boldsymbol\alpha) = \mathbf{1}^\top\boldsymbol\alpha - \tfrac12(\boldsymbol\alpha\odot\mathbf{y})^\top\mathbf{K}(\boldsymbol\alpha\odot\mathbf{y}).
```
If $`\mathbf{K}`$ is PSD this is **concave** ⇒ unique maximum value, solvable by convex programming (PD ⇒ unique maximiser). Otherwise KKT is no longer sufficient and you're stuck with non-linear programming (local optima, non-polynomial time).

### 20.4 Kernelised ridge regression (representer argument)
Ridge: $`\mathcal{L}_{\text{ridge}}(\mathbf{w}) = \sum_i(\mathbf{w}\cdot\mathbf{x}_i - y_i)^2 + \lambda\lVert\mathbf{w}\rVert_2^2`$. **Claim:** the optimum has the form $`\mathbf{w} = \sum_i\alpha_i\mathbf{x}_i`$. *Proof:* write $`\mathbf{w} = \mathbf{w}_s + \mathbf{w}_\perp`$ with $`\mathbf{w}_s \in \operatorname{span}\{\mathbf{x}_i\}`$ and $`\mathbf{w}_\perp`$ orthogonal to it. Then $`\mathbf{w}_\perp\cdot\mathbf{x}_i = 0`$ so the loss term is unaffected by $`\mathbf{w}_\perp`$, while $`\lVert\mathbf{w}\rVert_2^2 = \lVert\mathbf{w}_s\rVert_2^2 + \lVert\mathbf{w}_\perp\rVert_2^2`$. Setting $`\mathbf{w}_\perp = \mathbf{0}`$ can only decrease the objective. ∎ So one can solve for $`\boldsymbol\alpha`$ instead, and everything is expressed via $`\mathbf{x}_i\cdot\mathbf{x}_j`$ ⇒ replace with any Mercer kernel (Sheet 3). Kernel methods are **not** restricted to SVMs.

> [!TIP]
> **✅ Key takeaways**
> 1. SVM = maximise the minimum margin ⇒ minimise $`\tfrac12\lVert\mathbf{w}\rVert^2`$ with margin constraints; slack + $`C`$ handle non-separable data; equivalently **hinge loss + $`\ell_2`$ regulariser**.
> 2. The dual has $`N`$ box-constrained variables; **complementary slackness** ⇒ only support vectors ($`\alpha_i > 0`$) matter; $`\mathbf{w} = \sum_i\alpha_iy_i\mathbf{x}_i`$.
> 3. Inputs enter only via inner products ⇒ **kernel trick**: swap $`\mathbf{x}\cdot\mathbf{x}'`$ for $`\kappa(\mathbf{x},\mathbf{x}')`$ and get non-linear (even infinite-dimensional) features cheaply.
> 4. Use **Mercer** kernels so the dual stays concave. Build them with the closure rules; polynomial, RBF, string kernels are standard.
> 5. Any method whose solution lies in the span of the data (ridge, PCA, …) can be kernelised.

> [!WARNING]
> **⚠️ Common misconceptions & errors**
> - ❌ "The margin is $`\lVert\mathbf{w}\rVert`$." ✅ It's $`1/\lVert\mathbf{w}\rVert_2`$ (after normalising constraints to $`\ge 1`$); hence *minimise* $`\lVert\mathbf{w}\rVert^2`$.
> - ❌ Allowing $`\zeta_i < 0`$. ✅ Slack must be non-negative or correct points subsidise errors.
> - ❌ "Larger $`C`$ = more regularisation." ✅ Larger $`C`$ penalises violations more ⇒ **less** regularisation (narrower margin, fits training data harder).
> - ❌ "All training points are support vectors." ✅ Only those with $`\alpha_i > 0`$ (on or inside the margin).
> - ❌ "The kernel trick makes the problem smaller." ✅ The dual is still $`N^2`$; the saving is in computing features.
> - ❌ Using a non-Mercer kernel and expecting convexity.
> - ❌ "Hinge loss is differentiable." ✅ Kink at margin 1; use subgradients.

---

## 21. Multiclass strategies and measuring performance

### 21.1 From binary to $`K`$ classes
| Strategy | Classifiers trained | Prediction | Trade-offs |
|---|---|---|---|
| **One-vs-one (ovo)** | $`\binom K2`$, one per pair | **plurality** vote | more classifiers but each on a small 2-class subset (good if the learner scales badly, e.g. $`N^2`$ or $`N^3`$); more natural learning problems |
| **One-vs-rest (ovr)** | $`K`$ (class $`c`$ vs everything else) | ideally exactly one says "positive"; otherwise use the raw scores $`\mathbf{w}\cdot\mathbf{x}_{\text{new}}`$ and pick the largest | fewer classifiers but each sees all data; assumes "0 vs all other digits" is linearly separable, a strong assumption |

Both are "somewhat hacky" but work well. More principled: **error-correcting output codes** (train binary classifiers on pairs of disjoint class subsets; decode like an error-correcting code; Allwein et al. 2000). Logistic regression (softmax) and multiclass SVMs exist natively.

### 21.2 Measuring classification performance

> [!NOTE]
> **📖 Key definitions** (binary; rows = prediction, columns = actual)
> | | actual yes | actual no |
> |---|---|---|
> | **predict yes** | true positive (TP) | false positive (FP), *Type I* |
> | **predict no** | false negative (FN), *Type II* | true negative (TN) |
>
> - **True positive rate (TPR), sensitivity, recall:** $`\dfrac{\text{TP}}{\text{TP}+\text{FN}}`$ (fraction of actual positives found).
> - **False positive rate (FPR):** $`\dfrac{\text{FP}}{\text{FP}+\text{TN}}`$ (fraction of actual negatives wrongly flagged).
> - **Precision:** $`\dfrac{\text{TP}}{\text{TP}+\text{FP}}`$ (fraction of predicted positives that are right).
> - **ROC curve:** TPR vs FPR as the decision threshold varies, from $`(0,0)`$ (predict all negative) to $`(1,1)`$ (predict all positive). **AUC** (area under it) summarises: 1 best, 0 worst.
> - **Precision–recall curve:** precision vs recall; preferred when positives are rare (document retrieval).
> - **Confusion matrix** ($`K`$ classes): $`N_{ij}`$ = number of class-$`j`$ items predicted as class $`i`$ (rows = prediction, columns = actual, as in the notes' table and formula; their prose says the opposite, so **state your convention** in an exam); a good classifier is diagonal-heavy.

**Why measure differently from training?** Training minimises a surrogate (hinge, NLL); what we care about is **misclassification rate**, and often different mistakes cost differently (abuse detection, medical diagnosis). **Moving the threshold:** logistic regression predicts positive if $`\sigma(\mathbf{w}\cdot\mathbf{x}) \ge 0.5`$; raise it to 0.9 to cut false positives. Generative models threshold the ratio $`p(y=1\mid\mathbf{x})/p(y=0\mid\mathbf{x})`$ at 1 by default. SVMs: re-weight the objective to penalise one error type more. Sweeping the threshold traces the ROC curve. In both the ROC and precision–recall plots (Fig. 6) the **higher curve is the better classifier**; pick the operating point that matches your cost trade-off.

> [!TIP]
> **✅ Key takeaways**
> 1. ovo: many small problems, vote. ovr: $`K`$ problems, use scores to break ties.
> 2. Know the four cells of the confusion matrix and the three rates; recall = TPR.
> 3. Thresholds are tunable; the ROC/PR curve shows the trade-off; AUC compares classifiers independent of threshold.

> [!WARNING]
> **⚠️ Common misconceptions & errors**
> - ❌ Precision vs recall swapped. ✅ Precision = TP / **predicted** positives; recall = TP / **actual** positives.
> - ❌ "Accuracy is enough." ✅ With 99% negatives, predicting "negative" always gets 99% accuracy; use precision–recall.
> - ❌ "Type I = false negative." ✅ Type I = false **positive**.
