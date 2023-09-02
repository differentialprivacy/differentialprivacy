---
layout: post
title: "Beyond Global Sensitivity via Inverse Sensitivity"
comments: true
authors:
  - thomassteinke
timestamp: 10:00:00 -0700
categories: [Algorithms]
---

The most well-known and widely-used method for achieving differential privacy is to compute the true function value \\\(f(x)\\\) and then add Laplace or Gaussian noise scaled to the _global sensitivity_ of \\\(f\\\). 
This may be overly conservative. In this post we'll show how we can do better.

The global sensitivity of a function \\\(f : \\mathcal{X}^\* \to \\mathbb{R}\\\) is defined by \\\[ \mathsf{GS}\_f := \\sup\_\{x,x'\\in\\mathcal{X}^\* : \\mathrm{dist}(x,x') \\le 1\} \|f(x)-f(x')\|, \\tag{1}\\\] where \\\(\\mathrm{dist}(x,x')\\le 1\\\) denotes that \\\(x\\\) and \\\(x'\\\) are neighbouring datasets (i.e. they differ only by the addition or removal of one person's data); more generally, \\\(\\mathrm{dist}(\\cdot,\\cdot)\\\) is the corresponding metric on datasets (e.g., Hamming distance).[^1]

The global sensitivity considers datasests that have nothing to do with the dataset at hand and which could be completely unrealistic.
Many functions have infinite global sensitivity, but, on reasonably nice datasets, their _local sensitivity_ is much lower.

## Local Sensitivity

The local sensitivity of a function \\\(f : \\mathcal{X}^\* \to \\mathbb{R}\\\) at \\\(x \\in \\mathcal{X}^\*\\\) at distance \\\(k\\\) is defined by \\\[\\mathsf{LS}^k\_f(x) := \\sup\_\{x'\\in\\mathcal{X}^\* : \\mathrm{dist}(x,x') \\le k\} \|f(x)-f(x')\|. \\tag{2}\\\]
Often, we fix \\\(k=1\\\) and we may drop the superscript: \\\(\\mathsf{LS}\_f(x) := \\mathsf{LS}\_t^1(x)\\\).
Note that the local sensitivity is always at most the global sensitivity: \\\(\\mathsf{LS}\_f^k(x) \\le k \\cdot \\mathsf{GS}\_f\\\).

As a concrete example, the median has infinite global sensitivity, but for realistic data the local sensitivity is quite reasonable. 
Specifically, \\\[\\mathsf{LS}^k\_{\\mathrm{median}}(x\_1, \\cdots, x\_n) = \\max\\left\\\{ \\left\|x\_{\(\\tfrac{n+1}{2}\)}-x\_{\(\\tfrac{n+1}{2}+k\)}\\right\|, \\left\|x\_{\(\\tfrac{n+1}{2}\)}-x\_{\(\\tfrac{n+1}{2}-k\)}\\right\| \\right\\\},\\tag{3}\\\] where \\\( x\_{(1)} \\le x\_{(2)} \\le \\cdots \\le x\_{(n)}\\\) denotes the input in [sorted order](https://en.wikipedia.org/wiki/Order_statistic) and \\\(n\\\) is assumed to be odd, so, in particular, \\\(\\mathrm{median}(x\_1, \\cdots, x\_n) = x\_{\(\\tfrac{n+1}{2}\)}\\\).
For example, if \\\(X\_1, \\cdots X\_n\\\) are i.i.d. samples from a standard Gaussian and \\\(k \\ll n\\\), then \\\(\\mathsf{LS}^k\_{\\mathrm{median}}(X\_1, \\cdots, X\_n) \le O(k/n)\\\) with high probability.

## Using Local Sensitivity

Intuitively, the local sensitivity is the "real" sensitivity of the function and the global sensitivity is only a worst-case upper bound.
Thus it seems natural to add noise scaled to the local sensitivity instead of the global sensitivity. 

Unfortunately, na&iuml;vely adding noise scaled to local sensitivity doesn't satisfy differential privacy. 
The problem is that the local sensitivity itself can reveal information.
For example, consider the median on the inputs \\\(x=(1,2,2),x'=(2,2,2)\\\). The output distributions of the algorithm on these two inputs must be similar.
In both cases the median is \\\(2\\\), so that is a good start for ensuring that the distributions are similar. 
But the local sensitivity is different: \\\(\\mathsf{LS}^1\_{\\mathrm{median}}(x)=1\\\) versus \\\(\\mathsf{LS}^1\_{\\mathrm{median}}(x')=0\\\). 
So, if we add noise scaled to local sensitivity, then, on input \\\(x'\\\), we deterministically output \\\(2\\\), while, on input \\\(x\\\), we output a random number. If we use continuous Laplace or Gaussian noise, then the random number will be a non-integer almost surely. Thus the output perfectly distinguishes the two inputs, which is a catastrophic violation of differential privacy.

The good news is that we can exploit local sensitivity; we just need to do a bit more work.
In fact, there are many methods in the differential privacy literature to exploit local sensitivity.

The best-known methods for exploiting local sensitivity are _smooth sensitivity_ [[NRS07](https://cs-people.bu.edu/ads22/pubs/NRS07/NRS07-full-draft-v1.pdf "Kobbi Nissim, Sofya Raskhodnikova, Adam Smith. Smooth Sensitivity and Sampling in Private Data Analysis. STOC 2007.")][^2] and _propose-test-release_ [[DL09](https://www.stat.cmu.edu/~jinglei/dl09.pdf "Cynthia Dwork, Jing Lei. Differential Privacy and Robust Statistics. STOC 2009.")][^3].


In this post we will cover a different general-purpose technique. This technique is folklore.[^4] It was first systematically studied by Asi and Duchi [[AD20](https://arxiv.org/abs/2005.10630 "Hilal Asi, John Duchi. Near Instance-Optimality in Differential Privacy. 2020."),[AD20](https://papers.nips.cc/paper/2020/hash/a267f936e54d7c10a2bb70dbe6ad7a89-Abstract.html "Hilal Asi, John Duchi. Instance-optimality in differential privacy via approximate inverse sensitivity mechanisms. NeurIPS 2020.")], who also named the method the _inverse sensitivity mechanism_.

## The Inverse Sensitivity Mechanism

Consider a function \\\(f : \\mathcal{X}^\* \\to \\mathcal{Y}\\\).
Our goal is to estimate \\\(f(x)\\\) in a differentially private manner.
But we do not make any assumptions about the global sensitivity of the function.

For simplicity we will assume that \\\(\\mathcal{Y}\\\) is finite and that \\\(f\\\) is [surjective](https://en.wikipedia.org/wiki/Surjective_function).[^5]

Now we define a loss function \\\(\ell : \mathcal{X}^\* \times \mathcal{Y} \to \mathbb{Z}\_{\ge0}\\\) by \\\[\ell(x,y) := \\min\\left\\\{ \mathrm{dist}(x,\tilde{x}) : \tilde{x}\in\mathcal{X}^\*, f(\tilde{x})=y \\right\\\}.\\tag{4}\\\]
In other words, \\\(\\ell(x,y)\\\) measures how many entries of \\\(x\\\) we need to add or remove until \\\(f(x)=y\\\). 
Yet another way to think of it is that \\\(\\ell(x,y)\\\) is the distance from the point \\\(x\\\) to the set \\\(f^{-1}(y)\\\). (Hence the name inverse sensitivity.)

The loss is minimized by the desired answer: \\\(\ell(x,f(x))=0\\\). Intuitively, the loss \\\(\ell(x,y)\\\) increases as \\\(y\\\) moves further from \\\(f(x)\\\). So approximately minimizing this loss should produce a good approximation to \\\(f(x)\\\), as desired.

The trick is that this loss always has bounded global sensitivity -- i.e., \\\(\\mathsf{GS}\_\\ell \\le 1\\\) -- no matter what the sensitivity of \\\(f\\\) is!

> **Lemma 1.** Let \\\(f : \\mathcal{X}^\* \to \\mathcal{Y}\\\) be arbitrary and define \\\(\ell : \mathcal{X}^\* \times \mathcal{Y} \to \mathbb{Z}\_{\ge0}\\\) as in Equation 4. Then, for all \\\(x,x'\\in\\mathcal{X}^\*\\\) with \\\(\\mathrm{dist}(x,x')\\le 1\\\) and all \\\(y \\in \\mathcal{Y}\\\), we have \\\(\|\\ell(x,y)-\\ell(x',y)\|\\le 1\\\).

> _Proof._ 
> Fix \\\(x,x'\\in\\mathcal{X}^\*\\\) with \\\(\\mathrm{dist}(x,x')\\le 1\\\) and \\\(y \\in \\mathcal{Y}\\\).
> Let \\\(\\widehat{x} \\in\\mathcal{X}^\*\\\) satisfy \\\(\ell(x,y)=\\mathrm{dist}(x,\widehat{x})\\\) and \\\(f(\widehat{x})=y\\\).
> By definition, \\\[\ell(x',y) = \\min\\left\\\{ \mathrm{dist}(x',\tilde{x}) : f(\tilde{x})=y \\right\\\} \le \\mathrm{dist}(x',\widehat{x}).\\\]
> By the triangle inequality, \\\[\\mathrm{dist}(x',\widehat{x}) \le \\mathrm{dist}(x',x)+\\mathrm{dist}(x,\widehat{x}) \le 1 + \ell(x,y).\\\]
> Thus \\\(\ell(x',y) \le \\ell(x,y)+1\\\) and, by symmetry, \\\(\ell(x,y) \le \\ell(x',y)+1\\\), as required. &#8718;

This means that we can run the exponential mechanism [[MT07](https://ieeexplore.ieee.org/document/4389483 "Frank McSherry, Kunal Talwar. Mechanism Design via Differential Privacy. FOCS 2007.")] to select from \\\(\\mathcal{Y}\\\) using the loss \\\(\\ell\\\).[^6] That is, the inverse sensitivity mechanism is defined by 
\\\[\\forall y \\in \\mathcal{Y} ~~~~~ \\mathbb{P}\[M(x)=y\] ;= \\frac{\\exp\\left\(-\frac{\\varepsilon}{2}\\ell(x,y)\\right)}{\sum_{y'\\in\\mathcal{Y}}\\exp\\left\(-\frac{\\varepsilon}{2}\\ell(x,y')\\right)}.\\tag{5}\\\] 
By the properties of the exponential mechanism and Lemma 1, \\\(M\\\) satisfies differential privacy:

> **Theorem 2. (Privacy of the Inverse Sensitivity Mechanism)** Let \\\(M : \\mathcal{X}^\* \to \\mathcal{Y}\\\) be as defined in Equation 5 with the loss from Equation 4. Then \\\(M\\\) satisfies \\\(\\varepsilon\\\)-differential privacy ([and \\\(\\frac18\\varepsilon^2\\\)-zCDP](/exponential-mechanism-bounded-range/)).

## Utility Guarantee

The privacy guarantee of the inverse sensitivity mechanism is easy and, in particular, it doesn't depend on the properties of \\\(f\\\).
This means that the utility will need to depend on the properties of \\\(f\\\).

By the standard properties of the exponential mechanism, we can guaranatee that the output has low loss:

> **Lemma 3.** Let \\\(M : \\mathcal{X}^\* \to \\mathcal{Y}\\\) be as defined in Equation 5 with the loss from Equation 4. For all inputs \\\(x \\in \\mathcal{X}^\*\\\) and all \\\(\\beta\\in(0,1)\\\), we have \\\[\\mathbb{P}\\left\[\\ell(x,M(x)) < \\frac2\\varepsilon\\log\\left\(\\frac{\|\\mathcal{Y}\|}{\\beta}\\right\) \\right\] \ge 1-\beta.\\tag{6}\\\]

> _Proof._
> Let \\\(B_x = \\left\\{ y \in \\mathcal{Y} : \\ell(x,y) \\ge \\frac2\\varepsilon\\log\\left\(\\frac{\|\\mathcal{Y}\|}{\\beta}\\right\) \\right\\}\\\) be the subset of \\\(\\mathcal{Y}\\\) with high loss.
> Then \\\[ \\mathbb{P}\[M(x)\\in B_x\] = \\frac{\\sum_{y \in B_x} \\exp\\left\(-\frac{\\varepsilon}{2}\\ell(x,y)\\right)}{\sum_{y'\\in\\mathcal{Y}}\\exp\\left\(-\frac{\\varepsilon}{2}\\ell(x,y')\\right)} \\\]\\\[ \\le \\frac{\|B_x\| \cdot \\exp\\left\(-\frac{\\varepsilon}{2}\\frac2\\varepsilon\\log\\left\(\\frac{\|\\mathcal{Y}\|}{\\beta}\\right\) \\right)}{\\exp\\left\(-\frac{\\varepsilon}{2}\\ell(x,f(x))\\right)}\\\]\\[= \frac{\|B_x\| \cdot \frac{\beta}{\|\mathcal{Y}\|}}{1} \le \beta, \\\] as required. &#8718;

Now we need to translate this loss bound into something easier to interpret -- local sensitivity.

Suppose \\\(y \\gets M(x)\\\). Then we have some loss \\\(k=\\ell(x,y)\\\). What this means is that there exists \\\(\\tilde{x}\\in\\mathcal{X}^\*\\\) with \\\(f(\\tilde{x})=y\\\) and \\\(\\mathrm{dist}(x,\\tilde{x})\\le k\\\). By the definition of local sensitivity, \\\(\|f(x)-y\| = \|f(x)-f(\\tilde{x})\| \\le \\mathsf{LS}\_f^k(x)\\\). This means we can translate the loss guarantee of Lemma 3 into an accuracy guarantee in terms of local sensitivity:

> **Theorem 4. (Utility of the Inverse Sensitivity Mechanism)** Let \\\(M : \\mathcal{X}^\* \to \\mathcal{Y}\\\) be as defined in Equation 5 with the loss from Equation 4. For all inputs \\\(x \\in \\mathcal{X}^\*\\\) and all \\\(\\beta\\in(0,1)\\\), we have \\\[\\mathbb{P}\\left\[\\left\|M(x)-f(x)\\right\| \\le \\mathsf{LS}\_f^k(x) \\right\] \ge 1-\beta,\\tag{7}\\\] where \\\(k=\\left\\lfloor\\frac2\\varepsilon\\log\\left\(\\frac{\|\\mathcal{Y}\|}{\\beta}\\right\)\\right\\rfloor\\\).

We can tie this back to our concrete example of the median. Per Equation 3, \\\[\\mathsf{LS}^k\_{\\mathrm{median}}(x\_1, \\cdots, x\_n) \\le \\left\|x\_{\(\\tfrac{n+1}{2}+k\)}-x\_{\(\\tfrac{n+1}{2}-k\)}\\right\| .\\\]
Thus the error guarantee of Theorem 4 for the median would scale with the spread of the data. E.g., if \\\(k=\tfrac{n+1}{4}\\\), then  \\\(\\mathsf{LS}^k\_{\\mathrm{median}}(x\_1, \\cdots, x\_n)\\\) is at most the interquartile range of the data.

How does this compare with the usual global sensitivity approach?
The \\\(\\varepsilon\\\)-differentially private Laplace mechanism is given by \\\(\\widehat{M}(x):=f(x)+\mathsf{Laplace}(\\mathsf{GS}\_f/\varepsilon)\\\). For all \\\(x \\in \\mathcal{X}^\*\\\) and all \\\(\\beta\\in(0,1)\\\), we have the utility guarantee \\\[\\mathbb{P}\\left\[\\left\|\\widehat{M}(x)-f(x)\\right\| \\le \\mathsf{GS}\_f \\cdot \\frac1\\varepsilon \\log\\left\(\\frac{1}{2\\beta}\\right\) \\right\] \ge 1-\beta.\\tag{8}\\\]
Comparing Equations 7 and 8, we see that neither guarantee dominates the other. On one hand, the local sensitivity can be much smaller than the global sensitivity. On the other hand, we pick up a dependence on \\\(\\log\|\\mathcal{Y}\|\\\) and an extra factor of 2 in \\\(\\varepsilon\\\).

## Conclusion

In this post we've covered the inverse sensitivity mechanism and showed that it is private regardless of the sensitivity of the function \\\(f\\\) and we showed that it gives error guarantees that scale with the local sensitivity of \\\(f\\\), rather than its global sensitivity.

The inverse sensitivity mechanism is a simple demonstration that there is more to differential privacy than simply adding noise scaled to global sensitivity; there are many more techniques in the literature.

The inverse sensitivity mechanism has two main limitations. First, it is, in general, not computationally efficient. Computing the loss function is intractable for an arbitrary \\\(f\\\) (but can be done efficiently for simple examples like the median). Second, the \\\(\\log\|\\mathcal{Y}\|\\\) term in the accuracy guarantee is problematic when the output space is large, such as when we have high-dimensional outputs. 
While there are other techniques that can be used instead of inverse sensitivity, they suffer from some of the same limitations. Thus finding ways around these limitations is an [active research topic](/colt23-bsp/) [[BKSW19](https://arxiv.org/abs/1905.13229 "Mark Bun, Gautam Kamath, Thomas Steinke, Zhiwei Steven Wu. Private Hypothesis Selection. NeurIPS 2019."),[HKMN23](https://arxiv.org/abs/2212.05015 "Samuel B. Hopkins, Gautam Kamath, Mahbod Majid, Shyam Narayanan. Robustness Implies Privacy in Statistical Estimation. STOC 2023."),[FDY22](https://cse.hkust.edu.hk/~yike/ShiftedInverse.pdf "Juanru Fang, Wei Dong, Ke Yi. Shifted Inverse: A General Mechanism for Monotonic Functions under User Differential Privacy. CCS 2022."),[DHK23](https://arxiv.org/abs/2301.07078 "John Duchi, Saminul Haque, Rohith Kuditipudi. A Fast Algorithm for Adaptive Private Mean Estimation. COLT 2023."),[BHS23](https://arxiv.org/abs/2301.12250 "Gavin Brown, Samuel B. Hopkins, Adam Smith. Fast, Sample-Efficient, Affine-Invariant Private Mean and Covariance Estimation for Subgaussian Distributions. COLT 2023."),[AUZ23](https://arxiv.org/abs/2302.01855 "Hilal Asi, Jonathan Ullman, Lydia Zakynthinou. From Robustness to Privacy and Back. 2023.")].

We leave you with a riddle: What can we do if even the local sensitivity of our function is unbounded? For example, suppose we want to approximate \\\(f(x) = \\max\_i x_i\\\). Surprisingly, there are still things we can do and we intend to write a follow-up post on this.


---

[^1]: We define \\\(\\mathcal{X}^\* = \\bigcup\_{n = 0}^\\infty \\mathcal{X}^n\\\) to be the set of all input tuples of arbitrary size. For simplicity, we consider univariate functions here. But the definitions of global and local sensitivity easily extend to to vector-valued functions by taking a norm: \\\[ \mathsf{GS}\_f := \\sup\_\{x,x'\\in\\mathcal{X}^\* : \\mathrm{dist}(x,x') \\le 1\} \\\|f(x)-f(x')\\\|.\\\] If we use the 2-norm, then this cleanly corresponds to adding spherical Gaussian noise. The 1-norm corresponds to adding independent Laplace noise to the coordinates. 

[^2]: Briefly, smooth sensitivity is an upper bound on the local sensitivity which itself has low sensitivity in a multiplicative sense. That is, \\\(\mathsf{LS}\_f^1(x) \le \mathsf{SS}_f^t(x)\\\) and \\\(\mathsf{SS}_f^t(x) \le e^t \\cdot \mathsf{SS}_f^t(x') \\\) for neighbouring \\\(x,x'\\\). This suffices to ensure that we can add noise scaled to \\\(\mathsf{SS}_f^t(x)\\\). However, that noise usually needs to be more heavy-tailed than for global sensitivity [[BS19](https://proceedings.neurips.cc/paper/2019/hash/3ef815416f775098fe977004015c6193-Abstract.html "Mark Bun, Thomas Steinke. Average-Case Averages: Private Algorithms for Smooth Sensitivity and Mean Estimation. NeurIPS 2019.")]. 

[^3]: Roughly, the propose-test-release framework computes an upper bound on the local sensitivity in a differentially private manner and then uses this upper bound as the noise scale. (We hope to give more detail about both propose-test-release and smooth sensitivity in future posts.)

[^4]: Properly attributing the inverse sensitivity mechanism is difficult. The  earliest published instances of the inverse sensitivity mechanism of which we are aware of are from 2011 and 2013 [[MMNW11](https://www.cs.columbia.edu/~rwright/Publications/pods11.pdf "Darakhshan Mir, S. Muthukrishnan, Aleksandar Nikolov, Rebecca N. Wright. Pan-private algorithms via statistics on sketches. PODS 2011.")&sect;3.1,[JS13](hhttps://www.ncbi.nlm.nih.gov/pmc/articles/PMC4681528/ "Aaron Johnson, Vitaly Shmatikov. Privacy-preserving data exploration in genome-wide association studies. KDD 2013.")&sect;5]; but this was not novel even then. Asi and Duchi [[AD20](https://arxiv.org/abs/2005.10630 "Hilal Asi, John Duchi. Near Instance-Optimality in Differential Privacy. 2020.")&sect;1.2] state that McSherry and Talwar [[MT07](https://ieeexplore.ieee.org/document/4389483 "Frank McSherry, Kunal Talwar. Mechanism Design via Differential Privacy. FOCS 2007.")] considered it in 2007. In any case, the name we use was coined in 2020 [[AD20](https://arxiv.org/abs/2005.10630 "Hilal Asi, John Duchi. Near Instance-Optimality in Differential Privacy. 2020.")].

[^5]: Assuming that the output space \\\(\\mathcal{Y}\\\) is finite is a significant assumption. While it can be relaxed, it is to some extent an unavoidable limitation of the technique. To apply the inverse sensitivity mechanism to the median we must discretize and bound the inputs; bounding the inputs does impose a finite global sensitivity, but the dependence on the bound is logarithmic, so the bound can be fairly large. Assuming that the function is surjective is a minor assumption that ensures that the loss is always well-defined; otherwise we can define the loss to be infinite for points that are not in the range of the function.

[^6]: Note that we can use other selection algorithms, such as permute-and-flip [[MS20](https://arxiv.org/abs/2010.12603 "Ryan McKenna, Daniel Sheldon. Permute-and-Flip: A new mechanism for differentially private selection. NeurIPS 2020.")] or report-noisy-max [[DKSSWXZ21](https://arxiv.org/abs/2105.07260 "Zeyu Ding, Daniel Kifer, Sayed M. Saghaian N. E., Thomas Steinke, Yuxin Wang, Yingtai Xiao, Danfeng Zhang. The Permute-and-Flip Mechanism is Identical to Report-Noisy-Max with Exponential Noise. 2021.")] or gap-max [[CHS14](https://arxiv.org/abs/1409.2177 "Kamalika Chaudhuri, Daniel Hsu, Shuang Song. The Large Margin Mechanism for Differentially Private Maximization. NIPS 2014."),[BDRS18](https://dl.acm.org/doi/10.1145/3188745.3188946 " Mark Bun, Cynthia Dwork, Guy N. Rothblum, Thomas Steinke. Composable and versatile privacy via truncated CDP. STOC 2018."),[BKSW19](https://arxiv.org/abs/1905.13229 "Mark Bun, Gautam Kamath, Thomas Steinke, Zhiwei Steven Wu. Private Hypothesis Selection. NeurIPS 2019.")].
