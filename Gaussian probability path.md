---
categories:
  - "[[Evergreen]]"
title: "Gaussian probability path"
created: 2026-04-22
updated: 2026-04-22
tags:
  - 0🌲
  - generative-models
  - diffusion
  - flow-matching
  - probability-path
sources:
  - https://www.notion.so/Gaussian-probability-path-279d25c2a3b88004829ef0e4a6e11810
related:
  - "[[VAE and Diffusion]]"
  - "[[Diffusion]]"
  - "[[Flow matching and diffusion models]]"
---

# Gaussian probability path

A Gaussian probability path is one of the most useful constructions in diffusion and flow matching because it gives a simple interpolation between pure noise and clean data.

## 1. Definition

A common conditional path is

$$
p_t(x \mid z) = \mathcal N(\alpha_t z, \beta_t^2 I),
$$

which means a sample on the path can be written as

$$
x_t = \alpha_t z + \beta_t \epsilon, \qquad \epsilon \sim \mathcal N(0, I).
$$

Here:
- \(z\) is the target data sample,
- \(\alpha_t\) controls how much signal remains,
- \(\beta_t\) controls how much noise is injected.

## 2. Boundary conditions

To interpolate from noise to data, we usually want:

$$
p_0(x \mid z) = \mathcal N(0, I), \qquad p_1(x \mid z) = \delta_z.
$$

That is achieved by choosing

$$
\alpha_0 = 0, \quad \beta_0 = 1, \quad \alpha_1 = 1, \quad \beta_1 = 0.
$$

So:
- at \(t=0\), the sample is pure Gaussian noise,
- at \(t=1\), the sample collapses to the clean point \(z\).

## 3. Why this path is so convenient

The Gaussian path is popular because it is analytically friendly.

### 3.1 Easy to sample
Given \(z\), generating \(x_t\) is trivial.

### 3.2 Easy to differentiate
Both the score and the conditional transport field often have closed forms.

### 3.3 Easy to connect frameworks
This is the main reason it appears everywhere:
- diffusion uses it for noising and denoising,
- score matching uses it for analytic conditional scores,
- flow matching uses it for analytic conditional vector fields.

## 4. What it means geometrically

The path says that each data point is connected to noise by a time-dependent Gaussian cloud.

Early in time:
- the cloud is wide,
- the signal contribution is small,
- the sample mostly looks like noise.

Late in time:
- the cloud shrinks,
- the signal contribution grows,
- the sample becomes more concentrated near the data point.

So the model learns either:
- how to **pull samples through this cloud** (flow matching), or
- how to **reverse the cloud-corruption process** (diffusion).

## 5. Relation to DDPM-style notation

In DDPM-like notation, the path is often written as

$$
x_t = \bar\alpha_t x_0 + \bar\beta_t \epsilon.
$$

This is the same basic idea with different symbols:
- \(x_0\) plays the role of the clean sample \(z\),
- \(\bar\alpha_t\) is the signal coefficient,
- \(\bar\beta_t\) is the noise coefficient.

So the Gaussian probability path is not a separate concept from diffusion; it is one of diffusion's core building blocks.

## 6. What changes across papers

Different papers mainly differ in the schedule for \(\alpha_t\) and \(\beta_t\).

Changing the schedule changes:
- how quickly signal disappears,
- how quickly noise grows,
- how easy the target becomes for the network,
- and sometimes how stable or fast inference is.

That is why papers often propose different path or scheduler designs while keeping the same overall framework.

## 7. Bottom line

A Gaussian probability path is the standard bridge from data to noise and back again. It is popular not because it is the only possible path, but because it makes the math, training targets, and samplers all unusually clean.

If I want one sentence to remember it, it is this:

> **A Gaussian probability path gives a smooth, analyzable interpolation between clean data and Gaussian noise, which is why it underlies so much of diffusion and flow-matching theory.**
