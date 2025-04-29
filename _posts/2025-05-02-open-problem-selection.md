---
layout: post
title: "Open Problem: Selection via Low-Sensitivity Queries"
comments: true
authors:
  - thomassteinke
timestamp: 12:00:00 -0700
categories: [Open Problems]
---

Two of the basic tools for building differentially private algorithms are noise addition for answering low-sensitivity queries and the exponential mechanism for selection. 
Could we do away with the exponential mechanism and simply use low-sensitivity queries to perform selection?

## Formal Problem Statement

Recall that the exponential mechanism is a differentially private algorithm that takes a private dataset \\\(x \\in \\mathcal{X}^n\\\) and a public loss function \\\(\\ell : \\mathcal{X}^n \\times \\mathcal{Y} \\to \\mathbb{R}\\\) and returns \\\(Y \\in \\mathcal{Y}\\\) such that \\\(\mathbb{E}\_Y\[\\ell\(x, Y\)\] \\le \\min\_{y\\in\\mathcal{Y}} \\ell\(x, y\) + O\(\\frac{1}{\\varepsilon} \\log \|\\mathcal{Y}\|\)\\\), where \\\(\\varepsilon\\\) is the differential privacy parameter. (We will suppress the privacy parameter for simplicity.) Our goal is to replace this with an algorithm that is based only on low-sensitivity queries.

> **Problem 1:** <a id="prob1" />
> There is a \(private\) dataset \\\(x \\in \\mathcal{X}^n\\\) and a \(public\) loss function \\\(\\ell : \\mathcal{X}^n \\times \\mathcal{Y} \\to \\mathbb{R}\\\) that has sensitivity-\\\(1\\\) in its first argument. That is, for all \\\(x,x' \\in \\mathcal{X}^n\\\) differing in a single entry and all \\\(y \\in \\mathcal{Y}\\\), we have \\\(\|\\ell\(x,y\) − \\ell\(x',y\)\| \\le 1\\\). 
> 
> The goal is to construct an algorithm that outputs \\\(Y \\in \\mathcal{Y}\\\) such that \\\[\mathbb{E}\_Y\[\\ell\(x, Y\)\] \\le \\min\_{y\\in\\mathcal{Y}} \\ell\(x, y\) + O\(\\log \|\\mathcal{Y}\|\).\\tag{1}\\\]
> However, the algorithm cannot access \\\(x\\\) directly. Instead there is an oracle which provides noisy answers to \\\(k\\) sensitivity-\\\(1\\\) queries. Specifically, each query is specified by a sensitivity-\\\(1\\\) function \\\(q : \\mathcal{X}^n \\to \\mathbb{R}\\\), which is submitted to the oracle, and the oracle returns a sample from \\\(\\mathcal{N}\(q\(x\),k\)\\\). The number of queries \\\(k\\\) may be chosen arbitrarily and the queries may be specified adaptively.

Obviously, with direct access to \\\(x\\\), this problem would be trivial. And, if the oracle didn't add any noise, it would also be easy \(just query \\\(q\(x\)=\\ell\(x,y\)\\\) for all \\\(y\\in\\mathcal{Y}\\\) -- i.e., \\\(k=\|\\mathcal{Y}\|\\\) queries\).

The noise added by the oracle ensures that the algorithm is differentially private. Thus the goal of this algorithm is directly comparable with the guarantee of the exponential mechanism.

## Partial Solution
<a id="partsoln" />

As a starting point, the following binary-search-based algorithm attains expected excess loss \\\(O\(\(\\log\|\\mathcal{Y}\|\)^{3/2}\)\\\) instead of the desired \\\(O\(\\log\|\\mathcal{Y}\|\)\\\). 

Construct a complete binary tree with the leaves corresponding to the elements of \\\(\\mathcal{Y}\\\). Walk down the tree from the root to a leaf as follows and output the leaf’s element. 
At each node, query the oracle with <a id="eq2" />\\\[q\(x\) = \\frac{1}{2} \\big\(\\min\_{y\\in{\\rm left}} \\ell\(x, y\)\\big\) − \\frac{1}{2} \\big\(\\min\_{y\\in{\\rm right}} \\ell\(x, y\)\\big\).\\tag{2}\\\]  If the oracle’s answer is positive, move to the left child; otherwise, move right.

The number of queries for this algorithm is \\\(k=\\lceil \\log\_2\|\\mathcal{Y}\| \\rceil\\\). And it's easy to check that [Equation 2](#eq2) has sensitivity-\\\(1\\\).

For the utility analysis, let \\\(A\_1,A\_2,\\cdots,A\_k\\\) denote the nodes on the path from the root \\\(A\_1\\\) to the leaf \\\(A\_k\\\) that we output.
We track the minimum loss on the subtree rooted at the current node -- i.e., \\\(B\_i := \\min\_{y \\text{ in subtree rooted at } A\_i} \\ell\(x,y\)\\\).

Initially, we have \\\(B\_1 = \\min\_{y\\in\\mathcal{Y}} \\ell\(x, y\) \\\), which is the desired quantity. And \\\(B\_k\\\) is the loss of the final output.
We also have \\\(B\_1 \\le B\_2 \\le \\cdots \\le B\_k\\\). 
To complete the analysis we need only show that \\\(\\mathbb{E}\[B\_{i+1}\] \\le B\_i + O\(\\sqrt{\\log\|\\mathcal{Y}\|}\)\\\) for all \\\(i\\\).

If at step \\\(i\\\) the algorithm chooses the "correct" child, then \\\(B\_{i+1}=B\_i\\\).
But, if the algorithm chooses the "incorrect" child, we have \\\(B\_{i+1} = B\_i + 2\|q\_i\(x\)\|\\\), where \\\(q\_i\\\) is the query \(given in [Equation 2](#eq2)\) that was asked to the oracle in step \\\(i\\\).

What is the probability of choosing the wrong child? Well, it's the probability that the noise added to \\\(q\_i\(x\)\\\) flips the sign -- i.e., \\\(\\mathbb{P}\[\\mathsf{sign}\(\\mathcal{N}\(q\_i\(x\),k\)\) \\ne \\mathsf{sign}\(q\_i\(x\)\)\]\\\). Putting these together and doing a bit of algebraic manipulation, we have
\\\[ \\mathbb{E}\[B\_{i+1}\] = B\_i + 2\|q\_i\(x\)\| \cdot \\mathbb{P}\[\\mathcal{N}\(0,k\)\\ge\|q\_i\(x\)\|\] \\\]\\\[ ~~~~~~~~~~~~~~~ \\le B\_i + 2\\sqrt{k} \\cdot \\max_{v \\ge 0} v \cdot \\mathbb{P}\[\\mathcal{N}\(0,1\) \\ge v\].\\tag{3}\\\]
The quantity \\\(\\max_{v \\ge 0} v \cdot \\mathbb{P}\[\\mathcal{N}\(0,1\) \\ge v\] \\in \[0.169,0.17\]\\\) is a constant \(attained at \\\(v \\approx 0.75\\\)\).
Thus we have \\\(\\mathbb{E}\[B\_k] \\le 0.34 k^{3/2} = O\(\(\\log\|\\mathcal{Y}\|\)^{3/2}\)\\\).

## Who Cares?

That's a fair question -- why is this open problem interesting?

A positive solution to this open problem would demonstrate the power of low-sensitivity queries and illustrate how almost all differentially private tasks can be boiled down to noise addition. 

In practice, the exponential mechanism works fine, so we don't really _need_ this algorithm.
Nevertheless, I think this could lead to something useful/insightful: There are situations where we can do better than the exponental mechanism or at least better than the standard analysis of the exponential mechanism. An alternative algorithm might open up more avenues for improving on the exponential mechanism.

To give some examples where we know how to beat the standard analysis of the exponential mechanism: Suppose the loss function can be decomposed as \\\[\\ell\(x,y\) = \\ell\(x,\(y\_1,y\_2,\\cdots,y\_d\)\) = \\ell\_1\(x,y\_1\) + \\ell\_2\(x,y\_2\) + \\cdots + \\ell\_d\(x,y\_d\). \\tag{4}\\\]
Then the analysis of the exponential mechanism can also be decomposed into the composition of \\\(d\\\) independent exponential mechanisms, which yields better asymptotic results.
Another example is when there is one option \\\(y\_\* \\in \\mathcal{Y}\\\) that is significantly better than all the other options -- i.e., \\\(\\ell\(x,y\_\*\) \\le \\min\_{y \\in \\mathcal{Y} \\setminus \{y\_\*\}} \\ell\(x,y\) - c\\\), where \\\(c\\\) is sufficiently large. In this case we can privately output \\\(y\_\*\\\) with an improved dependence on the number of options \\\(\|\\mathcal{Y}\|\\\) \[[CHS14](https://arxiv.org/abs/1409.2177 "Kamalika Chaudhuri, Daniel Hsu, Shuang Song. The Large Margin Mechanism for Differentially Private Maximization. NIPS 2014."),[BDRS18](https://dl.acm.org/doi/10.1145/3188745.3188946 " Mark Bun, Cynthia Dwork, Guy N. Rothblum, Thomas Steinke. Composable and versatile privacy via truncated CDP. STOC 2018."),[BKSW19](https://arxiv.org/abs/1905.13229 "Mark Bun, Gautam Kamath, Thomas Steinke, Zhiwei Steven Wu. Private Hypothesis Selection. NeurIPS 2019.")\]. 

A negative solution -- that is, an impossibility result -- would show that selection is a fundamental and indivisible primitive of differentially private algorithms. This would be surprising and thus interesting. The proof technique would presumably also be novel.


## Remarks

This open problem was first [published in 2019](https://dataprivacyopenpro.wixsite.com/mysite/forum). I'm reposting it because, well, it's still open. (There are a few other open problems there, although some have been solved by now.) 

[Problem 1](#prob1) is stated in terms of Gaussian noise addition (and implicitly performs optimal/advanced composition).
The problem also makes sense with Laplace noise addition \(and [basic composition](/composition-basics/)\).
Let's state that formally:

> **Problem 2:** <a id="prob2" />
> There is a \(private\) dataset \\\(x \\in \\mathcal{X}^n\\\) and a \(public\) loss function \\\(\\ell : \\mathcal{X}^n \\times \\mathcal{Y} \\to \\mathbb{R}\\\) that has sensitivity-\\\(1\\\) in its first argument. That is, for all \\\(x,x' \\in \\mathcal{X}^n\\\) differing in a single entry and all \\\(y \\in \\mathcal{Y}\\\), we have \\\(\|\\ell\(x,y\) − \\ell\(x',y\)\| \\le 1\\\). 
> 
> The goal is to construct an algorithm that outputs \\\(Y \\in \\mathcal{Y}\\\) such that \\\[\mathbb{E}\_Y\[\\ell\(x, Y\)\] \\le \\min\_{y\\in\\mathcal{Y}} \\ell\(x, y\) + O\(\\log \|\\mathcal{Y}\|\).\\tag{5}\\\]
> However, the algorithm cannot access \\\(x\\\) directly. Instead there is an oracle which provides noisy answers to \\\(k\\) sensitivity-\\\(1\\\) queries. Specifically, each query is specified by a sensitivity-\\\(1\\\) function \\\(q : \\mathcal{X}^n \\to \\mathbb{R}\\\), which is submitted to the oracle, and the oracle returns a sample from \\\(q\(x\)+\\mathsf{Lap}\(k\)\\\). The number of queries \\\(k\\\) may be chosen arbitrarily and the queries may be specified adaptively.

For pure DP, the [binary tree algorithm](#partsoln) would achieve excess loss \\\(O\(\(\\log \|\\mathcal{Y}\|\)^2\)\\\) instead of \\\(O\(\\log \|\\mathcal{Y}\|\)\\\).

The contrast between pure DP and Gaussian DP is interesting because the exponential mechanism satisfies pure DP and [relaxing to approximate DP doesn't allow us to do any better](https://arxiv.org/abs/1704.03024). But, comparing [Problem 1](#prob1) and [Problem 2](#prob2), it seems like the Gaussian case should be easier.
I can't quite put my finger on it, but I feel like there's something interesting to say about this and I hope resolving this open problem would shed light on it.

A final remark: The reverse reduction is trivial. We can use the exponential mechanism to answer low-sensitivity queries. Namely, we can set \\\(\\ell\(x,y\)=\|q\(x\)-y\|\\\). Thus a positive solution to this problem would show an _equivalence_ between selection and low-sensitivity queries.
