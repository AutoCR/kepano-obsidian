---
categories:
  - "[[Evergreen]]"
title: "VAE and diffusion"
created: 2026-04-22
updated: 2026-04-22
tags:
  - 0🌲
  - generative-models
  - vae
  - diffusion
  - flow-matching
sources:
  - https://www.notion.so/VAE-Diffusion-210d25c2a3b880c29e80edb2baf302f3
  - https://zhuanlan.zhihu.com/p/345025351
  - https://zhuanlan.zhihu.com/p/346518942
  - https://zhuanlan.zhihu.com/p/345024301
  - https://zhuanlan.zhihu.com/p/348498294
related:
  - "[[Diffusion]]"
  - "[[Flow matching and diffusion models]]"
  - "[[Gaussian probability path]]"
  - "[[Adaptive Time Step Flow Matching 2602]]"
---

# VAE and diffusion

This note reorganizes the original Notion page into a cleaner learning path. The key idea is simple: **VAE and diffusion models are both generative models, but they approach generation from different directions**.

- **VAE** learns a latent-variable model and generates by decoding latent codes.
- **Diffusion** learns how to reverse a gradual corruption process.
- **Flow matching** provides a broader continuous-time view in which diffusion is one specific probability path.

## Executive summary

If I only keep four ideas in mind, they are these:

1. **Cross-entropy, KL, MLE, and MAP are the language of probabilistic learning.**
2. **VAE explains generation through latent variables and variational inference.**
3. **Diffusion explains generation through iterative denoising.**
4. **Flow matching unifies many diffusion-style models by directly learning the time-dependent transport field.**

## 1. Probability basics that appear everywhere

### 1.1 Information content
For an event with probability \(p(x)\), the information content is

$$
I(x) = -\log p(x).
$$

Rare events carry more information because they are more surprising.

### 1.2 Entropy
Entropy is the expected surprise under the true distribution:

$$
H(p) = \mathbb{E}_{x \sim p}[-\log p(x)] = -\sum_i p(x_i) \log p(x_i).
$$

It measures the intrinsic uncertainty of the distribution \(p\).

### 1.3 Cross-entropy
Cross-entropy measures how expensive it is to encode samples from \(p\) using a model \(q\):

$$
H(p, q) = \mathbb{E}_{x \sim p}[-\log q(x)].
$$

In supervised learning, \(p\) is usually fixed by the dataset, so minimizing cross-entropy is equivalent to making the model distribution \(q\) close to the data distribution.

### 1.4 KL divergence
KL divergence measures the gap between two distributions:

$$
D_{KL}(p \Vert q) = H(p, q) - H(p) = \sum_i p(x_i) \log \frac{p(x_i)}{q(x_i)} \ge 0.
$$

Important intuition:
- In **discriminative learning**, cross-entropy is convenient because \(H(p)\) is constant.
- In **generative modeling**, KL often appears explicitly because we care about matching entire distributions.

## 2. MLE and MAP

### 2.1 Maximum likelihood estimation
MLE chooses parameters that maximize the likelihood of observed data:

$$
\theta_{\text{MLE}} = \arg\max_\theta p(X \mid \theta)
= \arg\min_\theta \Big(-\sum_i \log p(x_i \mid \theta)\Big).
$$

This is the standard data-fitting objective.

### 2.2 Maximum a posteriori estimation
MAP adds a prior over parameters:

$$
\theta_{\text{MAP}} = \arg\max_\theta p(\theta \mid X)
= \arg\min_\theta \Big(-\log p(\theta) - \sum_i \log p(x_i \mid \theta)\Big).
$$

So MAP can be understood as **MLE plus regularization**.

### 2.3 Probability vs likelihood
A useful vocabulary distinction:
- \(p(X \mid \theta)\) is a **probability distribution over data** when \(\theta\) is fixed.
- The same expression is called a **likelihood function of parameters** when \(X\) is fixed and \(\theta\) varies.

## 3. Where VAE fits

A variational autoencoder models data through a latent variable \(z\):

$$
p_\theta(x) = \int p_\theta(x \mid z) p(z) \, dz.
$$

The main difficulty is that the posterior \(p_\theta(z \mid x)\) is usually intractable, so VAE introduces an approximate posterior \(q_\phi(z \mid x)\).

The big picture is:
- the **encoder** approximates the posterior over latent variables,
- the **decoder** reconstructs samples from latent codes,
- training balances **reconstruction quality** and **latent regularization**.

VAE is therefore a natural bridge from basic probabilistic learning to modern deep generative modeling.

## 4. Where diffusion fits

Diffusion models start from a different viewpoint.

Instead of introducing a latent code directly, they define a **forward noising process** that gradually destroys structure:

$$
x_0 \rightarrow x_1 \rightarrow \cdots \rightarrow x_T.
$$

The model then learns the **reverse process** that turns noisy samples back into clean data. In practice, the network often predicts one of:
- the noise \(\epsilon\),
- the clean sample \(x_0\),
- the score \(\nabla_x \log p_t(x)\), or
- a velocity-like target \(v\).

These parameterizations look different, but they are closely related.

See [[Diffusion]] for the detailed derivations.

## 5. How flow matching reframes diffusion

Flow matching provides a broader continuous-time perspective.

Instead of saying “learn to denoise step by step,” it says:

> choose a probability path from noise to data, then learn the time-dependent vector field that transports samples along that path.

From this perspective:
- diffusion corresponds to one family of stochastic paths,
- score matching learns the score field needed by the reverse stochastic dynamics,
- flow matching learns the transport field directly.

See [[Flow matching and diffusion models]] and [[Gaussian probability path]].

## 6. A practical reading path

If I want to understand the topic in the most stable order, I would read it like this:

1. This note for the probabilistic background.
2. [[Diffusion]] for DDPM, DDIM, Langevin, SDE, ODE, and CFG.
3. [[Flow matching and diffusion models]] for the unifying continuous-time view.
4. [[Gaussian probability path]] for the common path construction used in diffusion and flow-matching formulations.

## 7. Bottom line

VAE and diffusion are not competing buzzwords; they are two different answers to the same question:

> **How do we learn a distribution well enough to generate realistic samples?**

VAE answers with **latent-variable inference**. Diffusion answers with **progressive denoising**. Flow matching shows that many diffusion-style methods can be understood more generally as **learning how probability mass moves over time**.
