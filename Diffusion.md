---
categories:
  - "[[Evergreen]]"
title: "Diffusion"
created: 2026-04-22
updated: 2026-04-22
tags:
  - 0🌲
  - generative-models
  - diffusion
  - score-based
  - flow-matching
sources:
  - https://www.notion.so/Diffusion-23ed25c2a3b8803e8b2bfeeeeaf8a890
related:
  - "[[VAE and Diffusion]]"
  - "[[Flow matching and diffusion models]]"
  - "[[Gaussian probability path]]"
  - "[[DiffusionDrive 2411]]"
  - "[[Adaptive Time Step Flow Matching 2602]]"
---

# Diffusion

This note rewrites the original page into a more coherent story. The main idea is:

> **A diffusion model learns how to reverse a gradual corruption process.**

The details can be presented in discrete time, continuous time, stochastic form, or deterministic form. The names DDPM, DDIM, SDE, ODE, score matching, and classifier-free guidance are best understood as pieces of the same family rather than isolated tricks.

## 1. DDPM: the basic denoising picture

### 1.1 Forward process
The forward process gradually adds Gaussian noise:

$$
q(x_t \mid x_{t-1}) = \mathcal N(x_t; \alpha_t x_{t-1}, \beta_t^2 I),
$$

where $(\alpha_t^2 + \beta_t^2 = 1)$. If we define

$$
\bar\alpha_t = \prod_{s=1}^t \alpha_s, \qquad \bar\beta_t = \sqrt{1 - \bar\alpha_t^2},
$$

then we can jump directly from \(x_0\) to any noisy step \(x_t\):

$$
x_t = \bar\alpha_t x_0 + \bar\beta_t \epsilon, \qquad \epsilon \sim \mathcal N(0, I),
$$

so

$$
q(x_t \mid x_0) = \mathcal N(x_t; \bar\alpha_t x_0, \bar\beta_t^2 I).
$$

This closed form is what makes diffusion training practical.

### 1.2 Reverse process
Sampling needs the reverse transition

$$
p_\theta(x_{t-1} \mid x_t),
$$

which should remove a little noise at each step. The exact reverse distribution depends on the data distribution, so we approximate it with a neural network.

### 1.3 Why predict noise?
A common parameterization is to predict the injected noise \(\epsilon\) rather than predicting \(x_0\) directly.

From

$$
x_t = \bar\alpha_t x_0 + \bar\beta_t \epsilon,
$$

we can recover an estimate of the clean sample:

$$
\hat x_0 = \frac{x_t - \bar\beta_t \epsilon_\theta(x_t, t)}{\bar\alpha_t}.
$$

This turns the learning problem into a simple regression objective:

$$
\mathcal L_{\text{simple}} = \mathbb E\big[\lVert \epsilon - \epsilon_\theta(x_t, t) \rVert^2\big].
$$

This is the most common DDPM training view.

## 2. DDIM: same model, different sampler

DDIM keeps the same training idea but changes the reverse dynamics.

### 2.1 Core intuition
Instead of insisting on the original stochastic Markov chain, DDIM constructs a **non-Markovian reverse process** whose marginals remain compatible with the trained model.

The practical consequence is important:
- DDPM typically uses many small stochastic denoising steps.
- DDIM can take fewer, larger, often deterministic steps.

### 2.2 Deterministic sampling
When the DDIM noise term is set to zero, the update becomes deterministic. That means:
- repeated sampling from the same latent state gives the same trajectory,
- we can skip many timesteps,
- inference becomes much faster.

This is why DDIM is often interpreted as an **ODE-style sampler**.

### 2.3 Practical interpretation
A useful mental model is:
1. use the network to estimate the clean signal \(\hat x_0\),
2. re-noise that estimate to an earlier timestep,
3. optionally inject stochasticity.

DDPM corresponds to the fully stochastic nearby-step case. DDIM generalizes this so we can jump across timesteps.

## 3. Langevin sampling and score-based intuition

Before diffusion became dominant, score-based methods were often explained through Langevin dynamics:

$$
x_{k+1} = x_k + \frac{\eta}{2} \nabla_x \log p(x_k) + \sqrt{\eta} z_k,
$$

where \(z_k \sim \mathcal N(0, I)\).

The meaning is simple:
- the **score** \(\nabla_x \log p(x)\) points toward regions of higher probability density,
- the noise term keeps sampling stochastic,
- iterating this update moves samples toward the target distribution.

This is conceptually important because modern diffusion models can also be understood as learning time-dependent score functions.

## 4. SDE view: diffusion in continuous time

The discrete forward process can be generalized into a stochastic differential equation:

$$
dx = f_t(x) \, dt + g_t \, dw.
$$

Here:
- \(f_t(x)\) is the drift,
- \(g_t\) controls noise strength,
- \(dw\) is Brownian motion.

The reverse-time SDE becomes

$$
dx = \big(f_t(x) - g_t^2 \nabla_x \log p_t(x)\big) dt + g_t \, d\bar w,
$$

so generation requires the time-dependent score \(\nabla_x \log p_t(x)\).

This gives the central score-matching idea:

> train a neural network \(s_\theta(x_t, t)\) to approximate the score of the noisy data distribution.

That is why diffusion models, score-based models, and denoising models are so tightly connected.

## 5. ODE view: deterministic counterpart

The probability-flow ODE removes the stochastic term while preserving the same marginal distributions:

$$
dx = \Big(f_t(x) - \frac{1}{2} g_t^2 \nabla_x \log p_t(x)\Big) dt.
$$

This matters because:
- it gives a deterministic path from noise to data,
- it can be solved with standard ODE solvers,
- it explains why DDIM-style deterministic sampling works.

So a good summary is:
- **reverse SDE** = stochastic sampler,
- **probability-flow ODE** = deterministic counterpart.

## 6. Classifier-free guidance (CFG)

CFG is a practical way to strengthen conditioning without training a separate classifier.

The model is trained to handle both:
- a conditional input \(y\), and
- a null or dropped condition.

At inference time, we combine the two outputs:

$$
s_{\text{guided}} = s_{\text{uncond}} + w\big(s_{\text{cond}} - s_{\text{uncond}}\big),
$$

where \(w\) is the guidance strength.

Interpretation:
- \(w = 1\): standard conditional generation,
- \(w > 1\): stronger condition following,
- \(w = 0\): unconditional generation.

The same trick works whether the model predicts score, noise, \(x_0\), or \(v\).

## 7. The important unification

A lot of confusion disappears if I keep this table in mind:

| Concept | What is learned? | What is changed? |
|---|---|---|
| DDPM | usually noise \(\epsilon\) | stochastic discrete sampler |
| DDIM | same trained model | faster / often deterministic sampler |
| Score matching | score \(\nabla_x \log p_t(x)\) | training interpretation |
| Reverse SDE | score field | stochastic continuous-time sampler |
| Probability-flow ODE | score field | deterministic continuous-time sampler |
| CFG | conditional difference | stronger condition control |

So DDPM, DDIM, SDE, and ODE are not rival theories. They are different views or solvers built around the same denoising/score-learning core.

## 8. What I should remember in practice

1. **Training target and sampler are different design choices.** A model trained with one target can often be sampled in several ways.
2. **Noise prediction is popular because it is stable and simple.**
3. **DDIM is mainly about faster inference, not a totally different learning principle.**
4. **SDE and ODE views explain the theory behind diffusion and connect it to flow matching.**
5. **CFG is a guidance trick, not a separate generative family.**

## 9. Bottom line

A diffusion model learns a denoising rule that can be interpreted as noise prediction, score estimation, or continuous-time transport. The practical variants mainly differ in **how the reverse path is solved**: with more randomness, less randomness, more steps, or fewer steps.

For the broader bridge to continuous-time transport, see [[Flow matching and diffusion models]].
