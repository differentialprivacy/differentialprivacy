---
layout: post
title: "Limits of Privacy Amplification by Subsampling"
comments: true
authors:
  - thomassteinke
timestamp: 7:00:00 -0700
categories: [Algorithms]
---

In [our previous post](/subsampling) we gave a brief introduction to privacy amplification by subsampling.
The high-level story is that we can make differentially private algorithms faster by runninng them on a subsample of the dataset instead of the whole dataset and this comes at essentially no cost in privacy and accuracy. 
That story is pretty good. But now let's take a closer look at it.

Recall that we're comparing the standard Laplace mechanism \\\(M\(x\) := \\frac{1}{n}\\sum\_{x\_i \\in x} q\(x\_i\) + \\mathsf{Laplace}\\left\(\\frac{1}{\\varepsilon n}\\right\)\\\) to the subsampled Laplace mechanism \\\(\\widetilde{M}\_{p}\(x\) := \\frac{1}{pn} \\sum\_{x\_i \\in S\_p(x)} q\(x\_i\) + \\mathsf{Laplace}\\left\(\\frac{1}{\\varepsilon_p p n}\\right\)\\\), where \\\(S\_p\(x\)\\subseteq x\\\) is a random Poisson subsample that includes each person's data independently with probability \\\(p\\\).
The respective mean squared error guarantees are
<a id="eq1" />\\\[\\mathbb{E}\\left\[\\left\(M\(x\) - \\frac{1}{n}\\sum\_{x\_i \\in x} q\(x\_i\)\\right\)^2\\right\] = \\frac{2}{\\varepsilon^2 n^2}. \\tag{1}\\\]
and
<a id="eq2" />
\\\[ \\mathbb{E}\\left\[\\left\(\\widetilde{M}\_p\(x\) - \\frac{1}{n}\\sum\_{x\_i \\in x} q\(x\_i\) \\right\)^2\\right\]  \\le \\frac{\|x\|}{p n^2} + \\frac{2}{\\varepsilon\_p^2 p^2 n^2} \\approx \\frac{1}{p n} + \\frac{2}{\\varepsilon^2 n^2},\\tag{2}\\\]
where 
<a id="eq3" />\\\[\\varepsilon_p = \\log\\left\(1 + \\frac{1}{p} \\big\( e^{\\varepsilon}-1 \\big\)\\right\) \\approx \\frac{\\varepsilon}{p}. \\tag{3} \\\]

Comparing [Equation 1](#eq1) with [Equation 2](#eq2), there are two differences: The non-private statistical error \\\(\\frac{1}{p n}\\\) and the approximation from [Equation 3](#eq3). We'll ignore the non-private statistical error in this post, since it isn't the dominant error term for reasonable parameter regimes and, well, this is _DifferentialPrivacy.org_ not _Statistics.org_.

 the approximation we made in [Equation 6](#eq6):
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
