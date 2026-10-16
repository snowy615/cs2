# Machine Learning · Part 1: Linear Regression and Maximum Likelihood

*Lectures 1, 3, 4 (Varun Kanade, MT 2023 notes). [← Overview](ML-Summary.md) · [Part 2 →](ML-2-Generalisation-Regularisation-and-Bayes.md) · [Formula sheet](ML-7-Formula-Sheet.md)*

---

## Contents
1. [What is machine learning?](#1-what-is-machine-learning)
2. [Linear regression and least squares](#2-linear-regression-and-least-squares)
3. [Probability review](#3-probability-review)
4. [Maximum likelihood and the probabilistic view of regression](#4-maximum-likelihood-and-the-probabilistic-view-of-regression)

---

## 1. What is machine learning?

> [!NOTE]
> **📖 Key definitions**
> - **Machine learning (Murphy):** methods that *automatically detect patterns in data* and use them to *predict future data* or make decisions.
> - **Learning (Mitchell 1997):** a program **learns** from experience $`E`$ with respect to tasks $`T`$ and performance measure $`P`$ if its performance on $`T`$, measured by $`P`$, improves with $`E`$. *Face detection:* $`E`$ = images with boxes drawn round faces, $`T`$ = draw boxes on new images, $`P`$ = fraction of faces found.
> - **Supervised learning:** training data are **input–output pairs** $`(\mathbf{x}_i, y_i)`$; learn a map from input to output.
>   - **Regression:** output $`y`$ is a **real number** (house price).
>   - **Classification:** output is a **category** (which object is in the image). **Binary** = 2 classes; **multi-class** = more; **multi-label** = several labels per input.
> - **Unsupervised learning:** only inputs $`\mathbf{x}_1, \dots, \mathbf{x}_N`$, no outputs; find structure. **Clustering** (group similar points) and **dimensionality reduction** are the canonical examples (Novembre et al. 2008: European genomes cluster by geography).
> - **Other paradigms:** *active learning* (algorithm chooses which points to get labelled), *semi-supervised* (lots of unlabelled + few labelled), *collaborative filtering* (fill in a sparse user×movie rating matrix), *reinforcement learning* (sequential actions, delayed rewards; AlphaGo).

**Two historical roots you'll meet again:**
- **Fisher (1936), iris dataset:** classify flowers using a *linear function* of 4 measurements → Fisher Linear Discriminant Analysis (see LDA, Part 4). A linear combination of several measurements separates classes better than any single one.
- **Rosenblatt's perceptron:** output $`\mathrm{sign}(w_0 + w_1x_1 + \dots + w_Dx_D)`$; weighted sum, threshold. An algorithm adjusts weights when a prediction is wrong, and under certain assumptions (e.g. linearly separable data, cf. Part 4) it **provably converges** to a correct classifier. (Turing, 1950, had already argued that learning must be part of any intelligent system: simulate the child's mind, then educate it.) A *deep neural network* is many such units stacked in layers, with smooth activations instead of a hard threshold (Part 5).

**Practical concern:** a learning algorithm can only do useful things if given **useful data and features**. Classic failure: dogs photographed outside, cats inside ⇒ a classifier that just counts green pixels. Spam data from the 1990s is useless today.

---

## 2. Linear regression and least squares

### 2.1 The model

> [!NOTE]
> **📖 Key definitions**
> - **Data:** $`N`$ observations $`\langle(\mathbf{x}_i, y_i)\rangle_{i=1}^N`$, inputs $`\mathbf{x}_i \in \mathbb{R}^D`$ (column vectors), outputs $`y_i \in \mathbb{R}`$.
> - **Linear model:**
> ```math
> y = w_0 + w_1x_1 + \dots + w_Dx_D + \epsilon
> ```
> $`w_0`$ is the **bias/intercept** (constant term), $`\epsilon`$ is **noise** (the data won't lie exactly on a line).
> - **Bias trick:** add a feature $`x_0 = 1`$ to every input, so $`\mathbf{x} = (x_0, x_1, \dots, x_D) \in \mathbb{R}^{D+1}`$ and the model is just $`y = \mathbf{w}\cdot\mathbf{x} + \epsilon`$. (The notes often write $`D`$ for $`D+1`$.)
> - **One-hot encoding:** a categorical feature with $`K`$ values (day of the week) becomes $`K`$ binary features, exactly one of which is 1. Encoding days as $`0,\dots,6`$ would impose a fake ordering.
> - **Prediction:** $`\widehat{y}_{\text{new}} = \mathbf{w}\cdot\mathbf{x}_{\text{new}}`$.
> - **Residual:** the vertical gap between prediction and observation, $`\lvert\widehat{y}_i - y_i\rvert`$ in the notes (the sign is irrelevant for squared or absolute loss; elsewhere the signed $`\widehat{y}_i - y_i`$ is standard).
> - **Loss / cost / objective function:** a number measuring how badly the model fits; we **minimise** it.

Toy example: predict commute time from distance and weekday/weekend: $`y = w_0 + w_1 x_1 + w_2 x_2 + \epsilon`$.

### 2.2 Least squares in one dimension
Prediction $`\widehat{y}(x) = w_0 + w_1 x`$. The **least squares** estimate minimises the (half-)mean **residual sum of squares**:
```math
\mathcal{L}(w_0, w_1) = \frac{1}{2N}\sum_{i=1}^N (\widehat{y}_i - y_i)^2 = \frac{1}{2N}\sum_{i=1}^N (w_0 + w_1x_i - y_i)^2 .
```
(The $`\tfrac12`$ and $`\tfrac1N`$ don't change the minimiser; they tidy up derivatives.)

**Derivation.** Take partial derivatives and set them to 0 (the **normal equations**):
```math
\frac{\partial\mathcal{L}}{\partial w_0} = \frac1N\sum_i (w_0 + w_1x_i - y_i) = 0, \qquad \frac{\partial\mathcal{L}}{\partial w_1} = \frac1N\sum_i (w_0 + w_1x_i - y_i)\,x_i = 0 .
```
Define empirical statistics (each point has probability $`1/N`$):
```math
\bar{x} = \frac{\sum_i x_i}{N}, \quad \bar{y} = \frac{\sum_i y_i}{N}, \quad \widehat{\mathrm{var}}(x) = \frac{\sum_i x_i^2}{N} - \bar{x}^2, \quad \widehat{\mathrm{cov}}(x,y) = \frac{\sum_i x_iy_i}{N} - \bar{x}\bar{y}.
```
The first equation gives $`w_0 = \bar{y} - w_1\bar{x}`$. Substituting into the second and simplifying:
```math
w_1 = \frac{\widehat{\mathrm{cov}}(x,y)}{\widehat{\mathrm{var}}(x)}, \qquad w_0 = \bar{y} - w_1\bar{x}.
```
**Intuition:** the slope is "how much $`y`$ co-varies with $`x`$, per unit of variance in $`x`$"; the intercept makes the line pass through the mean point $`(\bar{x}, \bar{y})`$.

### 2.3 Least squares in $`D`$ dimensions (matrix form)
Stack the data: $`\mathbf{X}`$ is the $`N \times (D+1)`$ matrix whose $`i`$-th row is $`\mathbf{x}_i^\top`$ (first column all 1s), $`\mathbf{y}`$ is the column of outputs. Then all predictions at once are $`\widehat{\mathbf{y}} = \mathbf{X}\mathbf{w}`$ and
```math
\mathcal{L}(\mathbf{w}) = \frac{1}{2N}\sum_{i=1}^N(\mathbf{x}_i^\top\mathbf{w} - y_i)^2 = \frac{1}{2N}(\mathbf{X}\mathbf{w} - \mathbf{y})^\top(\mathbf{X}\mathbf{w} - \mathbf{y}) = \frac{1}{2N}\Big(\mathbf{w}^\top(\mathbf{X}^\top\mathbf{X})\mathbf{w} - 2\,\mathbf{y}^\top\mathbf{X}\mathbf{w} + \mathbf{y}^\top\mathbf{y}\Big).
```
(We used $`\mathbf{w}^\top\mathbf{X}^\top\mathbf{y} = \mathbf{y}^\top\mathbf{X}\mathbf{w}`$: both are the same scalar.)

**Two matrix-calculus rules** (memorise; derived by writing out the sums):
1. **Linear form:** $`\nabla_{\mathbf{w}}(\mathbf{c}^\top\mathbf{w}) = \mathbf{c}`$, since $`\partial(\sum_j c_jw_j)/\partial w_k = c_k`$.
2. **Quadratic form:** $`\nabla_{\mathbf{w}}(\mathbf{w}^\top\mathbf{A}\mathbf{w}) = \mathbf{A}\mathbf{w} + \mathbf{A}^\top\mathbf{w}`$, which is $`2\mathbf{A}\mathbf{w}`$ for symmetric $`\mathbf{A}`$. (Since $`\mathbf{w}^\top\mathbf{A}\mathbf{w} = \sum_i\sum_j w_iw_jA_{ij}`$, differentiating w.r.t. $`w_k`$ picks up the $`k`$-th column and the $`k`$-th row of $`\mathbf{A}`$.)

Apply them ($`\mathbf{X}^\top\mathbf{X}`$ is symmetric):
```math
\nabla_{\mathbf{w}}\mathcal{L} = \frac1N\Big((\mathbf{X}^\top\mathbf{X})\mathbf{w} - \mathbf{X}^\top\mathbf{y}\Big) = \mathbf{0} \quad\Longrightarrow\quad (\mathbf{X}^\top\mathbf{X})\mathbf{w} = \mathbf{X}^\top\mathbf{y}
```
> **Least squares solution (closed form):**
> ```math
> \mathbf{w}_{\text{LS}} = (\mathbf{X}^\top\mathbf{X})^{-1}\mathbf{X}^\top\mathbf{y} \qquad (\text{assuming the inverse exists}).
> ```
> Fitted values: $`\widehat{\mathbf{y}} = \mathbf{X}(\mathbf{X}^\top\mathbf{X})^{-1}\mathbf{X}^\top\mathbf{y}`$; the matrix $`\mathbf{X}(\mathbf{X}^\top\mathbf{X})^{-1}\mathbf{X}^\top`$ is the **hat matrix** (it puts a hat on $`\mathbf{y}`$).

**Why setting the gradient to zero is enough (Fig. 2 of the notes):** $`\mathcal{L}(\mathbf{w})`$ is a convex quadratic *bowl* (its Hessian $`\frac1N\mathbf{X}^\top\mathbf{X}`$ is positive semi-definite), so the single stationary point is the **global** minimum when $`\mathbf{X}^\top\mathbf{X}`$ is invertible.

**Cost:** $`O(D^2N)`$ when $`D < N`$ (forming $`\mathbf{X}^\top\mathbf{X}`$ dominates).

### 2.4 When does $`\mathbf{X}^\top\mathbf{X}`$ fail to be invertible?
$`\mathbf{X}^\top\mathbf{X}`$ is $`(D+1)\times(D+1)`$; it is invertible iff $`\mathbf{X}`$ has rank $`D+1`$, i.e. its columns are **linearly independent**. Problems:
- **Fewer data points than features** ($`N < D+1`$): rank can't reach $`D+1`$. (Fixed by regularisation, Part 2.)
- **Artificial linear dependence:** one-hot encoding all 7 days gives $`x_{\text{mon}} + \dots + x_{\text{sun}} = 1 = x_0`$. **Fix:** drop one category (it becomes the baseline when all others are 0).

### 2.5 Outliers and the absolute loss
Squared loss penalises a residual by its **square**, so one far-away point drags the whole line. Alternative: **absolute loss** $`\sum_i \lvert \widehat{y}_i - y_i\rvert`$, which penalises outliers less. No closed form exists; it's solved by linear programming or subgradient methods (Part 3). (Simply deleting outliers only works if you can spot them; Fig. 3 of the notes is a toy. Changing the loss needs no such judgement.)

### 2.6 Goal, model, algorithm (the three questions for every ML method)
- **Goal:** what do you actually want? Pure prediction (use a complex accurate model) or interpretation (simpler, explainable; sometimes legally required)?
- **Model:** linear model + choice of loss function are both *modelling assumptions*.
- **Algorithm:** here a closed form; elsewhere, iterative optimisation.

The least-squares approach is a pure **"optimisation view"**: pick a sensible loss, minimise it. The **probabilistic view** (next) models the noise explicitly and *derives* the loss.

> [!TIP]
> **✅ Key takeaways**
> 1. A linear model is $`\widehat{y} = \mathbf{w}\cdot\mathbf{x}`$ with a constant feature $`x_0 = 1`$ absorbing the intercept.
> 2. Least squares has a **closed form** $`\mathbf{w} = (\mathbf{X}^\top\mathbf{X})^{-1}\mathbf{X}^\top\mathbf{y}`$, found by setting the gradient to zero using the two matrix-calculus rules.
> 3. The 1-D answer $`w_1 = \widehat{\mathrm{cov}}(x,y)/\widehat{\mathrm{var}}(x)`$ is the same formula in disguise.
> 4. Invertibility needs linearly independent features and $`N > D`$; one-hot encode with one category dropped.
> 5. Squared loss is sensitive to outliers; absolute loss is robust but has no closed form.

> [!WARNING]
> **⚠️ Common misconceptions & errors**
> - ❌ "$`\nabla_{\mathbf{w}}(\mathbf{w}^\top\mathbf{A}\mathbf{w}) = 2\mathbf{A}\mathbf{w}`$ always." ✅ Only for **symmetric** $`\mathbf{A}`$; in general it's $`(\mathbf{A} + \mathbf{A}^\top)\mathbf{w}`$.
> - ❌ Writing $`\mathbf{w} = \mathbf{X}^{-1}\mathbf{y}`$. ✅ $`\mathbf{X}`$ is $`N\times(D+1)`$, not square; you need the normal equations.
> - ❌ Encoding categories as integers $`0,1,2,\dots`$. ✅ Use one-hot (minus one category).
> - ❌ "More features can't hurt least squares." ✅ Dependent or irrelevant features break invertibility / cause overfitting (Part 2).
> - ❌ Forgetting that the $`\frac{1}{2N}`$ factor is arbitrary and doesn't affect $`\arg\min`$.

---

## 3. Probability review

> [!NOTE]
> **📖 Key definitions**
> - **Covariance:** $`\mathrm{cov}(X,Y) = \mathbb{E}[(X-\mathbb{E}X)(Y-\mathbb{E}Y)] = \mathbb{E}[XY] - \mathbb{E}[X]\mathbb{E}[Y]`$. Measures *linear* relationship; depends on units (metres vs km changes it by 1000).
> - **Correlation (Pearson):** $`\mathrm{corr}(X,Y) = \dfrac{\mathrm{cov}(X,Y)}{\sqrt{\mathrm{var}(X)\mathrm{var}(Y)}} \in [-1, 1]`$, scale-free.
> - **Covariance matrix** of $`\mathbf{x} \in \mathbb{R}^D`$: $`\mathrm{cov}(\mathbf{x}) = \mathbb{E}[(\mathbf{x} - \mathbb{E}\mathbf{x})(\mathbf{x}-\mathbb{E}\mathbf{x})^\top]`$, entry $`(i,j)`$ = $`\mathrm{cov}(X_i, X_j)`$, diagonal = variances. (Correlation matrix: same with correlations; diagonal all 1.)
> - **Univariate Gaussian** $`\mathcal{N}(\mu, \sigma^2)`$: $`p(x) = \dfrac{1}{\sqrt{2\pi}\sigma}\exp\Big(-\dfrac{(x-\mu)^2}{2\sigma^2}\Big)`$.
> - **Multivariate Gaussian** $`\mathcal{N}(\boldsymbol\mu, \boldsymbol\Sigma)`$ in $`D`$ dimensions: $`p(\mathbf{x}) = \dfrac{1}{(2\pi)^{D/2}\lvert\boldsymbol\Sigma\rvert^{1/2}}\exp\Big(-\tfrac12(\mathbf{x}-\boldsymbol\mu)^\top\boldsymbol\Sigma^{-1}(\mathbf{x}-\boldsymbol\mu)\Big)`$.
> - **Laplace distribution** $`\mathrm{Lap}(\mu, b)`$: $`p(x) = \dfrac{1}{2b}\exp\Big(-\dfrac{\lvert x - \mu\rvert}{b}\Big)`$; mean $`\mu`$, variance $`2b^2`$. Heavier tails than the Gaussian.
> - **Likelihood** of i.i.d. observations under parameters $`\theta`$: $`p(x_1, \dots, x_N \mid \theta) = \prod_{i=1}^N p(x_i \mid \theta)`$.

**Independence vs correlation:** independent ⇒ correlation 0, but **not conversely**. Example: $`X \sim U[-1,1]`$, $`Y = X^2`$: $`\mathrm{cov}(X,Y) = \mathbb{E}X^3 - \mathbb{E}X\,\mathbb{E}X^2 = 0`$, yet $`Y`$ is a function of $`X`$.

**Eigen-structure of a covariance matrix.** Solve $`\boldsymbol\Sigma\mathbf{v}_i = \lambda_i\mathbf{v}_i`$ and order $`\lvert\lambda_1\rvert > \lvert\lambda_2\rvert > \dots`$. The eigenvector with the **largest eigenvalue is the direction of greatest variance**; the smallest, least variance. (This is the seed of PCA, Part 6.) Conversely you can *build* a covariance from chosen directions: $`\boldsymbol\Sigma = \mathbf{Q}\boldsymbol\Lambda\mathbf{Q}^{-1}`$ with $`\boldsymbol\Lambda = \mathrm{diag}(\lambda_1, \dots, \lambda_D)`$ and $`\mathbf{Q} = [\mathbf{v}_1 \cdots \mathbf{v}_D]`$.

---

## 4. Maximum likelihood and the probabilistic view of regression

> [!NOTE]
> **📖 Key definitions**
> - **Maximum likelihood principle:** choose the parameters $`\theta`$ under which the observed data are **most probable**: $`\theta_{\text{ML}} = \arg\max_\theta p(\mathcal{D}\mid\theta)`$.
> - **Log-likelihood** $`\mathrm{LL} = \log p(\mathcal{D}\mid\theta)`$; **negative log-likelihood** $`\mathrm{NLL} = -\mathrm{LL}`$. Since $`\log`$ is increasing, maximising LL = maximising likelihood = minimising NLL. Products become sums.
> - **Discriminative model:** model only $`p(y \mid \mathbf{x}, \mathbf{w})`$; inputs are treated as fixed. **Generative model:** model the joint $`p(\mathbf{x}, y)`$ too (Part 4).
> - **Aleatoric uncertainty:** randomness *in the data itself* (noise); not reducible with more data. (Epistemic uncertainty, about the parameters, is in Part 2.)

**Worked intuition.** Points $`0.3, 1.4, 1.7`$ came from $`\mathcal{N}(0,1)`$ or $`\mathcal{N}(2,1)`$? $`x_1`$ is as far from 0 as $`x_3`$ is from 2 (and vice versa), so only $`x_2`$ matters: it's closer to 2, so $`\mathcal{N}(2,1)`$ has the higher likelihood. (Works only because the variances are equal!)

### 4.1 Linear regression with Gaussian noise
Model the output as a random variable with mean $`\mathbf{w}^\top\mathbf{x}`$:
```math
\mathbb{E}[y \mid \mathbf{w}, \mathbf{x}] = \mathbf{w}^\top\mathbf{x}, \qquad p(y \mid \mathbf{w}, \mathbf{x}) = \mathcal{N}(y \mid \mathbf{w}^\top\mathbf{x}, \sigma^2), \qquad y \sim \mathbf{w}^\top\mathbf{x} + \mathcal{N}(0, \sigma^2).
```
Equivalently the noise $`\epsilon_i \sim \mathcal{N}(0, \sigma^2)`$, **independent** across data points. The regression function is the model mean $`f_{\mathbf{w}}(\mathbf{x}) = \mathbb{E}[y\mid\mathbf{x},\mathbf{w}]`$. Why bother with a distribution? It (i) makes assumptions explicit, (ii) quantifies risk (how far outputs can stray), (iii) lets you *sample* predictions, (iv) lets you **derive** the loss.

**Likelihood of the data** (independence ⇒ product):
```math
p(y_1,\dots,y_N \mid \mathbf{X}, \mathbf{w}, \sigma) = \prod_{i=1}^N \frac{1}{\sqrt{2\pi\sigma^2}}\exp\Big(-\frac{(y_i - \mathbf{w}^\top\mathbf{x}_i)^2}{2\sigma^2}\Big) = \Big(\frac{1}{2\pi\sigma^2}\Big)^{N/2}\exp\Big(-\frac{1}{2\sigma^2}\sum_{i=1}^N(y_i - \mathbf{w}^\top\mathbf{x}_i)^2\Big).
```
Geometrically (Fig. 5a of the notes): the likelihood is the *product of the density values* at each observed point; MLE picks the line maximising that product, whereas least squares minimises the sum of squared residuals. They turn out to agree:

**Negative log-likelihood:**
```math
\mathrm{NLL}(\mathbf{y}\mid\mathbf{X},\mathbf{w},\sigma) = \frac{1}{2\sigma^2}(\mathbf{X}\mathbf{w}-\mathbf{y})^\top(\mathbf{X}\mathbf{w}-\mathbf{y}) + \frac N2\log(2\pi\sigma^2).
```
As a function of $`\mathbf{w}`$, this is the least-squares objective $`\frac{1}{2N}(\mathbf{X}\mathbf{w}-\mathbf{y})^\top(\mathbf{X}\mathbf{w}-\mathbf{y})`$ **up to a positive multiplicative and an additive constant**. Hence

> ```math
> \mathbf{w}_{\text{ML}} = (\mathbf{X}^\top\mathbf{X})^{-1}\mathbf{X}^\top\mathbf{y} = \mathbf{w}_{\text{LS}}, \qquad \sigma^2_{\text{ML}} = \frac1N(\mathbf{X}\mathbf{w}_{\text{ML}} - \mathbf{y})^\top(\mathbf{X}\mathbf{w}_{\text{ML}} - \mathbf{y}) \ \ (\text{mean squared residual}).
> ```
> *(Derivation of $`\sigma^2_{\text{ML}}`$, Sheet 2: differentiate NLL w.r.t. $`\sigma^2`$: $`-\frac{1}{2\sigma^4}\text{RSS} + \frac{N}{2\sigma^2} = 0 \Rightarrow \sigma^2 = \text{RSS}/N`$.)*

**This is the justification for least squares:** it *is* maximum likelihood under Gaussian noise. (Whether Gaussian noise is appropriate is a separate question.)

**Prediction with uncertainty:** $`\widehat{y}_{\text{new}} = \mathbf{w}_{\text{ML}}\cdot\mathbf{x}_{\text{new}}`$ and $`y_{\text{new}} \sim \widehat{y}_{\text{new}} + \mathcal{N}(0, \sigma^2_{\text{ML}})`$, which gives confidence intervals.

### 4.2 Outliers and the Laplace noise model
Since MLE under Gaussian noise = least squares, it is equally **sensitive to outliers**. Reason: the Gaussian has very **light tails** (little mass a few σ from the mean), so an outlier is "impossible" under the model and the fit contorts to accommodate it. A **heavier-tailed** noise model expects occasional big deviations:
```math
p(y\mid\mathbf{w},\mathbf{x}) = \mathrm{Lap}(y \mid \mathbf{w}\cdot\mathbf{x}, b), \qquad p(\mathbf{y}\mid\mathbf{X},\mathbf{w},b) = \frac{1}{(2b)^N}\exp\Big(-\frac1b\sum_{i=1}^N\lvert y_i - \mathbf{w}^\top\mathbf{x}_i\rvert\Big),
```
```math
\mathrm{NLL}(\mathbf{y}\mid\mathbf{X},\mathbf{w},b) = \frac1b\sum_{i=1}^N\lvert y_i - \mathbf{w}^\top\mathbf{x}_i\rvert + N\log(2b).
```
So **MLE under Laplace noise = minimising the absolute loss**, exactly the robust alternative from §2.5. No closed form; see Part 3 (linear programming / subgradients). ($`b_{\text{ML}}`$ = mean absolute residual, by the same differentiation trick.)

> [!TIP]
> **✅ Key takeaways**
> 1. **Noise model ⇔ loss function.** Gaussian noise ⇒ squared loss; Laplace noise ⇒ absolute loss. This pattern (NLL = loss) recurs for logistic regression (cross-entropy) and neural networks.
> 2. MLE: write the likelihood as a product, take logs to get a sum, drop constants, minimise.
> 3. $`\sigma^2_{\text{ML}}`$ is the average squared residual, $`b_{\text{ML}}`$ the average absolute residual.
> 4. Heavier tails ⇒ more robust to outliers.
> 5. Covariance eigenvectors = directions of greatest variance (→ PCA).

> [!WARNING]
> **⚠️ Common misconceptions & errors**
> - ❌ "Correlation 0 ⇒ independent." ✅ Counterexample $`Y = X^2`$.
> - ❌ "Likelihood is the probability of the parameters." ✅ It's the probability of the **data** given the parameters, viewed as a function of the parameters.
> - ❌ Forgetting independence when writing the likelihood as a product.
> - ❌ "MLE under Gaussian noise differs from least squares." ✅ Identical $`\mathbf{w}`$; MLE additionally gives $`\sigma^2`$.
> - ❌ "Dividing by $`N-1`$ for $`\sigma^2_{\text{ML}}`$." ✅ The MLE divides by $`N`$ (it's biased, but that's what MLE gives).
> - ❌ Mixing up the Laplace scale $`b`$ with a variance: $`\mathrm{var} = 2b^2`$.
