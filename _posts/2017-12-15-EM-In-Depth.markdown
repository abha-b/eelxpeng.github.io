---
layout: post
title:  "EM Algorithm In-depth"
date:   2017-12-15 12:00:00 +0800
---

I find it necessary to understand EM more. As an optimization method for MLE/MAP, it is also related to variational bayes EM (VBEM), Gibbs sampling as well as Wake-sleep algorithm.

### EM Introduction
EM algorithm is extremely useful to deal with models with latent variables. It can be used for MLE and MAP estimates. The center idea of EM is that since we have latent variables, which we do not know their value, we can first complete the data by estimating the latent variables with probability, and then maximize the complete data loglikelihood over the parameters.

Let $x_i$ be the observed variables in case $i$, and let $z_i$ be the hidden variables. The goal is to maximize the loglikelihood of the observed data:

$$\mathcal{l}(\theta) = \sum_{i=1}^N \log p(x_i|\theta) = \sum_{i=1}^N \log [\sum_{z_i}p(x_i, z_i|\theta)].$$

This is hard to optimize since the log cannot be pushed inside. EM gets around this problem. If we complete the data by estimating the value of $z_i$, then we can get the complete data loglikelihood easily:

$$\mathcal{l}_c(\theta) =\sum_{i=1}^N \log p(x_i, z_i|\theta).$$

We can estimate the value of $z_i$ using old parameters $\theta^{t}$. But since our estimate of $z$ is not certain, but rather with probability, what we actually get is the so-called expected complete data loglikelihood:

$$Q(\theta, \theta^{t}) =\sum_{i=1}^N p(z_i|x_i,\theta^{t})\log p(x_i, z_i|\theta) = \mathbb{E}_{z|x,\theta^{t}}[\log p(x_i, z_i|\theta)].$$

This is exactly E-step. The goal of E-step is to compute $Q(\theta,\theta^{t})$, or rather, the terms inside of it which the MLE depends on (sufficient statistics). In the M-step, we optimize the Q function wrt $\theta$:

$$\theta^{t+1} = \underset{\theta}{\arg\max}Q(\theta, \theta^t).$$

That is EM algorithm. To give an concrete example, let me show the derivation of GMM. The expected completed data loglikelihood is given by

$$Q(\theta, \theta^{t}) =\sum_{i} \mathbb{E}_{z|x,\theta^t}[\log p(x_i, z_i|\theta)] = \sum_i \mathbb{E}[\log [\prod_{k=1}^K (\pi_k p(x_i|\theta_k))^\mathbb{I}(z_i=k)]]\\
=\sum_i \sum_k \mathbb{E}[\mathbb{I}(z_i=k)]\log[\pi_k p(x_i|\theta_k)]\\
=\sum_i \sum_k p(z_i=k|x_i,\theta_t)\log[\pi_k p(x_i|\theta_k)]\\
=\sum_i \sum_k r_{ik}\log[\pi_k p(x_i|\theta_k)],$$

where $r_{ik} = p(z_i=k\|x_i, \theta_t)$ is the responsibility that cluster k takes for data point i. This is computed in E-step. E-step:

$$r_{ik}=\frac{\pi_k p(x_i|\theta_k^t)}{\sum_{k'}\pi_{k'}p(x_i|\theta_{k'}^t)}.$$

M-step, we optimize Q wrt $\pi$ and $\theta$. For $\pi$, we should remember the constraint that $\sum_k \pi_k = 1$. Taking the Langagian and set the derivatives wrt $\theta$ to zero, we can get $\lambda=\sum_i\sum_k r_{ik}=N$ and $\pi_k = \frac{1}{N}\sum_{i}r_{ik}$. Same for $\theta$:

$$\mu_k = \frac{\sum_i r_{ik}x_i}{r_k}\\
\Sigma_k = \frac{\sum_i r_{ik}(x_i - \mu_k)(x_i - \mu_k)^T}{r_k}.$$

### Generalized EM: tight lower bound of data loglikelihood
As stated in Chapter 11 of Murphy's book, EM actually achieves the tight lower bound of data loglikelihood. The lower bound (or free energy) is

$$\mathcal{L}(\theta, q_i) = \sum_{z_i} q_i(z_i)\log \frac{p(x_i,z_i|\theta)}{q_i(z_i)}\\
= \log p(x_i|\theta) - \mathbb{KL}(q_i(z_i)||p(z_i|x_i,\theta)).$$

$\mathcal{L}$ is the lower bound of the data loglikelihood $\log p(x_i\|\theta)$, and the gap is $\mathbb{KL}(q_i(z_i)\|\|p(z_i\|x_i,\theta))$. We can maximize the lower bound by setting $q_i(z_i) = p(z_i\|x_i,\theta))$. Of course, $\theta$ is unknown, so instead we use $q_i^t(z_i) = p(z_i\|x_i,\theta^t)$, where $\theta^t$ is our estimate of the parameters at iteration t. With that, we plug back into the lower bound and get

$$Q(\theta, \theta^t) = \sum_i \mathbb{E}_{z_i|x_i,\theta^t}[\log p(x_i, z_i\theta)] + \mathbb{H}(p(z_i|x_i,\theta^t)).
$$

From above, we noticed that the first term is exactly the expected complete data loglikelihood in the EM algorithm. And since the second term is a constant wrt $\theta$, the M-step becomes

$$\theta^{t+1} = \underset{\theta}{\arg\max}Q(\theta, \theta^t)=\underset{\theta}{\arg\max}\sum_i \mathbb{E}_{z_i|x_i,\theta^t}[\log p(x_i, z_i|\theta)],$$

as before. Therefore, the E-step is to optimize over the $q$ function so as to achieve the tight lower bound of the data loglikelihood of given current model (model parameters) and the M-step is optimizing the tight lower bound over the parameters.

### VBEM: EM using variational approximation
Without any constraint on the form of $q_i(z_i)$, of course, the tight lower bound is achievable by simply setting $q_i^t(z_i) = p(z_i\|x_i,\theta^t)$. However, when the posterior has no closed form, is intractable and not easy to compute, the tight lower bound cannot be achieved. Variational Bayes makes mean-field assumption on the posterior in order to approximate the true posterior. That is, the constraint on $q_i(z_i)$ is factorial form: $q(z,\theta) = q(\theta)\prod_i q(z_i)$. Here, the full Bayesian inference is made, i.e. treating $\theta$ as a distribution like other variables instead of MAP estimate. The whole process then is exactly as EM, except integrating $\theta$ out in E-step and updating $q(\theta)$ in M-step. That's why it is called VBEM:

$$\text{Variational E-step:} \quad \log q(z) = \mathbb{E}_{q(\theta)}[\log p(x, z, \theta)] + \text{const} \\
\text{Variational M-step:} \quad \log q(\theta) = \mathbb{E}_{q(z)}[\log p(x, z, \theta)] + \text{const},$$

where $\log q_j(x_j) = \mathbb{E}_{-q_j}[\log \tilde{p}(x)] + const$ is the form we usually deal with in variational inference and it is obtained by maximizing lower bound.


### Gibbs Sampling: EM using Monte Carlo sampling
The steps of Gibbs sampling has surprising resemblance with EM. Gibbs sampling can be seen as the two sampling steps involving latent variable $z$ and parameter $\theta$:

$$z^{t+1} \sim p(z|x, \theta^t) \\
\theta^{t+1} \sim p(\theta|x, z^{t+1}).$$

The first step resembles E-step, and the second step resembles M-step. In EM, the E-step computes the exact posterior distribution $p(z\|x, \theta^t)$ for $z$, while the M-step computes the MAP estimate of $\theta$. In Gibbs sampling, the first step instead samples from $p(z\|x, \theta^t)$, while the second step instead samples from the distribution of $\theta$.

### Wake-sleep: EM for probabilistic deep model
Wake-sleep is an algorithm proposed by Hinton for training DBN and DBM (unsupervisedly). The network has recognition weights, which convert the input into representations in successive hidden layers, and generative weights, which reconstruct the representation. In the "wake" phase, neurons are driven by recognition connections, and generative weights are adapted to increase the probability that the would reconstruct the correct activity vector in the layer below. In the "sleep" phase, neurons are driven by generative connections, and recognition weights are adapted to increase the probability that they would produce the correct activity vector in the layer above. Wake-sleep algorithm is also optimizing the free energy:

$$\log P(d;G) \geq \log P(d;G) - \mathbb{KL}(Q(h|d;R), P(h|d;G))\\
F(d;R,G) = -\log P(d;G) + \mathbb{KL}(Q(h|d;R), P(h|d;G)).
$$

This exactly resembles the free energy of EM algorithm, and the "wake" phase corresponds to M-step, "sleep" phase corresponds to E-step. The distribution $Q(h\|d)$ produced by the recognition weights is a factorial distribution in each hidden layer because the recognition weights produce stochastic states of units within a hidden layer that are conditional independent, given the states in the layer below. It is natural to use factorial distribution in neural net because it allows the probability distribution over the $2^n$ alternative hidden representations to be specified with $n$ numbers instead of $2^n-1$. During "wake" phase, minimizing the free energy wrt the generative weights will tend to favor the model whole posterior distribution is most similar to $Q(h\|d)$. Thus, the effect of factorial distribution in $Q(h\|d)$ is not severe.

Although generally considered as approximating the normalization constant, the contrastive divergence (CD) algorithm for training RBM is also similar to wake-sleep algorithm if considering only generating one sample. 

In conclusion, many good ideas in machine learning are remarkablely similar. One has to find the underlying connections in order to develop deep understanding of the algorithms. I read some of the points from online or books, and I am not at all fully understand all of them. But the subtle connections are really interesting. And I think that, although many people are interested in deep neural networks without probabilistic, there are indeed many ideas that are more straightforward and more promising under probabilistic framework, especially unsupervised deep learning.
