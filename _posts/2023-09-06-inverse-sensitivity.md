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

The global sensitivity of a function \\\(f : \\mathcal{X}^\* \to \\mathbb{R}\\\) is defined by \\\[ \mathsf{GS}\_f := \\sup\_\{x,x'\\in\\mathcal{X}^\* : \\mathrm{dist}(x,x') \\le 1\} \|f(x)-f(x')\|,\\\] where \\\(\\mathrm{dist}(x,x')\\le 1\\\) denotes that \\\(x\\\) and \\\(x'\\\) are neighbouring datasets (i.e. they differ only by the addition or removal of one person's data); more generally, \\\(\\mathrm{dist}(\\cdot,\\cdot)\\\) is the corresponding metric on datasets.

The global sensitivity considers datasests that have nothing to do with the dataset at hand and which could be completely unrealistic.
Many functions have infinite global sensitivity, but, on reasonably nice datasets, their _local sensitivity_ is much lower.

## Local Sensitivity

The local sensitivity of a function \\\(f : \\mathcal{X}^\* \to \\mathbb{R}\\\) at \\\(x \\in \\mathcal{X}^\*\\\) at distance \\\(k\\\) is defined by \\\[\\mathsf{LS}^k\_f(x) := \\sup\_\{x'\\in\\mathcal{X}^\* : \\mathrm{dist}(x,x') \\le k\} \|f(x)-f(x')\|.\\\]
\(Usually, we fix \\\(k=1\\\) and we may drop the superscript: \\\(\\mathsf{LS}\_f(x) := \\mathsf{LS}\_t^1(x)\\\).\)

As a concrete example, the median has infinite global sensitivity, but for realistic data the local sensitivity is quite reasonable. 
Specifically, \\\[\\mathsf{LS}^k\_{\\mathrm{median}}(x\_1, \\cdots, x\_n) = \\max\\left\\\{ \\left\|x\_{\(\\tfrac{n+1}{2}\)}-x\_{\(\\tfrac{n+1}{2}+k\)}\\right\|, \\left\|x\_{\(\\tfrac{n+1}{2}\)}-x\_{\(\\tfrac{n+1}{2}-k\)}\\right\| \\right\\\},\\\] where \\\( x\_{(1)} \\le x\_{(2)} \\le \\cdots \\le x\_{(n)}\\\) denotes the input in [sorted order](https://en.wikipedia.org/wiki/Order_statistic) and \\\(n\\\) is assumed to be odd, so, in particular, \\\(\\mathrm{median}(x\_1, \\cdots, x\_n) = x\_{\(\\tfrac{n+1}{2}\)}\\\).
For example, if \\\(X\_1, \\cdots X\_n\\\) are i.i.d. samples from a standard Gaussian and \\\(k \\ll n\\\), then \\\(\\mathsf{LS}^k\_{\\mathrm{median}}(X\_1, \\cdots, X\_n) \le O(k/n)\\\) with high probability.

## Using Local Sensitivity

Intuitively, the local sensitivity is the "real" sensitivity of the function and the global sensitivity is only a worst-case upper bound.
Thus it seems natural to add noise scaled to the local sensitivity instead of the global sensisitivy. 

Unfortunately, na&iuml;vely adding noise scaled to local sensitivity doesn't satisfy differential privacy. 
The problem is that the local sensitivity itself can reveal information.
For example, consider the median on the inputs \\\(x=(1,2,2),x'=(2,2,2)\\\). In both cases the median is \\\(2\\\). But the local sensitivity is different: \\\(\\mathsf{LS}^1\_{\\mathrm{median}}(x)=1\\\) versus \\\(\\mathsf{LS}^1\_{\\mathrm{median}}(x')=0\\\). So, if we add noise scaled to local sensitivity, then, on input \\\(x'\\\), we deterministically output \\\(2\\\), while, on input \\\(x\\\), we output a random number. If we use continuous Laplace or Gaussian noise, then the random number will be a non-integer. Thus the output perfectly distinguishes the two inputs, which is a catastrophic violation of differential privacy.

The good news is that we can exploit local sensitivity; we just need to do a bit more work.
In fact, there are many methods in the differential privacy literature to exploit local sensitivity.

The best-known methods for exploiting local sensitivity are _smooth sensitivity_ [[NRS07](https://cs-people.bu.edu/ads22/pubs/NRS07/NRS07-full-draft-v1.pdf "Kobbi Nissim, Sofya Raskhodnikova, Adam Smith. Smooth Sensitivity and Sampling in Private Data Analysis. STOC 2007.")][^1] and _propose-test-release_ [[DL09](https://www.stat.cmu.edu/~jinglei/dl09.pdf "Cynthia Dwork, Jing Lei. Differential Privacy and Robust Statistics. STOC 2009.")][^2].


In this post we will cover a different general-purpose technique known as the _inversse sensitivity mechanism_. This technique is folklore.[^3] It was first systematically studied by Asi and Duchi [[AD20](https://arxiv.org/abs/2005.10630 "Hilal Asi, John Duchi. Near Instance-Optimality in Differential Privacy. 2020."),[AD20](https://papers.nips.cc/paper/2020/hash/a267f936e54d7c10a2bb70dbe6ad7a89-Abstract.html "Hilal Asi, John Duchi. Instance-optimality in differential privacy via approximate inverse sensitivity mechanisms. NeurIPS 2020.")], who also named the method.

## The Inverse Sensitivity Mechanism

Consider a function \\\(f : \\mathcal{X}^\* \\to \\mathcal{Y}\\\).
Our goal is to estimate \\\(f(x)\\\) in a differentially private manner.
But we do not make any assumptions about the global sensitivity of the function.

For simplicity we will assume that \\\(\\mathcal{Y}\\\) is finite and that \\\(f\\\) is [surjective](https://en.wikipedia.org/wiki/Surjective_function).[^4]

Now we define a loss function \\\(\ell : \mathcal{X}^\* \times \mathcal{Y} \to \mathbb{Z}\_{\ge0}\\\) by \\\[\ell(x,y) := \\min\\left\\\{ \mathrm{dist}(x,\tilde{x}) : f(\tilde{x})=y \\right\\\}.\\\]
The loss is minimized by the desired answer: \\\(\ell(x,f(x))=0\\\). Intuitively, the loss \\\(\ell(x,y)\\\) increases as \\\(y\\\) moves further from \\\(f(x)\\\). So approximately minimizing this loss should produce a good approximation to \\\(f(x)\\\), as desired.

The trick is that this loss always has bounded global sensitivity -- \\\(\\mathsf{GS}\_\\ell \\le 1\\\) -- no matter what the sensitivity of \\\(f\\\) is!

---

[^1]: Briefly, smooth sensitivity is an upper bound on the local sensitivity which itself has low sensitivity in a multiplicative sense. That is, \\\(\mathsf{LS}\_f^1(x) \le \mathsf{SS}_f^t(x)\\\) and \\\(\mathsf{SS}_f^t(x) \le e^t \\cdot \mathsf{SS}_f^t(x') \\\) for neighbouring \\\(x,x'\\\). This suffices to ensure that we can add noise scaled to \\\(\mathsf{SS}_f^t(x)\\\). However, that noise usually needs to be more heavy-tailed than for global sensitivity [[BS19](https://proceedings.neurips.cc/paper/2019/hash/3ef815416f775098fe977004015c6193-Abstract.html "Mark Bun, Thomas Steinke. Average-Case Averages: Private Algorithms for Smooth Sensitivity and Mean Estimation. NeurIPS 2019.")]. 

[^2]: Roughly, the propose-test-release framework computes an upper bound on the local sensitivity in a differentially private manner and then uses this upper bound as the noise scale. (We hope to give more detail about both propose-test-release and smooth sensitivity in future posts.)

[^3]: Properly attributing the inverse sensitivity mechanism is difficult. The  earliest published instance of the inverse sensitivity mechanism of which we are aware of is from 2011 [[MMNW11](https://www.cs.columbia.edu/~rwright/Publications/pods11.pdf "Darakhshan Mir, S. Muthukrishnan, Aleksandar Nikolov, Rebecca N. Wright. Pan-private algorithms via statistics on sketches. PODS 2011.") &sect;3.1]; but this was not novel even then. Asi and Duchi [[AD20](https://arxiv.org/abs/2005.10630 "Hilal Asi, John Duchi. Near Instance-Optimality in Differential Privacy. 2020.") &sect;1.2] state that McSherry and Talwar [[MT07](https://ieeexplore.ieee.org/document/4389483 "Frank McSherry, Kunal Talwar. Mechanism Design via Differential Privacy. FOCS 2007.")] considered it in 2007. In any case, the idea was only named in 2020 [[AD20](https://arxiv.org/abs/2005.10630 "Hilal Asi, John Duchi. Near Instance-Optimality in Differential Privacy. 2020.")].

[^4]: Assuming that the output space \\\(\\mathcal{Y}\\\) is finite is a significant assumption. While it can be relaxed, it is to some extent an unavoidabile limitation of the technique. To apply the inverse sensitivity mechanism to the median we must discretize and bound the inputs; bounding the inputs does impose a finite global sensitivity, but the dependence on the bound is logarithmic, so the bound can be a large.
