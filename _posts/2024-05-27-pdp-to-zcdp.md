---
layout: post
title: "Sharp RDP & zCDP from Pure DP"
comments: true
authors:
  - thomassteinke
timestamp: 10:00:00 -0700
categories: [Algorithms]
---

There are multiple ways to quantify differential privacy, including pure DP \[[DMNS06](https://journalprivacyconfidentiality.org/index.php/jpc/article/view/405 "Cynthia Dwork, Frank McSherry, Kobbi Nissim, Adam Smith. Calibrating Noise to Sensitivity in Private Data Analysis. 2006.")\], approximate DP \[[DKMMN06](https://link.springer.com/chapter/10.1007/11761679_29 "Cynthia Dwork, Krishnaram Kenthapadi, Frank McSherry, Ilya Mironov, Moni Naor. Our Data, Ourselves: Privacy Via Distributed Noise Generation. 2006.")\], Concentrated DP \[[DR16](https://arxiv.org/abs/1603.01887 "Cynthia Dwork, Guy N. Rothblum. Concentrated Differential Privacy. 2016."),[BS16](https://arxiv.org/abs/1605.02065 "Mark Bun, Thomas Steinke. Concentrated Differential Privacy: Simplifications, Extensions, and Lower Bounds. 2016.")\], Renyi DP \[[M17](https://arxiv.org/abs/1702.07476 "Ilya Mironov. Renyi Differential Privacy. 2017.")\], Gaussian DP \[[DRS19](https://arxiv.org/abs/1905.02383 "Jinshuo Dong, Aaron Roth, Weijie J. Su. Gaussian Differential Privacy. 2019.")\], & function-DP \[[DRS19](https://arxiv.org/abs/1905.02383 "Jinshuo Dong, Aaron Roth, Weijie J. Su. Gaussian Differential Privacy. 2019.")\].
Fortunately, these definitions are similar enough that we can convert between most of them with some loss in parameters.

In this post we will discuss converting from pure DP to Renyi DP and concentrated DP. In particular, we will provide optimal results, which are an improvement on what is currently in the literature.
But first, let's recap the relevant definitions.

## Definitions: Pure DP, Renyi DP, & zCDP

For notational simplicity, we will assume the output space of algorithms is discrete and the algorithms outputs have full support.[^1]

> **Definition 1 (Pure DP):**
> A randomized algorithm \\\(M : \\mathcal{X}^n \\to \\mathcal{Y}\\\) satisfies \\\(\\varepsilon\\\)-differential privacy if, for all pairs of inputs \\\(x, x' \\in \\mathcal{X}^n\\\) differing only on the data of a single individual, we have \\\[\\forall y \\in \\mathcal{Y} ~~~~~ \\log\\left(\\frac{\\mathbb{P}\[M\(x\)=y\]}{\\mathbb{P}\[M\(x'\)=y\]}\\right) \\le \\varepsilon.\\\]

Pure DP is the simplest (and first) definition and is very convenient for analysis. 
Pure DP can also be called pointwise DP because the guarantee holds for all points \\\(y\\\), whereas all the other definitions either bound some quantity averaged over \\\(y\\\) or quantify over sets \\\(S \\subseteq \\mathcal{Y}\\\).

> **Definition 2 (Renyi DP):**
> A randomized algorithm \\\(M : \\mathcal{X}^n \\to \\mathcal{Y}\\\) satisfies \\\(\(\\alpha,\\widehat\\varepsilon\)\\\)-Renyi differential privacy if, for all pairs of inputs \\\(x, x' \\in \\mathcal{X}^n\\\) differing only on the data of a single individual, we have \\\[ \\frac{1}{\\alpha-1} \\log \\left\( \\underset{Y \\gets M\(x'\)}{\\mathbb{E}}\\left\[ \\left\( \\frac{\\mathbb{P}\[M\(x\)=Y\]}{\\mathbb{P}\[M\(x'\)=Y\]} \\right\)^\\alpha \\right\] \\right\) \\le \\widehat\\varepsilon.\\\]

Renyi DP is a more flexible definition than pure DP. But flexibility comes at the cost of complexity.
The definition has two parameters, but we can usually trade off these parameters. Thus it is often better to think of it as being parameterized by a function \\\(\\widehat\\varepsilon\(\\alpha\)\\\), which gives us a \\\(\(\\alpha,\\widehat\\varepsilon\(\\alpha\)\)\\\)-RDP bound for all \\\(\\alpha>1\\\) simultaneously.
However, in many cases -- such as the Gaussian mechanism -- the function is linear, or can be bounded by a linear function.

> **Definition 3 (zero-Concentrated DP (zCDP)):**
> A randomized algorithm \\\(M : \\mathcal{X}^n \\to \\mathcal{Y}\\\) satisfies \\\(\\rho\\\)-zCDP if, for all pairs of inputs \\\(x, x' \\in \\mathcal{X}^n\\\) differing only on the data of a single individual, we have \\\[ \\forall \\alpha > 1 ~~~~~ \\frac{1}{\\alpha-1} \\log \\left\( \\underset{Y \\gets M\(x'\)}{\\mathbb{E}}\\left\[ \\left\( \\frac{\\mathbb{P}\[M\(x\)=Y\]}{\\mathbb{P}\[M\(x'\)=Y\]} \\right\)^\\alpha \\right\] \\right\) \\le \\alpha\\rho.\\\]

This definition is equivalent to satisfying \\\(\(\\alpha,\\rho\\alpha\)\\\)-RDP for all \\\(\\alpha>1\\\); zCDP can be thought of as a single-parameter version of RDP which gives us many of the benefits of RDP without the complexity.

## Converting Pure DP to Renyi DP




A basic and frequent task in data analysis is _selection_ -- given a set of options \\\(\\mathcal{Y}\\), output the (approximately) best one, where "best" is defined by some loss function \\\(\\ell : \\mathcal{Y} \\times \\mathcal{X}^n \\to \\mathbb{R}\\\) and a dataset \\\(x \\in \\mathcal{X}^n\\\). That is, we want to output some \\\(y \\in \\mathcal{Y}\\\) that approximately minimizes \\\(\\ell(y,x)\\\). Naturally, we are interested in _private selection_ -- i.e., the output should be differentially private in terms of the dataset \\\(x\\\).
This post discusses algorithms for private selection -- in particular, we give an improved privacy analysis of the popular exponential mechanism.

## The Exponential Mechanism

The most well-known algorithm for private selection is the [_exponential mechanism_](https://en.wikipedia.org/wiki/Exponential_mechanism_(differential_privacy)) [**[MT07]**](https://doi.org/10.1109/FOCS.2007.66 "Frank McSherry, Kunal Talwar. Mechanism Design via Differential Privacy. FOCS 2007."). The exponential mechanism \\\(M : \\mathcal{X}^n \\to \\mathcal{Y} \\\) is a randomized algorithm given by \\\[\\forall x \\in \\mathcal{X}^n ~ \\forall y \\in \\mathcal{Y} ~~~~~ \\mathbb{P}[M(x) = y] = \\frac{\\exp(-\\frac{\\varepsilon}{2\\Delta} \\ell(y,x))}{\\sum_{y' \\in \\mathcal{Y}} \\exp(-\\frac{\\varepsilon}{2\\Delta} \\ell(y',x)) }, \\tag{1}\\\] where \\\(\\Delta\\\) is the sensitivity of the loss function \\\(\\ell\\\) given by \\\[\\Delta = \\sup_{x,x' \\in \\mathcal{X}^n : d(x,x') \\le 1} \\max_{y\\in\\mathcal{Y}} \|\\ell(y,x) - \\ell(y,x')\|,\tag{2}\\\] where the supremum is taken over all datasets \\\(x\\\) and \\\(x'\\\) differing on the data of a single individual (which we denote by \\\(d(x,x')\\le 1\\\)).

In terms of utility, we can easily show that [**[BNSSSU16]**](https://arxiv.org/abs/1511.02513 "Raef Bassily, Kobbi Nissim, Adam Smith, Thomas Steinke, Uri Stemmer, Jonathan Ullman. Algorithmic Stability for Adaptive Data Analysis. STOC 2016.") \\\[\\mathbb{E}[\\ell(M(x),x)] \\le \\min_{y \\in \\mathcal{Y}} \\ell(y,x) + \\frac{2\\Delta}{\\varepsilon} \\log \|\\mathcal{Y}\|\\\] for all \\\(x \\in \\mathcal{X}^n\\\) (and we can also give high probability bounds).

It is easy to show that the exponential mechanism satisfies \\\(\\varepsilon\\\)-differential privacy.
But there is more to this story! We're going to look at a more refined privacy analysis.

## Bounded Range

The privacy guarantee of the exponential mechanism is more precisely characterized by _bounded range_. This was observed and defined by David Durfee and Ryan Rogers [**[DR19]**](https://arxiv.org/abs/1905.04273 "David Durfee, Ryan Rogers. Practical Differentially Private Top-k Selection with Pay-what-you-get Composition. NeurIPS 2019") and further analyzed later [**[DDR20]**](https://arxiv.org/abs/1909.13830 "Jinshuo Dong, David Durfee, Ryan Rogers. Optimal Differential Privacy Composition for Exponential Mechanisms. ICML 2020.").

> **Definition 1 (Bounded Range).**[^1] 
> A randomized algorithm \\\(M : \\mathcal{X}^n \\to \\mathcal{Y}\\\) satisfies \\\(\\eta\\\)-bounded range if, for all pairs of inputs \\\(x, x' \\in \\mathcal{X}^n\\\) differing only on the data of a single individual, there exists some \\\(t \\in \\mathbb{R}\\\) such that \\\[\\forall y \\in \\mathcal{Y} ~~~~~ \\log\\left(\\frac{\\mathbb{P}[M(x)=y]}{\\mathbb{P}[M(x')=y]}\\right) \in [t, t+\eta].\\\] Here \\\(t\\\) may depend on the pair of input datasets \\\(x,x'\\\), but not on the output \\\(y\\\).

To interpret this definition, we [recall the definition of the privacy loss random variable](/flavoursofdelta/): Define \\\(f : \\mathcal{Y} \\to \\mathbb{R}\\\) by \\\[f(y) = \\log\\left(\\frac{\\mathbb{P}[M(x)=y]}{\\mathbb{P}[M(x')=y]}\\right).\\\] Then the privacy loss random variable \\\(Z \\gets \\mathsf{PrivLoss}(M(x)\\\|M(x'))\\\) is given by \\\(Z = f(M(x))\\\).

Pure \\\(\\varepsilon\\\)-differential privacy is equivalent to demanding that the privacy loss is bounded by \\\(\\varepsilon\\\) -- i.e., \\\(\\mathbb{P}[\|Z\|\\le\\varepsilon]=1\\\). Approximate \\\(\(\\varepsilon,\\delta\)\\\)-differential privacy is, roughly, equivalent to demanding that \\\(\\mathbb{P}[Z\\le\\varepsilon]\\ge1-\\delta\\\).[^2]

Now \\\(\\eta\\\)-bounded range is simply demanding that the privacy loss \\\(Z\\\) is supported on some interval of length \\\(\\eta\\\). This interval \\\([t,t+\\eta]\\\) may depend on the pair \\\(x,x'\\\).

Bounded range and pure differential privacy are equivalent up to a factor of 2 in the parameters:

> **Lemma 2 (Bounded Range versus Pure Differential Privacy).** 
> - \\\(\\varepsilon\\\)-differential privacy implies \\\(\\eta\\\)-bounded range with \\\(\\eta \\le 2\\varepsilon\\\).
> - \\\(\\eta\\\)-bounded range implies \\\(\\varepsilon\\\)-differential privacy with \\\(\\varepsilon \\le \\eta\\\).

_Proof._ The first part of the equivalence follows from the fact that pure \\\(\\varepsilon\\\)-differential privacy implies the privacy loss is supported on the interval \\\([-\\varepsilon,\\varepsilon]\\\). Thus, if we set \\\(t=-\\varepsilon\\\) and \\\(\\eta=2\\varepsilon\\\), then \\\([t,t+\\eta] = [-\\varepsilon,\\varepsilon]\\\).
The second part follows from the fact that the support of the privacy loss \\\([t,t+\\eta]\\\) must straddle \\\(0\\\). That is, the privacy loss cannot be always positive nor always negative, so \\\(0 \\in [t,t+\\eta]\\\) and, hence, \\\([t,t+\\eta] \\subseteq [-\\eta,\\eta]\\\). Otherwise \\\(\\forall y ~ f(y)>0\\\) or \\\(\\forall y ~ f(y)<0\\\)  would imply \\\(\\forall y ~ \\mathbb{P}[M(x)=y]>\\mathbb{P}[M(x')=y]\\\) or \\\(\\forall y ~ \\mathbb{P}[M(x)=y]<\\mathbb{P}[M(x')=y]\\\), contradicting the fact that \\\(\\sum_{y \\in \\mathcal{Y}} \\mathbb{P}[M(x)=y] = 1\\\) and \\\(\\sum_{y \\in \\mathcal{Y}} \\mathbb{P}[M(x')=y] = 1\\\). &#8718;

OK, back to the exponential mechanism:

> **Lemma 3 (The Exponential Mechanism is Bounded Range).**
> The exponential mechanism (given in Equation 1 above) satisfies \\\(\\varepsilon\\\)-bounded range .[^3]

_Proof._
We have \\\[e^{f(y)} = \\frac{\\mathbb{P}[M(x)=y]}{\\mathbb{P}[M(x')=y]} = \\frac{\\exp(-\\frac{\\varepsilon}{2\\Delta}\\ell(y,x))}{\\exp(-\\frac{\\varepsilon}{2\\Delta}\\ell(y,x'))} \\cdot \\frac{\\sum_{y'} \\exp(-\\frac{\\varepsilon}{2\\Delta} \\ell(y',x'))}{\\sum_{y'} \\exp(-\\frac{\\varepsilon}{2\\Delta} \\ell(y',x))}.\\\]
Setting \\\(t = \\log\\left(\\frac{\\sum_{y'} \\exp(-\\frac{\\varepsilon}{2\\Delta} \\ell(y',x'))}{\\sum_{y'} \\exp(-\\frac{\\varepsilon}{2\\Delta} \\ell(y',x))}\\right) - \\frac{\\varepsilon}{2}\\\), we have \\\[ f(y) = \\frac{\\varepsilon}{2\\Delta} (\\ell(y,x')-\\ell(y,x)+\\Delta) + t.\\\]
By the definition of sensitivity (given in Equation 2), we have \\\( 0 \\le \\ell(y,x')-\\ell(y,x)+\\Delta \\le 2\\Delta\\\), whence \\\(t \\le f(y) \\le t + \\varepsilon\\\). &#8718;

Bounded range is not really a useful privacy definition on its own. Thus we're going to relate it to a relaxed version of differential privacy next.

## Concentrated Differential Privacy

Concentrated differential privacy [**[BS16]**](https://arxiv.org/abs/1605.02065 "Mark Bun, Thomas Steinke. Concentrated Differential Privacy: Simplifications, Extensions, and Lower Bounds. TCC 2016.") and its variants [**[DR16]**](https://arxiv.org/abs/1603.01887 "Cynthia Dwork, Guy N. Rothblum. Concentrated Differential Privacy. 2016.") [**[M17]**](https://arxiv.org/abs/1702.07476 "Ilya Mironov. Rényi Differential Privacy. CCS 2017.") are relaxations of pure differential privacy with many nice properties. In particular, it composes very cleanly.

> **Definition 4 (Concentrated Differential Privacy).**
> A randomized algorithm \\\(M : \\mathcal{X}^n \\to \\mathcal{Y}\\\) satisfies \\\(\\rho\\\)-concentrated differential privacy if, for all pairs of inputs \\\(x, x' \\in \\mathcal{X}^n\\\) differing only on the data of a single individual, 
> \\\[\\forall \\lambda > 0 ~~~~~ \\mathbb{E}[\\exp( \\lambda Z)] \le \\exp(\\lambda(\\lambda+1)\\rho),\\tag{3}\\\]
> where \\\(Z \\gets \\mathsf{PrivLoss}(M(x)\\\|M(x'))\\\) is the privacy loss random variable.[^4]

Intuitively, concentrated differential privacy requires that the privacy loss is subgaussian. Specifically, the bound on the moment generating function of \\\(\\rho\\\)-concentrated differential privacy is tight if the privacy loss \\\(Z\\\) follows the distribution \\\(\\mathcal{N}(\\rho,2\\rho)\\\). Indeed, the privacy loss random variable of the Gaussian mechanism has such a distribution.[^5] 

OK, back to the exponential mechanism:
We know that \\\(\\varepsilon\\\)-differential privacy implies \\\(\\frac12 \\varepsilon^2\\\)-concentrated differential privacy [**[BS16]**](https://arxiv.org/abs/1605.02065 "Mark Bun, Thomas Steinke. Concentrated Differential Privacy: Simplifications, Extensions, and Lower Bounds. TCC 2016.").
This, of course, applies to the exponential mechaism. A cool fact -- that we want to draw more attention to -- is that we can do better! 
Specifically, \\\(\\eta\\\)-bounded range implies \\\(\\frac18 \\eta^2\\\)-concentrated differential privacy [**[CR21]**](https://arxiv.org/abs/2004.07223 "Mark Cesar, Ryan Rogers. Bounding, Concentrating, and Truncating: Unifying Privacy Loss Composition for Data Analytics. ALT 2021.").
What follows is a proof of this fact following that of Mark Cesar and Ryan Rogers, but with some simplification.

> **Theorem 5 (Bounded Range implies Concentrated Differential Privacy).**
> If \\\(M\\\) is \\\(\\eta\\\)-bounded range, then it is \\\(\\frac18\\eta^2\\\)-concentrated differentially private.

_Proof._
Fix datasets \\\(x,x' \\in \\mathcal{X}^n\\\) differing on a single individual's data.
Let \\\(Z \\gets \\mathsf{PrivLoss}(M(x)\\\|M(x'))\\\) be the privacy loss random variable of the mechanism \\\(M\\\) on this pair of datasets.
By the definition of bounded range (Definition 1), there exists some \\\(t \\in \\mathbb{R}\\\) such that \\\(Z \\in [t, t+\\eta]\\\) with probability 1.
Now we employ [Hoeffding's Lemma](https://en.wikipedia.org/wiki/Hoeffding%27s_lemma) [**[H63]**](https://doi.org/10.1080%2F01621459.1963.10500830 "Wassily Hoeffding. Probability inequalities for sums of bounded random variables. JASA 1963."):
> **Lemma 6 (Hoeffding's Lemma).**
> Let \\\(X\\\) be a random variable supported on the interval \\\([a,b]\\\). Then, for all \\\(\\lambda \\in \\mathbb{R}\\\), we have \\\[\\mathbb{E}[\\exp(\\lambda X)] \\le \\exp \\left( \\mathbb{E}[X] \\cdot \\lambda + \\frac{(b-a)^2}{8} \\cdot \\lambda^2 \\right).\\\]

Applying the lemma to the privacy loss gives \\\[\\forall \\lambda \\in \\mathbb{R} ~~~~~  \\mathbb{E}[\\exp(\\lambda Z)] \\le \\exp \\left( \\mathbb{E}[Z] \\cdot \\lambda + \\frac{\\eta^2}{8} \\cdot \\lambda^2 \\right).\\\]
The only remaining thing we need is to show is that \\\(\\mathbb{E}[Z] \\le \\frac18 \\eta^2\\\).[^6]

If we set \\\(\\lambda = -1 \\\), then we get \\\( \\mathbb{E}[\\exp( - Z)] \\le \\exp \\left( -\\mathbb{E}[Z] + \\frac{\\eta^2}{8} \\right)\\\), which rearranges to \\\(\\mathbb{E}[Z] \\le \\frac18 \\eta^2 - \\log \\mathbb{E}[\\exp( - Z)]\\\). 
Now we have \\\[ \\mathbb{E}[\\exp( - Z)] \\!=\\! \\sum_y \\mathbb{P}[M(x)\\!=\\!y] \\exp(-f(y)) \\!=\\! \\sum_y \\mathbb{P}[M(x)\\!=\\!y]  \\!\\cdot\\! \\frac{\\mathbb{P}[M(x')\\!=\\!y]}{\\mathbb{P}[M(x)\\!=\\!y]} \\!=\\! 1.\\\]
&#8718;

This brings us to the TL;DR of this post:

> **Corollary 7.** The exponential mechanism (given by Equation 1) is \\\(\\frac18 \\varepsilon^2\\\)-concentrated differentially private.

This is great news. The standard analysis only gives \\\(\\frac12 \\varepsilon^2\\\)-concentrated differential privacy. Constants matter when applying differential privacy, and we save a factor of 4 in the concentrated differential privacy analysis of the exponential mechanism for free with this improved analysis.

Combining Lemma 2 with Theorem 5 also gives a simpler proof of the conversion from pure differential privacy to concentrated differential privacy [**[BS16]**](https://arxiv.org/abs/1605.02065 "Mark Bun, Thomas Steinke. Concentrated Differential Privacy: Simplifications, Extensions, and Lower Bounds. TCC 2016."):

> **Corollary 8.** \\\(\\varepsilon\\\)-differential privacy implies \\\(\\frac12 \\varepsilon^2\\\)-concentrated differential privacy.

## Beyond the Exponential Mechanism

The exponential mechanism is not the only algorithm for private selection. A closely-related algorithm is _report noisy max/min_:[^7] Draw independent noise \\\(\\xi_y\\\) from some distribution for each \\\(y \\in \\mathcal{Y}\\\) then output \\\[M(x) = \\underset{y \\in \\mathcal{Y}}{\\mathrm{argmin}} ~ \\ell(y,x) - \\xi_y.\\\]

If the noise distribution is an appropriate [Gumbel distribution](https://en.wikipedia.org/wiki/Gumbel_distribution), then report noisy max is exactly the exponential mechanism. (This equivalence is known as the "Gumbel max trick.")

We can also use the Laplace distribution or the exponential distribution. Report noisy max with the exponential distribution is equivalent to the _permute and flip_ algorithm [**[MS20]**](https://arxiv.org/abs/2010.12603 "Ryan McKenna, Daniel Sheldon. Permute-and-Flip: A new mechanism for differentially private selection
. NeurIPS 2020.") [**[DKSSWXZ21]**](https://arxiv.org/abs/2105.07260 "Zeyu Ding, Daniel Kifer, Sayed M. Saghaian N. E., Thomas Steinke, Yuxin Wang, Yingtai Xiao, Danfeng Zhang. The Permute-and-Flip Mechanism is Identical to Report-Noisy-Max with Exponential Noise. 2021."). However, these algorithms don't enjoy the same improved bounded range and concentrated differential privacy guarantees as the exponential mechanism.

There are also other variants of the selection problem. For example, in some cases we can assume that only a few options have low loss and the rest of the options have high loss -- i.e., there is a gap between the minimum loss and the second-lowest loss (or, more generally, the \\\(k\\\)-th lowest loss). In this case there are algorithms that attain better accuracy than the exponential mechanism under relaxed privacy definitions [**[CHS14]**](https://arxiv.org/abs/1409.2177 "Kamalika Chaudhuri, Daniel Hsu, Shuang Song. The Large Margin Mechanism for Differentially Private Maximization. NIPS 2014.") [**[BDRS18]**](https://dl.acm.org/doi/10.1145/3188745.3188946 " Mark Bun, Cynthia Dwork, Guy N. Rothblum, Thomas Steinke. Composable and versatile privacy via truncated CDP. STOC 2018.") [**[BKSW19]**](https://arxiv.org/abs/1905.13229 "Mark Bun, Gautam Kamath, Thomas Steinke, Zhiwei Steven Wu. Private Hypothesis Selection. NeurIPS 2019.").

There are a lot of interesting aspects of private selection, including questions for further research! We hope to have further posts about some of these topics.

---

[^1]: In general, we can replace \\\(\\frac{\\mathbb{P}\[M\(x\)=y\]}{\\mathbb{P}\[M\(x'\)=y\]}\\\) with the Radon-Nikodym derivative of the probability distribution given by \\\(M\(x\)\\\) with respect to the probability distribution given by \\\(M\(x'\)\\\) evaluated at \\\(y\\\). We handle division by zero by defining \\\(\\frac{0}{0} = 1\\\) and \\\(\\frac{\\eta}{0} = \\infty\\\) for \\\(\\eta>0\\\).

[^2]: To be more precise, \\\(\(\\varepsilon,\\delta\)\\\)-differential privacy is equivalent to demanding that \\\(\\mathbb{E}[\\max\\{0,1-\\exp(\\varepsilon-Z)\\}]\\le\\delta\\\) [**[CKS20]**](https://arxiv.org/abs/2004.00010 "Clément L. Canonne, Gautam Kamath, Thomas Steinke. The Discrete Gaussian for Differential Privacy. NeurIPS 2020."). (To be completely precise, we must appropriately deal with the \\\(Z=\\infty\\\) case, which we ignore in this discussion for simplicity.)

[^3]: This proof actually gives [a slightly stronger result](https://dongjs.github.io/2020/02/10/ExpMech.html): We can replace the sensitivity \\\(\\Delta\\\) (defined in Equation 2) by half the range \\\[\\hat\\Delta = \\frac12 \\sup_{x,x' \\in \\mathcal{X}^n : d(x,x') \\le 1} \\left( \\max_{\\overline{y}\\in\\mathcal{Y}} \\ell(\\overline{y},x) - \\ell(\\overline{y},x') - \\min_{\\underline{y}\\in\\mathcal{Y}} \\ell(\\underline{y},x) - \\ell(\\underline{y},x') \\right).\\\] We always have \\\(\\hat\\Delta \\le \\Delta\\\) but it is possible that \\\(\\hat\\Delta < \\Delta\\\) and the privacy analysis of the exponential mechanism still works if we replace \\\(\\Delta\\\) by \\\(\\hat\\Delta\\\).

[^4]: Equivalently, a randomized algorithm \\\(M : \\mathcal{X}^n \\to \\mathcal{Y}\\\) satisfies \\\(\\rho\\\)-concentrated differential privacy if, for all pairs of inputs \\\(x, x' \\in \\mathcal{X}^n\\\) differing only on the data of a single individual, \\\[\\forall \\lambda > 0 ~~~~~ \\mathrm{D}\_{\\lambda+1}(M(x)\\\|M(x')) \\le (\\lambda+1)\\rho,\\\] where \\\(\\mathrm{D}\_{\\lambda+1}(M(x)\\\|M(x')))\\\) is the order \\\(\\lambda+1\\\) Rényi divergence of \\\(M(x)\\\) from \\\(M(x')\\\).

[^5]: To be precise, if \\\(M(x) = q(x) + \\mathcal{N}(0,\\sigma^2I)\\\), then \\\(M : \\mathcal{X}^n \\to \\mathbb{R}^d\\\) satisfies \\\(\\frac{\\Delta\_2^2}{2\\sigma^2}\\\)-concentrated differential privacy, where \\\(\\Delta\_2 = \\sup\_{x,x'\\in\\mathcal{X}^n : d(x,x')\\le1} \\\|q(x)-q(x')\\\|\_2\\\) is the 2-norm sensitivity of \\\(q:\\mathcal{X}^n \\to \\mathbb{R}^d\\\). Furthermore, the privacy loss of the Gaussian mechanism is itself a Gaussian and it makes the inequality defining concentrated differential privacy (Equation 3) an equality for all \\\(\\lambda\\\)

[^6]: Note that the expectation of the privacy loss is simply the [KL divergence](https://en.wikipedia.org/wiki/Kullback%E2%80%93Leibler_divergence): \\\(\\mathbb{E}[Z] = \\mathrm{D}\_1( M(x) \\\| M(x') )\\\).

[^7]: We have presented selection here in terms of minimization, but most of the literature is in terms of maximization.
