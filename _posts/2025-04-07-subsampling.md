---
layout: post
title: "Privacy Amplification by Subsampling"
comments: true
authors:
  - thomassteinke
timestamp: 7:00:00 -0700
categories: [Algorithms]
---

Privacy Amplification by Subsampling is an important property of differential privacy. 
It is key to making many algorithms efficient -- particularly in machine learning applications.
Thus a lot of work has gone into analyzing this phenomenon.
In this post we will give a quick introduction to privacy amplification by subsampling and its applications. 
Furthermore, we're going to look at the limitations of privacy amplification by subsampling -- i.e., what _can't_ it do.

## What is Privacy Amplification by Subsampling?

The premise of privacy amplification by subsampling is that we start with a (large) dataset \\\(x\\\) and we pick a (small) random subset \\\(S\(x\) \\subseteq x\\\) and run a DP algorithm \\\(M\\\) on that subset.[^notation]
The question is: _What are the privacy properties of the combined algorithm \\\(M \\circ S\\\)?_
The answer depends on both the privacy properties of base algorithm \\\(M\\\) and the subsampling procedure \\\(S\\\).

Intuitively, there are two reasons why the combined algorithm \\\(M \\circ S\\\) should have better privacy properties than the base algorithm \\\(M\\\):
First, there is some probability \\\(p\\\) that your data \\\(x\_i\\\) is included in the subsample -- i.e. \\\(p = \\mathbb{P}\[x\_i\\in S\(x\)\]\\\).[^up] But, with probability \\\(1-p\\\), your data is _not_ included. And, when your data is not included, you have perfect privacy.
Second, the privacy adversary does not know whether or not your data is included in the subsample. This ambiguity enhances your privacy even in the case where your data is included.[^amb] 

There are different possible subsampling procedures \\\(S\\\).
A natural subsampling scheme is for the subsample \\\(S\(x\)\\\) to be a fixed-size subset of the dataset \\\(x\\\) that is otherwise uniformly random.
However, it turns out to work better if each person's data is included _independently_. This subsampling procedure is known as Poisson subsampling.[^poisson] We denote Poisson subsampling by \\\(S\_p\\\), where \\\(p\\in\[0,1\]\\\) is the probability of inclusion. 
In this case, the size of the subsample is not fixed. Assuming each person's data is included with the same probability \\\(p\\\), the size is binomially distributed: <a id="eq1" />\\\[\|S\_p\(x\)\| \\sim \\mathsf{Binomial}\(\|x\|,p\).\\tag{1}\\\]
It also turns out to be easier to analyze differential privacy with respect to addition or removal of one person's data, rather than with respect to replacement.

There are _many_ privacy ampliffication by subsampling results in the literature. The gist of them is pretty much the same; the differences are about the specific assumptions they make and how tight they are. Next we'll state and prove a very simple version.

> **Theorem 1 (Privacy Amplification by Subsampling for Poisson-Subsampled Approximate DP).** <a id="thm1" />
> Let \\\(S\_p : \\mathcal{X}^\* \\to \\mathcal{X}^\*\\\) be the Poisson subsampling operation with probability \\\(p\\\).[^notation] That is, for all inputs \\\(x\\\), we have \\\(S\_p\(x\) \\subseteq x\\\) where each \\\(x\_i \\in x\\\) is included in \\\(S\_p\(x\)\\\) independently with probability \\\(p\\\).
> Let \\\(M : \\mathcal{X}^\* \\to \\mathcal{Y}\\\) satisfy \\\(\(\\varepsilon,\\delta\)\\\)-differential privacy with respect to addition or removal of one person's data.
> Let \\\(M \\circ S\_p : \\mathcal{X}^\* \\to \\mathcal{Y}\\\) denote the combined algorithm that first subsamples and then runs \\\(M\\\) -- i.e., \\\(M \\circ S\_p \(x\) = M\(S\_p\(x\)\)\\\) for all \\\(x\\\).
> Then \\\(M \\circ S\_p\\\) satisfies \\\(\(\\varepsilon',\\delta'\)\\\)-differential privacy with respect to addition or removal of one person's data for <a id="eq2" />\\\[\\varepsilon' = \\log\\big\(1+p\(\\exp\(\\varepsilon\)-1\)\\big\) \~\~\~\~ \\text{ and } \~\~\~\~ \\delta' = p \\delta. \\tag{2}\\\]

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
Let's assume we have a large dataset \\\(x\\in\\mathcal{X}^\*\\\) and a query \\\(q:\\mathcal{X}\\to\[0,1\]\\\) and our goal is to privately estimate the average value of the query on the dataset \\\(\\frac{1}{n}\\sum\_{x\_i \\in x} q\(x\_i\)\\\).

The obvious solution is the Laplace mechanism: <a id="eq3" />\\\[M\(x\) := \\frac{1}{n}\\sum\_{x\_i \\in x} q\(x\_i\) + \\mathsf{Laplace}\\left\(\\frac{1}{\\varepsilon n}\\right\).\\tag{3}\\\]
This is \\\(\\varepsilon\\\)-differentially private and has mean squared error <a id="eq4" />\\\[\\mathbb{E}\\left\[\\left\(M\(x\) - \\frac{1}{n}\\sum\_{x\_i \\in x} q\(x\_i\)\\right\)^2\\right\] = \\frac{2}{\\varepsilon^2 n^2}. \\tag{4}\\\]
However, this takes time linear in the size of the dataset \\\(x\\\); that may be OK for one query, but, if we need to answer \\\(k\\\) queries \\\(q\_1,q\_2,\\cdots,q\_k\\\), this would take \\\(\\Omega\(k\|x\|\)\\\) time.

Suppose we can subsample from the dataset in sublinear time.[^supp] Ideally, suppose we can compute \\\(S\_p\(x\)\\\) in \\\(O\(p\|x\|\)\\\) time (on average).
Then we can run the Laplace mechanism on the subsample: <a id="eq5" />
\\\[\\widetilde{M}\_{p}\(x\) := \\frac{1}{pn} \\sum\_{x\_i \\in S\_p(x)} q\(x\_i\) + \\mathsf{Laplace}\\left\(\\frac{1}{\\varepsilon_p p n}\\right\) .\\tag{5}\\\]
This is faster, but how does it compare in terms of privacy and accuracy?

Before privacy amplification by subsampling, \\\(\\widetilde{M}_p\\\) satisfies \\\(\\varepsilon\_p\\\)-differential privacy.
Applying [Theorem 1](#thm1) we conclude that it satisfies \\\(\\varepsilon'\\\)-differential privacy with \\\(\\varepsilon' = \\log\(1+p\(e^{\\varepsilon\_p}-1\)\)\\\).
If we want to set \\\(\\varepsilon_p\\\) to achieve \\\(\\varepsilon'=\\varepsilon\\\), we can invert this formula to get <a id="eq6" />\\\[\\varepsilon_p = \\log\\left\(1 + \\frac{1}{p} \\big\( e^{\\varepsilon}-1 \\big\)\\right\) \\approx \\frac{\\varepsilon}{p}. \\tag{6} \\\]
The approximation comes from the first order Taylor series: \\\(\\log\(1+v\) = v+O\(v^2\)\\\) and \\\(e^v-1 = v+O\(v^2\)\\\) for \\\(v\\to0\\\).

On the accuracy front, we have <a id="eq7" />\\\[ \\mathbb{E}\\big\[\\widetilde{M}\_p\(x\)\\big\] = \\frac{1}{n}\\sum\_{x\_i \\in x} q\(x\_i\) .\\tag{7}\\\] That is, \\\(\\widetilde{M}\_p\\\) is unbiased.
In terms of variance, we have <a id="eq8" />
\\\[ \\mathbb{E}\\left\[\\left\(\\widetilde{M}\_p\(x\) - \\frac{1}{n}\\sum\_{x\_i \\in x} q\(x\_i\) \\right\)^2\\right\] = \\frac{p\(1-p\)}{p^2 n^2} \\sum\_{x\_i \\in x} q\(x\_i\)^2 + \\frac{2}{\\varepsilon\_p^2 p^2 n^2}\\\]
\\\[\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~ \\le \\frac{\|x\|}{p n^2} + \\frac{2}{\\varepsilon\_p^2 p^2 n^2}\\\]
\\\[\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~ \\approx \\frac{1}{p n} + \\frac{2}{\\varepsilon^2 n^2}.\\tag{8}\\\]
In the last step we substitute in the approximation from [Equation 6](#eq6).[^nx]

Now let's compare the linear-time mechanism \\\(M\\\) with the subsampled mechanism \\\(\\widetilde{M}\_p\\\): 
We have the same privacy guarantee.
Comparing the accuracy guarantee in [Equation 4](#eq4) with that in [Equation 8](#eq8) we see two differences -- the approximation (more on that shortly) and the extra \\\(\\frac{1}{pn}\\\) term.
This extra term is a low order term when <a id="eq9" />
 \\\[\\frac{1}{pn} \\le \\frac{1}{\\varepsilon^2 n^2} \\iff p \\ge \\varepsilon^2 n \\iff \\varepsilon \\le \\sqrt{\\frac{p}{n}}.\\tag{9}\\\]
 In other words, when \\\(\\varepsilon\\\) is sufficiently small, the statistical error \\\(\\frac{1}{\\sqrt{pn}}\\\) is dominated by the scale of the noise added for privacy \\\(\\frac{1}{\\varepsilon\_p p n}\\approx\\frac{1}{\\varepsilon n}\\\). 
 The statistical error is unrelated to privacy; it is something people are used to and we don't need to worry about it too much.
 
  **The upshot is that, for sufficiently small values of \\\(\\varepsilon\\\), the error of the subsampled Laplace mechanism \\\(\\widetilde{M}\_p\\\) is approximately the same as the standard Laplace mechanism \\\(M\\\).
 Thus we get a faster algorithm with essentially no cost in privacy and accuracy.**
 
This is very useful in machine learning applications, where the query \\\(q\\\) computes a gradient.
However, gradients are usually higher-dimensional, rather than one-dimensional.
This adds a bit of complexity, but doesn't fundamentally alter the story; essentially we need to analyze Gaussian noise rather than Laplace noise.

## The Limits of Privacy Amplification by Subsamplig
 
The story so far is pretty good. But let's take a closer look at the approximation we made in [Equation 6](#eq6):
<a id="eq6prime" />\\\[\\varepsilon_p = \\log\\left\(1 + \\frac{1}{p} \\big\( e^{\\varepsilon}-1 \\big\)\\right\) \\approx \\frac{\\varepsilon}{p}. \\tag{6'} \\\]
Specifically, we're interested in the scale of the Laplace noise: <a id="eq10" />\\\[\\frac{1}{\\varepsilon\_p p} = \\frac{1}{p\\log\\left\(1 + \\frac{1}{p} \\big\( e^{\\varepsilon}-1 \\big\)\\right\)} \\approx \\frac{1}{\\varepsilon}. \\tag{10} \\\]
To get an idea of how good this approximation is, let's plot the ratio <a id="eq11" />\\\[\\frac{p\\varepsilon\_p}{\\varepsilon} = \\frac{p}{\\varepsilon} \\log\\left\(1 + \\frac{1}{p} \\big\( e^{\\varepsilon}-1 \\big\)\\right\) \\approx 1:\\tag{11}\\\]
<p align="center"><img src="/images/subsampling-ratio-p.png" alt="Plot of p*eps\_p/eps as a function of p for eps=0.01,0.1,1,2" width="768" height="576"/></p> 
<p align="center"><img src="/images/subsampling-ratio-eps.png" alt="Plot of p*eps\_p/eps as a function of eps for p=0.001,0.01,0.1,0.5"  width="768" height="576"/></p> 
This doesn't look so good!
The approximation we made in [Equation 6](#eq6) tells us that all of the plotted lines should be close to 1.
But this seems to only be accurate when \\\(p \\approx 1\\\) or when \\\(\\varepsilon\\\) is _very_ small.
Large subsampling probability \\\(p\\\) doesn't make much sense for subsampling; we don't get much speedup. So the question is _how small does the privacy parameter \\\(\\varepsilon\\\) need to be?_

Roughly, if we want the approximation in [Equation 6](#eq6) to be good within constant factors, then  the privacy parameter \\\(\\varepsilon\\\) needs to scale linearly with the subsampling probability \\\(p\\\). I.e., \\\(\\varepsilon=cp\\\) for a constant \\\(c\\\). Let's see what the ratio looks like for various constants:
<p align="center"><img src="/images/subsampling-ratio-c.png" alt="Plot of p*eps\_p/eps as a function of p for eps=p*const where const=0.2,0.5,2,5"  width="768" height="576"/></p> 
This looks better. In particular, if \\\(\\varepsilon \\le 2p\\\), then \\\(\\frac{p\\varepsilon\_p}{\\varepsilon} \\ge \\frac{1}{2}\\\), which means the subsampled Laplace mechanism adds at most twice as much noise as the standard Laplace mechanism.
 
In general, if we set \\\(\\varepsilon=cp\\\) and take the infimum over all \\\(p\\in\[0,1\]\\\) of the ratio in [Equation 11](#eq11), we get 
<a id="eq12" />\\\[\\inf\_{p\\in\[0,1\],\\varepsilon=cp}\\frac{p}{\\varepsilon} \\log\\left\(1 + \\frac{1}{p} \\big\( e^{\\varepsilon}-1 \\big\)\\right\) = \\frac{1}{c} \\log\\big\( 1+c\\big\).\\tag{12}\\\] In other words, if \\\(\\varepsilon \\le cp\\\), then the subsampled Laplace mechanism adds at most \\\(\\frac{c}{\\log\(1+c\)}\\\) times as much noise as the standard Laplace mechanism. 
Here's what this function looks like:
<p align="center"><img src="/images/subsampling-ratio-lim.png" alt="Plot of c/log(1+c as a function of c"  width="768" height="576"/></p> 

This bound on the ratio seems reasonable as long as \\\(c\\\) isn't large. 
However, assuming \\\(\\varepsilon \\le cp\\\) is a pretty strong assumption!
This is the big limitation of privacy amplification by subsampling -- subsampling is free only when the privacy parameter is tiny.

## Conclusion

To summarize, we showed that privacy amplification by subsampling can be used to make differentially private algorithms faster. 
This comes at essentially no cost in privacy and accuracy, but only if the privacy parameter is tiny. Specifically, the privacy parameter needs to be on the order of the subsampling probability -- i.e., \\\(\\varepsilon=O\(p\)\\\) -- for the claim to hold.


 
---

[^up]: For simplicity, we assume that the probability of inclusion \\\(\\mathbb{P}\[x\_i\\in S\_p\(x\)\]\\\) is the same for all individuals \\\(i\\\). In general, it can be different, in which case we would work with the largest probability \\\(p = \\max\_i \\mathbb{P}\[x\_i\\in S\_p\(x\)\]\\\).  

[^amb]: Under pure differential privacy, there is no privacy amplification by subsampling when the adversary knows whether or not your data was included in the subsample. (However, under approximate or R&eacute;nyi differential privacy there is some amplification, but less than when the subsample remains secret.)

[^poisson]: Intuitively, the reason independent inclusion is better than having a fixed-size subsample is that, if the size of the subsample is known, then knowing whether other people's data is included or excluded reveals information about whether your data is included or excluded. I have no idea why it's called Poisson subsampling instead of Binomial subsampling.

[^notation]: This post uses set notation \\\(x\_i \\in S\(x\) \\subseteq x\\\) somewhat informally. Things become a bit imprecise if there are duplicates -- i.e., \\\(x\_i=x\_j\\\) for \\\(i \\ne j\\\), so we assume this issue doesn't arise. To make things formal we could define the index set \\\(S\\\) of the subsample separate from the subsample \\\(S\(x\)\\\); then we would condition on \\\(i \\in S\\\) instead of \\\(x\_i \\in S\(x\)\\\). We use \\\(\\mathcal{X}^\* = \\bigcup\_{n=0}^\\infty \\mathcal{X}^n\\\) to denote the set of all finite tuples/multisets with elements in \\\(\\mathcal{X}\\\)

[^supp]: This is a nontrivial supposition. Often different subsampling schemes are used in practice because they are easier to implement than Poisson subsampling.

[^nx]: Sweeping details under the rug: Since we're defining differential privacy with respect to addition or removal of one person's data, the size of the dataset \\\(\|x\|\\\) is itself private. Thus we only assume that \\\(n \\approx \|x\|\\\).
