---
title: "Nothing-At-Stake Attack"
date: 2025-11-27 20:24:00 +0000
categories: [computer science, blockchain]
tags: [blockchain, math]     # TAG names should always be lowercase
math: true
katex: true
description: We take a look at some of the math behind the Nothing-At-Stake attack
---

## Object of this Post

I recently finished watching Tim Roughgarden's brilliant exposition of the Proof-of-Stake sybil resistance mechanism in his lecture series [Foundations of Blockchain](https://youtu.be/KNJGPI0fuFA?si=cUlG7x0hPzDYhno-) on YouTube. In lecture 12, he discusses the classic Nothing-at-Stake attack on the aforementioned protocol.  

One of the claims which struck me was that in such an attack, the details of which we will get into in a minute, the attacker can grow his chain at a rate proportional to $e \cdot \lambda_a \$, where $\lambda_a \$ is the adversary's stake share and $ e $ is Euler's number. In this post we will show the lower bound, which can be done fairly straightforwardly.

## How Nothing-At-Stake Works

I will assume some basic familiarity Proof-of-Stake here. Moreover, I assume a small amount of mathematical maturity. 

We work under the following assumptions about our Proof-of-Stake system. 

Our PoS Lottery will look like the following. Stakeholder with public key $p_k $ gets to mine a block $B$ with hash = prev if:

$$
H(prev, timestamp, p_k) < T
$$

Where H is a hash function and T some difficulty parameter.

Then this hash will go into the new block's metadata as 'prev'.

Now to be clear, the result we show does not apply to all PoS blockchains. Just the ones which have pseudorandomness as defined above. The crux idea is that as a result of the above lottery ticket idea, we have an independent lottery at each block every timestep. If we had an ideal randomness beacon that we baked into our lottery, they would obviously not be independent.

The idea of a Nothing-At-Stake attack is fairly straightforward: An adversarial node tries to orphan off some target block by extending it's own chain to some block before the target. What makes this special compared to an ordinary nakamoto style double spend attack is a compounding effect afforded to the adversarial node by the Proof-of-Stake system. Namely, instead of extending just one competing chain, the adversary will extend an exponentially increasing number of chains all at the same time.

With all this out of the way we can plow on to some definitions and modelling assumptions.

## Setup

We will narrow our focus to a separate blockchain being built by an adversarial node with stake $\lambda_a \$. For each block in our blockchain, every so often our adversary will be able to add another block to it. For a given block we call the time between creation of the parent block $B$ and the birth of its $i^{th}$ child the random variable $Y_{B,i}$. Notice that $Y_{B,0} \sim \mathrm{Exp}(\lambda_a) \ $ and moreover that $Y_{B,i} = Y_{B,i-1} + X_i \$ where $ X_i \sim \mathrm{Exp}(\lambda) $.

Let's make some helpful function definitions:

We define a path $\pi$ to be a sequence of integers $(\pi(i))$ from some root block $B_0$ say. To follow such a path it means at the $i^{th}$ block you choose the ${\pi(i)}^{th}$ child, in order of birth(youngest to oldest).

The total time taken along a path $\pi$ of child-blocks starting from a root block:

$$
S(\pi) = \sum_{i=0}^{|\pi|-1} Y_{\pi(i),\,\pi(i+1)}
$$

$$
T(n) = \min_{\substack{\pi:\,|\pi| = n}} S(\pi)
$$

Then the goal of our proof is to show $\lim\limits_{n \to \infty} \frac{T(n)}{n} = \frac{1}{\lambda_a\, e}.$

In fact we will prove something weaker, $\liminf\limits_{n \to \infty} \frac{T(n)}{n} <= \frac{1}{\lambda_a\, e}.$

This allows us to stick to elementary mathematics and still partially demystify this spectacular result.

## The Crux Move

The crux move here in getting a result is to consider the laplace transform of $(S(\pi))$ and sum this up over all paths $\pi$ of given length n. This helps us because the complicated recursive nature of our dynamics undergoes a dramatic simplification under this transform. Let's see it:

We define first $\phi_n$

$$
\phi_n(s)
\;=\;
\sum_{\pi\in\Pi_n} \mathcal{L}_{S(\pi)}(s)
\;=\;
\sum_{\pi\in\Pi_n} \mathbb{E}\!\left[ e^{-s S(\pi)} \right]
\;=\;
\sum_{\pi\in\Pi_n} \mathbb{E}\!\left[
\exp\!\Big(-s\sum_{i=0}^{n-1} Y_{\pi(i),\pi(i+1)}\Big)
\right].
$$

Note that $Y_{B, i}$ is independent and identically distributed to $Y_{B', i}$ for any other block B'. Hence we can factor the expectation into the following:

$$
\phi_n(s)
\;=\;
\sum_{\pi\in\Pi_n} \mathbb{E}\!\left[
\exp\!\Big(-s\sum_{i=0}^{n-2} Y_{\pi(i),\pi(i+1)}\Big)
\right] \cdot \mathbb{E}\!\left[
\exp\!\Big(-s Y_{B_0,\pi(n)}\Big)
\right]
$$

Here in the last expression I replaced $\pi(0)$ with $B_0$. Now we split the sum in two using Fubini's theorem(In real life one would have to check this sum satisies the conditions of Fubini).

$$
\sum_{\pi \in \Pi_n}
\;=\;
\sum_{\pi' \in \Pi_{n-1}}
\;\sum_{\pi(n) \in \{1,\ldots,n\}}.
$$

This gives:

$$
\phi_n(s) = \phi_{n-1}(s)\,\phi_{1}(s) 
$$

$$
\phi_n(s) = \big(\phi_{1}(s)\big)^{n}
$$

Pretty nice, huh? Now let's evaluate our $\phi_n(s)$

$$
\phi_{1}(s) = \sum_{i=1}^{\infty} \mathbb{E} \left[ \exp\!\Big(-s Y_{B_0, n}\Big)\right]
$$

Since $Y_i = X_1 + ... + X_n$ and all the $X_i$ are IID we get the following simplification

$$
\phi_{1}(s) = \sum_{i=1}^{\infty} \mathbb{E} \left[ {\exp\!\Big(-s X\Big)}\right]^n
$$

Using the formula for the Laplace transform of an exponential random variable with rate $\lambda$ we get:

$$
\phi_{1}(s) = \sum_{i=1}^{\infty} {(\frac{\lambda}{\lambda + s})}^n = \frac{\lambda}{s}
$$

## Deriving the bound

Our approach is to show $\sum_{n=1}^{\infty} \mathbb{P} (T(n) < n \cdot x) < \infty$ and induce the first Borel-Cantelli lemma to show $\mathbb{P}\ \left(\frac{T(n)}{n} < x \ \text{infinitely often}\right) = 0.$ Then we can find the maximum such $x$ and use this to derive our desired result.

Let's begin with the left hand side:

$$
\mathbb{P} (T(n) < nx) = \mathbb{P}\big(\exists\, \pi \text{ with } |\pi| = n : S(\pi) > nx \big)
$$

Now we use the trick that for $Z$ a random variable with values in the integers 

$$\mathbb{P}(Z \geq 1) \leq \mathbb{E} (Z)$$

So plugging this in gives:

$$
... \leq \mathbb{E} \left[ \# \text{no. of } \pi : |\pi| = n \text{ and } S(\pi) < nx \right]

$$

$$
= \mathbb{E}\!\left[ \sum_{\pi : |\pi| = n} \mathbf{1}_{\{ S(\pi) < nx \}} \right].

$$

Now we can take advantage of the trivial bound $ \mathbf{1}_{\{ S(\pi) < nx \}} \leq \exp(snx - sS(\pi) \)$ for $s \ge 0$ which yields:

$$
... \leq \exp(snx) \cdot \mathbb{E} \left[ \sum_{\pi : |\pi| = n} \exp(-sS(\pi)) \right] 
$$

Now we can use our work from earlier to bound the whole lot

$$
... \leq ({\frac{\exp(sx)}{\lambda s}})^n
$$

So the series will converge if ${\frac{\exp(sx)}{\lambda s}} \lt 1$. Using elementary calculus one can show that the maximum value of x allowed is $\frac{1}{\lambda \cdot e}$. We are basically done now.

What we have show is that $\mathbb{P}\ \left(\frac{T(n)}{n} < \frac{1}{\lambda \cdot e} \ \text{infinitely often}\right) = 0.$ Taking the complements we get  $\mathbb{P}\ \left(\frac{T(n)}{n} > \frac{1}{\lambda \cdot e} \ \text{eventually}\right) = 1.$ Hence we get our desired liminf result. 

## Summary

I thought the math here was pretty slick. I haven't done a huge amount of applied probability, mostly just general measure theory stuff but I found this really interesting and count myself inspired to look into more probability on trees. Stay tuned for the upper bound in a future post!

