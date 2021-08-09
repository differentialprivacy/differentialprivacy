---
layout: post
title: "One-shot DP Top-k mechanisms"
comments: true
authors:
  - daviddurfee
  - ryanrogers
timestamp: 10:00:00 -0700
categories: [Algorithms]
---


In the last [_blog post_](https://differentialprivacy.org/exponential-mechanism-bounded-range/), we showed that the exponential mechanism enjoys improved composition bounds over general pure DP mechanisms due to a property called **bounded range**.  For this post, we will present another useful, and somewhat surprising, property of the exponential mechanism in its application of top-\\\(k\\\) selection.

## Differentially Private Top-\\\(k\\\) Selection

We will focus on datasets that are a vector of counts \\\(h = \(h_1, \\cdots, h_d\) \\in \\mathbb{N}^d\\\), which consist of counts \\(h_i\\\) for elements from a universe \\\(\\mathcal{U}\\\) where \\\(\|\\mathcal{U}\| = d\\\).  Let's assume that a user's data can modify each count by at most 1, yet can change all \\\(d\\\) counts, i.e. the \\\(\\ell_\\infty\\\)-sensitivity is 1 and the \\\(\\ell_0\\\)-sensitivity is \\\(d\\\).  The task here is to return the top-\\\(k\\\) elements from the input counts in a differentially private way.  

For top-\\\(1\\\), this is simply returning the element with the max count, and this is precisely the problem that the exponential mechanism is set up to solve.  Let's write out the exponential mechanism \\\(M^{(1)}: \mathbb{N}^d \to \[d\]\\\) for this instance:
\\\[
\mathbb{P}[M^{(1)}(h) = i] = \frac{e^{ \varepsilon h_i }}{\\sum_{j \\in [d] } e^{ \\varepsilon h_j } }, \\qquad \\forall i \\in \[d\].
\\\]
For those wondering why this formula omits the factor of \\\(1/2\\\) in the exponent, we are using the [stronger result](https://dongjs.github.io/2020/02/10/ExpMech.html) of the exponential mechanism which replaces global sensitivity with the range of the loss function \\\(\\ell(i,h) = - h_i\\\), which is \\\(1\\\) in this case.  Recall from the last blog post that the exponential mechanism is \\\(\\varepsilon^2/8\\\)-CDP.  


Hence, to generalize this to top-\\\(k\\\) selection, we can simply iteratively apply this exponential mechanism by removing the _discovered_ element from each previous round.  That is, we write \\\(M^{\(k\)}: \\mathbb{N}^d \\to \[d\]^k\\\) as the following for any outcome \\\( \(i_1, i_2, \\cdots, i_k\) \\in \[d\]^k\\\),

\\\[
\\mathbb{P}\[M^{(k)}(h) = \(i_1, i_2, \cdots, i_k\)\] \\qquad \\qquad \\qquad \\qquad \\qquad \\qquad \\qquad \\qquad 
\\\]
<a name="eq:peelingEM"></a>
\\\[\qquad = \\frac{e^{ \\varepsilon h_{i_1} }}{\\sum_{j \\in \[d\] } e^{ \\varepsilon h_j } } \\cdot  \\frac{ e^{\\varepsilon h_{i_2} } }{\\sum_{j\\in \[d\]\\setminus \\{ i_1\\}} e^{\\varepsilon h_j } } \\cdot \\cdots \\cdot \\frac{ e^{ \\varepsilon h_{i_2} } }{\\sum_{j\\in \[d\]\\setminus \\{ i_1, \\cdots, i_{k-1}\\}} e^{ \\varepsilon h_j } }. 
\\tag{1}
\\\]

We can then apply composition to conclude that \\\(M^{\(k\)}\\\) is \\\(k \\varepsilon^2/8\\\)-CDP.  



## Gumbel Noise and the Exponential Mechanism

As we discussed in our last post, we can implement the exponential mechanism by adding [Gumbel](https://en.wikipedia.org/wiki/Gumbel_distribution) noise to each count and reporting the noisy max element.  A Gumbel random variable \\\(X \\sim \\text{Gumbel}\(\\beta\) \\\), parameterized by scale parameter \\\(\\beta>0\\\), has the following density function
<a name="eq:GumbelDensity"></a>
\\\[
p(x;\\beta) = \\frac{1}{\\beta} \\exp\\left\( - x/\\beta - e^{-x/\\beta} \\right\), \\qquad \\forall x \\in \mathbb{R}.
\\tag{2}
\\\]

Hence, we can write the exponential mechanism in the following way
\\\[
M^{\(1\)}\(h\) = \\arg\\max \\{ h_i + X_i : i \\in \[d\] \\}, \\qquad \\{X_i \\} \\stackrel{i.i.d.}{\\sim} \\text{Gumbel}\(1/\\varepsilon\).
\\\]

We can then extend this to top-\\\(k\\\) by repeatedly adding independent Gumbel noise to each count and removing the discovered element for the next round.  However, something that would significantly improve run time would be to add Gumbel noise to each count _once_ and then take the elements with the top-\\\(k\\\) noisy counts.  We could then add only \\\(d\\\) many noise terms, rather than \\\(O\(d^k\)\\\) noise terms if we were to iteratively run \\\(k\\\) different exponential mechanisms.  The question is, does this one-shot top-\\\(k\\\) Gumbel noise mechanism ensure the same level of privacy?

Let's denote the one-shot Gumbel mechanism as \\\(\\tilde{M}^{\(k\)}\\\).  At first glance, it does not seem like the one-shot Gumbel mechanism \\\(\\tilde{M}^{\(k\)}\\\) should be just as private as the iterative exponential mechanism \\\(M^{\(k\)}\\\), but it turns out they are exactly the same mechanism!  The following result is due to [**[DR19]**](https://arxiv.org/abs/1905.04273 "David Durfee, Ryan Rogers. Practical Differentially Private Top-k Selection with Pay-what-you-get Composition. NeurIPS 2019").

> **Theorem 1**
>For any input vector of counts \\\(h \\in \\mathbb{N}^d\\\), the one-shot Gumbel mechanism \\\(\\tilde{M}^{\(k\)}\(h\)\\\) and iteratively applying the exponential mechanism \\\(M^{\(k\)}\(h\)\\\) are equal in distribution.


_Proof._ 
Recall the distribution of the iterative exponential mechanism \\\(M^{\(k\)}\(h\)\\\) from [\(1\)](#eq:peelingEM).
Now we consider the one-shot Gumbel mechanism \\\(\\tilde{M}^{\(k\)}\(h\)\\\) where we use the density of \\\(X \\sim \\\) Gumbel\\\( \(1/\\varepsilon\)\\\) from [\(2\)](#eq:GumbelDensity).  
\\\[
\mathbb{P}[\tilde{M}^{(k)}(h) = (i_1, \cdots, i_k)] \\qquad \\qquad \\qquad \\qquad \\qquad \\qquad \\qquad \\qquad 
\\\]
\\\[
\\qquad = \\int_{-\\infty}^\\infty p\(u_1 - h_{i_1}) \\int_{-\\infty}^{u_1}  p\(u_2 - h_{i_2}\) \\cdots \\int_{-\\infty}^{u_{k-1}} p\(u_k - h_k\) 
\\\]
<a name="eq:integral"></a>
\\\[
\\qquad \\qquad \\cdot \\prod_{j \\in \[d\] \\setminus \\{i_1, \\cdots, i_k \\} } \\mathbb{P}\[ X < u_k - h_j\]du_k \\cdots du_2 du_1.
\\tag{3}
\\\]
Note that we have 
\\\[
\\mathbb{P}\[X < y\] = \\exp\\left\( - \\exp\\left\( -\\varepsilon y \\right\) \\right\).
\\\]
Let's focus on the inner integral over \\\(u_k\\\) in [\(3\)](#eq:integral).  
\\\[
\\int_{-\\infty}^{u_{k-1}} p\(u_k - h_{i_k} \)\\prod_{j \\in \[d\] \\setminus \\{i_1, \\cdots, i_k \\} } \\mathbb{P}\[X < u_k - h_j \]du_k \\qquad \\qquad \\qquad \\qquad \\qquad \\qquad \\qquad \\qquad 
\\\]
\\\[
\\quad = \\int_{-\\infty}^{u_{k-1}} \\varepsilon \\cdot  \\exp\\left\( - \\varepsilon \(u_k - h_{i_k}\) - e^{ -\\varepsilon \(u_k - h_{i_k}) } \\right\) \\cdot  \\exp\\left\( -e^{-\\varepsilon u_k}  \\sum_{j \\in \[d\] \\setminus \\{i_1, \cdots, i_k \\} } e^{\\varepsilon h_j}  \\right\) du_k 
\\\]
\\\[
\\quad  = \\varepsilon e^{\\varepsilon h_{i_k}}  \\int_{-\\infty}^{u_{k-1}} \\exp\\left\( -\\varepsilon u_k - e^{-\\varepsilon u_k} \\left( e^{\\varepsilon h_{i_k}} + \\sum_{j \\in \[d\] \\setminus \\{i_1, \\cdots i_k \\} } e^{\\varepsilon h_j} \\right) \\right) du_k \\qquad
\\\]
<a name="eq:lastLine"></a>
\\\[
\\qquad =  \\varepsilon e^{\\varepsilon h_{i_k}}  \\int_{-\\infty}^{u_{k-1}} \\exp\\left\( -\\varepsilon u_k - e^{-\\varepsilon u_k} \\left(\\sum_{j \\in \[d\] \\setminus \\{i_1, \\cdots i_{k-1} \\} } e^{\\varepsilon h_j} \\right\) \\right\) du_k. \\qquad \\qquad
\\tag{4}
\\\]
We now integrate with a \\\(v\\\)-substitution,
\\\[
v =e^{-\\varepsilon u_{k}} \\sum_{j \in \[d\] \\setminus \\{i_1, \\cdots i_{k-1} \\} } e^{\\varepsilon h_j}  
\\\]
\\\[
dv = - \\varepsilon \\sum_{j \\in \[d\] \\setminus \\{i_1, \\cdots i_{k-1} \\} } e^{\\varepsilon h_j}  \\cdot e^{-\\varepsilon u_{k}} du_{k}.
\\\]

Continuing with [\(4\)](#eq:lastLine), we get
\\\[
\\int_{-\\infty}^{u_{k-1}} p\(u_k - h_{i_k} \)\\prod_{j \\in \[d\] \\setminus \\{i_1, \\cdots, i_k \\} } \\mathbb{P}\[X < u_k - h_j \]du_k \\qquad \\qquad \\qquad \\qquad \\qquad
\\\]
\\\[
\\qquad = \\frac{e^{\\varepsilon h_{i_k} }}{\\sum_{j \\in \[d\] \\setminus \\{i_1, \\cdots, i_{k-1} \\}} e^{\\varepsilon h_j}} \\cdot \\exp\\left\( - e^{-\\varepsilon u_{k-1}} \\cdot \\sum_{j \\in \[d\] \\setminus \\{i_1, \\cdots, i_{k-1} \\}} e^{\\varepsilon h_j} \\right\)
\\\]
\\\[
\\qquad = \\frac{e^{\\varepsilon h_{i_k} }}{\\sum_{j \\in \[d\] \\setminus \\{i_1, \\cdots, i_{k-1} \\}} e^{\\varepsilon h_j}}  \\cdot \\prod_{j \\in \[d\] \\setminus \\{i_1, \\cdots, i_{k-1} \\} } \\mathbb{P}\[X < u_{k-1} - h_j \] .
\\\] 
Note how this line has the last term in the expression for \\\(M^{\(k\)}\(h\)\\\) in [\(1\)](#eq:peelingEM), which is independent of \\\(u_{k-1}\\\) and can hence be pulled out of the larger integral in [\(3\)](#eq:integral).  By induction, we have
\\\[
\\mathbb{P}\[\\tilde{M}^{\(k\)}\(h\) = \(i_1, \\cdots, i_k\)\] \\qquad \\qquad \\qquad \\qquad \\qquad \\qquad \\qquad \\qquad 
\\\]
\\\[
\\qquad  = \\frac{e^{\\varepsilon h_{i_1} }}{\\sum_{j \\in \[d\]} e^{\\varepsilon h_j}}  \\cdot \\frac{e^{\\varepsilon h_{i_2} }}{\\sum_{j \\in \[d\] \\setminus \\{ i_1\\}} e^{\\varepsilon h_j}}  \\cdot \\cdots \\cdot \\frac{e^{\\varepsilon h_k }}{\\sum_{j \\in \[d\] \\setminus \\{i_1, \\cdots, i_{k-1} \\}} e^{\\varepsilon h_j}} 
\\\]
\\\[
\\qquad = \\mathbb{P}\[M^{\(k\)}\(h\) =\(i_1, \cdots, i_k\)\].
\\\] &#8718;

So that's great!  We can now run the one-shot Gumbel mechanism for top-\\\(k\\\) and still get the improved composition bounds of the exponential mechanism.  In addition to this achieving better runtime, this analysis can help with proving top-\\\(k\\\) DP algorithms over a large domain universe despite giving access to only the true top-\\\(\\bar{k}\\\) items and their counts where \\\(\\bar{k} > k \\\), see [**[DR19]**](https://arxiv.org/abs/1905.04273 "David Durfee, Ryan Rogers. Practical Differentially Private Top-k Selection with Pay-what-you-get Composition. NeurIPS 2019") for more details.



## Report Noisy Max for DP Top-\\\(k\\\)

We now turn to comparing this algorithm to some natural alternatives.  As we discussed in the last post, there is a family of mechanisms, report noisy max (RNM) mechanisms, that ensure differential privacy for the selection problem, and hence the top-\\\(1\\\) problem.  We showed that the exponential mechanism is equivalent to RNM with Gumbel noise, there is also RNM with Laplace and with Exponential noise, the last being the recently discovered _permute-and-flip_ mechanism [**[MS20]**](https://arxiv.org/abs/2010.12603 "Ryan McKenna, Daniel Sheldon. Permute-and-Flip: A new mechanism for differentially private selection
. NeurIPS 2020.") [**[DKSSWXZ21]**](https://arxiv.org/abs/2105.07260 "Zeyu Ding, Daniel Kifer, Sayed M. Saghaian N. E., Thomas Steinke, Yuxin Wang, Yingtai Xiao, Danfeng Zhang. The Permute-and-Flip Mechanism is Identical to Report-Noisy-Max with Exponential Noise. 2021.").  

To then use RNM mechanisms for top-\\\(k\\\), we can again iteratively apply them and use composition to get the overall privacy guarantee.  However, it turns out that you can also use the Laplace noise version of RNM in one-shot [**[QSZ21]**](https://arxiv.org/abs/2105.08233 "Gang Qiao, Weijie J. Su, Li Zhang. Oneshot Differentially Private Top-k Selection. ICML 2021.").

We can compare the relative noise that is added to each count in both the Laplace and Gumbel versions.  Since [**[QSZ21]**](https://arxiv.org/abs/2105.08233 "Gang Qiao, Weijie J. Su, Li Zhang. Oneshot Differentially Private Top-k Selection. ICML 2021.") gives their privacy guarantee in terms of approximate \\\(\(\varepsilon,\delta \)\\\)-DP, we will now make the comparison there.  We first look at the standard deviation for Laplace \\\( \\sigma_{\\text{Lap}}\\\) \(using Theorem 2.2 in [**[QSZ21]**](https://arxiv.org/abs/2105.08233 "Gang Qiao, Weijie J. Su, Li Zhang. Oneshot Differentially Private Top-k Selection. ICML 2021.")\).
\\\[
\sigma_{\text{Lap}} =  \\frac{8 \\sqrt{2k \\ln\(d/\\delta\)}}{\\varepsilon}.
\\\]
Note that the one-shot Laplace mechanism returns counts as well as the indices of the top-\\\(k\\\), both of which use Laplace noise with standard deviation \\\(\sigma_{\text{Lap}}\\\), so we will also include Laplace noise to the discovered elements in the Gumbel version. That is, we add Gumbel noise with scale \\\(\sqrt{k}/\varepsilon'\\\) for the discovery portion and Laplace noise with scale \\\(2\sqrt{k}/\varepsilon'\\\) for obtaining their counts, resulting in standard deviation noise \\\(\sigma_{\text{Gumb}}'\\\) and \\\(\sigma_{\text{Lap}}'\\\), respectively.  
\\\[
\\sigma_{\\text{Gumb}}'  = \\frac{\\pi \\sqrt{k} }{\\sqrt{6}\\varepsilon'}, \\qquad 
\\sigma_{\\text{Lap}}' = \\frac{2\\sqrt{2k}}{\\varepsilon'}.
\\\]
Recall that adding this scale of Gumbel noise and Laplace noise will ensure \\\(\tfrac{\varepsilon'^2}{8} \\\)-CDP each, so combining will ensure \\\(\tfrac{\varepsilon'^2}{4}\\\)-CDP.  We could also use Gaussian noise to return the counts since we are using CDP, but we will analyze it with Laplace noise for comparison.  To ensure \\\(\(\varepsilon,\delta\)\\\)-DP, we use the CDP to DP conversion from Lemma 3.5 in [**[BS16]**](https://arxiv.org/abs/1605.02065 "Mark Bun, Thomas Steinke. Concentrated Differential Privacy: Simplifications, Extensions, and Lower Bounds. TCC 2016.") and solve for \\\(\varepsilon'\\\).  Hence, we get for any \\\(\delta>0\\\)
\\\[
\varepsilon'^2/4 = \left\( \sqrt{\ln\(1/\delta\) + \varepsilon} - \sqrt{\ln\(1/\delta\)} \right\)^2 
\\\]
\\\[ \implies \varepsilon' = 2 \sqrt{\ln\(1/\delta\)} \left\( \sqrt{1 + \tfrac{\varepsilon}{\ln\(1/\delta\)}} - 1 \right\).
\\\]

Let's consider a typical privacy setting where \\\(\\varepsilon <  \\ln\(1/\\delta\)\\\), and use the inequality \\\(\\sqrt{1+x} \\geq 1 + x/4\\\) for \\\(0\<x\<1\\\).  Here is a short proof of this inequality:
\\\[
(1 + x/4)^2 = 1 + x/2  + x^2/16  \leq 1 + x/2 + x/2 = 1 + x.  
\\\]
Note that the privacy guarantee for one-shot Laplace noise only holds when \\\(\\varepsilon < 0.2\\\) and \\\(\\delta < 0.05\\\) as stated in Theorem 2.2 in [**[QSZ21]**](https://arxiv.org/abs/2105.08233 "Gang Qiao, Weijie J. Su, Li Zhang. Oneshot Differentially Private Top-k Selection. ICML 2021.").  In this case, we have 
\\\[
\varepsilon' \geq 2 \sqrt{\ln\(1/\delta\)} \left\( 1 + 1/4 \cdot \tfrac{\varepsilon}{\ln\(1/\delta\)} - 1 \right\) = 1/2 \cdot \tfrac{\varepsilon}{\sqrt{\ln\(1/\delta\)}}.
\\\]
Plugging \\\(\varepsilon'\\\) into the standard deviation of Gumbel and Laplace, we get
\\\[
\\sigma_{\\text{Gumb}}'  \leq \\frac{2\\pi\\sqrt{k \\ln\(1/\\delta\)}}{\\sqrt{6}\cdot \varepsilon}, \qquad \\sigma_{\\text{Lap}}' \leq \\frac{4\\sqrt{2k\ln\(1/\delta\)}}{\\varepsilon}.
\\\]

Putting this together, we can show that we add significantly less noise for the discovery and releasing noisy count phases, 
\\\[
\sigma_{\text{Gumb}}'\leq \sigma_{\text{Lap}}/4 ,\qquad  \sigma_{\text{Lap}}' \leq \sigma_{\text{Lap}}/2.
\\\]
Note that these bounds can be improved further with similar analysis.

Although it has not been studied yet whether the permute-and-flip mechanism \\\(M_{\\text{PF}} \\\) can also ensure DP in one shot by using Exponential noise, we briefly discuss whether it can be bounded range for a similar parameter as the Exponential Mechanism, and hence achieve similar composition bounds.  Consider running permute-and-flip on two items \\\(\\{1,2 \\}\\\) with a monotonic quality score \\\(q: \\mathcal{X} \\times \\{1,2 \\} \\to \\mathbb{R} \\\) whose sensitivity is 1.  Let \\\(x, x' \\in \\mathcal{X} \\\) be neighbors where 
\\\[
q(x,1) = q(x,2) = 0
\\\]
\\\[
q(x',1) = 0 , \\quad q(x',2) = 1.
\\\]
Hence, permute-and-flip will return outcome \\\(1\\\) or \\\(2\\\) with half probability each on dataset \\\(x\\\), while with dataset \\\(x'\\\) outcome \\\(1\\\) occurs with probability \\\(1/2 \\cdot  e^{-\\varepsilon}\\\) and outcome \\\(2\\\) occurs with probability \\\( 1/2 + 1/2 \\cdot \(1 - e^{-\\varepsilon}\)\\\).  We can then compute the bounded range parameter \\\(\\alpha\\\) as
\\\[
\\frac{\\mathbb{P}\[M_{\\text{PF}}\(x'\) = 2 \] }{\\mathbb{P}\[M_{\\text{PF}}\(x\) = 2 \]}\\leq e^{\\alpha}\\frac{\\mathbb{P}\[M_{\\text{PF}}\(x'\) = 1 \]}{\\mathbb{P}\[M_{\\text{PF}}\(x\) = 1 \]} \\implies  \\alpha \\geq \\varepsilon + \\ln\(2 - e^{-\\varepsilon} \).
\\\]
Note that with \\\(\\varepsilon \gg 1\\\), we get \\\(\\alpha \\\) close to \\\(\\varepsilon\\\), which would be the same bounded range parameter as the exponential mechanism.  However, with \\\(\\varepsilon< 1\\\), we get \\\(\\alpha\\\) close to \\\(2\\varepsilon\\\).  This example provides a lower bound on the BR parameter for permute-and-flip.

## Conclusion

We have looked at the top-\\\(k\\\) selection problem subject to differential privacy and although there are many different mechanisms to use, the exponential mechanism stands out for several reasons:
1. The exponential mechanism is \\\(\\varepsilon\\\)-DP and \\\( \\varepsilon^2/8\\\)-CDP and hence gets improved composition.
2. Iteratively applying the exponential mechanism for top-\\\(k\\\) can be implemented by adding Gumbel noise to each count and returning the elements with the top-\\\(k\\\) noisy counts in one-shot.
3. The one-shot Gumbel mechanism returns a ranked list of \\\(k\\\) elements, rather than a set of \\\(k\\\) elements.  


