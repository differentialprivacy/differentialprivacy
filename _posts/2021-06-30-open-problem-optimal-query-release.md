---
layout: post
title: "Open Problem - Optimal Query Release for Pure Differential Privacy"
comments: true
authors:
  - sashonikolov
  - jonullman
bibtexauthors: "Aleksandar Nikolov and Jonathan Ullman"
timestamp: 19:00:00 -0400
categories: [Open Problems]
---

Releasing large sets of statistical queries is a centerpiece of the theory of differential privacy.  Here, we are given a <em>dataset</em> \\\(x = (x_1,\dots,x_n) \in [T]^n\\\), and a set of <em>statistical queries</em> \\\(f_1,\dots,f_k\\\), where each query is defined by some bounded function \\\(f_j : [T] \to [-1,1]\\\), and (abusing notation) is defined as
\\\[
f_j(x) = \frac{1}{n} \sum_{i=1}^{n} f_j(x_i).
\\\]
We use \\\(f(x) = (f_1(x),\dots,f_k(x))\\\) to denote the vector consisting of the true answers to all these queries.
Our goal is to design an \\\((\varepsilon, \delta)\\\)-differentially private algorithm \\\(M\\\) that takes a dataset \\\(x\in [T]^n\\\) and outputs a random vector \\\(M(x)\in \mathbb{R}^k\\\) such that \\\(\\\| M(x) - f(x) \\\|\\\) is small in expectation for some norm \\\(\\\|\cdot\\\|\\\). Usually algorithms for this problem also give high probability bounds on the error, but we focus on expected error for simplicity.

This problem has been studied for both <em>pure differential privacy</em> (\\\(\delta = 0\\\)) and <em>appproximate differential privacy</em> (\\\(\delta > 0\\\)), and for both \\\(\ell_\infty\\\)-error
\\\[
\mathbb{E}( \\\| M(x) - f(x)\\\|\_{\infty} ) \leq \alpha,
\\\]
and \\\(\ell_2\\\)-error
\\\[
\mathbb{E}( \\\| M(x) - f(x)\\\|\_{2} ) \leq \alpha k^{1/2},
\\\]
giving four variants of the problem.  By now we know tight worst-case upper and lower bounds for two of these variants, and nearly tight bounds (up to logarithmic factors) for a third. The tightest known upper bounds are given in the following table.

|      | Pure DP | Approx DP |
| \\\( \ell_2 \\\)<br>error      | \\\( \alpha \lesssim \left(\frac{\log^2 k ~\cdot~ \log^{3/2}T}{\varepsilon n} \right)^{1/2} \\\) <br> [[NTZ13](https://arxiv.org/abs/1212.0297)]     | \\\( \alpha \lesssim \left(\frac{\log^{1/2} T}{\varepsilon n} \right)^{1/2} \\\) <br> [[DRV10](https://guyrothblum.files.wordpress.com/2014/11/drv10.pdf)] |
| \\\( \ell_\infty \\\)<br>error  | \\\( \alpha \lesssim \left(\frac{\log k ~\cdot~ \log T}{\varepsilon n} \right)^{1/3} \\\)  <br> [[BLR13](https://arxiv.org/abs/1109.2229)]      | \\\( \alpha \lesssim \left(\frac{\log k ~\cdot~ \log^{1/2} T}{\varepsilon n} \right)^{1/2} \\\) <br> [[HR10](https://guyrothblum.files.wordpress.com/2014/11/hr10.pdf), [GRU12](https://arxiv.org/abs/1107.3731)] |

The bounds for approximate DP are known to be tight [[BUV14](https://arxiv.org/abs/1311.3158)].  Our two open problems both involve improving the best known upper bounds for pure differential privacy.

>    <b>Open Problem 1:</b> What is the best possible \\\(\ell_\infty\\\)-error for answering a worst-case set of \\\(k\\\) statistical queries over a domain of size \\\(T\\\) subject to \\\((\varepsilon,0)\\\)-differential privacy?

We conjecture that the known upper bound in the table can be improved to
\\\[
\alpha = \left(\frac{\log k \cdot \log T}{\varepsilon n} \right)^{1/2},
\\\]
which is known to be the best possible [[Har11](https://dataspace.princeton.edu/handle/88435/dsp01vq27zn422), Theorem 4.5.1].


>    <b>Open Problem 2:</b> What is the best possible \\\(\ell_2\\\)-error for answering a worst-case set of \\\(k\\\) statistical queries over a domain of size \\\(T\\\) subject to \\\((\varepsilon,0)\\\)-differential privacy?

We conjecture that the upper bound can be improved to
\\\[
\alpha = \left(\frac{\log T}{\varepsilon n} \right)^{1/2}.
\\\]
The construction used in [[Har11](https://dataspace.princeton.edu/handle/88435/dsp01vq27zn422), Theorem 4.5.1] can be analyzed to show this bound would be tight. Note, in particular, that this conjecture implies that the tight upper bound has no dependence on the number of queries, similarly to the case of \\\(\ell_2\\\) error and approximate DP.
