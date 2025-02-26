---
layout: post
title: "Open Problem: Is the Exponential Mechanism Really Optimal?"
comments: true
authors:
  - thomassteinke
timestamp: 10:00:00 -0700
categories: [Open Problems]
---

The Exponential Mechanism [[MT07](https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=a0ac083b0368237a25c042a0e0efc07160ef822d "Frank McSherry, Kunal Talwar. Mechanism Design via Differential Privacy. FOCS 2007.")] is a key tool in the algorithmic toolbox of differential privacy. We have [written about it previously](/exponential-mechanism-bounded-range/) and it has its own [Wikipedia article](https://en.wikipedia.org/wiki/Exponential_mechanism). It's generally believed to be optimal in some sense. In this post we're going to talk about what exactly it means for the exponential mechanism to be optimal and what kind of optimality guarantees we can prove.

## Recap: The Exponential Mechanism
First, here's a quick summary of what the exponential mechanism is and what it guarantees:
In the problem of *private selection*, we are given a private dataset \\\(x \\in \\mathcal{X}^n\\\), a (public, finite) set of options \\\(\\mathcal{Y}\\\), and a loss function \\\(\\ell : \\mathcal{Y} \\times \\mathcal{X}^n \\to \\mathbb{R}\\\); and our goal is to find an option \\\(y \\in \\mathcal{Y}\\\) that approximately minimizes the loss \\\(\\ell(y,x)\\\) subject to differential privacy with respect to the dataset \\\(x\\\).   
The exponential mechanism \\\(M : \\mathcal{X}^n \\to \\mathcal{Y} \\\) is a randomized algorithm given by <a id="expmech" />\\\[\\forall x \\in \\mathcal{X}^n ~ \\forall y \\in \\mathcal{Y} ~~~~~ \\mathbb{P}[M(x) = y] := \\frac{\\exp(-\\frac{\\varepsilon}{2\\Delta} \\ell(y,x))}{\\sum_{y' \\in \\mathcal{Y}} \\exp(-\\frac{\\varepsilon}{2\\Delta} \\ell(y',x)) }, \\tag{1}\\\] where \\\(\\Delta\\\) is the sensitivity of the loss function \\\(\\ell\\\) given by \\\[\\Delta := \\sup_{x,x' \\in \\mathcal{X}^n : d(x,x') \\le 1} \\max_{y\\in\\mathcal{Y}} \|\\ell(y,x) - \\ell(y,x')\|,\tag{2}\\\] where the supremum is taken over all datasets \\\(x\\\) and \\\(x'\\\) differing on the data of a single individual (which we denote by \\\(d(x,x')\\le 1\\\)).
The exponential mechanism given in [Equation 1](#expmech) satisfies \\\(\(\\varepsilon,0\)\\\)-differential privacy (and [\\\(\\frac18\\varepsilon^2\\\)-zCDP](/exponential-mechanism-bounded-range/)).
In terms of utility, we have \\\[\\mathbb{P}\\left\[\\ell(M(x),x) \\le \\min_{y \\in \\mathcal{Y}} \\ell(y,x) + \\frac{2\\Delta}{\\varepsilon} \\log \\left(\\frac{\|\\mathcal{Y}\|}{\\beta}\\right) \\right\] \ge 1-\\beta \tag{3}\\\] and <a id="exutil" />\\\[\\mathbb{E}[\\ell(M(x),x)] \\le \\min_{y \\in \\mathcal{Y}} \\ell(y,x) + \\frac{2\\Delta}{\\varepsilon} \\log \|\\mathcal{Y}\| \tag{4}\\\] for all \\\(x \\in \\mathcal{X}^n\\\) and all \\\(\\beta>0\\\) [[DR14](https://www.cis.upenn.edu/~aaroth/Papers/privacybook.pdf "Cynthia Dwork, Aaron Rooth. The Algorithmic Foundations of Differential Privacy. 2014.") Corollary 3.12, [BNSSSU16](https://arxiv.org/abs/1511.02513 "Raef Bassily, Kobbi Nissim, Adam Smith, Thomas Steinke, Uri Stemmer, Jonathan Ullman. Algorithmic Stability for Adaptive Data Analysis. STOC 2016.") Lemma 7.1].

These utility guarantees are pretty good! The excess loss scales logarithmically with the number of options \\\(|\\mathcal{Y}|\\\) and it scales linearly in the sensitivity \\\(\\Delta\\\) and the reciprocal of the privacy loss \\\(1/\\varepsilon\\\), which is what we would expect.
The question we're looking at in this post is when these utility guarantees are _optimal_. That is, for what tasks can we prove that no mechanisms achieves better privacy and utility (up to constants)?

## Warmup: Packing Lower Bound for Pure DP
Let's start with a result for pure \\\(\\varepsilon,0\\\)-differential privacy.
We will instantiate \\\(\\mathcal{X} := \\{0,1\\}^d\\\), \\\(\\mathcal{Y} := \\{1,2,\\cdots,d\\}\\\), and \\\[\\ell\(y,x\) := \\sum\_i^n (x\_i)\_y\tag{5},\\\] so we can think of an input \\\(x \\in \\mathcal{X}^n\\\) as a 0/1-valued matrix \\\(x \\in \\{0,1\\}^{n \\times d}\\\) with \\\(n\\\) rows (corresponding to people) and \\\(d\\\) columns (corresponding to options); the goal of private selection is thus to pick a column with as few 1s as posssible subject to row-level differential privacy.

> **Theorem 1.** <a id="packinglb" />
> Let \\\(\\mathcal{X} := \\{0,1\\}^d\\\), \\\(\\mathcal{Y} := \\{1,2,\\cdots,d\\}\\\), and \\\(\\ell(y,x) := \\sum\_i^n (x_i)_y\\\).
> Let \\\(M : \\mathcal{X}^n \to \\mathcal{Y}\\\) satisfy \\\(\(\\varepsilon,0\)\\\)-differential privacy.
> If \\\(n \\le \\frac1\\varepsilon \\log\(d-1\)\\\), then there exists an input \\\(x \\in \\mathcal{X}^n\\\) such that 
\\\[\\mathbb{E}\[\\ell(M\(x\),x)\] \\ge \\min\_{y \\in \\mathcal{Y}} \\ell(y,x) + n/2 . \tag{6}\\\]

Before jumping into the proof, let's compare [Theorem 1](#packinglb) with [Equation 4](#exutil): If we set \\\(n = \\frac4\\varepsilon \\log\(d\)\\\) in [Equation 4](#exutil), we get \\\(\\mathbb{E}[\\ell(M(x),x)] \\le \\min_{y \\in \\mathcal{Y}} \\ell(y,x) + n/2\\\), which matches [Theorem 1](#packinglb) up to the factor of \\\(4\\\) in \\\(n\\\).

_Proof._
We will restrict ourselves to inputs \\\(x \\in \\{0,1\\}^{n \\times d}\\\) with one column of 0s and the other \\\(d-1\\\) columns being all 1s.
We enumerate these inputs as \\\(x^1, x^2, \\cdots, x^d\\\), where \\\(x^k\\\) has 0s in the \\\(k\\\)-th column and 1s everywhere else.
For these inputs the loss is always either \\\(0\\\) or \\\(n\\\). Namely, \\\(\\ell(y,x^y)=0\\\) and \\\(\\ell(y,x^k)=n\\\) for all \\\(k \\ne y\\\). Furthermore, the optimal loss is always \\\(0\\\).

For the sake of contradiction, assume that, for all \\\(k \in \\{1,2,\cdots,d\\}\\\), \\\[\\mathbb{E}\[\\ell(M\(x^k\),x^k)\] &lt; \\min\_{y \\in \\mathcal{Y}} \\ell(y,x^k) + \\alpha n,\\\] where the value of \\\(\\alpha\\\) will be set later.

By our choice of datasets, we have \\\(\\min\_{y \\in \\mathcal{Y}} \\ell(y,x^k) = 0 \\\) and \\\(\\mathbb{E}\[\\ell\(M\(x^k\),x^k\)\] = \(1 - \\mathbb{P}\[M\(x^k\)=k\]\) n \\\) for all \\\(k\\\).
Thus our assumption implies \\\(\\mathbb{P}\[M\(x^k\)=k\] &gt; 1-\\alpha\\\) for all \\\(k\\\).

Now we apply group privacy: The distance between \\\(x^1\\\) and \\\(x^k\\\) is \\\(n\\\). Thus \\\(\\mathbb{P}\[M\(x^k\)=k\] \\le e^{n\\varepsilon} \\mathbb{P}\[M\(x^1\)=k\]\\\) for all \\\(k\\\).
This rearranges to \\\[ \\forall k ~~~~~ \\mathbb{P}\[M\(x^1\)=k\] \\ge e^{-n\\varepsilon} \\mathbb{P}\[M\(x^k\)=k\] &gt; e^{-n\\varepsilon}(1-\\alpha).\\\]
Finally, we sum up probabilities: \\\[ 1 = \\mathbb{P}\[M\(x^1\)=1\] + \\sum\_{k=2}^d \\mathbb{P}\[M\(x^1\)=k\] &gt; (1-\\alpha) + (d-1) e^{-n\\varepsilon}(1-\\alpha).\\\]
This rearranges to \\\[\alpha &gt; 1-\\frac{1}{1 + (d-1)e^{-n\\varepsilon}} = \\frac{d-1}{e^{n\\varepsilon}+d-1}.\\\]
If \\\(n \\le \\frac1\\varepsilon \\log\(d-1\)\\\), then we have \\\(\\alpha &gt; \\frac12\\\).
Thus setting \\\(\\alpha=\\frac12\\\) yields a contradiction to the assumption and proves our result.
&#8718;
