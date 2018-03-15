---
layout: post
title:  "Exponential Family"
date:   2018-03-14 12:00:00 +0800
---

The exponential family distributions are very important in graphical models and Bayesian learning. They have nice properties, like conjugacy and finite sufficient statistics, which enable the convenience and efficiency of the inference and learning process. Yet, I only almost know the details of exponential family distribution. The minor gap between "almost know" and "know" prevents me from understanding it completely. Here I try to close the gap.

## Basics

The canonical/natural form of exponential family distribution is of the form

$$p(x|\theta) = \frac{1}{Z(\theta)}h(x)\exp[\theta^T \phi(x)] = h(x)\exp[\theta^T \phi(x) - A(\theta)]$$

where $\theta$ is the natural parameter, $\phi(x)$ is sufficient statistics, $Z(\theta)$ is the partition function, $A(\theta)$ is the log partition function. Let's write the univariate Gaussian distribution in  exponential family form

$$\mathcal{N}(x|\mu, \sigma^2) = \frac{1}{\sqrt{2\pi \sigma^2}}\exp[-\frac{1}{2\sigma^2}(x-\mu)^2] = \frac{1}{\sqrt{2\pi \sigma^2}}\exp[-\frac{1}{2\sigma^2}\mu^2]\exp[-\frac{1}{2\sigma^2}x^2 + \frac{\mu}{\sigma^2}x]$$

Therefore, it is easy to tell that

$$\theta = \begin{pmatrix} \frac{\mu}{\sigma^2} \\ -\frac{1}{2\sigma^2}\end{pmatrix} \\
\phi(x) = \begin{pmatrix}x \\ x^2\end{pmatrix}\\
Z(\theta) = \sqrt{2\pi \sigma^2}\exp[-\frac{1}{2\sigma^2}\mu^2]$$

There is one beautiful property of the log partition function $\nabla_{\theta}A(\theta) = \mathbb{E}_{p(x)}[\phi(x)]$.

## MLE Estimation

Given dataset $\mathcal{D}=\{x_1, x_2,\cdots,x_N\}$, the data likelihood of an exponential family model has the form

$$p(\mathcal{D}|\theta) = [\prod_{i=1}^N h(x_i)] \frac{1}{Z(\theta)^N} \exp[\theta^T (\sum_{i=1}^N \phi(x_i))] $$

We see that the sufficient statistics are $N$ and $\sum_{i=1}^N \phi(x_i)$.


## MLE for Gaussian distribution


## EM for Gaussian Mixture

