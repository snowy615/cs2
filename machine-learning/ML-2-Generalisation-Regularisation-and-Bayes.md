# Machine Learning · Part 2: Generalisation, Regularisation and the Bayesian View

*Lectures 5, 6, 7. [← Part 1](ML-1-Linear-Regression-and-Maximum-Likelihood.md) · [Overview](ML-Summary.md) · [Part 3 →](ML-3-Optimisation.md) · [Formula sheet](ML-7-Formula-Sheet.md)*

---

## Contents
5. [Basis expansion: making linear models non-linear](#5-basis-expansion-making-linear-models-non-linear)
6. [Overfitting, bias–variance, learning curves](#6-overfitting-biasvariance-learning-curves)
7. [Regularisation: ridge and lasso](#7-regularisation-ridge-and-lasso)
8. [Model selection: validation, cross-validation, feature selection](#8-model-selection-validation-cross-validation-feature-selection)
9. [The Bayesian approach and MAP](#9-the-bayesian-approach-and-map)

---

## 5. Basis expansion: making linear models non-linear

> [!NOTE]
> **📖 Key definitions**
> - **Basis (feature) expansion:** replace the input $`\mathbf{x}`$ by a fixed vector of features $`\phi(\mathbf{x})`$ and fit a **linear** model $`\mathbf{w}\cdot\phi(\mathbf{x})`$. The model is linear in $`\mathbf{w}`$ but non-linear in $`\mathbf{x}`$.
> - **Polynomial basis** (1-D, degree $`d`$): $`\phi(x) = [1, x, x^2, \dots, x^d]^\top`$. In 2-D, degree 2: $`\phi(\mathbf{x}) = [1, x_1, x_2, x_1^2, x_2^2, x_1x_2]^\top`$. Degree $`d`$ in $`D`$ dimensions needs roughly $`D^d`$ features.
> - **Kernel (for now):** any function that takes two points $`\mathbf{x}, \mathbf{x}' \in \mathbb{R}^D`$ and returns a non-negative real number (a formal definition comes with SVMs in Part 4).
> - **RBF (radial basis function) kernel** with width parameter $`\gamma`$: $`\kappa(\mathbf{x}', \mathbf{x}) = \exp\big(-\gamma\lVert\mathbf{x} - \mathbf{x}'\rVert^2\big)`$. Depends only on the **distance** between the points.
> - **Kernel basis expansion** with centres $`\boldsymbol\mu_1, \dots, \boldsymbol\mu_M`$: $`\phi(\mathbf{x}) = [1, \kappa(\boldsymbol\mu_1, \mathbf{x}), \dots, \kappa(\boldsymbol\mu_M, \mathbf{x})]^\top`$, giving $`\widehat{y}(\mathbf{x}) = w_0 + \sum_{i=1}^M w_i\,\kappa(\boldsymbol\mu_i, \mathbf{x})`$: a weighted sum of bumps.
> - **Universal approximators:** polynomials (as degree grows) and RBF expansions (as the number of centres grows) can approximate **any continuous function**. Powerful, maybe *too* powerful.

**Why it works:** mapping 1-D $`x`$ to 3-D $`[1, x, x^2]`$ turns a quadratic relationship into a *linear* one in the new space. Same least-squares machinery, same closed form.

**Choices:**
- **Centres:** choose the **data points themselves** (standard; a centre far from all data gives a nearly-constant, useless feature).
- **Width $`\gamma`$:** small $`\gamma`$ ⇒ wide kernel ⇒ every point looks the same ⇒ **underfitting**. Large $`\gamma`$ ⇒ narrow spikes ⇒ fits training points, wiggles elsewhere ⇒ **overfitting**. In high dimensions it's hard to find the middle ground (curse of dimensionality).
- **Curse of dimensionality (Remark 2):** the ratio of the volume of the unit-diameter ball to the unit cube is $`\pi/4`$ in 2-D but **exponentially small** in high $`D`$: sample uniformly from a 100-D box and almost nothing lands in the inscribed ball. Intuitions about "near" and "far" break down.

---

## 6. Overfitting, bias–variance, learning curves

> [!NOTE]
> **📖 Key definitions**
> - **Overfitting:** the model fits the *training* data very well (including its noise) but performs badly on *unseen* data. In statistical-learning terms: **high variance**.
> - **Underfitting:** the model is too simple to capture the true relationship (fitting a line to a quadratic). **High bias**.
> - **Variance** (of a learning method): how much the fitted model changes when trained on **different random samples** of data of the same size.
> - **Bias:** systematic error: the model's predictions are inherently off (too high/too low) across the input space, however much data you have.
> - **Bias–variance trade-off:** choosing model complexity to avoid both. (Sheet 2: bias and variance of the least-squares estimator.)
> - **Training set / test set:** data used to fit the model / data held back to measure performance on unseen inputs.
> - **Learning curve:** training error and test error plotted against the **amount of training data**.

**Pictures to remember (Fig. 7 of the notes):** fit high-degree polynomials to 5 noisy samples from the same quadratic. Each fit is wild, but their **average** is close to the truth: high variance, low bias. Fit lines instead: all similar, all wrong in the same way: low variance, high bias.

**Reading learning curves (Fig. 8):**
| Pattern | Diagnosis | Fix |
|---|---|---|
| Train and test errors converge quickly and stay level | **no overfitting**: underfitting (high bias) iff the plateau error is too high; otherwise the model is fitting correctly | if underfitting: more complex model (basis expansion); more data won't help |
| Train error low, test error high, gap closes only slowly as data grows | high variance / overfitting | simpler model, regularisation, or more data |

**Two more lessons from the polynomial pictures (Fig. 1):** with $`N`$ points, a polynomial of degree $`N-1`$ **interpolates them exactly**: training error 0, pure noise-fitting. And given lots of data (Fig. 1d) the degree 2, 7 and 14 polynomials all track the true quadratic (overfitting disappears), yet the straight line still underfits: **more data cures variance, not bias**.

### 6.1 Why does overfitting happen? (the coin-flip thought experiment)
Flip two independent fair coins 100 times; count matches. Expect ≈50, but you'll see something like 43 or 57. If you see 43 you could "conclude" that coin 1 predicts coin 2 with 57% accuracy. Absurd, but that is exactly what a flexible model does with a **finite, noisy** dataset: it finds spurious patterns.

**Toy experiment (Fig. 9):** $`N = 100`$ points in $`D = 100`$ dimensions, $`y = x_1 + \epsilon`$ (only feature 1 matters). Fit least squares using the first $`k`$ features, $`k = 1, \dots, 100`$. Training error falls to 0; **test error rises**; the weights on irrelevant features grow until, at $`k = 100`$, the model has fitted the noise perfectly using junk features. Von Neumann: "with four parameters I can fit an elephant". For scale: inputs in $`\mathbb{R}^{100}`$ with degree-10 expansion ⇒ $`\approx 10^{20}`$ parameters; no dataset or computer could fit that.

> [!TIP]
> **✅ Key takeaways**
> 1. Basis expansion = non-linear features + linear model. Closed form survives; the **number of parameters explodes** ($`\approx D^d`$).
> 2. Model complexity is a dial: too low ⇒ bias (underfit), too high ⇒ variance (overfit).
> 3. **Diagnose with learning curves**, then act: underfitting ⇒ richer model; overfitting ⇒ simplify, regularise, or get more data (more data closes the train–test gap).
> 4. Overfitting is caused by finite data + noise; flexible models fit the noise.

> [!WARNING]
> **⚠️ Common misconceptions & errors**
> - ❌ "Low training error means a good model." ✅ Training error is optimistically biased; only held-out error counts.
> - ❌ "More data always helps." ✅ Not when underfitting (curves already converged).
> - ❌ "Wide kernels (small $`\gamma`$) overfit." ✅ Wide = smooth = **underfit**; narrow = overfit.
> - ❌ "A universal approximator is always the best choice." ✅ It needs enough data and compute; it may be too powerful.

---

## 7. Regularisation: ridge and lasso

> [!NOTE]
> **📖 Key definitions**
> - **Regularisation:** add a **penalty on the size of the weights** to the loss, so the optimiser must trade fit against complexity. We don't know which features are irrelevant, so penalise all of them.
> - **Ridge regression** ($`\ell_2`$ penalty), $`\lambda > 0`$:
> ```math
> \mathcal{L}_{\text{ridge}}(\mathbf{w}) = (\mathbf{X}\mathbf{w} - \mathbf{y})^\top(\mathbf{X}\mathbf{w}-\mathbf{y}) + \lambda\sum_{i=1}^D w_i^2 .
> ```
> - **Lasso** (Least Absolute Shrinkage and Selection Operator, $`\ell_1`$ penalty):
> ```math
> \mathcal{L}_{\text{lasso}}(\mathbf{w}) = (\mathbf{X}\mathbf{w} - \mathbf{y})^\top(\mathbf{X}\mathbf{w}-\mathbf{y}) + \lambda\sum_{i=1}^D \lvert w_i\rvert .
> ```
> - **Hyperparameter:** a setting chosen *before* training that the training algorithm doesn't learn ($`\lambda`$, polynomial degree, kernel width $`\gamma`$).
> - **Standardising inputs:** transform every feature to mean 0, variance 1 (on the training data; apply the **same** transformation to test data). **Centring outputs:** subtract the mean of $`y`$.

### 7.1 Why not penalise $`w_0`$, and why standardise?
- The penalty excludes $`w_0`$: a **translation of the output** (°C vs K adds 273) is not extra model complexity.
- Scale matters: with $`\widehat{y} = w_0 + w_1x`$ in °C, switching to °F gives $`\widehat{y} = (w_0 - \tfrac{160}{9}w_1) + \tfrac59 w_1 x'`$, so the penalty $`w_1^2`$ becomes $`25w_1^2/81`$, less than a third. The penalty would depend on your units. **So standardise inputs.** If you also centre $`y`$, ridge gives $`w_0 = 0`$ automatically (Sheet 2), and the objective becomes simply $`(\mathbf{X}\mathbf{w}-\mathbf{y})^\top(\mathbf{X}\mathbf{w}-\mathbf{y}) + \lambda\mathbf{w}^\top\mathbf{w}`$.

### 7.2 Ridge: closed form
Expand and differentiate (same two rules as least squares):
```math
\mathcal{L}_{\text{ridge}} = \mathbf{w}^\top\mathbf{X}^\top\mathbf{X}\mathbf{w} - 2\mathbf{y}^\top\mathbf{X}\mathbf{w} + \mathbf{y}^\top\mathbf{y} + \lambda\mathbf{w}^\top\mathbf{w},
\qquad
\nabla_{\mathbf{w}}\mathcal{L}_{\text{ridge}} = 2(\mathbf{X}^\top\mathbf{X})\mathbf{w} - 2\mathbf{X}^\top\mathbf{y} + 2\lambda\mathbf{w} = 2\big((\mathbf{X}^\top\mathbf{X} + \lambda\mathbf{I}_D)\mathbf{w} - \mathbf{X}^\top\mathbf{y}\big).
```
> ```math
> \mathbf{w}_{\text{ridge}} = (\mathbf{X}^\top\mathbf{X} + \lambda\mathbf{I}_D)^{-1}\mathbf{X}^\top\mathbf{y}.
> ```
**Always invertible for $`\lambda > 0`$** (exercise: $`\mathbf{X}^\top\mathbf{X}`$ is positive semi-definite, so adding $`\lambda\mathbf{I}`$ makes every eigenvalue $`\ge \lambda > 0`$). This fixes the $`N < D`$ problem of least squares.

**Reading $`\lambda`$:** $`\lambda \to 0`$ recovers least squares; $`\lambda \to \infty`$ forces $`\mathbf{w} \to \mathbf{0}`$. Pick something in between by validation (§8).

**Constrained form** (equivalent, the penalty form is its **Lagrangian**):
```math
\text{minimise } (\mathbf{X}\mathbf{w}-\mathbf{y})^\top(\mathbf{X}\mathbf{w}-\mathbf{y}) \quad\text{subject to } \mathbf{w}^\top\mathbf{w} \le R .
```
For $`R \ge \mathbf{w}_{\text{LS}}^\top\mathbf{w}_{\text{LS}}`$ the constraint is inactive and you get least squares; for smaller $`R`$ the solution has $`\mathbf{w}^\top\mathbf{w} = R`$ and the loss contours are **tangent to the circle** $`\mathbf{w}^\top\mathbf{w} = R`$ (Fig. 1a).

### 7.3 Lasso: sparsity
No closed form (the $`\lvert w_i\rvert`$ terms aren't differentiable at 0); use subgradient methods (Part 3). Constrained form: minimise RSS subject to $`\sum_i\lvert w_i\rvert \le R`$. The constraint set is a **diamond** (in 2-D) with **corners on the axes**. Loss contours typically touch the diamond **at a corner**, where some $`w_i`$ are **exactly zero**. Hence lasso performs **variable selection**: it switches irrelevant features off. Ridge's circle has no corners, so ridge shrinks weights but rarely zeroes them. **Regularisation path** (Figs. 1b/2b of the notes, diabetes dataset): plot each $`w_i`$ against $`-\log\lambda`$. Ridge curves bend smoothly towards 0 and never reach it; lasso curves sit at **exactly 0** until $`\lambda`$ drops below a feature-specific threshold, then switch on one by one.

**Toy problem revisited (Fig. 3):** with 99 irrelevant features, least squares and ridge both overfit as features are added; **lasso stays good** even with all 100, because it keeps the irrelevant weights at zero. (Ridge still spreads small weights over the junk features.)

> [!TIP]
> **✅ Key takeaways**
> 1. Regularisation trades **fit vs complexity** through $`\lambda`$; the optimum is found by validation.
> 2. Ridge: closed form with $`\lambda\mathbf{I}`$ added; always invertible; shrinks weights smoothly.
> 3. Lasso: $`\ell_1`$ penalty, corners ⇒ **sparse** solutions ⇒ feature selection; no closed form.
> 4. **Standardise inputs and centre outputs** before regularising; don't penalise the intercept.
> 5. Geometric picture: loss contours tangent to a circle (ridge) or diamond (lasso).

> [!WARNING]
> **⚠️ Common misconceptions & errors**
> - ❌ Penalising $`w_0`$. ✅ Excluded; a shift in output isn't complexity.
> - ❌ Standardising test data with *its own* mean/variance. ✅ Use the **training** statistics.
> - ❌ "Ridge does feature selection." ✅ Ridge shrinks; **lasso** zeroes.
> - ❌ "Ridge needs $`\mathbf{X}^\top\mathbf{X}`$ invertible." ✅ $`\mathbf{X}^\top\mathbf{X} + \lambda\mathbf{I}`$ is always invertible for $`\lambda > 0`$.
> - ❌ Choosing $`\lambda`$ by minimising **training** error (always picks $`\lambda = 0`$). ✅ Use a validation set.

---

## 8. Model selection: validation, cross-validation, feature selection

> [!NOTE]
> **📖 Key definitions**
> - **Validation set:** part of the data, not used to fit $`\mathbf{w}`$, used to **choose hyperparameters**.
> - **Test set:** untouched until the very end; used **once** to report performance. (In the real world, the test "presents itself".)
> - **Grid search:** try every combination of hyperparameter values on a grid (often log-scaled, e.g. $`\lambda \in \{0.01, 0.1, 1, 10, 100\}`$), pick the best validation error. Exponential in the number of hyperparameters; random search or Bayesian optimisation are alternatives (not covered).
> - **$`K`$-fold cross-validation:** split the training data into $`K`$ folds; train on $`K-1`$, validate on the remaining one; repeat for each fold; **average** the validation errors. Common: $`K = 5`$ or $`10`$.
> - **Leave-one-out CV (LOOCV):** $`K = N`$. Expensive (train $`N`$ times) unless the model allows cheap removal of one point (least squares does).
> - **Forward search (feature selection):** a greedy *wrapper* around any learner. Start with $`F = \emptyset`$; at each step, for every unused feature $`i`$ train on $`F\cup\{i\}`$ alone and measure validation error; add the best $`i`$; continue until all features are in (or $`\lvert F\rvert`$ hits a cap); **return the best subset seen at any stage**. $`O(n^2)`$ training runs instead of $`2^n`$ subsets.
> - **Filter feature selection:** score each feature individually (e.g. by **mutual information** with $`y`$) and keep the top $`k`$.
> - **Mutual information:**
> ```math
> I(X, Y) = \sum_x\sum_y p(X=x, Y=y)\,\log\frac{p(X=x, Y=y)}{p(X=x)\,p(Y=y)} \quad (= 0 \text{ iff independent}).
> ```
> (Equivalently $`\operatorname{KL}\big(p(x,y)\,\|\,p(x)p(y)\big)`$; probabilities estimated from the training set.)

**Procedure:** (1) train on the training set for each hyperparameter value; (2) evaluate on validation; (3) pick the best; (4) **retrain on training + validation** with the chosen hyperparameter (more data = better model); (5) report on the test set once.

**The validation curve is U-shaped** (Fig. 4b): large error at small $`\lambda`$ (overfitting: training error ≈ 0, validation high) and at large $`\lambda`$ (underfitting: both high); pick the bottom. Made-up table: $`\lambda = 10`$ had training error 10% and validation 8%, better than $`\lambda = 1`$'s 2%/12%.

**Why cross-validate?** When data are scarce, a single small validation set gives a **noisy** estimate; averaging over folds reduces the variance.

**Leakage warning:** every time you look at test performance and then change the model, you leak information from the test set and can overfit to it. Kaggle teams that top the public leaderboard often fall on the private one for exactly this reason.

**Different kinds of error cost differently** (tumour: false negative worse than false positive), so model selection may weight error types (Part 4, §16). In **unsupervised** learning there is no ground truth at all, so model selection needs different criteria (Part 6: the elbow method).

> [!TIP]
> **✅ Key takeaways**
> 1. Three-way split: **train** (fit $`\mathbf{w}`$) / **validation** (choose hyperparameters) / **test** (report once).
> 2. Validation error vs hyperparameter is U-shaped; choose the minimum, then retrain on all non-test data.
> 3. Cross-validation when data are scarce; LOOCV is the extreme.
> 4. Feature selection: forward search (wrapper, greedy) or filter by mutual information.

> [!WARNING]
> **⚠️ Common misconceptions & errors**
> - ❌ Tuning hyperparameters on the test set. ✅ That's what validation is for; the test set is sacred.
> - ❌ Forgetting to retrain on all available data after choosing $`\lambda`$.
> - ❌ "Mutual information = correlation." ✅ MI captures any dependence (non-linear too); zero MI ⇔ independence.
> - ❌ Trying all $`2^n`$ feature subsets. ✅ Infeasible; use greedy or filter methods.

---

## 9. The Bayesian approach and MAP

> [!NOTE]
> **📖 Key definitions**
> - **Frequentist:** probability = long-run frequency. Parameters $`\mathbf{w}`$ are fixed but unknown; data are random; design methods around the **likelihood** $`p(\mathcal{D}\mid\mathbf{w})`$.
> - **Bayesian:** probability = **degree of belief**. Data are fixed; parameters are uncertain and described by a distribution. Goal: the **posterior** $`p(\mathbf{w}\mid\mathcal{D})`$.
> - **Prior** $`p(\mathbf{w})`$: belief before seeing data. **Bayes' rule:**
> ```math
> p(\mathbf{w}\mid\mathcal{D}) = \frac{p(\mathcal{D}\mid\mathbf{w})\,p(\mathbf{w})}{p(\mathcal{D})}, \qquad p(\mathcal{D}) = \int p(\mathcal{D}\mid\mathbf{w})\,p(\mathbf{w})\,d\mathbf{w}.
> ```
> "posterior ∝ likelihood × prior".
> - **Aleatoric uncertainty:** randomness in the data (noise in $`y`$); **not reducible** with more data. **Epistemic uncertainty:** uncertainty about the parameters/model; **reduced** by more data. Both camps model aleatoric; only Bayesians model epistemic (via the posterior).
> - **Marginalisation:** averaging predictions over the posterior, $`p(y\mid\mathbf{x},\mathcal{D}) = \int p(y\mid\mathbf{x},\mathbf{w})\,p(\mathbf{w}\mid\mathcal{D})\,d\mathbf{w}`$: "the heart of any true Bayesian method".
> - **Conjugate prior:** one for which the posterior has the same functional form (Gaussian prior + Gaussian likelihood ⇒ Gaussian posterior), giving closed forms.
> - **MAP (maximum a posteriori) estimate:** the mode of the posterior,
> ```math
> \mathbf{w}_{\text{MAP}} \in \arg\max_{\mathbf{w}} p(\mathcal{D}\mid\mathbf{w})\,p(\mathbf{w}) = \arg\min_{\mathbf{w}}\big(-\log p(\mathcal{D}\mid\mathbf{w}) - \log p(\mathbf{w})\big).
> ```
> No need to compute $`p(\mathcal{D})`$. A point estimate: not fully Bayesian.
> - **Improper prior:** one that doesn't integrate to 1 (e.g. uniform over all of $`\mathbb{R}^D`$).

### 9.1 Convenience-store example (Gaussian mean with known variance)
Tomato shelf-life at shop $`s`$: $`p(t\mid s) = \mathcal{N}(\mu_s, \sigma^2)`$ with $`\sigma^2`$ known. Prior $`p(\mu_s) = \mathcal{N}(\mu_{0,s}, \sigma_{0,s}^2)`$: $`\mu_{0,s}`$ is your initial guess, $`\sigma^2_{0,s}`$ how unsure you are (narrow = confident). After $`N`$ observations $`t_1, \dots, t_N`$ the posterior is Gaussian (Murphy Ch. 7):
```math
p(\mu_s\mid\mathcal{D}) = \mathcal{N}(\mu_{N,s}, \sigma^2_{N,s}), \qquad \mu_{N,s} = \frac{1}{\frac{\sigma^2}{\sigma^2_{0,s}} + N}\Big(\frac{\sigma^2}{\sigma^2_{0,s}}\mu_{0,s} + \sum_{i=1}^N t_i\Big), \qquad \sigma^2_{N,s} = \Big(\frac{1}{\sigma^2_{0,s}} + \frac{N}{\sigma^2}\Big)^{-1}.
```
**Read it:** the posterior mean is a **weighted average of the prior mean and the data**, the prior counting as $`\sigma^2/\sigma^2_{0,s}`$ pseudo-observations. $`N = 0`$ gives the prior back; as $`N \to \infty`$, $`\mu_{N,s} \to \frac1N\sum t_i`$ (the MLE) and $`\sigma^2_{N,s} \to 0`$: the posterior **concentrates on the MLE**. (General version: **Bernstein–von Mises**: with a prior that is non-zero everywhere, the posterior → $`\mathcal{N}(\mathbf{w}_{\text{mle}}, N^{-1}I_F^{-1})`$, $`I_F`$ = Fisher information.)

**Bayesian prediction** (marginalise over $`\mu_s`$):
```math
p(t\mid s, \mathcal{D}) = \int p(t\mid s, \mu_s, \sigma^2)\,p(\mu_s\mid\mathcal{D})\,d\mu_s = \mathcal{N}\big(\mu_{N,s},\ \underbrace{\sigma^2}_{\text{aleatoric}} + \underbrace{\sigma^2_{N,s}}_{\text{epistemic}}\big).
```
Decision: choose the shop with the largest predictive mean, $`s_{\text{Bayesian}} = \arg\max_{s'}\mu_{N,s'}`$. Simulation: true means $`\mu_A = 6`$, $`\mu_B = 4.5`$, $`\mu_C = 5`$ (Abdul's is best), equal optimistic priors $`\mathcal{N}(10, 2)`$. After 3 choices every shop has been visited once (posteriors too wide to prefer any: **exploration**); after 10 choices shop A has been visited 8 times, $`\mu_{10,A} \approx 6`$ with a sharp posterior, while B and C (once each) keep wide posteriors.

### 9.2 Bayesian linear regression
Likelihood $`p(\mathbf{y}\mid\mathbf{X},\mathbf{w}) = (2\pi\sigma^2)^{-N/2}\exp\big(-\frac{(\mathbf{y}-\mathbf{X}\mathbf{w})^\top(\mathbf{y}-\mathbf{X}\mathbf{w})}{2\sigma^2}\big)`$ with $`\sigma^2`$ fixed; prior $`p(\mathbf{w}) = \mathcal{N}(\mathbf{0}, \boldsymbol\Lambda)`$. The posterior is Gaussian (Murphy 7.6):
```math
p(\mathbf{w}\mid\mathcal{D}) = \mathcal{N}(\mathbf{w}_N, \boldsymbol\Sigma_N), \qquad \boldsymbol\Sigma_N = \Big(\boldsymbol\Lambda^{-1} + \frac{1}{\sigma^2}\mathbf{X}^\top\mathbf{X}\Big)^{-1}, \qquad \mathbf{w}_N = \frac{1}{\sigma^2}\boldsymbol\Sigma_N\mathbf{X}^\top\mathbf{y}.
```
Predictive distribution for a new input:
```math
p(y\mid\mathcal{D}, \mathbf{x}_{\text{new}}, \sigma^2) = \mathcal{N}\big(\mathbf{w}_N^\top\mathbf{x}_{\text{new}},\ \sigma^2_N(\mathbf{x}_{\text{new}})\big), \qquad \sigma^2_N(\mathbf{x}_{\text{new}}) = \sigma^2 + \mathbf{x}_{\text{new}}^\top\boldsymbol\Sigma_N\mathbf{x}_{\text{new}}.
```
The variance has an **aleatoric** part $`\sigma^2`$ (constant) and an **epistemic** part $`\mathbf{x}^\top\boldsymbol\Sigma_N\mathbf{x}`$ that is **small near observed data and large far from it** (Fig. 5: error bars fan out where data are scarce). Sampling several $`\mathbf{w}`$ from the posterior gives a spread of plausible curves.

### 9.3 MAP for linear regression: regularisers are priors
With a general prior $`p(\mathbf{w}) \propto \exp(-\mathcal{R}(\mathbf{w}))`$ and Gaussian likelihood:
```math
p(\mathbf{w}\mid\mathbf{X},\mathbf{y}) \propto \exp\Big(-\frac{(\mathbf{y}-\mathbf{X}\mathbf{w})^\top(\mathbf{y}-\mathbf{X}\mathbf{w})}{2\sigma^2} - \mathcal{R}(\mathbf{w})\Big)
\ \Longrightarrow\ \mathcal{L}_{\text{MAP}}(\mathbf{w}) = \underbrace{(\mathbf{y}-\mathbf{X}\mathbf{w})^\top(\mathbf{y}-\mathbf{X}\mathbf{w})}_{\mathcal{L}_{\text{LS}}} + \underbrace{2\sigma^2\mathcal{R}(\mathbf{w})}_{\text{regularisation}}
```
(after scaling by $`2\sigma^2`$, which doesn't change the minimiser). So every regularised estimator is a MAP estimate under some prior:

| Regulariser $`\mathcal{R}(\mathbf{w})`$ | Prior $`p(\mathbf{w})`$ | Estimator |
|---|---|---|
| $`\frac{\lambda}{2\sigma^2}\mathbf{w}^\top\mathbf{w}`$ | $`\mathcal{N}\big(\mathbf{0}, \operatorname{diag}(\sigma^2/\lambda)\big)`$, Gaussian | **Ridge** |
| $`\frac{\lambda}{2\sigma^2}\sum_i\lvert w_i\rvert`$ | $`\prod_i \operatorname{Lap}(0, 2\sigma^2/\lambda)`$, Laplace | **Lasso** |
| $`\lim_{\lambda\to0}\frac{\lambda}{2\sigma^2}\mathbf{w}^\top\mathbf{w}`$ | Gaussian with infinite variance = **improper uniform** prior | **MLE / least squares** |

**Interpretation:** both priors are centred at $`\mathbf{w} = \mathbf{0}`$ ("weights are probably small"), with $`\lambda`$ controlling confidence: larger $`\lambda`$ = tighter prior = more shrinkage. For equal variance the Laplace is more peaked at 0 and heavier-tailed than the Gaussian (Part 1, Fig. 7 of lecture 4). What really produces **sparsity** is the **cusp at 0**: the Laplace penalty $`-\log p(w_i) = \lvert w_i\rvert/b + \text{const}`$ has slope $`\pm1/b`$ at $`w_i = 0`$, so small weights get pushed *exactly* to 0, whereas the Gaussian penalty $`w_i^2`$ has zero slope at 0 and only shrinks.

### 9.4 Choosing a prior
Priors matter most with **little data**; with lots of data the likelihood dominates. Use background knowledge if you have it; otherwise a non-informative (uniform-ish) prior. A wrong-but-confident (peaked) prior needs more data to overturn than a vague one. In practice priors are often chosen for mathematical convenience (conjugacy), which is a standard criticism of the approach.

> [!TIP]
> **✅ Key takeaways**
> 1. Bayes: posterior ∝ likelihood × prior. The posterior **concentrates on the MLE** as data grow; the prior matters when data are scarce.
> 2. Two kinds of uncertainty: aleatoric (noise, irreducible) and epistemic (about parameters, reducible). Bayesian predictive variance = sum of both, and epistemic variance grows away from the data.
> 3. Full Bayes **marginalises**; MAP takes the **mode** (cheap, avoids the intractable $`p(\mathcal{D})`$, but no uncertainty).
> 4. **MAP with Gaussian prior = ridge; Laplace prior = lasso; flat prior = MLE.** Regularisation *is* a prior belief that weights are small.
> 5. Bayesian linear regression's posterior mean is ridge in disguise: substituting $`\boldsymbol\Sigma_N`$ gives $`\mathbf{w}_N = (\mathbf{X}^\top\mathbf{X} + \sigma^2\boldsymbol\Lambda^{-1})^{-1}\mathbf{X}^\top\mathbf{y}`$, so $`\sigma^2\boldsymbol\Lambda^{-1}`$ plays the role of $`\lambda\mathbf{I}`$; with the ridge prior $`\boldsymbol\Lambda = (\sigma^2/\lambda)\mathbf{I}`$ it is *exactly* $`\mathbf{w}_{\text{ridge}}`$.

> [!WARNING]
> **⚠️ Common misconceptions & errors**
> - ❌ "The posterior is the likelihood." ✅ Posterior = likelihood × prior / evidence.
> - ❌ "MAP is fully Bayesian." ✅ It's a point estimate; it ignores posterior spread.
> - ❌ "More data reduces all uncertainty." ✅ Only **epistemic**; aleatoric noise stays.
> - ❌ Gaussian prior ↔ lasso / Laplace prior ↔ ridge (swapped). ✅ Gaussian ↔ ridge ($`\ell_2`$), Laplace ↔ lasso ($`\ell_1`$).
> - ❌ "Larger prior variance = stronger regularisation." ✅ Larger variance = **weaker** (λ smaller); variance $`\sigma^2/\lambda`$.
