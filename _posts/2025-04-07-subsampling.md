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
However, it turns out to work better if each person's data is included independently. This subsampling procedure is known as Poisson subsampling.[^poisson] In this case, the size of the subsample is not fixed. Assuming each person's data is included with the same probability \\\(p\\\), the size is binomially distributed: \\\[\|S\_p\(x\)\| \\sim \\mathsf{Binomial}\(\|x\|,p\).\\\]
It also turns out to be easier to analyze differential privacy with respect to addition or removal of one person's data, rather than with respect to replacement.

There are _many_ privacy ampliffication by subsampling results in the literature. The gist of them is pretty much the same; the differences are about the specific assumptions they make and how tight they are. Next we'll state and prove a very simple version.

> **Theorem 1 (Privacy Amplification by Subsampling for Poisson-Subsampled Approximate DP).** <a id="thm1" />
> Let \\\(S\_p : \\mathcal{X}^\* \\to \\mathcal{X}^\*\\\) be the Poisson subsampling operation with probability \\\(p\\\). That is, for all inputs \\\(x\\\), we have \\\(S\_p\(x\) \\subseteq x\\\) where each \\\(x\_i \\in x\\\) is included in \\\(S\_p\(x\)\\\) independently with probability \\\(p\\\).
> Let \\\(M : \\mathcal{X}^\* \\to \\mathcal{Y}\\\) satisfy \\\(\(\\varepsilon,\\delta\)\\\)-differential privacy with respect to addition or removal of one person's data.
> Let \\\(M \\circ S\_p : \\mathcal{X}^\* \\to \\mathcal{Y}\\\) denote the combined algorithm that first subsamples and then runs \\\(M\\\) -- i.e., \\\(M \\circ S\_p \(x\) = M\(S\_p\(x\)\)\\\) for all \\\(x\\\).
> Then \\\(M \\circ S\_p\\\) satisfies \\\(\(\\varepsilon',\\delta'\)\\\)-differential privacy for \\\[\\varepsilon' = \\log\\big\(1+p\(\\exp\(\\varepsilon\)-1\)\\big\) \~\~\~\~ \\text{ and } \~\~\~\~ \\delta' = p \\delta. \\\]

_Proof._
Let \\\(x \\in \\mathcal{X}^\*\\\) and \\\(x\_i \\in x \\\) be arbitrary. Let \\\(x'=x\\setminus\\{x\_i\\}\\\) be \\\(x\\\) with \\\(x\_i\\\) removed. Let \\\(T \\subseteq \\mathcal{Y}\\\) be arbitrary.
We have <br/>
\\\(\\mathbb{P}\[M\(S\_p\(x\)\) \\in T \] = \(1-p\) \\mathbb{P}\[M\(S\_p\(x\)\) \\in T \\mid x\_i \\notin S\_p\(x\)\] + p \\mathbb{P}\[M\(S\_p\(x\)\) \\in T \\mid x\_i \\in S\_p\(x\)\] \\\)
\\\(\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~ = \(1-p\) \\mathbb{P}\[M\(S\_p\(x'\)\) \\in T\] + p \\mathbb{P}\[M\(S\_p\(x'\)\\cup\{x\_i\}\) \\in T\]\\\)<br/>
\\\(\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~ \\le \(1-p\) \\mathbb{P}\[M\(S\_p\(x'\)\) \\in T\] + p \(e^\\varepsilon \\mathbb{P}\[M\(S\_p\(x'\)\) \\in T\] + \\delta \) \\\)<br/>
\\\(\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~ = \(1-p + p e^\\varepsilon \) \\mathbb{P}\[M\(S\_p\(x'\)\) \\in T\] + p \\delta. \\\) <br/>
Here we are using the fact that \\\(S\_p\(x\)\\\) conditioned on \\\(x\_i \\notin S\_p\(x\)\\\) is just \\\(S\_p\(x'\)\\\) and the fact that \\\(S\_p\(x\)\\\) conditioned on \\\(x\_i \\in S\_p\(x\)\\\) is just \\\(S\_p\(x'\)\\cup\{x\_i\}\\\).
This establishes half of the result. The other direction is similar:<br/>
\\\(\\mathbb{P}\[M\(S\_p\(x\)\) \\in T \] = \(1-p\) \\mathbb{P}\[M\(S\_p\(x\)\) \\in T \\mid x\_i \\notin S\_p\(x\)\] + p \\mathbb{P}\[M\(S\_p\(x\)\) \\in T \\mid x\_i \\in S\_p\(x\)\] \\\)
\\\(\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~ = \(1-p\) \\mathbb{P}\[M\(S\_p\(x'\)\) \\in T\] + p \\mathbb{P}\[M\(S\_p\(x'\)\\cup\{x\_i\}\) \\in T\]\\\)<br/>
\\\(\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~ \\ge \(1-p\) \\mathbb{P}\[M\(S\_p\(x'\)\) \\in T\] + p e^{-\\varepsilon}\( \\mathbb{P}\[M\(S\_p\(x'\)\) \\in T\] - \\delta \) \\\)<br/>
\\\(\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~ = \(1-p+p e^{-\\varepsilon}\) \\mathbb{P}\[M\(S\_p\(x'\)\) \\in T\] - p e^{-\\varepsilon} \\delta \\\)<br/>
This rearranges to<br/>
\\\( \\mathbb{P}\[M\(S\_p\(x'\)\) \\in T\] \\le \\frac{\\mathbb{P}\[M\(S\_p\(x\)\) \\in T \]+p e^{-\\varepsilon}\\delta}{1-p+p e^{-\\varepsilon}} \\le \(1-p+pe^\\varepsilon\)\\mathbb{P}\[M\(S\_p\(x\)\) \\in T \] + p\\delta,\\\)<br/>
as required. (The inequalities \\\(\\frac{1}{1-p+pe^{-\\varepsilon}} \\le 1-p+pe^\\varepsilon\\\) and \\\(\\frac{e^{-\\varepsilon}}{1-p+pe^{-\\varepsilon}} \\le 1\\\) are left as exercises for the reader.)
&#8718;

## Why is Privacy Amplification by Subsampling Useful?

Lets work out a simplified illustrative example for why privacy amplification by subsampling is useful.
Let's assume we have a large dataset \\\(x\\in\\mathcal{X}^n\\\) and a query \\\(q:\\mathcal{X}\\to\[0,1\]\\\) and our goal is to privately estimate \\\(\\frac{1}{n}\\sum\_{x\_i \\in x} q\(x\_i\)\\\).

The obvious solution is the Laplace mechanism: \\\[M\(x\) := \\frac{1}{n}\\sum\_{x\_i \\in x} q\(x\_i\) + \\mathsf{Laplace}\(\\frac{1}{\\varepsilon n}\).\\\]
This is \\\(\\varepsilon\\\)-differentially private and has error \\\(O\(1/\\varepsilon n\)\\\).
However, this takes time linear in the size of the dataset \\\(x\\\); that may be OK for one query, but, if we need to answer \\\(k\\\) queries \\\(q\_1,q\_2,\\cdots,q\_k\\\), this would take \\\(\\Omega\(k\|x\|\)\\\) time.

Suppose we can subsample from the dataset in sublinear time.[^supp] Ideally, suppose we can compute \\\(S\_p\(x\)\\\) in \\\(O\(p\|x\|\)\\\) time.
Then we can run the Laplace mechanism on the subsample:
\\\[\\widetilde{M}\_{p}\(x\) := \\frac{1}{pn} \\sum\_{x\_i \\in S\_p(x)} q\(x\_i\) + \\mathsf{Laplace}\(\\frac{1}{\\varepsilon_p p n}\) .\\\]
This is faster, but how does it compare in terms of privacy and accuracy?

Before privacy amplification by subsampling, \\\(\\widetilde{M}_p\\\) satisfies \\\(\\varepsilon\_p\\\)-differential privacy.
Applying [Theorem 1](#thm1) we conclude that it satisfies \\\(\\varepsilon'\\\)-differential privacy with \\\(\\varepsilon' = \\log\(1+p\(e^{\\varepsilon\_p}-1\)\)\\\).
If we want to set \\\(\\varepsilon_p\\\) to achieve a certain value of \\\(\\varepsilon'\\\), we can invert this formula to get \\\(\\varepsilon_p = \\log\\big\(1 + \\frac{1}{p} \\big\( e^{\\varepsilon'}-1 \\big\)\\big\)\\\).

On the accuracy front, we have \\\[ \\mathbb{E}\\big\[\\widetilde{M}\_p\(x\)\\big\] = \\frac{1}{n}\\sum\_{x\_i \\in x} q\(x\_i\) .\\\] That is, \\\(\\widetilde{M}\_p\\\) is unbiased.
 
---

[^up]: For simplicity we assume that the probability of inclusion \\\(\\mathbb{P}\[x\_i\\in S\_p\(x\)\]\\\) is the same for all individuals \\\(i\\\). In general, it can be different, in which case we would work with the largest probability \\\(p = \\max\_i \\mathbb{P}\[x\_i\\in S\_p\(x\)\]\\\).  

[^amb]: Under pure differential privacy, there is no privacy amplification by subsampling when the adversary knows whether or not your data was included in the subsample. (However, under approximate or R&eacute;nyi differential privacy there is some amplification, but less than when the subsample remains secret.)

[^poisson]: I have no idea why it's called Poisson subsampling instead of Binomial subsampling. Intuitively, the reason independent inclusion is better than having a fixed-size subsample is that, if the size of the subsample is known, then knowing whether other people's data is included or excluded reveals information about whether your data is included or excluded. 

[^notation]: This post uses set notation \\\(x\_i \\in S\_p\(x\) \\subset x\\\) somewhat informally. 

[^supp]: This is a nontrivial supposition. Often different subsampling schemes are used in practice because they are easier to implement than Poisson subsampling.
