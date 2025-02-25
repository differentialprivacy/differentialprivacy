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
In terms of utility, we have \\\[\\mathbb{P}\\left\[\\ell(M(x),x) \\le \\min_{y \\in \\mathcal{Y}} \\ell(y,x) + \\frac{2\\Delta}{\\varepsilon} \\log \\left(\\frac{\|\\mathcal{Y}\|}{\\beta}\\right) \\right\] \ge 1-\\beta \tag{3}\\\] and \\\[\\mathbb{E}[\\ell(M(x),x)] \\le \\min_{y \\in \\mathcal{Y}} \\ell(y,x) + \\frac{2\\Delta}{\\varepsilon} \\log \|\\mathcal{Y}\| \tag{4}\\\] for all \\\(x \\in \\mathcal{X}^n\\\) and all \\\(\\beta>0\\\) [[DR14](https://www.cis.upenn.edu/~aaroth/Papers/privacybook.pdf "Cynthia Dwork, Aaron Rooth. The Algorithmic Foundations of Differential Privacy. 2014.") Corollary 3.12, [BNSSSU16](https://arxiv.org/abs/1511.02513 "Raef Bassily, Kobbi Nissim, Adam Smith, Thomas Steinke, Uri Stemmer, Jonathan Ullman. Algorithmic Stability for Adaptive Data Analysis. STOC 2016.") Lemma 7.1].

These utility guarantees are pretty good! The excess loss scales logarithmically with the number of options \\\(|\\mathcal{Y}|\\\) and it scales linearly in the sensitivity \\\(\\Delta\\\) and the reciprocal of the privacy loss \\\(1/\\varepsilon\\\), which is what we would expect.
The question we're looking at in this post is when these utility guarantees are _optimal_. That is, for what tasks can we prove that no mechanisms achieves better privacy and utility (up to constants)?

## Packing Lower Bound for Pure DP
Let's start with a result for pure \\\(\\varepsilon,0\\\)-differential privacy.
