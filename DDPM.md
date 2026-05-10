---
categories:
  - "[[Evergreen]]"
title: "DDPM"
created: 2026-05-10
updated: 2026-05-10
tags:
  - 0🌲
  - generative-models
  - diffusion
  - ddpm
  - ddim
sources:
  - https://arxiv.org/abs/2006.11239
  - https://arxiv.org/abs/2010.02502
related:
  - "[[Flow matching and diffusion models]]"
  - "[[Diffusion]]"
  - "[[VAE and Diffusion]]"
---

# DDPM

DDPM, **Denoising Diffusion Probabilistic Model**, is a latent-variable generative model that learns to reverse a gradual Gaussian noising process. The big picture is:

> Start from real data, add noise until it becomes almost standard Gaussian; then train a neural network to reverse that process step by step.

This note lists the main equations first, then gives the conceptual picture, DDPM-vs-DDIM comparison, and detailed derivations in the appendices.

## Main equations

### 1. Forward noising process

One-step noising:

$$
q(x_t \mid x_{t-1}) = \mathcal N\left(x_t; \sqrt{\alpha_t}x_{t-1}, \beta_t I\right), \qquad \alpha_t = 1 - \beta_t.
$$

Closed-form noising from clean data:

$$
q(x_t \mid x_0) = \mathcal N\left(x_t; \sqrt{\bar\alpha_t}x_0, (1-\bar\alpha_t)I\right),
$$

where

$$
\bar\alpha_t = \prod_{s=1}^t \alpha_s.
$$

Equivalent sampling form:

$$
x_t = \sqrt{\bar\alpha_t}x_0 + \sqrt{1-\bar\alpha_t}\epsilon, \qquad \epsilon \sim \mathcal N(0,I).
$$

### 2. Reverse generative model

The reverse joint distribution is:

$$
p_\theta(x_{0:T}) = p(x_T)\prod_{t=1}^T p_\theta(x_{t-1}\mid x_t),
$$

with the prior

$$
p(x_T)=\mathcal N(0,I).
$$

The learned reverse transition is Gaussian:

$$
p_\theta(x_{t-1}\mid x_t)=\mathcal N\left(x_{t-1};\mu_\theta(x_t,t),\Sigma_\theta(x_t,t)\right).
$$

### 3. True reverse posterior

The true posterior for one reverse step is:

$$
q(x_{t-1}\mid x_t,x_0)=\mathcal N\left(x_{t-1};\tilde\mu_t(x_t,x_0),\tilde\beta_t I\right),
$$

where

$$
\tilde\mu_t(x_t,x_0)=
\frac{\sqrt{\bar\alpha_{t-1}}\beta_t}{1-\bar\alpha_t}x_0
+
\frac{\sqrt{\alpha_t}(1-\bar\alpha_{t-1})}{1-\bar\alpha_t}x_t,
$$

and

$$
\tilde\beta_t = \frac{1-\bar\alpha_{t-1}}{1-\bar\alpha_t}\beta_t.
$$

### 4. ELBO / variational loss

The DDPM negative ELBO is:

$$
L_{\text{vlb}}
= \mathbb E_q\left[
D_{\mathrm{KL}}(q(x_T\mid x_0)\Vert p(x_T))
+ \sum_{t=2}^{T}D_{\mathrm{KL}}\left(q(x_{t-1}\mid x_t,x_0)\Vert p_\theta(x_{t-1}\mid x_t)\right)
- \log p_\theta(x_0\mid x_1)
\right].
$$

### 5. Noise-prediction parameterization

Predict clean data from predicted noise:

$$
\hat x_0(x_t,t)=\frac{x_t-\sqrt{1-\bar\alpha_t}\epsilon_\theta(x_t,t)}{\sqrt{\bar\alpha_t}}.
$$

Parameterize the reverse mean by predicted noise:

$$
\mu_\theta(x_t,t)
=
\frac{1}{\sqrt{\alpha_t}}\left(
x_t - \frac{\beta_t}{\sqrt{1-\bar\alpha_t}}\epsilon_\theta(x_t,t)
\right).
$$

### 6. Simplified DDPM training loss

The practical DDPM objective is:

$$
L_{\text{simple}}
=
\mathbb E_{x_0,t,\epsilon}\left[
\left\|\epsilon-\epsilon_\theta(x_t,t)\right\|^2
\right],
$$

where

$$
x_t=\sqrt{\bar\alpha_t}x_0+\sqrt{1-\bar\alpha_t}\epsilon,
\qquad
\epsilon\sim\mathcal N(0,I),
\qquad
t\sim \mathrm{Uniform}\{1,\ldots,T\}.
$$

### 7. DDIM sampling update

Given a chosen previous timestep \(s<t\), DDIM uses:

$$
x_s = \sqrt{\bar\alpha_s}\hat x_0
+ \sqrt{1-\bar\alpha_s-\sigma_t^2}\epsilon_\theta(x_t,t)
+ \sigma_t z,
\qquad z\sim\mathcal N(0,I).
$$

For deterministic DDIM, \(\eta=0\), so \(\sigma_t=0\):

$$
x_s = \sqrt{\bar\alpha_s}\hat x_0
+ \sqrt{1-\bar\alpha_s}\epsilon_\theta(x_t,t).
$$

## Big picture

DDPM has two processes:

1. **Forward process** \(q\): fixed, not learned. It slowly corrupts data into Gaussian noise.
2. **Reverse process** \(p_\theta\): learned. It starts from Gaussian noise and denoises back to data.

The forward process is designed so that after many small noising steps,

$$
q(x_T\mid x_0) \approx \mathcal N(0,I).
$$

Therefore generation can start from

$$
x_T\sim \mathcal N(0,I),
$$

then repeatedly sample

$$
x_{t-1}\sim p_\theta(x_{t-1}\mid x_t).
$$

The most important training insight is that the hard variational objective reduces to a simple supervised regression problem:

> Given noisy \(x_t\) and timestep \(t\), predict the exact Gaussian noise \(\epsilon\) that was added to \(x_0\).

## Why the loss becomes noise prediction

The ELBO contains KL terms of the form:

$$
D_{\mathrm{KL}}\left(q(x_{t-1}\mid x_t,x_0)\Vert p_\theta(x_{t-1}\mid x_t)\right).
$$

Both distributions are Gaussian. If the variance is fixed, the trainable part of the KL is just mean matching:

$$
\left\|\tilde\mu_t(x_t,x_0)-\mu_\theta(x_t,t)\right\|^2.
$$

The true posterior mean can be written using the true noise \(\epsilon\), while the model mean is written using predicted noise \(\epsilon_\theta\). Therefore mean matching becomes noise matching:

$$
\left\|\epsilon-\epsilon_\theta(x_t,t)\right\|^2.
$$

This is why DDPM training is simple even though the underlying model is a variational latent-variable model.

## DDPM vs DDIM

DDIM, **Denoising Diffusion Implicit Model**, usually uses the same trained noise predictor \(\epsilon_\theta(x_t,t)\) as DDPM, but changes the sampling trajectory.

| Aspect | DDPM | DDIM |
|---|---|---|
| Training objective | Usually same noise-prediction loss | Usually reuses DDPM-trained model |
| Reverse process | Markovian | Non-Markovian |
| Sampling step | Usually \(t\to t-1\) | Can jump \(t\to s\), where \(s<t\) |
| Randomness | Stochastic ancestral sampling | Can be deterministic if \(\eta=0\) |
| Speed | Often many steps | Often fewer steps |
| Same initial noise gives same output? | Usually no | Yes when \(\eta=0\) |
| Main idea | Learn reverse Gaussian transitions | Predict \(x_0\) and jump along a chosen schedule |

### How DDIM skips steps

From the predicted noise, estimate clean data:

$$
\hat x_0=\frac{x_t-\sqrt{1-\bar\alpha_t}\epsilon_\theta(x_t,t)}{\sqrt{\bar\alpha_t}}.
$$

Then choose any earlier timestep \(s<t\) and construct \(x_s\) directly:

$$
x_s \approx \sqrt{\bar\alpha_s}\hat x_0 + \sqrt{1-\bar\alpha_s}\epsilon_\theta(x_t,t).
$$

This lets DDIM skip from, for example,

$$
1000\to 900\to 800\to \cdots \to 0
$$

instead of

$$
1000\to 999\to 998\to \cdots \to 0.
$$

### Is DDIM equal to DDPM if it does not skip?

Not necessarily. Using all timesteps only makes the timestep schedule the same.

- **Deterministic DDIM** with \(\eta=0\) is still different from DDPM because it does not inject fresh Gaussian noise at every step.
- **Stochastic DDIM** with \(\eta=1\), all timesteps, and posterior variance \(\tilde\beta_t\) can match DDPM-style ancestral sampling.

So:

$$
\text{same timestep schedule} \ne \text{same sampler}.
$$

To match DDPM, DDIM must also match the reverse variance and stochastic noise injection.

## Mental model

DDPM:

```text
learn to reverse every small noising step
x_T -> x_{T-1} -> ... -> x_0
```

DDIM:

```text
at each step, predict x_0, then jump to a chosen earlier noise level
x_T -> x_s -> ... -> x_0
```

The same neural network can be used in both because it predicts the noise at arbitrary timestep \(t\).

---

# Appendix A: ELBO to KL terms

Start with the marginal likelihood:

$$
\log p_\theta(x_0)=\log\int p_\theta(x_{0:T})dx_{1:T}.
$$

Insert the forward process:

$$
\log p_\theta(x_0)
=\log\mathbb E_{q(x_{1:T}\mid x_0)}\left[\frac{p_\theta(x_{0:T})}{q(x_{1:T}\mid x_0)}\right].
$$

By Jensen's inequality:

$$
\log p_\theta(x_0)
\ge
\mathbb E_q\left[\log p_\theta(x_{0:T})-\log q(x_{1:T}\mid x_0)\right].
$$

So the ELBO is:

$$
\mathrm{ELBO}=\mathbb E_q\left[\log p_\theta(x_{0:T})-\log q(x_{1:T}\mid x_0)\right].
$$

Expand the reverse joint:

$$
\log p_\theta(x_{0:T})=
\log p(x_T)+\sum_{t=1}^T\log p_\theta(x_{t-1}\mid x_t).
$$

Expand the forward chain:

$$
\log q(x_{1:T}\mid x_0)=\sum_{t=1}^T\log q(x_t\mid x_{t-1}).
$$

Use Bayes' rule:

$$
q(x_{t-1}\mid x_t,x_0)=
\frac{q(x_t\mid x_{t-1})q(x_{t-1}\mid x_0)}{q(x_t\mid x_0)}.
$$

Therefore:

$$
\log q(x_t\mid x_{t-1})
=
\log q(x_{t-1}\mid x_t,x_0)
+
\log q(x_t\mid x_0)
-
\log q(x_{t-1}\mid x_0).
$$

Summing over \(t=2,\ldots,T\) telescopes and gives:

$$
\sum_{t=1}^T\log q(x_t\mid x_{t-1})
=
\sum_{t=2}^T\log q(x_{t-1}\mid x_t,x_0)+\log q(x_T\mid x_0).
$$

Substitute back:

$$
\mathrm{ELBO}
=
\mathbb E_q\left[
\log p_\theta(x_0\mid x_1)
+
\log p(x_T)-\log q(x_T\mid x_0)
+
\sum_{t=2}^T\left(\log p_\theta(x_{t-1}\mid x_t)-\log q(x_{t-1}\mid x_t,x_0)\right)
\right].
$$

Recognize

$$
D_{\mathrm{KL}}(q\Vert p)=\mathbb E_q[\log q-\log p].
$$

Thus:

$$
\mathrm{ELBO}
=
\mathbb E_q\left[
\log p_\theta(x_0\mid x_1)
-D_{\mathrm{KL}}(q(x_T\mid x_0)\Vert p(x_T))
-
\sum_{t=2}^TD_{\mathrm{KL}}(q(x_{t-1}\mid x_t,x_0)\Vert p_\theta(x_{t-1}\mid x_t))
\right].
$$

Taking the negative gives \(L_{\text{vlb}}\).

# Appendix B: KL divergence between two Gaussians

Let

$$
q(x)=\mathcal N(\mu_q,\Sigma_q),\qquad p(x)=\mathcal N(\mu_p,\Sigma_p).
$$

The KL definition is:

$$
D_{\mathrm{KL}}(q\Vert p)=\mathbb E_q[\log q(x)-\log p(x)].
$$

The multivariate Gaussian log density is:

$$
\log \mathcal N(x;\mu,\Sigma)
=-\frac d2\log(2\pi)-\frac12\log\det\Sigma-
\frac12(x-\mu)^T\Sigma^{-1}(x-\mu).
$$

Substituting and taking expectations under \(q\) gives:

$$
D_{\mathrm{KL}}(q\Vert p)=
\frac12\left[
\log\frac{\det\Sigma_p}{\det\Sigma_q}
-d
+\mathrm{Tr}(\Sigma_p^{-1}\Sigma_q)
+(\mu_p-\mu_q)^T\Sigma_p^{-1}(\mu_p-\mu_q)
\right].
$$

For isotropic Gaussians

$$
q=\mathcal N(\mu_q,\sigma_q^2I),\qquad p=\mathcal N(\mu_p,\sigma_p^2I),
$$

this becomes:

$$
D_{\mathrm{KL}}(q\Vert p)=
\frac12\left[
d\log\frac{\sigma_p^2}{\sigma_q^2}
+d\frac{\sigma_q^2}{\sigma_p^2}
+\frac{\|\mu_q-\mu_p\|^2}{\sigma_p^2}
-d
\right].
$$

In DDPM, if variances are fixed, only the mean term depends on the network.

# Appendix C: Deriving the true posterior mean

We want:

$$
q(x_{t-1}\mid x_t,x_0).
$$

By Bayes' rule and the Markov property:

$$
q(x_{t-1}\mid x_t,x_0)\propto q(x_t\mid x_{t-1})q(x_{t-1}\mid x_0).
$$

Let \(z=x_{t-1}\). Then:

$$
q(x_t\mid z)=\mathcal N(x_t;\sqrt{\alpha_t}z,\beta_tI),
$$

and

$$
q(z\mid x_0)=\mathcal N(z;\sqrt{\bar\alpha_{t-1}}x_0,(1-\bar\alpha_{t-1})I).
$$

Ignoring constants independent of \(z\):

$$
\log q(z\mid x_t,x_0)
= -\frac{1}{2\beta_t}\|x_t-\sqrt{\alpha_t}z\|^2
-\frac{1}{2(1-\bar\alpha_{t-1})}\|z-\sqrt{\bar\alpha_{t-1}}x_0\|^2+C.
$$

Collect terms in \(z\):

$$
\log q(z\mid x_t,x_0)=
C-\frac12Az^Tz+b^Tz,
$$

where

$$
A=\frac{\alpha_t}{\beta_t}+\frac{1}{1-\bar\alpha_{t-1}},
$$

and

$$
b=\frac{\sqrt{\alpha_t}}{\beta_t}x_t+
\frac{\sqrt{\bar\alpha_{t-1}}}{1-\bar\alpha_{t-1}}x_0.
$$

Complete the square. The variance is \(A^{-1}I\), and the mean is \(A^{-1}b\).

Simplify \(A\):

$$
A=\frac{1-\bar\alpha_t}{\beta_t(1-\bar\alpha_{t-1})}.
$$

So:

$$
\tilde\beta_t=A^{-1}=\frac{\beta_t(1-\bar\alpha_{t-1})}{1-\bar\alpha_t}.
$$

The mean is:

$$
\tilde\mu_t=A^{-1}b
=
\frac{\sqrt{\bar\alpha_{t-1}}\beta_t}{1-\bar\alpha_t}x_0
+
\frac{\sqrt{\alpha_t}(1-\bar\alpha_{t-1})}{1-\bar\alpha_t}x_t.
$$

# Appendix D: From KL to the DDPM noise loss

The trainable KL term is:

$$
D_{\mathrm{KL}}\left(q(x_{t-1}\mid x_t,x_0)\Vert p_\theta(x_{t-1}\mid x_t)\right).
$$

Assume:

$$
q=\mathcal N(\tilde\mu_t,\tilde\beta_tI),\qquad p_\theta=\mathcal N(\mu_\theta,\sigma_t^2I).
$$

The network-dependent part is:

$$
\frac{1}{2\sigma_t^2}\|\tilde\mu_t-\mu_\theta\|^2.
$$

The true posterior mean can be written as:

$$
\tilde\mu_t(x_t,x_0)=
\frac{1}{\sqrt{\alpha_t}}\left(x_t-\frac{\beta_t}{\sqrt{1-\bar\alpha_t}}\epsilon\right).
$$

The learned mean is parameterized as:

$$
\mu_\theta(x_t,t)=
\frac{1}{\sqrt{\alpha_t}}\left(x_t-\frac{\beta_t}{\sqrt{1-\bar\alpha_t}}\epsilon_\theta(x_t,t)\right).
$$

Subtract:

$$
\tilde\mu_t-\mu_\theta
=
\frac{\beta_t}{\sqrt{\alpha_t(1-\bar\alpha_t)}}\left(\epsilon_\theta(x_t,t)-\epsilon\right).
$$

Therefore:

$$
\|\tilde\mu_t-\mu_\theta\|^2
=
\frac{\beta_t^2}{\alpha_t(1-\bar\alpha_t)}
\|\epsilon-\epsilon_\theta(x_t,t)\|^2.
$$

Substitute into the KL term:

$$
L_{t-1}
=
\frac{\beta_t^2}{2\sigma_t^2\alpha_t(1-\bar\alpha_t)}
\|\epsilon-\epsilon_\theta(x_t,t)\|^2.
$$

So the exact KL objective gives a weighted noise-prediction loss. DDPM's simplified objective drops the timestep weight:

$$
L_{\text{simple}}
=
\mathbb E_{x_0,t,\epsilon}\left[\|\epsilon-\epsilon_\theta(x_t,t)\|^2\right].
$$

# Related notes

- [[Flow matching and diffusion models]]
- [[Diffusion]]
- [[VAE and Diffusion]]
- [[score-based e2e autonomous driving review]]
