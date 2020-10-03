---
layout: post
title: "Open Problem - Avoiding the Union Bound for Multiple Queries"
comments: true
authors: gautamkamath
timestamp: 19:00:00 -0400
categories: [Open Problems]
---

**Background:** Perhaps the best-studied problem in differential privacy is answering multiple counting queries.
The standard approach is to add independent, appropriately-calibrated (Laplace or Gaussian) noise to each query result and apply a composition theorem.
To bound the maximum error over the query answers, one takes a union bound over the independent noise samples.
However, this is *not* optimal.
The problem is to identify the optimal method (up to constant factors).

**Problem 1:** Is there a randomized algorithm \\(M : \\{0,1\\}^{n \times k} \rightarrow \mathbb{R}^k\\) that is differentially private[^1] and satisfies
\\[
\forall x \in \\{0, 1\\}^{n \times k} \quad \mathbb{E}\left[ \left\\|M(x) - \sum_{i=1}^n x_i \right\\|_\infty \right] \leq c \sqrt{k}
\\]
for some constant \\( c > 0\\) depending only on the privacy parameters?[^2]

Adding independent Gaussian noise to each coordinate/query yields \\(c \sqrt{k \log k}\\) in place of \\(c \sqrt{k}\\) above. 
Steinke and Ullman \[[SU17](https://arxiv.org/abs/1501.06095)\] showed that the union bound can *almost* be avoided and obtained \\(c \sqrt{k \log \log k}\\) using correlated noise.
The algorithm is nonetheless based on independent Gaussian noise, with the added step of using the exponential mechanism to identify high-error answers and correct them.

Note that a \\(\Omega(\sqrt{k})\\) lower bound is known \[[BUV18](https://arxiv.org/abs/1311.3158), [SU17](https://arxiv.org/abs/1501.06095)\]. By \[[BDKT12](http://www.cs.utah.edu/~bhaskara/files/privacy.pdf)\] it suffices to consider mechanisms \\(M\\) that add *instance-independent noise*. That is, \\(M(x) = \sum_{i=1}^n x_i + Z\\) where \\(Z\\) is some fixed noise distribution over \\(\mathbb{R}^k\\) that is independent of \\(x\\).

**Reward:** For a positive solution, an all-you-can-eat sushi dinner at a sushi restaurant of your choice.
If the solution is an efficiently-sampleable distribution with a closed-form density, alcohol will be included.
For a negative solution, alcohol only.[^3]

**Other related work:** \[[DSSUV15](https://privacytools.seas.harvard.edu/files/privacytools/files/robust.pdf), [Vad17](https://privacytools.seas.harvard.edu/files/privacytools/files/complexityprivacy_1.pdf), [AS18](https://arxiv.org/abs/1801.09236)\]. 
A very recent work by Ganesh and Zhao \[[GZ20](https://people.eecs.berkeley.edu/~arunganesh/papers/generalizedgaussians.pdf)\] improves the best upper bound from \\(c\sqrt{k \log \log k}\\) to \\(c\sqrt{k \log \log \log k}\\).

_Submitted by [Thomas Steinke](http://www.thomas-steinke.net/) and [Jonathan Ullman](https://www.ccs.neu.edu/home/jullman/) on April 9, 2019._

---

[^1]: Specifically, \\(M\\) is either 1-zCDP [[BS16](https://arxiv.org/abs/1605.02065)] with \\(c\\) an absolute constant or, for every \\(\delta > 0\\), there is an \\(M_\delta\\) that is \\((1, \delta)\\)-DP with \\(c = c(\delta) = c' \cdot \sqrt{\log (1/\delta)}\\) for an absolute constant \\(c'\\).

[^2]: Here \\(x_i \in \\{0, 1\\}^k \subset \mathbb{R}^k\\) denotes the data vector of individual \\(i\\) and \\(x = (x_1, x_2, \dots, x_n) \in \\{0,1\\}^{n \times k}\\). For simplicity, we only consider expected error; high-probability error bounds are an immediate consequence.

[^3]: Restaurant need not necessarily be all-you-can-eat. Maximum redeemable value US$500.
