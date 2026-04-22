---
categories:
  - "[[Evergreen]]"
title: "Flow matching and diffusion models"
created: 2026-04-22
updated: 2026-04-22
tags:
  - 0🌲
  - generative-models
  - diffusion
  - flow-matching
  - score-based
sources:
  - https://www.notion.so/An-introduction-to-flow-mathcing-and-diffusion-models-278d25c2a3b8803db102d8c9df2907b8
related:
  - "[[VAE and Diffusion]]"
  - "[[Diffusion]]"
  - "[[Gaussian probability path]]"
  - "[[Adaptive Time Step Flow Matching 2602]]"
  - "[[GoalFlow 2503]]"
---

# Flow matching and diffusion models

This note rewrites the original Notion page into a cleaner conceptual bridge.

The most useful perspective is:

> **Diffusion models and flow-matching models both describe how to move probability mass from a simple distribution to the data distribution.**

The main difference is what the model learns directly.

- **Diffusion / score-based view:** learn the score field needed to reverse a noisy stochastic process.
- **Flow-matching view:** learn the velocity field that transports samples along a chosen probability path.

## 1. Starting point: a probability path

We choose a family of intermediate distributions \(p_t(x)\) for \(t \in [0,1]\):

- \(p_0\) is a simple base distribution, often Gaussian,
- \(p_1\) is the data distribution.

This family is called a **probability path**.

A generative model then needs a rule for moving samples along that path.

## 2. Flow view: learn the vector field

In the flow-matching formulation, samples evolve according to an ODE:

$$
\frac{d x_t}{dt} = u_t(x_t),
$$

where \(u_t(x)\) is a time-dependent vector field.

If I knew the true marginal vector field \(u_t^{\text{target}}(x)\), I could transport noise into data by integrating this ODE.

So the learning problem becomes:

> train a network to approximate the vector field that moves particles along the desired probability path.

## 3. Conditional vs marginal vector fields

The original note emphasizes an important practical point.

The object we ultimately want is the **marginal vector field** over \(p_t(x)\), but directly computing it is hard.

Instead, we often use a **conditional probability path** \(p_t(x \mid z)\), where \(z\) is a data sample or latent anchor. Then we derive a **conditional vector field** that is easy to write down.

Training with the conditional field works because, after averaging over \(z\), it matches the marginal transport we need. This plays a role similar to stochastic gradients in optimization:
- each conditional sample is only one local view,
- but the expectation recovers the correct global target.

## 4. Why continuity equations matter

The continuity equation says that the density changes according to the divergence of the flow:

$$
\partial_t p_t(x) + \nabla \cdot \big(p_t(x) u_t(x)\big) = 0.
$$

This is the mathematical statement that “probability mass is conserved while flowing.”

Its conceptual role is crucial:
- it links a vector field to the evolution of the whole distribution,
- it justifies why learning the right velocity field is enough,
- it explains how conditional and marginal descriptions can be consistent.

## 5. Diffusion view: learn the score field

Diffusion introduces stochasticity and usually writes the dynamics as an SDE. In continuous time, the reverse process depends on the **score function**

$$
\nabla_x \log p_t(x).
$$

This score tells us where the density increases most sharply. If the model learns the score well, it can reverse the noising process.

So the score-based learning objective is different from flow matching in form, but similar in spirit:
- flow matching learns **how particles should move**,
- score matching learns **which direction increases probability density**.

Under common Gaussian paths, these two quantities are tightly related.

## 6. The key bridge between flow and diffusion

The biggest conceptual payoff is that **flow matching and diffusion are not disconnected families**.

For many Gaussian probability paths:
- the conditional vector field can be written analytically,
- the conditional score can also be written analytically,
- one can convert between vector-field learning and score-based learning.

That is why many diffusion papers can be reinterpreted as transport problems, and many flow-matching papers look diffusion-like.

## 7. Training targets in practice

### 7.1 Flow matching objective
Flow matching trains a network \(u_\theta(x_t, t)\) to approximate the target vector field:

$$
\mathcal L_{\text{FM}} = \mathbb E\big[\lVert u_\theta(x_t, t) - u_t^{\text{target}}(x_t) \rVert^2\big].
$$

In practice, the marginal target is often replaced by a conditional target because the latter is tractable.

### 7.2 Score matching objective
Score matching trains a network \(s_\theta(x_t, t)\) to match the score:

$$
\mathcal L_{\text{SM}} = \mathbb E\big[\lVert s_\theta(x_t, t) - \nabla_x \log p_t(x_t \mid z) \rVert^2\big]
$$

or an equivalent denoising form.

The practical message is that the two frameworks often differ more in **parameterization and solver choice** than in their final modeling goal.

## 8. Gaussian probability paths are the common workhorse

The original note points out that much of the theory becomes simple under a Gaussian conditional path:

$$
p_t(x \mid z) = \mathcal N(\alpha_t z, \beta_t^2 I).
$$

This path is useful because:
- sampling is easy,
- conditional scores are analytic,
- conditional vector fields are analytic,
- diffusion and flow-matching formulations become easy to connect.

See [[Gaussian probability path]] for a dedicated note.

## 9. How to think about the landscape

A clean mental hierarchy is:

1. **Choose a probability path** from noise to data.
2. **Choose what to learn** along that path:
   - a score field,
   - a vector field,
   - or an equivalent denoising target.
3. **Choose how to sample**:
   - stochastic SDE-like updates,
   - deterministic ODE integration,
   - or discrete denoising steps.

This viewpoint makes many model families feel much less fragmented.

## 10. Bottom line

Flow matching and diffusion models are two compatible ways to describe the same high-level task: **transporting a simple base distribution into the data distribution over time**.

- Diffusion emphasizes **stochastic corruption and score estimation**.
- Flow matching emphasizes **deterministic transport and vector-field regression**.
- Gaussian probability paths make the bridge especially explicit.

If diffusion tells me *how to undo noise*, flow matching tells me *how probability mass should move*.
