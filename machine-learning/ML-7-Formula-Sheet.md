# Machine Learning · Formula Sheet, Concepts Not to Confuse, Top Mistakes

*Quick reference for the whole course. [Overview](ML-Summary.md) · [Part 1](ML-1-Linear-Regression-and-Maximum-Likelihood.md) · [Part 2](ML-2-Generalisation-Regularisation-and-Bayes.md) · [Part 3](ML-3-Optimisation.md) · [Part 4](ML-4-Classification.md) · [Part 5](ML-5-Neural-Networks.md) · [Part 6](ML-6-Unsupervised-Learning.md)*

*Exam note (lecture 20): the exam tests **understanding of ideas**, not memorised formulas; you should know the basic models (linear/logistic regression, …) and only the **Bernoulli and Gaussian** distributions by heart.*

---

## 1. Models and their estimators

| Model | Prediction | Objective (minimise) | Solution |
|---|---|---|---|
| Linear regression (least squares) | $`\widehat{y} = \mathbf{w}\cdot\mathbf{x}`$ | $`\frac{1}{2N}(\mathbf{X}\mathbf{w}-\mathbf{y})^\top(\mathbf{X}\mathbf{w}-\mathbf{y})`$ | $`\mathbf{w} = (\mathbf{X}^\top\mathbf{X})^{-1}\mathbf{X}^\top\mathbf{y}`$ |
| 1-D least squares | $`w_0 + w_1x`$ | same | $`w_1 = \widehat{\operatorname{cov}}(x,y)/\widehat{\operatorname{var}}(x)`$, $`w_0 = \bar y - w_1\bar x`$ |
| Gaussian-noise MLE | same | NLL $`= \frac{1}{2\sigma^2}\text{RSS} + \frac N2\log(2\pi\sigma^2)`$ | $`\mathbf{w}_{\text{ML}} = \mathbf{w}_{\text{LS}}`$, $`\sigma^2_{\text{ML}} = \text{RSS}/N`$ |
| Laplace-noise MLE | same | $`\frac1b\sum_i\lvert y_i - \mathbf{w}^\top\mathbf{x}_i\rvert + N\log 2b`$ | absolute loss; LP / subgradient; $`b_{\text{ML}} = \frac1N\sum_i\lvert y_i - \mathbf{w}^\top\mathbf{x}_i\rvert`$ |
| Ridge | same | $`\text{RSS} + \lambda\lVert\mathbf{w}\rVert_2^2`$ | $`\mathbf{w} = (\mathbf{X}^\top\mathbf{X} + \lambda\mathbf{I})^{-1}\mathbf{X}^\top\mathbf{y}`$ |
| Lasso | same | $`\text{RSS} + \lambda\lVert\mathbf{w}\rVert_1`$ | no closed form; sparse |
| Bayesian linear regression | $`\mathbf{w}_N^\top\mathbf{x}`$ | prior $`\mathcal{N}(\mathbf{0},\boldsymbol\Lambda)`$ | $`\boldsymbol\Sigma_N = (\boldsymbol\Lambda^{-1} + \sigma^{-2}\mathbf{X}^\top\mathbf{X})^{-1}`$, $`\mathbf{w}_N = \sigma^{-2}\boldsymbol\Sigma_N\mathbf{X}^\top\mathbf{y}`$, pred. var $`\sigma^2 + \mathbf{x}^\top\boldsymbol\Sigma_N\mathbf{x}`$ |
| MAP | | $`-\log p(\mathcal{D}\mid\mathbf{w}) - \log p(\mathbf{w})`$ $`\propto \text{RSS} + 2\sigma^2\mathcal{R}(\mathbf{w})`$ | prior $`\mathcal{N}(\mathbf{0}, (\sigma^2/\lambda)\mathbf{I})`$ ⇒ ridge; $`\prod_i\operatorname{Lap}(0, 2\sigma^2/\lambda)`$ ⇒ lasso; flat ⇒ MLE |
| Gaussian mean, known $`\sigma^2`$ (conjugate) | | prior $`\mathcal{N}(\mu_0, \sigma_0^2)`$ | $`\mu_N = \frac{(\sigma^2/\sigma_0^2)\mu_0 + \sum_it_i}{\sigma^2/\sigma_0^2 + N}`$, $`\sigma_N^2 = (1/\sigma_0^2 + N/\sigma^2)^{-1}`$, predictive var $`\sigma^2 + \sigma_N^2`$ |
| Generative classifier | $`\arg\max_c\pi_c\,p(\mathbf{x}\mid c)`$ | log-lik $`\sum_cN_c\log\pi_c + \sum_i\log p(\mathbf{x}_i\mid y_i)`$ | $`\pi_c = N_c/N`$ |
| Naïve Bayes | same with $`p(\mathbf{x}\mid c) = \prod_jp(x_j\mid c)`$ | per-feature 1-D MLEs | params linear in $`D`$ ($`O(CD)`$ binary, $`O(CDK)`$ categorical) |
| QDA / LDA | Gaussian class-conditionals | $`\widehat{\boldsymbol\mu}_c`$ = class mean, $`\widehat{\boldsymbol\Sigma}_c`$ = class covariance | quadratic / linear boundaries; LDA ⇒ softmax |
| Logistic regression | $`p(y=1\mid\mathbf{x}) = \sigma(\mathbf{w}\cdot\mathbf{x})`$ | NLL $`= -\sum_i\big(y_i\log\mu_i + (1-y_i)\log(1-\mu_i)\big)`$ | gradient $`\mathbf{X}^\top(\boldsymbol\mu-\mathbf{y})`$, Hessian $`\mathbf{X}^\top\mathbf{S}\mathbf{X}`$; Newton = IRLS |
| Multiclass logistic | $`\operatorname{softmax}([\mathbf{w}_c\cdot\mathbf{x}]_c)`$ | convex NLL | gradient methods |
| SVM (primal) | $`\operatorname{sign}(\mathbf{w}\cdot\mathbf{x} + w_0)`$ | $`\frac12\lVert\mathbf{w}\rVert^2 + C\sum_i\zeta_i`$ s.t. $`y_i(\mathbf{w}\cdot\mathbf{x}_i+w_0) \ge 1-\zeta_i`$, $`\zeta_i\ge0`$ | QP; $`=\frac12\lVert\mathbf{w}\rVert^2 + C\sum_i\text{hinge}_i`$ |
| SVM (dual) | $`\sum_i\alpha_iy_i\kappa(\mathbf{x}_i,\mathbf{x}) + w_0`$ | max $`\sum_i\alpha_i - \frac12\sum_{ij}\alpha_i\alpha_jy_iy_j\kappa(\mathbf{x}_i,\mathbf{x}_j)`$ s.t. $`\sum_i\alpha_iy_i = 0`$, $`0\le\alpha_i\le C`$ | $`\mathbf{w} = \sum_i\alpha_iy_i\mathbf{x}_i`$; $`\alpha_i > 0`$ ⇔ support vector |
| Neural net | $`\mathbf{a}^L`$ after FE1–FE2 | cross-entropy / squared error | backprop + SGD |
| k-means | nearest mean | $`\sum_j\sum_{i\in C_j}\lVert\mathbf{x}_i - \boldsymbol\mu_j\rVert^2`$ | Lloyd's (local opt.) |
| PCA | $`\mathbf{z} = \mathbf{U}_k^\top(\mathbf{x}-\boldsymbol\mu)`$ | $`\lVert\mathbf{X} - \widetilde{\mathbf{X}}\rVert_F^2`$ | truncated SVD (Eckart–Young) |

## 2. Distributions you must know
```math
\mathcal{N}(x\mid\mu,\sigma^2) = \frac{1}{\sqrt{2\pi}\sigma}e^{-(x-\mu)^2/2\sigma^2}, \qquad \mathcal{N}(\mathbf{x}\mid\boldsymbol\mu,\boldsymbol\Sigma) = \frac{1}{(2\pi)^{D/2}\lvert\boldsymbol\Sigma\rvert^{1/2}}e^{-\frac12(\mathbf{x}-\boldsymbol\mu)^\top\boldsymbol\Sigma^{-1}(\mathbf{x}-\boldsymbol\mu)}, \qquad \operatorname{Bernoulli}(y\mid\theta) = \theta^y(1-\theta)^{1-y}
```
Also seen: Laplace $`\frac{1}{2b}e^{-\lvert x-\mu\rvert/b}`$ (variance $`2b^2`$); multinoulli $`p(x = l) = \mu_l`$. Empirical statistics: $`\widehat{\operatorname{var}}(x) = \frac1N\sum_ix_i^2 - \bar x^2`$, $`\widehat{\operatorname{cov}}(x,y) = \frac1N\sum_ix_iy_i - \bar x\bar y`$.

## 3. Calculus toolkit
| Rule | Formula |
|---|---|
| Linear form | $`\nabla_{\mathbf{w}}(\mathbf{c}^\top\mathbf{w}) = \mathbf{c}`$ |
| Quadratic form | $`\nabla_{\mathbf{w}}(\mathbf{w}^\top\mathbf{A}\mathbf{w}) = (\mathbf{A} + \mathbf{A}^\top)\mathbf{w}`$ ($`= 2\mathbf{A}\mathbf{w}`$ if symmetric) |
| Sigmoid | $`\sigma(t) = \frac{1}{1+e^{-t}}`$, $`\sigma' = \sigma(1-\sigma) \le \frac14`$, $`1 - \sigma(t) = \sigma(-t)`$ |
| tanh | $`\tanh' = 1 - \tanh^2`$ |
| ReLU | $`\max(0,z)`$; derivative $`\mathbb{1}(z>0)`$; subgradient at 0 is $`[0,1]`$ |
| Softmax | $`\operatorname{softmax}(\mathbf{z})_c = e^{z_c}/\sum_{c'}e^{z_{c'}}`$; shift-invariant, not scale-invariant |
| Cross-entropy + sigmoid/softmax output | $`\partial\ell/\partial\mathbf{z}^L = \mathbf{a}^L - \mathbf{y}`$ |
| Chain rule (Jacobians) | $`\frac{\partial h}{\partial\mathbf{x}} = \frac{\partial h}{\partial\mathbf{z}}\frac{\partial\mathbf{z}}{\partial\mathbf{x}}`$ (row-vector convention for scalars) |
| Subgradient | $`\mathbf{g}`$ with $`f(\mathbf{w}) \ge f(\mathbf{w}_0) + \mathbf{g}^\top(\mathbf{w}-\mathbf{w}_0)`$; $`\partial\lvert w\rvert`$ at 0 is $`[-1,1]`$ |
| Lagrangian (equality) | $`\Lambda = f + \lambda g`$; stationary ⇒ $`\nabla f = -\lambda\nabla g`$ |
| KKT (inequalities $`g_i \ge 0`$) | stationarity $`\nabla_{\mathbf{z}}\Lambda = \mathbf{0}`$, primal feasible, $`\alpha_i \ge 0`$, $`\alpha_ig_i = 0`$; sufficient for convex problems |

## 4. Optimisation updates
```math
\text{GD: } \mathbf{w}_{t+1} = \mathbf{w}_t - \eta_t\nabla f(\mathbf{w}_t) \qquad
\text{Newton: } \mathbf{w}_{t+1} = \mathbf{w}_t - \mathbf{H}_t^{-1}\mathbf{g}_t \qquad
\text{SGD: same as GD with } \nabla\ell(\mathbf{w};\mathbf{x}_i,y_i) + \lambda\nabla\mathcal{R},\ i \text{ random}
```
Costs per step: GD $`O(ND)`$, SGD $`O(D)`$, Newton needs $`D + \binom D2`$ second derivatives + inversion. Convex ⇒ local = global. Hessian eigenvalues: all $`>0`$ min, all $`<0`$ max, mixed saddle.

**Backprop:** FE1 $`\mathbf{z}^l = \mathbf{W}^l\mathbf{a}^{l-1} + \mathbf{b}^l`$; FE2 $`\mathbf{a}^l = f_l(\mathbf{z}^l)`$; BE1 $`\frac{\partial\ell}{\partial\mathbf{z}^L} = \frac{\partial\ell}{\partial\mathbf{a}^L}\frac{\partial\mathbf{a}^L}{\partial\mathbf{z}^L}`$; BE2 $`\frac{\partial\ell}{\partial\mathbf{z}^l} = \frac{\partial\ell}{\partial\mathbf{z}^{l+1}}\mathbf{W}^{l+1}\frac{\partial\mathbf{a}^l}{\partial\mathbf{z}^l}`$; BE3 $`\frac{\partial\ell}{\partial\mathbf{W}^l} = \big(\mathbf{a}^{l-1}\frac{\partial\ell}{\partial\mathbf{z}^l}\big)^\top`$; BE4 $`\frac{\partial\ell}{\partial\mathbf{b}^l} = \frac{\partial\ell}{\partial\mathbf{z}^l}`$.

## 5. Kernels
| Kernel | Formula | Notes |
|---|---|---|
| Linear | $`\mathbf{x}\cdot\mathbf{x}'`$ | |
| Polynomial degree $`d`$ | $`(1 + \mathbf{x}\cdot\mathbf{x}')^d`$ | all monomials up to degree $`d`$; $`O(D\log d)`$ |
| RBF / Gaussian | $`\exp(-\lVert\mathbf{x}-\mathbf{x}'\rVert^2/2\sigma^2) = \exp(-\gamma\lVert\mathbf{x}-\mathbf{x}'\rVert^2)`$ | bandwidth $`\sigma`$; small $`\gamma`$ = wide = underfit |
| String | $`\sum_sw_s\phi_s(\mathbf{x})\phi_s(\mathbf{x}')`$ | substring counts; $`O(\lvert\mathbf{x}\rvert+\lvert\mathbf{x}'\rvert)`$ |
| Closure | $`\kappa_1+\kappa_2`$, $`\alpha\kappa_1`$, $`\kappa_1\kappa_2`$, $`\mathbf{x}^\top\mathbf{B}\mathbf{x}'`$ ($`\mathbf{B}`$ PD) | all Mercer |

Mercer ⇔ Gram matrix always PSD ⇔ an inner product in some feature space ⇒ dual is concave.

## 6. Evaluation
| Quantity | Formula |
|---|---|
| TPR = recall = sensitivity | $`\text{TP}/(\text{TP}+\text{FN})`$ |
| FPR | $`\text{FP}/(\text{FP}+\text{TN})`$ |
| Precision | $`\text{TP}/(\text{TP}+\text{FP})`$ |
| ROC curve / AUC | TPR vs FPR as threshold varies / area (1 best) |
| Mutual information | $`I(X,Y) = \sum_{x,y}p(x,y)\log\frac{p(x,y)}{p(x)p(y)}`$ |
| Perplexity | $`2^{H}`$, $`H = -\frac1N\log_2p(w_1^N)`$ |
| k-fold CV error | mean validation error over $`K`$ folds |
| PCA relative error | $`\sum_{i>k}\sigma_i^2/\sum_i\sigma_i^2`$ |

## 7. Linear algebra for PCA
$`\mathbf{A} = \mathbf{U}\boldsymbol\Sigma\mathbf{V}^\top = \sum_i\sigma_i\mathbf{u}_i\mathbf{v}_i^\top`$; $`\mathbf{A}^\top\mathbf{A}\mathbf{v}_i = \sigma_i^2\mathbf{v}_i`$; $`\lVert\mathbf{A}\rVert_F^2 = \sum_i\sigma_i^2`$; $`\lVert\mathbf{A}-\mathbf{A}_k\rVert_F^2 = \sum_{i>k}\sigma_i^2`$ (minimal over rank-$`k`$ matrices); PCA: $`\mathbf{X}_k = \mathbf{U}_k\mathbf{U}_k^\top\mathbf{X}`$; power iteration: with probability $`\ge 9/10`$ over $`\mathbf{x}_0`$, $`1 - \lvert\mathbf{y}_k^\top\mathbf{v}_1\rvert \le 20n(\lambda_2/\lambda_1)^k`$ for all $`k`$ ($`\lambda_i = \sigma_i^2`$); graph Laplacian $`L = D - W`$, PSD, $`L\mathbf{1} = \mathbf{0}`$.

---

## 8. Concepts you must not confuse
| Pair | Difference |
|---|---|
| Regression vs classification | real-valued output vs categorical output |
| Supervised vs unsupervised | labelled pairs vs inputs only |
| Parameters vs hyperparameters | learned by the training algorithm ($`\mathbf{w}`$) vs set beforehand and chosen by validation ($`\lambda, \gamma, \eta, k`$, degree) |
| Training / validation / test set | fit $`\mathbf{w}`$ / choose hyperparameters / report once |
| Bias vs variance | systematically wrong (underfit) vs unstable across datasets (overfit) |
| Loss vs likelihood | minimised vs maximised; NLL = loss under a noise model |
| Gaussian vs Laplace noise | squared loss, outlier-sensitive vs absolute loss, robust |
| Ridge vs lasso | $`\ell_2`$, closed form, shrinks vs $`\ell_1`$, no closed form, sparse |
| $`\lambda`$ vs $`C`$ | more $`\lambda`$ = more regularisation; more $`C`$ = **less** (SVM) |
| MLE vs MAP vs full Bayes | max likelihood / max posterior (point) / integrate over posterior |
| Aleatoric vs epistemic uncertainty | noise, irreducible vs parameter uncertainty, reducible |
| Frequentist vs Bayesian | parameters fixed, data random vs data fixed, parameters uncertain |
| Generative vs discriminative | model $`p(\mathbf{x},y)`$ (NB, LDA/QDA) vs $`p(y\mid\mathbf{x})`$ (logistic regression) |
| LDA vs QDA | shared covariance, linear boundary vs per-class covariance, quadratic boundary |
| LDA vs logistic regression | same sigmoid/softmax form, parameters from class statistics vs from NLL minimisation |
| Naïve Bayes assumption | **conditional** independence given the class, not independence |
| Convex vs non-convex | local = global vs local minima and saddles (neural nets) |
| GD vs SGD vs Newton | full gradient / random point gradient / Hessian step |
| Gradient vs subgradient | unique at smooth points vs a set at kinks |
| Primal vs dual SVM | $`N+D+1`$ vars, polytope vs $`N`$ vars, box constraints, kernelisable |
| Margin vs slack | margin of a point = its distance to the boundary; after normalising $`y_i(\mathbf{w}\cdot\mathbf{x}_i + w_0) \ge 1`$ the *smallest* margin is $`1/\lVert\mathbf{w}\rVert_2`$; slack = allowed violation $`\zeta_i \ge 0`$ |
| Hinge vs logistic loss | exactly 0 beyond margin 1 vs approaches 0 |
| Kernel vs feature map | $`\kappa(\mathbf{x},\mathbf{x}') = \phi(\mathbf{x})\cdot\phi(\mathbf{x}')`$; the trick avoids computing $`\phi`$ |
| ovo vs ovr | $`\binom K2`$ small classifiers, vote vs $`K`$ classifiers, use scores |
| Precision vs recall | TP / predicted positive vs TP / actual positive |
| Type I vs Type II | false positive vs false negative |
| Saturation vs vanishing gradient | one unit's $`\sigma'\approx0`$ vs product of many $`\sigma'\le\frac14`$ across layers |
| Dropout vs bagging | random units dropped per step vs models trained on bootstrap resamples; both average ⇒ less variance |
| Convolution channels vs filters | input depth vs number of output maps |
| n-gram vs RNN | fixed Markov history vs unbounded history in hidden state |
| BPTT vs TBPTT | full unrolled backprop vs truncated at length $`k`$ (forward still exact) |
| Singular values vs eigenvalues | $`\sigma_i = \sqrt{\lambda_i(\mathbf{A}^\top\mathbf{A})}`$ |
| PCA vs kernel PCA vs spectral clustering | SVD of centred data / same via kernel Gram matrix / bottom eigenvectors of graph Laplacian |
| k-means vs k-median vs k-center | squared Euclidean sum / Euclidean sum / max distance |
| Single vs complete vs average linkage | min pair / max pair / mean pair dissimilarity |
| Flat vs hierarchical clustering | partition vs dendrogram |

## 9. Top 20 mistakes
1. Tuning on the test set (leakage) or picking $`\lambda`$ by training error.
2. Standardising test data with its own statistics; forgetting to standardise before ridge/lasso.
3. Penalising the intercept.
4. $`\nabla(\mathbf{w}^\top\mathbf{A}\mathbf{w}) = 2\mathbf{A}\mathbf{w}`$ for non-symmetric $`\mathbf{A}`$.
5. Forgetting independence when writing a likelihood as a product; dropping the minus in NLL.
6. "Correlation 0 ⇒ independent".
7. Swapping Gaussian↔ridge / Laplace↔lasso; thinking larger prior variance = stronger regularisation.
8. "Naïve Bayes assumes independent features".
9. Thresholding logistic regression at $`\mathbf{w}\cdot\mathbf{x} \ge 0.5`$ instead of $`\ge 0`$.
10. Claiming a closed form for logistic regression / lasso / SVM.
11. Sign errors in GD/Newton; forgetting the regulariser gradient in SGD.
12. "Newton always decreases the objective".
13. Margin $`= \lVert\mathbf{w}\rVert`$; negative slack allowed; "large $`C`$ = more regularisation".
14. Non-Mercer kernel ⇒ expecting a concave dual.
15. Treating layer superscripts as powers; wrong Jacobian order; missing transpose in BE3.
16. All-zero initialisation; squared loss on sigmoid outputs; no weight rescaling after dropout.
17. Max-pool gradient spread over the window instead of the argmax.
18. Not centring before PCA; $`\sqrt{\sigma_{k+1} + \dots}`$ instead of $`\sqrt{\sigma_{k+1}^2 + \dots}`$.
19. Choosing $`k`$ by minimising the k-means objective.
20. Using the largest Laplacian eigenvectors in spectral clustering.
