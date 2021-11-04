---
layout: post
title: "A simple recipe for private synthetic data generation"
comments: true
authors:
  - ryanmckenna
timestamp: 10:00:00 -0700
categories: [Tools, Synthetic Data]
---

![](/images/select-measure-reconstruct.png)

In the last blog post, we covered the potential pitfalls of synthetic data without formal privacy guarantees, and motivated the need for differentially private synthetic data mechanisms.  Many mechanisms for this problem follow the so-called **select-measure-generate** paradigm.  The three steps underlying the select-measure-generate paradigm are illustrated in the figure above, and explained below.

1. **Select** a collection of queries to measure --- typically low-dimensional marginals.
2. **Measure** the selected queries privately using a noise-addition mechanism.
3. **Generate** synthetic data that best explains the noisy measurements.

Mechanisms in this class differ primarily in their methodology for selecting queries and their algorithm for generating synthetic data from noisy measurements.  The focus of this blog post is the final **Generate** step, and specifically, the open-source **[Marginal-Based Inference (MBI)](https://github.com/ryan112358/private-pgm){:target="_blank"}** repository that implements several generic and scalable solutions to this problem.


Separating the Generate subroutine from the existing synthetic data generation mechanisms greatly simplifies the design space of new differentially private mechanisms.  It allows the mechanism designer to focus on *selecting the queries* to maximize utility of the synthetic data, rather than *how to generate synthetic data* that explain the noisy measurements well.  Both are challenging problems, and MBI provides principled solutions to the latter problem, while exposing a simple interface that can be readily adopted by mechanism designers. 


Among the methods implemented in MBI is [PGM](https://arxiv.org/abs/1901.09136){:target="\_blank"}, which was a key component of the first-place solution in the 2018 NIST Differential Privacy [Synthetic Data Competition](https://www.nist.gov/ctl/pscr/open-innovation-prize-challenges/past-prize-challenges/2018-differential-privacy-synthetic){:target="\_blank"} and in both the first and second-place solutions in the follow-up [Temporal Map Competition](https://www.nist.gov/ctl/pscr/open-innovation-prize-challenges/current-and-upcoming-prize-challenges/2020-differential){:target="\_blank"}.[^2]  Other methods implemented in MBI have seen success in the academic settings as well. 

[^2]: Other top-performing mechanisms followed the select-measure-generate paradigm as well, but used other methods for the generate step.

After reading this blog post you will:

* Be able to use MBI to design your first synthetic data mechanism. 
* Understand how different methods for Generate are related and differ.
* Be ready to compete in the next synthetic data competition!


# Getting Started

We will begin by introducing the basic concepts necessary to understand and develop mechanisms in the select-measure-generate paradigm.  We will accomplish this using real Python code based on the open source MBI repository.

### Discrete Data

For the purpose of this blog post, assume we have a preprocessed discrete dataset, such that each attribute \\\( i \\\) can take on values from the set \\\(\\\{0, 1, \dots, n_i-1 \\\} \\\) where \\\( n_i \\\) is the number of possible values for a given column.  The open source MBI repository contains a preprocessed adult dataset.  We can load it in using the following code:

{% highlight python %}
>>> from mbi import Dataset
>>> data = Dataset.load('adult.csv', 'adult-domain.json')
{% endhighlight %}

The Dataset object is essentially a wrapper for a pandas data frame, with utility functions to compute marginals.  It also keeps track of domain information, or the set of columns along with the number of possible values for each column.   Inspecting this domain object shows that age has 85 possible values, workclass has 9, education-num has 16, etc.

{% highlight python %}
>>> data.domain
Domain(age: 85, workclass: 9, fnlwgt: 100, education-num: 16, marital-status: 7, occupation: 15, relationship: 6, race: 5, sex: 2, capital-gain: 100, capital-loss: 100, hours-per-week: 99, native-country: 42, income>50K: 2)
{% endhighlight %}

### Marginals

A marginal for a subset of attributes counts the number of records in the dataset that match each setting of possible values.  It is essentially a histogram over a subset of attributes.  Marginals can easily be obtained in MBI using the `project` and `datavector` functions.  For example:

{% highlight python %}
>>> data.project('relationship').datavector()
array([ 2331.,  7581., 19716., 12583.,  1506.,  5125.])
{% endhighlight %}

The relationship marginal \\\( \mu \\\) is a length 6 array, where \\\( \mu_j \\\) counts the number of records with relationship \\\( = j \\\).  We can similarly calculate marginals involving more than one attribute as follows:

{% highlight python %}
>>> data.project(['marital-status', 'sex']).datavector(flatten=False)
array([[ 2480., 19899.],
       [ 4001.,  2632.],
       [ 7218.,  8899.],
       [  931.,   599.],
       [ 1233.,   285.],
       [  304.,   324.],
       [   25.,    12.]])
{% endhighlight %}

This marginal is a \\\( 7 \times 2 \\\) array \\\( \mu \\\), where \\\( \mu\_{jk} \\\) counts the number of records with marital-status \\\( = j \\\) and sex \\\( = k \\\).  Low-dimensional marginals are particularly useful for differentially private synthetic data for a number of reasons:

* They capture low-dimensional structure common in real world data distributions.
* Each cell in a marginal is a count, a statistic that is fairly robust to noise.   
* One individual can only contribute to a single cell of a marginal, so all cells have low sensitivity and can be measured simultaneously with low privacy cost.

These observations together make low-dimensional marginals an ideal statistic to measure with differential privacy.  We can readily invoke the Gaussian mechanism to privately answer a marginal using standard numpy code, as follows:

{% highlight python %}
>>> marginal = data.project(['marital-status', 'sex']).datavector(flatten=False)
>>> noisy_marginal = marginal + np.random.normal(loc=0, scale=50, size=marginal.shape)
>>> noisy_marginal
array([[ 2568.2026173 , 19919.00786042],
       [ 4049.93689921,  2744.04465996],
       [ 7311.37789951,  8850.13610601],
       [  978.50442088,   591.43213959],
       [ 1227.83905741,   305.5299251 ],
       [  311.20217856,   396.71367535],
       [   63.05188626,    18.08375082]])
{% endhighlight %}

The computation above is a \\\( \rho \\\)-zCDP for \\\( \rho = \frac{1}{2 \sigma^2} = \frac{1}{5000} \\\) under the add/remove definition of DP.

# Preparing Noisy Measurements 

The input to any method for Generate is a collection of noisy measurements.  We show below how to prepare these measurements in a format compatible with the methods for Generate implemented in the MBI repository.  The measurements are represented as a list, where each element of the list is a noisy marginal (represented as a numpy array), along with relevant metadata including the attributes in the marginal and the amount of noise used to answer it.  In the code snippet below, the selected marginals are hard-coded in "cliques", but in general these can be any collection of attribute subsets. 

{% highlight python %}
import numpy as np
from scipy import sparse
from mbi import Dataset

data = Dataset.load('adult.csv', 'adult-domain.json')

# SELECT the marginals we'd like to measure
cliques = [('marital-status', 'sex'),
           ('education-num', 'race'),
           ('sex', 'hours-per-week'),
           ('workclass',),
           ('marital-status', 'occupation', 'income>50K')]

# MEASURE the marginals and log the noisy answers
sigma = 50 
measurements = []
for cl in cliques:
    x = data.project(cl).datavector()
    y = x + np.random.normal(loc=0, scale=sigma, size=x.shape)
    I = sparse.eye(x.size)
    measurements.append( (I, y, sigma, cl) )
{% endhighlight %}

The above mechanism is a 5-fold composition of \\\( \frac{1}{5000} \\\)-zCDP mechanisms, so the entire mechanism satisfies \\\( \frac{1}{1000} \\\)-zCDP.

# Generating Synthetic Data from Measurements

Given measurements represented in the format above, we can readily generate synthetic data using one of several methods.  For example, the code snippet below generates synthetic data that approximately matches the noisy measurements: 

{% highlight python %}
from mbi import FactoredInference
from mbi import MixtureInference
from mbi import SparseInference
from mbi import LocalInference

# GENERATE synthetic data using MBI 
engine = FactoredInference(data.domain, iters=2500)
model = engine.estimate(measurements)
synth = model.synthetic_data()
{% endhighlight %}

```FactoredInference```  is the original method for Generate implemented in MBI, corresponding to the technique known as PGM.  The other Inference methods imported represent novel alternative techniques introduced after the introduction of PGM that can be usd in place of it if desired.  These will be explained in more detail in the [last section](#the-generate-subproblem-a-unifying-view).

Inspecting the (marital-status, sex) marginal reveals that the counts in the synthetic data approximately match the real counts, as desired.

{% highlight python %}
>>> synth.project(['marital-status','sex']).datavector(flatten=False)
array([[ 2474., 19874.],
       [ 4050.,  2583.],
       [ 7211.,  8821.],
       [  812.,   627.],
       [ 1271.,   227.],
       [  435.,   256.],
       [   59.,    20.]])
{% endhighlight %}

Some marginals are not preserved as well as others, however. For example, the relationship marginal is uniformly distributed in the synthetic data, but not in the real data.  This is not a failure of the Generate algorithm, but rather an artifact of what queries were selected: since none of the selected marginals include the relationship attribute, it is not surprising that the synthetic data does not preserve its distribution.

{% highlight python %}
>>> data.project(['relationship']).datavector()
array([ 2331.,  7581., 19716., 12583.,  1506.,  5125.])

>>> synth.project(['relationship']).datavector()
array([8120., 8120., 8120., 8120., 8120., 8120.])
{% endhighlight %}

This example shows that the quality of the synthetic data crucially depends on the selected queries.  Selecting a good set of queries to measure remains an important open problem that will be the topic of the next blog post.

# The Generate Subproblem: A Unifying View

Now that we have described how to generate synthetic data from noisy measurements using real code, we will offer some insight into what this code does under the hood.  We will provide a unifying view of several approaches to the generate subproblem.  Underlying all of these approaches is the following optimization problem:

\\\[\hat{P} \in \text{arg} \min\_{P \in \mathcal{S}} L(P, \tilde{Q}) \\tag{1} \label{eq1} \\\]

Here, \\\( \tilde{Q} \\\) are the noisy query answers, and \\\( \mathcal{S} \\\) is the set of all distributions over the data domain.  We seek to find a data distribution \\\( \hat{P} \\\) that *best explains* the noisy observations \\\( \tilde{Q} \\\) according to the loss function \\\( L \\\).  Different loss functions are possible, but by default MBI uses the \\\( L_2 \\\) squared loss,
\\\( L(P) = \|\| Q(P) - \tilde{Q} \|\|\_2^2 \\\).  Intuitively, we seek to find a data distribution \\\( P \\\) such that \\\( Q(P) \\\), the query answers under \\\( P \\\), are close to the noisy query answers \\\( \tilde{Q} \\\), subject to the constraint that \\\( P \\\) is a probability distribution.  

### Direct

If the queries \\\( Q \\\) are linear, then this optimization problem is convex, and hence can be solved in theory.  However, solving it directly is challenging on high-dimensional domains because the size of \\\( P \\\) is equal to the size of the domain, which is often intractably large.  For example, the domain size of the adult dataset considered earlier is  \\\( 6.4 \times 10^{17} \\\).  Writing down a single distribution over this domain is intractable, let alone optimizing over the space of all distributions.  

There are several methods described below that attempt to overcome the curse of dimensionality inherent in Problem \ref{eq1}.  These methods scale by solving a relaxation of Problem \ref{eq1} that is tractable to solve.  One way to do this is to restrict the search space of the optimization from all high-dimensional joint distributions, to a subset of joint distributions.  This subset of distributions can be characterized by a smaller set of parameters, which enables tractable optimization over it.  The sections below describe the different (implicit) assumptions each method makes, as well as the consequences of those assumptions.  

### PGM (```FactoredInference```)

PGM, short for Probabilistic Graphical Model, works by finding a graphical model \\\( P\_{\theta} \\\).  The key observation of PGM is that when \\\( Q \\\) only depends on \\\( P \\\) through it's low-dimensional marginals, then one of the optimizers of Problem \ref{eq1} is a graphical model with parameters \\\( \theta \\\).  Problem \ref{eq1} typically has infinitely many solutions, and it turns out that the solution found by PGM has maximum entropy among all solutions to the problem --- a very natural way to break ties among equally good solutions. Remarkably, these facts are true for any dataset --- they do not require the underlying data to be generated from a graphical model with the same structure.  For more information on this matter, please refer to Section 4.2 of [[MMS21]](https://arxiv.org/abs/2108.04978){:target="\_blank"}.

The complexity of PGM depends on \\\( Q \\\), and in the worst case is the same as the Direct method [^6].  However, in many common cases of practical interest, the complexity of PGM is exponentially better than that of Direct, in which case we can efficiently solve the optimization problem above, finding \\\( \theta \\\) and thus a tractable representation of \\\( \hat{P} \\\).  The complexity of PGM ultimately depends on the size of the junction tree derived from the measured marginals, and understanding the relationship between \\\( Q \\\) and the size of the junction tree requires some expertise in graphical models.  However, the MBI code allows you to quickly check how big the required junction tree is, based on which cliques were measured.  The size of the junction tree in the example above would be only 944, whereas the size of \\\( P \\\) would be \\\( 6.4 \times 10^{17} \\\).  

[^6]: For example, this worst-case behavior is realized if **all** 2-way marginals are measured.  While this can be seen as a limitation of PGM, [it is known](http://people.seas.harvard.edu/~salil/research/synthetic-Feb2010.pdf) that generating synthetic data that preserves all 2-way marginals is computationally hard in the worst-case.

{% highlight python %}
>>> from mbi import GraphicalModel
>>> model = GraphicalModel(data.domain, cliques)
>>> model.size
944
>>> data.domain.size()
641263392000000000
{% endhighlight %}

The size of the junction tree increases with the number and size of the selected cliques, but it also depends on the structure of the cliques as well.  This dependence is explained well in a recent paper, [PrivMRF](http://vldb.org/pvldb/vol14/p2190-cai.pdf){:target="\_blank"}.  A more detailed but still accessible overview of MBI is given in another recent paper, [MST](https://arxiv.org/abs/2108.04978){:target="\_blank"}.  

### Relaxed Tabular (```MixtureInference```)

An alternative approach was proposed in the recent [RAP](https://arxiv.org/abs/2103.06641){:target="\_blank"} paper.  The key idea is to restrict attention to "pseudo-distributions" that can be represented in a relaxed tabular format.  The format is similar to the one-hot encoding of a discrete dataset, although the entries need not be \\\( 0 \\\) or \\\( 1 \\\), which enables gradient-based optimization to be performed on the cells in this table.  The number of rows is a tunable knob that can be set to trade off expressive capacity with computational efficiency.  With a sufficiently large knob size, the true minimizer of the original problem can be expressed in this way, but there is no guarantee that gradient-based optimization will converge to it because this representation introduces non-convexity.  This idea was refined into [RAP<sup>softmax</sup>](https://arxiv.org/abs/2106.07153){:target="\_blank"} in follow-up-work, and was recently added to the MBI repository under the name ```MixtureInference```.  It can be used as a drop-in replacement for ```FactoredInference``` in situations where PGM fails to scale.  

### Generative Networks 

Another approach proposed in the recent [GEM](https://arxiv.org/abs/2106.07153){:target="\_blank"} paper is to search over the space of distributions representable as a generative network.  The distribution is implicitly encoded via the parameters of the network, which are learned through gradient-based optimization.  It can be seen as a compact parameterization of a mixture of products with an infinite number of mixture components.  It shares some similarity with the previous approach, although optimization is done indirectly via the network parameters instead of directly over the probabilities in the mixture components.  We hope it can be added to the MBI repository in the future. 

### Local Consistency (```LocalInference```)

Finally, [GUM](https://arxiv.org/abs/2106.07153){:target="\_blank"} and [APPGM](https://arxiv.org/abs/2109.06153){:target="\_blank"} do not search over any space of distributions, but instead impose *local consistency* constraints on the noisy measurements.  That is, they optimize over the space of pseudo-marginals, rather than distributions.  The pseudo-marginals are required to be internally consistent, but there is no guarantee that there is a distribution which realizes those pseudo-marginals.  Synthetic data can be obtained from a post-processing heuristic to translate these locally consistent pseudo-marginals into synthetic tabular data.  This approach was used by team DPSyn in both NIST competitions.  APPGM was recently added to the MBI repository under the name ```LocalInference```.  It can be used as a drop-in replacement for ```FactoredInference``` in situations where PGM fails to scale.

### Summary

Among the alternatives discussed here, only Direct and MBI can be expected to solve Problem \ref{eq1}.  PGM scales by exploiting the structure in the marginals measured, whereas other methods scale by restricting the space of distributions considered in a way that does not depend on the measurements taken.  While these ideas are related, there is a subtle but important distinction between them.  PGM can be seen as restricting the search space, but it does so carefully to ensure that the solution obtained still solves the original optimization problem (over the space of all distributions).  The other methods do not share this property, instead the solutions found solve different relaxations of the original optimization problem, and the solutions found will generally be suboptimal in the original problem.  This distinguishing property of PGM comes at a cost: it's scalability can suffer when the number of marginals measured becomes too large, a drawback not suffered as severely by the other methods.  When using PGM, some care must be taken when selecting marginals to avoid the worst-case behavior, as is done in two state-of-the-art mechanisms: [PrivMRF](http://vldb.org/pvldb/vol14/p2190-cai.pdf){:target="\_blank"} and [MST](https://arxiv.org/abs/2108.04978){:target="\_blank"}.  

# Coming up Next

In this blog post, we focused on the **Generate** step of the select-measure-generate paradigm.  By leveraging the MBI repository for this step, you can develop end-to-end mechanisms for synthetic data with surprisingly little code.  In the next blog post, we will focus on state-of-the-art approaches to the **Select** sub-problem.  If you have any comments, questions, or remarks, please feel free to share them in the comments section below.  If you would like to try generating synthetic data with MBI, check out this [jupyter notebook](https://colab.research.google.com/drive/1c8gT5m_GWfQoa_mx8eXh4sPD48Y0z3ML?usp=sharing) on Google Colab!  

---
