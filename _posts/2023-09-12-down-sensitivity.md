---
layout: post
title: "Beyond Local Sensitivity via Down Sensitivity"
comments: true
authors:
  - thomassteinke
timestamp: 10:00:00 -0700
categories: [Algorithms]
---

In [our previous post](/inverse-sensitivity/), we discussed local sensitivity and how we can get accuracy guarantees that scale with local sensitivity, which can be much better than the global sensitivity guarantees attained via standard noise addition mechanisms.
In this post we will look at what we can do when even the local sensitivity is unbounded. This is obviously a challenging setting, but it turns out that not all hope is lost.

As a motivating example, suppose we have a dataset \\\(x=\(x\_1,x\_2,\\cdots,x\_n\)\\\) and we want to approximate \\\(\\max\_i x\_i \\\) in a differentially private manner.
The difficulty is that adding a single element to \\\(x\\\) can increase the maximum arbitrarily. That is, if \\\(x'=\(x\_1,x\_2,\\cdots,x\_n,\\infty\)\\\), then \\\(\\max\_i x'\_i=\\infty\\\). Differential privacy requires us to make the outputs \\\(M\(x\)\\\) and \\\(M\(x'\)\\\) indistinguishable, which seems to directly contradict our accuracy goal \\\(M\(x\) \\approx \\max\_i x\_i\\\).

One solution to the problem of unbounded sensitivity is to clip the inputs, so that the sensitivity becomes bounded. But this requires knowing a good a priori approximate upper bound on the \\\(x\_i\\\)s. Trying to find such an upper bound is probably the very reason we want to approximate the maximum in the first place!

Another solution is to "aim lower:" Instead of aiming to approximate the largest element \\\(x\_{\(n\)} := \\max\_i x\_i\\\), we can aim to approximate the \\\(k\\\)-th largest element \\\(x\_{\(n-k+1\)}\\\).
The \\\(k\\\)-th largest element has bounded local sensitivity (up to distance \\\(k-1\\\)), which means we can apply [the inverse sensitivity mechanism](/inverse-sensitivity/) or similar tools.
And -- spoiler alert -- this is essentially what we will do. However, we will present an algorithm that is more general than just for approximating the maximum.

The algorithm we present is due to Fang, Dong, and Yi [[FDY22](https://cse.hkust.edu.hk/~yike/ShiftedInverse.pdf "Juanru Fang, Wei Dong, Ke Yi. Shifted Inverse: A General Mechanism for Monotonic Functions under User Differential Privacy. CCS 2022.")].
In terms of applications, a natural setting where we may need to approximate functions of unbouned local sensitivity is when each person can contribute multiple items to the dataset. This setting is often referred to as "user-level differential privacy" or "user DP."[^1]
For example, if we have a collection of web browsing histories, we may wish to estimate the total number of webpages visited; this has unbounded local sensitivity because a single person could visit an arbitrary number of webpages.

## Down Sensitivity

Observe that, while _adding_ one element to the input can increase the maximum arbitrarily, _removing_ one element can only decrease it by the gap between the largest and second-largest elements \\\(x\_{\(n\)}-x\_{\(n-1\)}\\\). In other words, the maximum satisfies some kind of one-sided local sensitivity bound. This is the general property we will rely on.

We define the _down sensitivity_[^2] of the function \\\(f : \\mathcal{X}^\* \\to \\mathbb{R}\\\) at the input \\\(x\\in\\mathcal{X}^\*\\\) at distance \\\(k\\\) as
\\\[\\mathsf{DS}^k\_f(x) := \\sup\_\{x' \\subseteq x : \\mathrm{dist}(x,x') \\le k\} \|f(x)-f(x')\|. \\tag{1}\\\]
Here \\\(\\mathrm{dist} : \\mathcal{X}^\* \times \\mathcal{X}^\* \\to \\mathbb{R}\\\) is the size of the symmetric difference between the two input tuples/multisets \\\(\\mathrm{dist}\(x,x'\) = \|x \\setminus x'\| + \| x' \\setminus x \|\\\), which defines a metric. In other words, it measures how many people's data must be added or removed to get from one dataset to the other.
For comparison, the local sensitivity is
\\\[\\mathsf{LS}^k\_f(x) := \\sup\_\{x'\\in\\mathcal{X}^\* : \\mathrm{dist}(x,x') \\le k\} \|f(x)-f(x')\|. \\tag{2}\\\]
The difference between Equations 1 and 2 is simply that down sensitivity only considers removing elements from \\\(x\\\), while local sensitivity considers both addition and removal.
Thus, the down sensitivity is at most the local sensitivity, which is, in turn, upper bounded by the global sensitivity: \\\(\\mathsf{DS}^k\_f(x) \\le \\mathsf{LS}^k\_f(x) \\le k \\cdot \\mathsf{GS}\_f\\\).

Our goal now is to estimate \\\(f\(x\)\\\) in a differentially private manner, where the accuracy guarantee scales with the down sensitivity.

## Monotonicity Assumption

In order to do anything, we need some assumptions about the function \\\(f : \\mathcal{X}^\* \\to \\mathcal{Y}\\\) that we are trying to approximate.
First we will assume that \\\(\\mathcal{Y} \subseteq \\mathbb{R}\\\) is finite.[^3]
The main assumption is monotonicity:
\\\[\\forall x' \\subseteq x \\in \\mathcal{X}^\* ~~~ f(x') \le f(x). \tag{3}\\\]
The maximum and many other example functions satisfy this assumption.

Intuitively, we need this assumption to ensure that the down sensitivity is itself low sensitivity (in terms of the distance \\\(k\\\)).
Specifically, Lemma 1 below requires monotonicity.

As an example of what could happen if we don't make this assumption, consider the function \\\(\mathrm{sum}(x) := \sum\_i x_i\\\) and the pair of neighbouring inputs \\\(x=(1,1,\cdots,1)\in\mathcal{Y}^n,x'=(1,1,\cdots,1,-n)\in\mathcal{Y}^{n+1}\\\). Then, for all \\\(1 \le k\\le n\\\), we have \\\(\mathsf{DS}\_{\mathrm{sum}}^k(x)=k\\\), but \\\(\mathsf{DS}\_{\mathrm{sum}}^k(x')=n\\\).

Note that the sum is monotone if we restrict to non-negative inputs. In general, we can take any function \\\(g\\\) and convert it into a monotone function \\\(f\\\) by defining \\\(f\(x\) = \max\\{ g\(\\check{x}\) : \\check{x} \\subseteq x \\}\\\). Depending on the context, \\\(f\\\) may or may not be a good proxy for \\\(g\\\).

## A Loss With Bounded Global Sensitivity

Given a monotone function \\\(f : \\mathcal{X}^\* \\to \\mathbb{R}\\\), we define a loss function \\\(\\ell : \\mathcal{X}^\* \\times \\mathbb{R} \\to \\mathbb{Z}\_{\\ge 0}\\\) by
\\\[\\ell(x,y) := \\min\\{ \\mathrm{dist}(x,\\tilde{x}) : \\tilde{x} \\subseteq x, f\(\\tilde{x}\) \le y \\}. \\tag{4}\\\]
In other words, \\\(\\ell(x,y)\\\) measures how many entries of \\\(x\\\) we need to remove until \\\(f\(x\) \\le y\\\). 
Yet another way to think of it is that \\\(\\ell(x,y)\\\) is the distance from the point \\\(x\\\) to the set \\\(f^{-1}((-\\infty,y]) \\cap \\{ \\tilde{x} : \\tilde{x} \\subseteq x \\} \\\). 

The key property we need is that this loss has bounded sensitivity. We split the proof into Lemmas 1 and 2.

> **Lemma 1.**
> Let \\\(f : \\mathcal{X}^\* \\to \\mathbb{R}\\\) satisfy the monotonicity property in Equation 3.
> Define \\\(\\ell : \\mathcal{X}^\* \\times \\mathbb{R} \\to \\mathbb{Z}\_{\\ge 0}\\\) as in Equation 4. <br>
> Let \\\(x' \subseteq x \in \mathcal{X}^\*\\\).
> Then \\\(\ell(x',y)\le\ell(x,y)\\\) for all \\\(y \in \mathbb{R}\\\).

> _Proof._
> Fix \\\(y \in \mathbb{R}\\\) and \\\(x' \subseteq x \in \mathcal{X}^\*\\\).
> Let \\\(x\_\\Delta = x \\setminus x' \\subseteq x\\\), so that \\\(x' = x \\setminus x\_\\Delta \\\).
> 
> Let \\\(\\widehat{x} \\subseteq x\\\) satisfy \\\(f\(\\widehat{x}\)\\le y\\\) and \\\(\\mathrm{dist}\(x,\\widehat{x}\)=\\ell\(x,y\)\\\).
> Define \\\(\\widehat{x}' = \\widehat{x} \\setminus x\_\\Delta\\\). This ensures \\\(\\widehat{x}' \\subseteq x'\\\) and \\\[\\mathrm{dist}(x',\\widehat{x}') = \\mathrm{dist}\(x  \\setminus x\_\\Delta , \\widehat{x}  \\setminus x\_\\Delta \) \\le \\mathrm{dist}(x,\\widehat{x}).\\\]
> 
> By monotonicity, \\\(f\(\\widehat{x}'\) \\le f\(x\) \\le y\\\). 
> Thus \\\[\\ell(x',y) = \\min\\{ \\mathrm{dist}(x',\\tilde{x}') : \\tilde{x}' \\subseteq x', f\(\\tilde{x}'\) \le y \\}\\\]\\\[ \\le \\mathrm{dist}(x',\\widehat{x}')  \\le \\mathrm{dist}(x,\\widehat{x}) = \\ell\(x,y\).\\\]
> &#8718;

> **Lemma 2.**
> Let \\\(f : \\mathcal{X}^\* \\to \\mathbb{R}\\\).
> Define \\\(\\ell : \\mathcal{X}^\* \\times \\mathbb{R} \\to \\mathbb{Z}\_{\\ge 0}\\\) as in Equation 4. <br>
> Let \\\(x' \subseteq x \in \mathcal{X}^\*\\\).
> Then \\\(\ell(x,y)\le\ell(x',y)+\mathrm{dist}(x,x')\\\) for all \\\(y \in \mathbb{R}\\\).

> _Proof._
> Fix \\\(y \in \mathbb{R}\\\) and \\\(x' \subseteq x \in \mathcal{X}^\*\\\).
> 
> Let \\\(\\widehat{x}' \\subseteq x'\\\) satisfy \\\(f\(\\widehat{x}'\)\\le y\\\) and \\\(\\mathrm{dist}\(x',\\widehat{x}'\)=\\ell\(x',y\)\\\).
> Since \\\(\\widehat{x}' \\subseteq x' \\subseteq x\\\), we have 
> \\\[\\ell(x,y) = \\min\\{ \\mathrm{dist}(x,\\tilde{x}) : \\tilde{x} \\subseteq x, f\(\\tilde{x}\) \le y \\}  \\le \\mathrm{dist}(x,\\widehat{x}') \\\]\\\[ \\le \\mathrm{dist}(x,x') + \\mathrm{dist}(x',\\widehat{x}') = \\ell\(x,y\)+\mathrm{dist}(x,x'),\\\]
> by the triangle inequality, as required.
> &#8718;

Note that we only needed the monotonicity assumption for Lemma 1. 
Combining the two lemmas gives \\\[ \forall  x' \\subseteq x ~ \forall y ~~~~~ \ell(x',y) \le \ell(x,y) \le \ell(x',y) + \mathrm{dist}(x,x').\\\]
Overall we have the following guarantee.

> **Proposition 3. (Global Sensitivity of the Loss)**
> Let \\\(f : \\mathcal{X}^\* \\to \\mathbb{R}\\\) satisfy the monotonicity property in Equation 3.
> Define \\\(\\ell : \\mathcal{X}^\* \\times \\mathbb{R} \\to \\mathbb{Z}\_{\\ge 0}\\\) as in Equation 4. <br>
> Then, for all \\\(x, x' \\in \\mathcal{X}^\*\\\) and all \\\(y \in \mathbb{R}\\\), we have \\\[\|\ell(x,y)-\ell(x',y)\| \le \mathrm{dist}(x,x').\\\]

> _Proof._
> Fix \\\(x, x' \\in \\mathcal{X}^\*\\\) and \\\(y \\in \\mathbb{R}\\\).
> Let \\\(x'' = x \\cap x'\\\). 
> Since \\\(x'' \\subset x'\\\) and \\\(f\\\) is assumed to be monotone, Lemma 1 gives \\\(\\ell\(x'' ,y\) \\le \\ell\(x',y\)\\\).
> Also \\\(x'' \\subset x\\\), whence Lemma 2 gives \\\(\\ell\(x,y\) \\le \\ell\(x'' , y\) + \\mathrm{dist}\(x , x'' \)\\\).
> Note that \\\( \\mathrm{dist}\(x , x'' \) = \| x \\setminus x'' \| = \| x \\setminus x' \| \\le \\mathrm{dist}\(x , x' \).\\\)
> Combining inequalities gives \\\(\\ell\(x,y\) \\le \\ell\(x' , y\) + \\mathrm{dist}\(x , x' \)\\\). The other direction is symmetric.
> &#8718;


## The Shifted Inverse Sensitivity Mechanism

Let's recap where we are: We have a monotone function \\\(f : \\mathcal{X}^\* \\to \\mathcal{Y}\\\), where \\\(\\mathcal{Y} \\subseteq \\mathbb{R}\\\) is finite. We want to approximate \\\(f(x)\\\) privately. Equation 4 gives us a loss \\\(\\ell\\\) that is low-sensitivity.
We have \\\(\\ell(x,f(x))=0\\\) and, if \\\(y < f\(x\)\\\) decreases, the loss \\\(\\ell(x,y)\\\) increases (depending on the down sensitivity of \\\(f\\\)).
So far so good. The problem is that if \\\(y > f\(x\)\\\) increases, the loss \\\(\\ell(x,y)\\\) doesn't increase. This means we can't just throw this loss into the exponential mechanism.

Intuitively, the way we get around this problem is by looking for a value \\\(y\\\) such that the loss \\\(\\ell\(x,y\)\\\) is greater than zero, but not too large. That is, we "shift" our goal from trying to minimize \\\(\\ell(x,y)\\\) to minimizing something like \\\(\|\\ell(x,y)-\\tau\|\\\) for some constant \\\(\\tau>0\\\).
Going back to the example of the maximum, this corresponds to aiming for the \\\(\(\\tau+1\)\\\)-th largest value instead of the largest value.
The hope is that we get an output with \\\(\|\\ell(x,y)-\\tau\|<\\tau\\\), which for the maximum example corresponds roughly to getting a value between the largest value and the \\\(2\tau\\\)-th largest value.

Alas, there is one more technical issue we need to deal with: The loss function is far from continuous, so there may not exist any \\\(y\\\) such that \\\(\|\\ell(x,y)-\\tau\|<\\tau\\\).
For example, computing the maximum of the dataset \\\(x=\(1,1,\\cdots,1\)\\\) gives a loss function with \\\(\\ell(x,y)=0\\\) for all \\\(y \\ge 1\\\) and \\\(\\ell(x,y)=n\\\) for all \\\(y < 1\\\).


---

[^1]: We emphasize that user-level differential privacy is not an alternative privacy definition, rather it is the standard definition of differential privacy with a data schema allowing multiple data items per person. In contrast, most of the differential privacy literature assumes a one-to-one correspondence between people and data items. Note that we prefer the terminology "person"/"people" rather than "user"/"users." The "user" terminology is specific to the tech industry and may be confusing in other contexts; e.g., in the context of the US Census Bureau, "users" are the entities that use data provided by the bureau, rather than the people whose data the bureau collects.

[^2]: The name "down sensitivity" is due to Cummings and Durfee [[CD20](https://arxiv.org/abs/1804.08645 "Rachel Cummings, David Durfee. Individual sensitivity preprocessing for data privacy. SODA 2020.")], who attribute the idea to Raskhodnikova and Smith [[RS16](https://arxiv.org/abs/1504.07912 "Sofya Raskhodnikova, Adam Smith. Efficient lipschitz extensions for highdimensional graph statistics and node private degree distributions. FOCS 2016.")]. 

[^3]: The finiteness assumption can be relaxed somewhat, but since we rely on the exponential mechanism we do need some kind of constraint on the output space to ensure utility. Note that we define \\\(\\mathcal{X}^\* := \\bigcup\_{n=0}^\\infty \\mathcal{X}^n\\\) to be the set of all finite tuples of elements in \\\(\\mathcal{X}\\\); we use subset notation \\\(x' \\subseteq x \\\) to denote that \\\(x'\\\) can be obtained by removing elements from \\\(x\\\) (and potentially permuting).
