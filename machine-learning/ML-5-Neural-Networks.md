# Machine Learning · Part 5: Neural Networks

*Lectures 14–17. [← Part 4](ML-4-Classification.md) · [Overview](ML-Summary.md) · [Part 6 →](ML-6-Unsupervised-Learning.md) · [Formula sheet](ML-7-Formula-Sheet.md)*

---

## Contents
22. [Artificial neurons and multilayer perceptrons](#22-artificial-neurons-and-multilayer-perceptrons)
23. [Backpropagation](#23-backpropagation)
24. [Training neural networks: problems and tricks](#24-training-neural-networks-problems-and-tricks)
25. [Convolutional neural networks](#25-convolutional-neural-networks)
26. [Language models and recurrent neural networks](#26-language-models-and-recurrent-neural-networks)

---

## 22. Artificial neurons and multilayer perceptrons

> [!NOTE]
> **📖 Key definitions**
> - **Artificial neuron / unit:** output $`f(b + w_1x_1 + \dots + w_Dx_D)`$: a **linear function** of the inputs followed by a non-linear **activation function** $`f`$. Perceptron: $`f = \mathrm{sign}`$. Logistic regression: $`f = \sigma`$ (a single-unit network).
> - **Multilayer perceptron (MLP):** units arranged in **layers**. Layer 1 = **input** $`\mathbf{x}`$; last layer $`L`$ = **output**; the rest are **hidden** (not observed in the data). **Fully connected:** every unit in layer $`l-1`$ feeds every unit in layer $`l`$.
> - **Notation:** $`w^l_{ij}`$ = weight from unit $`j`$ of layer $`l-1`$ to unit $`i`$ of layer $`l`$ (superscripts are **layer indices, not powers**); matrix $`\mathbf{W}^l`$ of size $`n_l\times n_{l-1}`$; bias vector $`\mathbf{b}^l`$.
> - **Pre-activation** $`\mathbf{z}^l = \mathbf{W}^l\mathbf{a}^{l-1} + \mathbf{b}^l`$; **activation** $`\mathbf{a}^l = f_l(\mathbf{z}^l)`$ (usually unit-wise, $`a^l_i = f_l(z^l_i)`$; softmax acts on the whole layer).
> - **Softmax (layer-wise activation, usually only at the output):** $`\mathrm{softmax}(\mathbf{z})_i = e^{z_i}/\sum_je^{z_j}`$; its Jacobian $`\partial\mathbf{a}^L/\partial\mathbf{z}^L`$ is a full $`n_L\times n_L`$ matrix.
> - **Common activations:** sigmoid $`\sigma(z) = 1/(1+e^{-z})`$, $`\sigma' = \sigma(1-\sigma) \le \tfrac14`$; $`\tanh`$, $`\tanh' = 1 - \tanh^2`$; **ReLU** $`f(z) = \max(0, z)`$, derivative 1 for $`z > 0`$, 0 otherwise.
> - **Cross-entropy loss** (binary, $`\widehat{y}`$ = predicted probability): $`\ell = -\big(y\log\widehat{y} + (1-y)\log(1-\widehat{y})\big)`$. $`C`$ classes with softmax output: $`\ell = -\sum_c\mathbb{1}(y = c)\log a^L_c`$. Regression: squared error $`(a^L - y)^2`$.

**Why compose units?** Artificial neurons can implement Boolean gates (Sheet 1), so networks can compute anything a computer can. Unlike Boolean circuits, networks with **continuous activations are differentiable end to end**, so gradient methods can train them. (The notes stress this is intuition, not a theorem: Boolean circuits aren't differentiable, so they'd need heuristic or genetic search, and far less effort has gone into training them.)

**Toy MLP (Fig. 3):** inputs $`x_1, x_2`$ → hidden layer of 2 tanh units → one sigmoid output:
```math
\mathbf{a}^1 = \mathbf{z}^1 = \mathbf{x}, \quad \mathbf{z}^2 = \mathbf{W}^2\mathbf{a}^1 + \mathbf{b}^2, \quad \mathbf{a}^2 = \tanh(\mathbf{z}^2), \quad \mathbf{z}^3 = \mathbf{W}^3\mathbf{a}^2 + \mathbf{b}^3, \quad y = \mathbf{a}^3 = \sigma(\mathbf{z}^3).
```
On data that no line can separate (Fig. 2a), logistic regression gets ≈50%; the MLP gets it right. **Why:** the hidden layer maps $`(x_1, x_2) \mapsto (a^2_1, a^2_2)`$ non-linearly, and in the new coordinates the data are nearly linearly separable (Fig. 2c); the output unit is just logistic regression on $`\mathbf{a}^2`$. The hidden map is **non-linear and non-local** (each $`a^2_i`$ depends on *both* $`x_1, x_2`$), and the learned decision region (Fig. 2d) is a diagonal **band**: two tanh units give a boundary made of two roughly parallel lines, impossible for a single logistic unit. So a hidden layer performs **learned basis expansion**: instead of hand-designing features, the features are trained.

**Objective:** $`\mathcal{L}(\boldsymbol\theta; \mathcal{D}) = \sum_{i=1}^N\ell(\mathbf{x}_i, y_i\mid\boldsymbol\theta)`$, so $`\partial\mathcal{L}/\partial\boldsymbol\theta = \sum_i\partial\ell_i/\partial\boldsymbol\theta`$: it suffices to differentiate the loss for **one** data point. Architecture (depth, widths, activations) is a hyperparameter set; proper model selection is expensive, so defaults are often used.

---

## 23. Backpropagation

### 23.1 Derivative conventions (Jacobians)
- For $`f : \mathbb{R}^n\to\mathbb{R}`$: $`\dfrac{\partial f}{\partial\mathbf{z}} = \big[\partial f/\partial z_1, \dots, \partial f/\partial z_n\big]`$, a **row** vector.
- For $`\mathbf{f} : \mathbb{R}^n\to\mathbb{R}^m`$: $`\dfrac{\partial\mathbf{f}}{\partial\mathbf{z}}`$ is the $`m\times n`$ **Jacobian**, entry $`(i,j) = \partial f_i/\partial z_j`$.
- For $`f : \mathbb{R}^{n\times m}\to\mathbb{R}`$: $`\dfrac{\partial f}{\partial\mathbf{W}}`$ is the $`n\times m`$ matrix of partials.
- **Chain rule:** if $`h = g\circ f`$ with $`\mathbf{z} = f(\mathbf{x})`$, then $`\dfrac{\partial h}{\partial\mathbf{x}} = \dfrac{\partial h}{\partial\mathbf{z}}\dfrac{\partial\mathbf{z}}{\partial\mathbf{x}}`$ (product of an $`m\times k`$ and a $`k\times n`$ Jacobian).

### 23.2 The toy model, worked
Suppose we already have the row vectors $`\partial\ell/\partial\mathbf{z}^2`$ and $`\partial\ell/\partial\mathbf{z}^3`$. Since $`z^2_i = w^2_{i1}x_1 + w^2_{i2}x_2 + b^2_i`$,
```math
\frac{\partial\ell}{\partial w^2_{ij}} = \frac{\partial\ell}{\partial z^2_i}\cdot\frac{\partial z^2_i}{\partial w^2_{ij}} = \frac{\partial\ell}{\partial z^2_i}\cdot x_j, \qquad \frac{\partial\ell}{\partial b^2_i} = \frac{\partial\ell}{\partial z^2_i}\cdot 1 .
```
In matrix form (outer product of the column $`\mathbf{x}`$ and the row $`\partial\ell/\partial\mathbf{z}^2`$, transposed so entry $`(i,j)`$ is $`\partial\ell/\partial w^2_{ij}`$):
```math
\frac{\partial\ell}{\partial\mathbf{W}^2} = \Big(\mathbf{x}\frac{\partial\ell}{\partial\mathbf{z}^2}\Big)^\top, \qquad \frac{\partial\ell}{\partial\mathbf{W}^3} = \Big(\mathbf{a}^2\frac{\partial\ell}{\partial\mathbf{z}^3}\Big)^\top, \qquad \frac{\partial\ell}{\partial\mathbf{b}^2} = \frac{\partial\ell}{\partial\mathbf{z}^2}, \qquad \frac{\partial\ell}{\partial\mathbf{b}^3} = \frac{\partial\ell}{\partial\mathbf{z}^3}.
```
So everything reduces to the $`\partial\ell/\partial\mathbf{z}^l`$, computed **backwards from the output**:
1. Cross-entropy $`\ell(\mathbf{a}^3, y) = -(y\log a^3_1 + (1-y)\log(1 - a^3_1))`$ gives $`\dfrac{\partial\ell}{\partial\mathbf{a}^3} = \Big[\dfrac{a^3_1 - y}{a^3_1(1-a^3_1)}\Big]`$.
2. $`a^3_1 = \sigma(z^3_1)`$ so $`\dfrac{\partial\mathbf{a}^3}{\partial\mathbf{z}^3} = [\sigma'(z^3_1)] = [a^3_1(1 - a^3_1)]`$, hence
```math
\frac{\partial\ell}{\partial\mathbf{z}^3} = \frac{\partial\ell}{\partial\mathbf{a}^3}\frac{\partial\mathbf{a}^3}{\partial\mathbf{z}^3} = [a^3_1 - y] \qquad (\text{the sigmoid's derivative cancels the denominator: "prediction minus target"}).
```
3. $`\mathbf{z}^3 = \mathbf{W}^3\mathbf{a}^2 + \mathbf{b}^3`$ so $`\dfrac{\partial\mathbf{z}^3}{\partial\mathbf{a}^2} = \mathbf{W}^3`$; and $`a^2_i = \tanh(z^2_i)`$ so $`\dfrac{\partial\mathbf{a}^2}{\partial\mathbf{z}^2} = \mathrm{diag}\big(1 - (a^2_1)^2,\ 1 - (a^2_2)^2\big)`$. Therefore
```math
\frac{\partial\ell}{\partial\mathbf{z}^2} = \frac{\partial\ell}{\partial\mathbf{z}^3}\frac{\partial\mathbf{z}^3}{\partial\mathbf{a}^2}\frac{\partial\mathbf{a}^2}{\partial\mathbf{z}^2} = [a^3_1 - y]\,\mathbf{W}^3\begin{bmatrix}1 - (a^2_1)^2 & 0\\ 0 & 1 - (a^2_2)^2\end{bmatrix}.
```

### 23.3 The general algorithm
> **Forward equations** (for $`l = 2, \dots, L`$, with $`\mathbf{a}^1 = \mathbf{x}`$):
> ```math
> \mathbf{z}^l = \mathbf{W}^l\mathbf{a}^{l-1} + \mathbf{b}^l \quad\text{(FE1)}, \qquad \mathbf{a}^l = f_l(\mathbf{z}^l) \quad\text{(FE2)}.
> ```
> **Backward equations:**
> ```math
> \frac{\partial\ell}{\partial\mathbf{z}^L} = \frac{\partial\ell}{\partial\mathbf{a}^L}\frac{\partial\mathbf{a}^L}{\partial\mathbf{z}^L} \quad\text{(BE1)}, \qquad
> \frac{\partial\ell}{\partial\mathbf{z}^l} = \frac{\partial\ell}{\partial\mathbf{z}^{l+1}}\,\mathbf{W}^{l+1}\,\frac{\partial\mathbf{a}^l}{\partial\mathbf{z}^l} \quad\text{(BE2)}, \qquad
> \frac{\partial\ell}{\partial\mathbf{W}^l} = \Big(\mathbf{a}^{l-1}\frac{\partial\ell}{\partial\mathbf{z}^l}\Big)^\top \quad\text{(BE3)}, \qquad
> \frac{\partial\ell}{\partial\mathbf{b}^l} = \frac{\partial\ell}{\partial\mathbf{z}^l} \quad\text{(BE4)}.
> ```
**Where BE2 comes from:** $`\dfrac{\partial\ell}{\partial\mathbf{z}^l} = \dfrac{\partial\ell}{\partial\mathbf{z}^{l+1}}\dfrac{\partial\mathbf{z}^{l+1}}{\partial\mathbf{a}^l}\dfrac{\partial\mathbf{a}^l}{\partial\mathbf{z}^l}`$ with $`\partial\mathbf{z}^{l+1}/\partial\mathbf{a}^l = \mathbf{W}^{l+1}`$ from FE1. The Jacobian $`\partial\mathbf{a}^l/\partial\mathbf{z}^l`$ is **diagonal** for unit-wise activations (don't build it explicitly; multiply element-wise), but a full matrix for softmax. BE3 is just $`\partial\ell/\partial W^l_{ij} = (\partial\ell/\partial z^l_i)\,a^{l-1}_j`$.

**What backprop really is:** the multivariate chain rule applied to a computation graph. There are many paths from a weight $`W^l_{ij}`$ to the loss; the chain rule sums over all of them. Backprop organises this by **storing and reusing** the intermediate quantities $`\partial\ell/\partial\mathbf{z}^l`$, trading extra memory (all $`\mathbf{z}^l, \mathbf{a}^l`$ must be kept from the forward pass) for a running time dominated by the matrix multiplications. In practice you process a **minibatch** at once (matrices → tensors with a batch dimension), sized to fit GPU memory; shuffle data and cycle through batches (SGD, Part 3).

---

## 24. Training neural networks: problems and tricks
Networks have huge numbers of parameters (≈2 million for MNIST problems on Sheet 4 vs 50,000 images; AlexNet 60 million vs 1.2 million images) and the objective is **non-convex**, so training "may feel more art than science".

### 24.1 Saturation
Toy: one sigmoid unit $`a = \sigma(z)`$, $`z = wx + b`$, learning NOT ($`x \in \{-1, 1\}`$, target $`y = (1-x)/2`$), with squared loss $`\ell = (a - y)^2`$, so $`\partial\ell/\partial z = 2(a - y)\,\sigma'(z)`$. Suppose the initialisation is such that for the input $`x = -1`$ (target $`y = 1`$) the pre-activation is $`z = -5`$. Then $`a \approx 0`$: the prediction is badly **wrong** and the loss $`(a - y)^2 \approx 1`$ is large. But $`z = -5`$ sits where **the sigmoid is flat**, so $`\sigma'(z) \approx 0`$ and the gradient $`\partial\ell/\partial z \approx 0`$: gradient steps barely move **even though the loss is large**. This is **saturation**. *(The notes write $`w = -5`$, $`b = 0`$ for this, which actually gives $`z = +5`$ at $`x = -1`$; a sign slip. The point is unaffected.)* Fix: with **cross-entropy** the derivative is $`\partial\ell/\partial z = a - y`$ (the $`\sigma'`$ cancels, §23.2), so a big error gives a big gradient.

### 24.2 Vanishing and exploding gradients
Chain the NOT network through three sigmoid layers. Expanding the chain rule:
```math
\frac{\partial\ell}{\partial z^2_1} = (a^4_1 - y)\cdot w^4_1\,\sigma'(z^3_1)\cdot w^3_1\,\sigma'(z^2_1).
```
(Cross-entropy at the output, so $`\partial\ell/\partial z^4_1 = a^4_1 - y`$ with no extra $`\sigma'`$ factor.) Each $`\sigma'(\cdot) \le \tfrac14`$ (max of $`ab`$ with $`a + b = 1`$), so the product of many such factors **shrinks towards 0 exponentially with depth**: the gradient **vanishes**. Large weights can instead make it **explode**. Either way, early layers learn slowly or unstably.

### 24.3 Tricks
| Problem | Remedy |
|---|---|
| Saturation / vanishing gradient | **ReLU** $`\max(0,z)`$: saturates only on one side, derivative exactly 1 on the other; cross-entropy loss |
| Bad initial state | Weights into a unit with $`D`$ inputs (its fan-in, $`n_{l-1}`$): $`\sim\mathcal{N}(0, \sigma^2)`$ with $`\sigma^2 = 1/D`$ (assuming inputs have $`\mathbb{E}[x_i^2] \approx 1`$), same for sigmoid and ReLU units; bias 0 or small for sigmoid, small **positive** for ReLU so units start un-saturated |
| Symmetric units | **Random** initialisation breaks symmetry; identical initial weights ⇒ identical units forever |
| Overfitting | $`\ell_1`$/$`\ell_2`$ penalties; **early stopping** (watch validation error after each step; stop when it plateaus; Hardt et al. 2016 give theory; but a plateau may just be a local minimum, try changing the learning rate); **more data** incl. **augmentation** (rotations/translations don't change the label; Google's translation app trained on synthetic letters; only works when such label-preserving transformations exist); **dropout**; weight constraints; gradient clipping; noise injection |

**Dropout** (Srivastava et al. 2014): at each gradient step, randomly "drop" a fraction (typically ½) of the hidden units and update only the surviving weights; at test time use the full network with weights **scaled** (halved for rate ½). Intuition: prevents **co-adaptation** of units. It's a form of **model averaging**, like **bagging** (Breiman 1996): draw $`k`$ datasets of size $`N`$ from $`\mathcal{D}`$ **with replacement** (so each has duplicates and omissions), train $`k`$ models, and average (regression) or take the plurality (classification). Averaging reduces **variance** ⇒ less overfitting.

> [!TIP]
> **✅ Key takeaways**
> 1. A hidden layer = **learned basis expansion**; the output layer is logistic/softmax/linear regression on learned features.
> 2. Backprop = chain rule with caching: forward pass stores $`\mathbf{z}^l, \mathbf{a}^l`$; backward pass computes $`\partial\ell/\partial\mathbf{z}^l`$ from layer $`L`$ down; weight gradients are outer products (BE3).
> 3. For sigmoid/softmax outputs with cross-entropy, $`\partial\ell/\partial\mathbf{z}^L = \mathbf{a}^L - \mathbf{y}`$.
> 4. Sigmoid saturates and $`\sigma' \le \tfrac14`$ ⇒ vanishing gradients in deep nets; ReLU + careful random init + cross-entropy help.
> 5. Overfitting controls: early stopping, data augmentation, dropout (≈ bagging ≈ variance reduction), penalties.

> [!WARNING]
> **⚠️ Common misconceptions & errors**
> - ❌ Treating $`w^l_{ij}`$'s superscript as a power.
> - ❌ "$`\partial\ell/\partial\mathbf{z}`$ is a column vector." ✅ The notes use **row** vectors for derivatives of scalars; Jacobians are $`m\times n`$; get the order right in the chain rule.
> - ❌ Forgetting the transpose in BE3 (entry $`(i,j)`$ must be $`\partial\ell/\partial W_{ij}`$).
> - ❌ "Backprop finds the global minimum." ✅ Non-convex; local minima and saddles.
> - ❌ Initialising all weights to zero (symmetry never breaks).
> - ❌ "Squared loss is fine for sigmoid outputs." ✅ It saturates; use cross-entropy.
> - ❌ Forgetting to rescale weights at test time after dropout.

---

## 25. Convolutional neural networks

> [!NOTE]
> **📖 Key definitions**
> - **Tensor:** a $`d`$-dimensional array of shape $`n_1\times\dots\times n_d`$. A grey image is an $`m\times n`$ tensor of pixel intensities in $`[0,1]`$; an RGB image is $`m\times n\times3`$ (3 **channels**).
> - **Convolutional filter (kernel):** a small tensor, e.g. $`W\times H`$ (or $`W\times H\times c`$ for $`c`$ input channels). Applied to a patch by **element-wise multiply and sum** (a dot product), plus a bias, then a non-linearity.
> - **Stride:** step size when sliding the filter (may differ per direction). **Padding:** zeros added around the border so edge pixels are covered.
> - Convnets are also used on audio (1-D convolutions), video (3-D) and text; the lecture treats images, i.e. 2-D convolutions.
> - **Weight tying / sharing:** the same filter weights are used at every position. Convnets reduce overfitting by exploiting the geometry.
> - **Max pooling:** replace each small window (e.g. $`2\times2`$, stride 2) by its **maximum** (Fig. 9). Average pooling is an alternative.

**Example:** $`100\times100`$ image, $`5\times5`$ filter, stride 2 ⇒ output $`48\times48`$ (positions $`(1,1), (1,3), \dots, (1,95)`$; the 100th column is never covered unless you pad).

**Hand-designed filters** (binary image, bias $`-1`$, ReLU): filter $`\begin{bmatrix}-1&1\\-1&1\end{bmatrix}`$ outputs 1 exactly on a black–white vertical edge; $`\begin{bmatrix}-1&1\\-1&-1\end{bmatrix}`$ with bias 0 detects a black "L". In practice filters are **learned by backprop**.

**Channels:** a layer with input $`m\times n\times c`$ and $`c'`$ filters of shape $`W\times H\times c`$ outputs $`m'\times n'\times c'`$; the number of output channels is the number of filters. Still a "2-D convolution" because the filter moves in two directions only.

**Forward equation** (layer $`l`$ output $`a^l_{i,j,f}`$ of shape $`m_l\times n_l\times F_l`$, $`F_{l+1}`$ filters of shape $`W_f\times H_f\times F_l`$, stride 1, no padding):
```math
z^{l+1}_{i',j',f'} = b^{l+1,f'} + \sum_{i=1}^{W_f}\sum_{j=1}^{H_f}\sum_{f=1}^{F_l}a^l_{i'+i-1,\ j'+j-1,\ f}\;w^{l+1,f'}_{i,j,f}.
```
**Backward equations** (given $`\partial\ell/\partial z^{l+1}_{i',j',f'}`$):
```math
\frac{\partial\ell}{\partial w^{l+1,f'}_{i,j,f}} = \sum_{i',j'}\frac{\partial\ell}{\partial z^{l+1}_{i',j',f'}}\,a^l_{i'+i-1,\,j'+j-1,\,f} \quad(\text{sum over every position the shared weight is used}),
```
```math
\frac{\partial\ell}{\partial a^l_{i,j,f}} = \sum_{i',j',f'}\frac{\partial\ell}{\partial z^{l+1}_{i',j',f'}}\,w^{l+1,f'}_{i-i'+1,\,j-j'+1,\,f}, \qquad \frac{\partial\ell}{\partial z^l_{i,j,f}} = f'(z^l_{i,j,f})\cdot\frac{\partial\ell}{\partial a^l_{i,j,f}}.
```
Same structure as the fully-connected case, except the weight gradient **sums over positions** because of sharing. Biases likewise: $`\partial\ell/\partial b^{l+1,f'} = \sum_{i',j'}\partial\ell/\partial z^{l+1}_{i',j',f'}`$.

**Pooling** (no parameters): $`\Omega(i',j')`$ = the input indices in the pool window, e.g. for $`2\times2`$ pools $`\Omega(1,1) = \{(1,1),(1,2),(2,1),(2,2)\}`$; Fig. 9: a $`2\times2`$, stride-2 max pool of $`\begin{bmatrix}4&2&8&2\\1&5&8&7\\8&4&3&7\\3&9&2&2\end{bmatrix}`$ gives $`\begin{bmatrix}5&8\\9&7\end{bmatrix}`$. Forward $`s^{l+1}_{i',j'} = \max_{(i,j)\in\Omega(i',j')}a^l_{i,j}`$; backward $`\partial s^{l+1}_{i',j'}/\partial a^l_{i,j} = \mathbb{1}\big((i,j) = \arg\max_{\Omega(i',j')}a^l\big)`$: the gradient flows **only to the max element**. Motivation: an edge detected at one position is also detected at neighbours; pooling removes this redundancy. A typical convnet: several conv (+pool) layers, then fully connected layers to the output.

---

## 26. Language models and recurrent neural networks

> [!NOTE]
> **📖 Key definitions**
> - **Language model:** a probability distribution over word sequences, $`\sum_{\mathbf{w}\in\Sigma^*}p(\mathbf{w}) = 1`$. Answers "how probable is this utterance?" (MT: $`p(\text{he likes apples}) > p(\text{apples likes he})`$; ASR: likes vs licks).
> - **Chain rule (autoregressive decomposition)** (not the only option: context-free-grammar models impose a hierarchical decomposition instead): $`p(w_1, \dots, w_N) = p(w_1)\,p(w_2\mid w_1)\,p(w_3\mid w_1, w_2)\cdots p(w_N\mid w_1, \dots, w_{N-1})`$: exact; models the next word given the **history**.
> - **Cross-entropy** of a model on a sequence: $`H(w_1^N) = -\frac1N\log_2 p(w_1^N)`$ = bits per token. **Perplexity** $`= 2^{H(w_1^N)}`$: how "surprised" the model is per word. Sensitive to tokenisation and vocabulary size.
> - **Markov assumption / n-gram:** only the last $`n-1`$ words matter. Bigram: $`p(w_1,\dots,w_n) \approx p(w_1)p(w_2\mid w_1)p(w_3\mid w_2)\cdots p(w_n\mid w_{n-1})`$. The conditioning context is the **history**.
> - **Recurrent neural network (RNN):** hidden state depends on the current input **and the previous hidden state**: $`\mathbf{h}_n = g(V[\mathbf{x}_n; \mathbf{h}_{n-1}] + c)`$, $`\widehat{\mathbf{y}}_n = W\mathbf{h}_n + b`$. No Markov assumption; $`\mathbf{h}_n`$ compresses the whole prefix.
> - **Unrolling:** viewing the RNN over a sequence as one very deep feed-forward network with an output at every layer and **shared weights**. **Backpropagation through time (BPTT):** backprop on the unrolled graph. **Truncated BPTT (TBPTT):** cut the sequence into chunks of length $`k`$; forward pass carries the hidden state across chunks (exact), backward pass stops at chunk boundaries (approximate).

**Why model $`p(\mathbf{w})`$ at all?** Bayes: to map input $`\mathbf{x}`$ to utterance $`\mathbf{w}`$, use $`p(\mathbf{w})p(\mathbf{x}\mid\mathbf{w})`$; monolingual text for $`p(\mathbf{w})`$ is far more plentiful than paired data for $`p(\mathbf{w}\mid\mathbf{x})`$. Many tasks reduce to continuation: MT, QA, dialogue ($`p_{\text{LM}}(\text{What do dogs love?} \mid\mid\mid \text{bones} \mid \beta)`$). With enough context, "There she built a" → *sandcastle* needs resolving *she* = Alice and *beach*: **long-range dependencies** (Turing and Good estimated word frequencies to crack Enigma).

**Count-based n-grams** (MLE: $`p(w_n\mid\text{history}) = \#(\text{history}, w_n)/\#(\text{history})`$; unigram ignores context, bigram uses the previous word, trigram the previous two) fail because most contexts are never seen (zero probabilities). **Neural n-gram** (trigram, Fig. 2b): inputs are one-hot vectors of the previous two words, $`\mathbf{h}_n = g(V[\mathbf{w}_{n-1}; \mathbf{w}_{n-2}] + c)`$, $`\widehat{\mathbf{p}}_n = \mathrm{softmax}(W\mathbf{h}_n + b)`$ over a vocabulary of size $`> 10^5`$. Trained by SGD + backprop on i.i.d. (context, next word) pairs, objective
```math
\mathcal{F} = -\frac1N\sum_n\mathrm{cost}_n(\mathbf{w}_n, \widehat{\mathbf{p}}_n), \qquad \mathrm{cost}(\mathbf{a}, \mathbf{b}) = \mathbf{a}^\top\log\mathbf{b} \ (\text{log-prob of the true word}),
```
```math
\frac{\partial\mathcal{F}}{\partial W} = -\frac1N\sum_n\frac{\partial\mathrm{cost}_n}{\partial\widehat{\mathbf{p}}_n}\frac{\partial\widehat{\mathbf{p}}_n}{\partial W}, \qquad \frac{\partial\mathcal{F}}{\partial V} = -\frac1N\sum_n\frac{\partial\mathrm{cost}_n}{\partial\widehat{\mathbf{p}}_n}\frac{\partial\widehat{\mathbf{p}}_n}{\partial\mathbf{h}_n}\frac{\partial\mathbf{h}_n}{\partial V}.
```
Each time step's gradient is independent ⇒ trivially parallel on GPUs. **Sampling:** feed $`n-1`$ start symbols `<s>`, sample a word, feed it back, repeat until `</s>` (the training text must contain these start/end symbols so the model learns to begin and finish utterances) (sample from a discrete distribution via a uniform $`z`$ and the cumulative probabilities).

**RNN vs n-gram:** RNNs can represent **unbounded** dependencies in $`\mathbf{h}_n`$ (also usable as a string encoding), but are hard to train, need large hidden layers for long inputs, and cost grows **quadratically** in hidden size. In BPTT the gradient at step $`n`$ depends on all **future** steps (Fig. 7):
```math
\frac{\partial\mathcal{F}}{\partial\mathbf{h}_2} = \frac{\partial\mathcal{F}}{\partial\text{cost}_2}\frac{\partial\text{cost}_2}{\partial\widehat{\mathbf{p}}_2}\frac{\partial\widehat{\mathbf{p}}_2}{\partial\mathbf{h}_2} + \frac{\partial\mathcal{F}}{\partial\text{cost}_3}\frac{\partial\text{cost}_3}{\partial\widehat{\mathbf{p}}_3}\frac{\partial\widehat{\mathbf{p}}_3}{\partial\mathbf{h}_3}\frac{\partial\mathbf{h}_3}{\partial\mathbf{h}_2} + \frac{\partial\mathcal{F}}{\partial\text{cost}_4}\frac{\partial\text{cost}_4}{\partial\widehat{\mathbf{p}}_4}\frac{\partial\widehat{\mathbf{p}}_4}{\partial\mathbf{h}_4}\frac{\partial\mathbf{h}_4}{\partial\mathbf{h}_3}\frac{\partial\mathbf{h}_3}{\partial\mathbf{h}_2}.
```
The unrolled RNN is a DAG, so ordinary backprop applies. BPTT costs linear in the longest sequence and a minibatch is an $`m\times N\times\lvert\mathbf{x}_i\rvert`$ tensor ($`N`$ = longest sequence) padded with a special symbol, wasting memory; TBPTT is constant in the truncation length, batches cleanly, and though approximate works well for *medium*-range dependencies.

### 26.1 Exploding and vanishing gradients in RNNs
Write $`\mathbf{h}_n = g(\mathbf{z}_n)`$ with $`\mathbf{z}_n = V_x\mathbf{x}_n + V_h\mathbf{h}_{n-1} + c`$. Then $`\dfrac{\partial\mathbf{h}_n}{\partial\mathbf{z}_n} = \mathrm{diag}(g'(\mathbf{z}_n))`$ and $`\dfrac{\partial\mathbf{z}_n}{\partial\mathbf{h}_{n-1}} = V_h`$, so
```math
\frac{\partial\text{cost}_N}{\partial\mathbf{h}_1} = \frac{\partial\text{cost}_N}{\partial\widehat{\mathbf{p}}_N}\frac{\partial\widehat{\mathbf{p}}_N}{\partial\mathbf{h}_N}\prod_{n\in\{N,\dots,2\}}\mathrm{diag}\big(g'(\mathbf{z}_n)\big)\,V_h .
```
The core is the **repeated multiplication by the same matrix $`V_h`$**. Raising a matrix to a power: largest eigenvalue $`= 1`$ ⇒ gradient propagates; $`> 1`$ ⇒ **explodes**; $`< 1`$ ⇒ **vanishes**. Usually the spectral radius is small, the gradient vanishes, and long-range dependencies aren't learned (the non-linearities $`g'`$ shrink it further). Remedies: second-order methods (don't scale), careful (orthogonal) initialisation of $`V_h`$, and above all **gated units** (LSTMs; beyond this course).

> [!TIP]
> **✅ Key takeaways**
> 1. Convnets = weight sharing + locality: a filter slides over the input, pooling summarises; same forward/backward logic with sums over positions. Far fewer parameters than fully connected layers on images.
> 2. Language models factorise $`p(\mathbf{w})`$ with the chain rule; n-grams truncate the history (Markov), RNNs carry it in a hidden state.
> 3. Train with cross-entropy (= MLE); evaluate with perplexity.
> 4. RNN training = backprop on the unrolled network (BPTT); truncate for efficiency.
> 5. Repeated multiplication by $`V_h`$ ⇒ gradients explode or vanish depending on its largest eigenvalue; gating fixes it.

> [!WARNING]
> **⚠️ Common misconceptions & errors**
> - ❌ "A 2-D convolution on an RGB image uses a 2-D filter." ✅ The filter is $`W\times H\times3`$; "2-D" refers to the directions it moves in.
> - ❌ "Number of output channels = number of input channels." ✅ = number of **filters**.
> - ❌ Forgetting that the weight gradient of a shared filter **sums over all positions**.
> - ❌ "Max-pool gradient spreads over the window." ✅ Only the argmax receives it.
> - ❌ "The chain-rule decomposition of $`p(\mathbf{w})`$ is an approximation." ✅ It's exact; the **Markov** assumption is the approximation.
> - ❌ "Lower perplexity is worse." ✅ Lower = less surprised = better (but compare only with the same tokenisation/vocabulary).
> - ❌ "TBPTT's forward pass is approximate." ✅ Forward is exact (state carried over); only the backward pass is truncated.
