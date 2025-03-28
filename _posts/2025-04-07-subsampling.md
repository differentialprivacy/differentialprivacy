---
layout: post
title: "Limits of Privacy Amplification by Subsampling"
comments: true
authors:
  - thomassteinke
timestamp: 7:00:00 -0700
categories: [Algorithms]
---

Privacy Amplification by Subsampling is an important property of differential privacy. 
It is key to making many algorithms efficient.
Thus a lot of work has gone into analyzing this phenomenon.
In this post we will give a quick introduction to privacy amplification by subsampling and its applications. Furthermore we're going to look at the limitations of privacy amplification by subsampling -- i.e., what _can't_ it do.

## What is Privacy Amplification by Subsampling?

The premise of privacy amplification by subsampling is that we start with a (large) dataset \\\(x\\\) and we pick a (small) random subset \\\(S\(x\) \\subseteq x\\\) and run a DP algorithm \\\(M\\\) on that subset.
The question is: _What are the privacy properties of the combined algorithm \\\(M \\circ S\\\)?_
The answer depends on both the privacy properties of base algorithm \\\(M\\\) and the subsampling procedure \\\(S\\\).

Intuitively, there are two reasons why the combined algorithm \\\(M \\circ S\\\) should have better privacy properties than the base algorithm \\\(M\\\):
First, there is some probability \\\(p\\\) that your data \\\(x\_i\\\) is included in the subsample i.e. \\\(p = \\mathbb{P}\[x\_i\\in S\(x\)\]\\\).[^up] But, with probability \\\(1-p\\\), your data is _not_ included. When your data is not included, you have perfect privacy.
Second, the privacy adversary does not know whether or not your data is included in the subsample. This ambiguity enhances your privacy even in the case where your data is included.[^amb] 

There are different possible subsampling procedures \\\(S\\\).
A natural subsampling scheme is for the subsample \\\(S\(x\)\\\) to be a fixed-size subset of the dataset \\\(x\\\) that is otherwise uniformly random.
However, it turns out to work better if each person's data is included independently. This subsampling procedure is known as Poisson subsampling.[^poisson] In this case, the size of the subsample is not fixed. Assuming each person's data is included with the same probability \\\(p\\\), the size is binomially distributed: \\\[\|S\(x\)\| \\sim \\mathsf{Binomial}\(\|x\|,p\).\\\]
It also turns out to be easier to analyze differential privacy with respect to addition or removal of one person's data, rather than with respect to replacement.

There are _many_ privacy ampliffication by subsampling results in the literature. The gist of them is pretty much the same; the differences are about the specific assumptions they make and how tight they are. Next we'll state and prove a very simple version.

> **Theorem 1 (Privacy Amplification by Subsampling for Poisson-Subsampled Approximate DP).** <a id="thm1" />
> Let \\\(S : \\mathcal{X}^\* \\to \\mathcal{X}^\*\\\) be the Poisson subsampling operation with probability \\\(p\\\). That is, for all inputs \\\(x\\\), we have \\\(S\(x\) \\subseteq x\\\) where each \\\(x\_i \\in x\\\) is included in \\\(S\(x\)\\\) independently with probability \\\(p\\\).
> Let \\\(M : \\mathcal{X}^\* \\to \\mathcal{Y}\\\) satisfy \\\(\(\\varepsilon,\\delta\)\\\)-differential privacy with respect to addition or removal of one person's data.
> Let \\\(M \\circ S : \\mathcal{X}^\* \\to \\mathcal{Y}\\\) denote the combined algorithm that first subsamples and then runs \\\(M\\\) -- i.e., \\\(M \\circ S \(x\) = M\(S\(x\)\)\\\) for all \\\(x\\\).
> Then \\\(M \\circ S\\\) satisfies \\\(\(\\varepsilon',\\delta'\)\\\)-differential privacy for \\\[\\varepsilon' = \\log\\big\(1+p\(\\exp\(\\varepsilon\)-1\)\\big\) \~\~\~\~ \\text{ and } \~\~\~\~ \\delta' = p \\delta. \\\]


---

[^up]: For simplicity we assume that the probability of inclusion \\\(\\mathbb{P}\[x\_i\\in S\(x\)\]\\\) is the same for all individuals \\\(i\\\). In general, it can be different, in which case we would work with the largest probability \\\(p = \\max\_i \\mathbb{P}\[x\_i\\in S\(x\)\]\\\). 

[^amb]: Under pure differential privacy, there is no privacy amplification by subsampling when the adversary knows whether or not your data was included in the subsample. (However, under approximate or R&eacute;nyi differential privacy there is some amplification, but less than when the subsample remains secret.)

[^poisson]: I have no idea why it's called Poisson subsampling instead of Binomial subsampling. Intuitively, the reason independent inclusion is better than having a fixed-size subsample is that, if the size of the subsample is known, then knowing whether other people's data is included or excluded reveals information about whether your data is included or excluded. 

[^replace]: 
