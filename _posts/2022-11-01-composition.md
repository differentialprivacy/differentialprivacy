---
layout: post
title: Composition
comments: true
authors: 
    - thomassteinke
categories: [Surveys]
timestamp: 11:45:00 -0400
---

Our data is subject to many different uses. Many entities will have access to our data and those entities will perform many different analyses that involve our data. The greatest risk to privacy is that an attacker will combine multiple pieces of information from the same or different sources and that the combination of these will reveal sensitive details about us.
Thus we cannot study privacy leakage in a vacuum; it is important that we can reason about the accumulated privacy leakage over multiple independent analyses, which is known as _composition_. We have [previously discussed](/privacy-composition/) why composition is so important for differential privacy.
    
This is the first in a series of posts on _composition_ in which we will explain in more detail how compositoin analyses work.
    
Composition is quantitative. The differential privacy guarantee of the overall system will depend on the number of analyses and the privacy parameters that they each satisfy. The exact relationship between these quantities can be complex. There are various composition theorems that give bounds on the overall parameters in terms of the parameters of the parts of the system.
    
The simplest composition theorem is what is known as basic composition, which applies to pure \\\(\\varepsilon\\\)-DP (although it can be extended to approximate \\\((\\varepsilon,\\delta)\\\)-DP):
    
> **Theorem** (Basic Composition)
> Let \\\(M\_1, M\_2, \\cdots, M\_k : \\mathcal{X}^n \\to \\mathcal{Y}\\\) be randomized algorithms. Suppose \\\(M\_j\\\) is \\\(\\varepsilon\_j\\\)-DP for each \\\(j \\in \[k\]\\\).
> Define \\\(M : \\mathcal{X}^n \\to \\mathcal{Y}^k\\\) by \\\(M(x)=(M\_1(x),M\_2(x),\\cdots,M\_k(x))\\\), where each algorithm is run independently. Then \\\(M\\\) is \\\(\\varepsilon\\\)-DP for \\\(\\varepsilon = \\sum\_{j=1}^k \\varepsilon\_j\\\).

*Proof.*
Fix an arbitrary pair of neighbouring datasets \\\(x,x' \\in \\mathcal{X}^n\\\) and output \\\(y \\in \\mathcal{Y}^k\\\).
To establish that \\\(M\\\) is \\\(\\varepsilon\\\)-DP, we must show that \\\(e^{-\\varepsilon} \\le \\frac{\\mathbb{P}\[M(x)=y\]}{\\mathbb{P}\[M(x')=y\]} \\le e^\\varepsilon\\\). By independence, we have \\\[\\frac{\\mathbb{P}\[M(x)=y\]}{\\mathbb{P}\[M(x')=y\]} = \\frac{\\prod\_{j=1}^k\\mathbb{P}\[M\_j(x)=y\_j\]}{\\prod\_{j=1}^k\\mathbb{P}\[M\_j(x')=y\_j\]} =  \\prod\_{j=1}^k \\frac{\\mathbb{P}\[M\_j(x)=y\_j\]}{\\mathbb{P}\[M\_j(x')=y\_j\]} \\le \\prod\_{j=1}^k e^{\\varepsilon\_j} = e^{\\sum\_{j=1}^k \\varepsilon\_j} = e^\\varepsilon,\\\] where the inequality follows from the fact that each \\\(M\_j\\\) is \\\(\\varepsilon\_j\\\)-DP and, hence, \\\(e^{-\\varepsilon\_j} \\le \\frac{\\mathbb{P}\[M\_j(x)=y\_j\]}{\\mathbb{P}\[M\_j(x')=y\_j\]} \\le e^{\\varepsilon\_j}\\\). Similarly, \\\(\\prod\_{j=1}^k \\frac{\\mathbb{P}\[M\_j(x)=y\_j\]}{\\mathbb{P}\[M\_j(x')=y\_j\]} \\ge \\prod\_{j=1}^k e^{-\\varepsilon\_j}\\\), which completes the proof. &#8718;
    
Basic composition is already a powerful result, despite its simple proof; it establishes the versatility of differential privacy and allows us to begin reasoning about complex systems in terms of their building blocks. For example, suppose we have \\\(k\\\) functions \\\(f\_1, \\cdots, f\_k : \\mathcal{X}^n \\to \\mathbb{R}\\\) each of sensitivity \\\(1\\\). For each \\\(j \\in \[k\]\\\), we know that adding \\\(\\mathsf{Laplace}(1/\\varepsilon)\\\) noise to the value of \\\(f\_j(x)\\\) satisfies \\\(\\varepsilon\\\)-DP. Thus, if we add independent \\\(\\mathsf{Laplace}(1/\\varepsilon)\\\) noise to each value \\\(f\_j(x)\\\) for all \\\(j \\in \[k\]\\\), then basic composition tells us that releasing this vector of \\\(k\\\) noisy values satisfies \\\(k\\varepsilon\\\)-DP. If we want the overall system to be \\\(\\varepsilon\\\)-DP, then we should add independent \\\(\\mathsf{Laplace}(k/\\varepsilon)\\\) noise to each value \\\(f\_j(x)\\\).
    
## Is Basic Composition Optimal?
    
If we want to release \\\(k\\\) values each of sensitivity \\\(1\\\) (as above) and have the overall release be \\\(\\varepsilon\\\)-DP, then, using basic composition, we can add \\\(\\mathsf{Laplace}(k/\\varepsilon)\\\) noise to each value. The variance of the noise for each value is \\\(2k^2/\\varepsilon^2\\\), so the standard deviation is \\\(\\sqrt{2} k /\\varepsilon\\\). In other words, the scale of the noise must grow linearly with the number of values \\\(k\\\) if the overall privacy and each value's sensitivity is fixed. It is natural to wonder whether the scale of the Laplace noise can be reduced by improving the basic composition result. We now show that this is not possible.
    
For each \\\(j \\in \[k\]\\\), let \\\(M\_j : \\mathcal{X}^n \\to \\mathbb{R}\\\) be the algorithm that releases \\\(f\_j(x)\\\) with \\\(\\mathsf{Laplace}(k/\\varepsilon)\\\) noise added. Let \\\(M : \\mathcal{X}^n \\to \\mathbb{R}^k\\\) be the composition of these \\\(k\\\) algorithms. Then \\\(M\_j\\\) is \\\(\\varepsilon/k\\\)-DP for each \\\(j \\in \[k\]\\\) and basic composition tells us that \\\(M\\\) is \\\(\\varepsilon\\\)-DP. The question is whether \\\(M\\\) satisfies a better DP guarantee than this -- i.e., does \\\(M\\\) satisfy \\\(\\varepsilon\_*\\\)-DP for some \\\(\\varepsilon\_\*<\\varepsilon\\\)?
Suppose we have neighbouring datasets \\\(x,x'\\in\\mathcal{X}^n\\\) such that \\\(f\_j(x) = f\_j(x')+1\\\) for each \\\(j \\in \[k\]\\\). Let \\\(y=(a,a,\\cdots,a) \\in \\mathbb{R}^k\\\) for some \\\(a \\ge \\max\_{j=1}^k f\_j(x)\\\).
Then 
\\\[
        \\frac{\\mathbb{P}\[M(x)=y\]}{\\mathbb{P}\[M(x')=y\]} = \\frac{\\prod\_{j=1}^k \\mathbb{P}\[f\_j(x)+\\mathsf{Laplace}(k/\\varepsilon)=y\_j]}{\\prod\_{j=1}^k \\mathbb{P}\[f\_j(x')+\\mathsf{Laplace}(k/\\varepsilon)=y\_j\]} 
\\\]
\\\[
         = \\prod\_{j=1}^k \\frac{\\frac{\\varepsilon}{2k}\\exp\\left(-\\frac{\\varepsilon}{k} |y\_j-f\_j(x)| \\right)}{\\frac{\\varepsilon}{2k}\\exp\\left(-\\frac{\\varepsilon}{k} |y\_j-f\_j(x')| \\right)} 
         = \\prod\_{j=1}^k \\frac{\\exp\\left(-\\frac{\\varepsilon}{k} (y\_j-f\_j(x)) \\right)}{\\exp\\left(-\\frac{\\varepsilon}{k} (y\_j-f\_j(x')) \\right)} 
\\\]
\\\[
         = \\prod\_{j=1}^k \\exp\\left(\\frac{\\varepsilon}{k}\\left(f\_j(x)-f\_j(x')\\right)\\right)
         = \\exp\\left( \\frac{\\varepsilon}{k} \\sum\_{j=1}^k \\left(f\_j(x)-f\_j(x')\\right)\\right)= e^\\varepsilon,
\\\]
where the third equality removes the absolute values because \\\(y\_j \\ge f\_j(x)\\\) and \\\(y\_j \\ge f\_j(x')\\\).
This shows that basic composition is optimal. For this example, we cannot prove a better guarantee than what is given by basic composition.
    
Is there some other way to improve upon basic composition that circumvents this example? Note that we assumed that there are neighbouring datasets \\\(x,x'\\in\\mathcal{X}^n\\\) such that \\\(f\_j(x) = f\_j(x')+1\\\) for each \\\(j \\in \[k\]\\\). In some settings, no such worst case datasets exist. In that case, instead of scaling the noise linearly with \\\(k\\\), we can scale the Laplace noise according to the \\\(\\ell\_1\\\) sensitivity \\\(\\Delta\_1 := \\sup\_{x,x' \\in \\mathcal{X}^n \\atop \\text{neighbouring}} \\sum\_{j=1}^k \|f\_j(x)-f\_j(x')\|\\\). 
    
Instead of adding assumptions to the problem, we will look more closely at the example above.
We showed that there exists some output \\\(y \\in \\mathbb{R}^d\\\) such that \\\(\\frac{\\mathbb{P}\[M(x)=y\]}{\\mathbb{P}\[M(x')=y\]} = e^\\varepsilon\\\).
However, such outputs \\\(y\\\) are very rare, as we require \\\(y\_j \\ge \\max\\{f\_j(x),f\_j(x')\\}\\\) for each \\\(j \\in \[k\]\\\) where \\\(y\_j = f\_j(x) + \\mathsf{Laplace}(k/\\varepsilon)\\\). Thus, in order to observe an output \\\(y\\\) such that the likelihood ratio is maximal, all of the \\\(k\\\) Laplace noise samples must be positive, which happens with probability \\\(2^{-k}\\\). 
The fact that outputs \\\(y\\\) with maximal likelihood ratio are exceedingly rare turns out to be a general phenomenon and not specific to the example above. 
    
Can we improve on basic composition if we only ask for a high probability bound? That is, instead of demanding \\\(\\frac{\\mathbb{P}[M(x)=y\]}{\\mathbb{P}\[M(x')=y\]} \\le e^{\\varepsilon\_\*}\\\) for all \\\(y \\in \\mathcal{Y}\\\), we demand \\\(\\mathbb{P}\_{Y \\gets M(x)}\\left\[\\frac{\\mathbb{P}\[M(x)=Y\]}{\\mathbb{P}\[M(x')=Y\]} \\le e^{\\varepsilon\_\*}\\right\] \\ge 1-\\delta\\\) for some \\\(0 < \\delta \\ll 1\\\). Can we prove a better bound \\\(\\varepsilon\_\* < \\varepsilon\\\) in this relaxed setting? The answer turns out to be yes.
    
The limitation of pure \\\(\\varepsilon\\\)-DP is that events with tiny probability -- which are negligible in real-world applications -- can dominate the privacy analysis. This motivates us to move to relaxed notions of differential privacy, such as approximate \\\((\\varepsilon,\\delta)\\\)-DP and concentrated DP, which are less sensitive to low probability events. 
    
## Preview: Advanced Composition
    
By moving to approximate \\\((\\varepsilon,\\delta)\\\)-DP with \\\(\\delta>0\\\), we can prove an asymptotically better composition theorem, which is known as _the advanced composition theorem_ **[[DRV10]](https://ieeexplore.ieee.org/document/5670947 "Cynthia Dwork, Guy Rothblum, Salil Vadhan. Boosting and Differential Privacy. FOCS 2010.")**.
    
> **Theorem** (Advanced Composition Starting from Pure DP[^1])
> Let \\\(M\_1, M\_2, \\cdots, M\_k : \\mathcal{X}^n \\to \\mathcal{Y}\\\) be randomized algorithms. Suppose \\\(M\_j\\\) is \\\(\\varepsilon\_j\\\)-DP for each \\\(j \\in \[k\]\\\).
> Define \\\(M : \\mathcal{X}^n \\to \\mathcal{Y}^k\\\) by \\\(M(x)=(M\_1(x),M\_2(x),\\cdots,M\_k(x))\\\), where each algorithm is run independently. Then \\\(M\\\) is \\\((\\varepsilon,\\delta)\\\)-DP for any \\\(\\delta>0\\\) with \\\[\\varepsilon = \\frac12 \\sum\_{j=1}^k \\varepsilon\_j^2 + \\sqrt{2\\log(1/\\delta) \\sum\_{j=1}^k \\varepsilon\_j^2}.\\\]
    
Recall that basic composition gives \\\(\\delta=0\\\) and \\\(\\varepsilon = \\sum\_{j=1}^k \\varepsilon\_j\\\). That is, basic composition scales with the 1-norm of the vector \\\((\\varepsilon\_1, \\varepsilon\_2, \\cdots, \\varepsilon\_k)\\\), whereas advanced composition scales with the 2-norm of this vector (and the squared 2-norm).
Neither bound strictly dominates the other. However, asymptotically (in a sense we will make precise in the next paragraph) advanced composition dominates basic composition.
    
Suppose we have a fixed \\\((\\varepsilon,\\delta)\\\)-DP guarantee for the entire system and we must answer \\\(k\\\) queries of sensitivity \\\(1\\\).
Using basic composition, we can answer each query by adding \\\(\\mathsf{Laplace}(k/\\varepsilon)\\\) noise to each answer.
However, using advanced composition, we can answer each query by adding \\\(\\mathsf{Laplace}(\\sqrt{k/2\\rho})\\\) noise to each answer, where[^2]
\\\[\\rho = \\frac{\\varepsilon^2}{4\\log(1/\\delta)+4\\varepsilon}.\\\]
If the privacy parameters \\\(\\varepsilon,\\delta>0\\\) are fixed (which implies \\\(\\rho\\\) is fixed) and \\\(k \\to \\infty\\\), we can see that asymptotically advanced composition gives noise per query scaling as \\\(\\Theta(\\sqrt{k})\\\), while basic composition results in noise scaling as \\\(\\Theta(k)\\\).
    
&nbsp;
    
In the next few posts we will explain how advanced composition works. We hope this conveys an intuitive understanding of composition and, in particular, how this \\\(\\sqrt{k}\\\) asymptotic behaviour arises. If you want to read ahead, these posts are extracts from [this book chapter](https://arxiv.org/abs/2210.00597).
    
---

[^1]: This result generalizes to approximate DP. If instead we assume \\\(M\_j\\\) is \\\((\\varepsilon\_j,\\delta\_j)\\\)-DP for each \\\(j \\in \[k\]\\\), then the final composition is \\\((\\varepsilon,\\delta+\\sum\_{j=1}^k \\delta\_j)\\\)-DP with \\\(\\varepsilon\\\) as before.

[^2]: Adding \\\(\\mathsf{Laplace}(\\sqrt{k/2\\rho})\\\) noise to a sensitivity-1 query ensures \\\(\\varepsilon\_j\\\)-DP for \\\(\\varepsilon\_j = \\sqrt{2\\rho/k}\\\). Hence \\\(\\sum\_{j=1}^k \\varepsilon\_j^2 = 2\rho\\\). Setting \\\(\\rho = \\frac{\\varepsilon^2}{4\\log(1/\\delta)+4\\varepsilon}\\\) ensures that \\\(\\frac12 \\sum\_{j=1}^k \\varepsilon\_j^2 + \\sqrt{2\\log(1/\\delta) \\sum\_{j=1}^k \\varepsilon\_j^2} = \\rho + \\sqrt{4\\rho\\log(1/\\delta)} \\le \\varepsilon\\\).
