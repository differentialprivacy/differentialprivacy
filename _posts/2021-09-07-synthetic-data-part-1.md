---
layout: post
title: "A simple recipe for private synthetic data generation"
comments: true
authors:
  - ryanmckenna
timestamp: 10:00:00 -0700
categories: [Tools, Synthetic Data]
---

Differentially private synthetic data is an open research problem that is of great importance to the privacy community, and has received increased attention in recent years.  In 2018, the National Institute of Standards and Technology (NIST) organized the [Differential Privacy Synthetic Data Competition](https://www.nist.gov/ctl/pscr/open-innovation-prize-challenges/past-prize-challenges/2018-differential-privacy-synthetic){:target="_blank"}, a three round competition with $150,000 in prizes where participants were challenged with developing novel and practical mechanisms for synthetic data generation.  In 2020, NIST organized the followup [Temporal Map Competition](https://www.nist.gov/ctl/pscr/open-innovation-prize-challenges/current-and-upcoming-prize-challenges/2020-differential){:target="_blank"}, which was similar in spirit to the original competition with new technical challenges to overcome.  While the details of these competitions are out of scope for this blog post, we will talk about the basic approach employed by the best-performing mechanisms in both competitions.  These mechanisms, along with many others in the literature, follow the so-called **select-measure-generate** paradigm.

# The Select-Measure-Generate Paradigm

![](/images/select-measure-reconstruct.png)

The three steps underlying the select-measure-generate paradigm are illustrated in the figure above, and explained below.

1. **Select** a collection of queries (typically low-dimensional marginals) to measure, either manually by a domain expert, or automatically by an algorithm.   
2. **Measure** the selected queries using a noise-addition mechanism (e.g., Laplace/Gaussian).
3. **Generate** synthetic data that best explains the noisy measurements.

Mechanisms in this class differ primarily in their methodology for selecting queries and their algorithm for generating synthetic data from noisy measurements.  The focus of this blog post is the final **Generate** step, and specifically, the open-source **[Private-PGM](https://github.com/ryan112358/private-pgm){:target="_blank"}** tool that solves this problem in a generic and scalable way.  

Private-PGM was a key component of the first-place solution in the 2018 competition, and in both the first and second-place solution in the 2020 competition.[^2]  It allows the mechanism designer to focus on *what queries to measure* to maximize utility of the synthetic data, rather than *how to post-process* the noisy measurements to obtain synthetic data.  Both are challenging problems, and Private-PGM provides a principled, robust, and elegant solution to the latter problem, while exposing a simple interface that can be readily used by mechanism designers. 

[^2]: Other top-performing mechanisms followed the select-measure-generate paradigm as well, but used other methods for the generate step.

After reading this blog post you will:

* Be able to use Private-PGM to design your first synthetic data mechanism.
* Understand what Private-PGM does, how it works, and what it's limitations are.
* Learn about alternatives to Private-PGM for the generate step.
* Be ready to compete in the next synthetic data competition!

# Discrete Data

For the purpose of this blog post, assume we have a preprocessed discrete dataset, such that each attribute \\\( i \\\) can take on values from the set \\\(\\\{0, 1, \dots, n_i-1 \\\} \\\) where \\\( n_i \\\) is the number of possible values for a given column.  The open source Private-PGM repository contains a preprocessed adult dataset.  We can load it in using the following code:

{% highlight python %}
>>> from mbi import Dataset
>>> data = Dataset.load('adult.csv', 'adult-domain.json')
{% endhighlight %}

The Dataset object is essentially a wrapper for pandas data frame, with utility functions to compute marginals.  It also keeps track of domain information, or the set of columns along with the number of possible values for each column.   Inspecting this domain object shows that age has 85 possible values, workclass has 9, education-num has 16, etc.

{% highlight python %}
>>> data.domain
Domain(age: 85, workclass: 9, fnlwgt: 100, education-num: 16, marital-status: 7, occupation: 15, relationship: 6, race: 5, sex: 2, capital-gain: 100, capital-loss: 100, hours-per-week: 99, native-country: 42, income>50K: 2)
{% endhighlight %}

# Marginals

A marginal for a subset of attributes counts the number of records in the dataset that match each setting of possible values.  It is essentially a histogram over a subset of attributes.  Marginals can easily be obtained in Private-PGM using the `project` and `datavector` functions.  For example:

{% highlight python %}
>>> data.project('relationship').datavector()
array([ 2331.,  7581., 19716., 12583.,  1506.,  5125.])
{% endhighlight %}

The relationship marginal \\\( \mu \\\) is a length 6 array, where \\\( \mu_j \\\) counts the number of records with relationship=j.  We can similarly calculate marginals involving more than one attribute as follows:

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

This marginal is a \\\( 7 \times 2 \\\) array \\\( \mu \\\), where \\\( \mu\_{jk} \\\) counts the number of records with marital-status=j and sex=k.  Low-dimensional marginals are particularly useful for differentially private synthetic data for a number of reasons:

* They capture low-dimensional structure common in real world data distributions.
* Each count is a coarse-grained aggregation, so the statistic is fairly robust to noise.   
* One individual can only contribute to a single cell of a marginal, so we can measure every cell of the marginal without using extra privacy budget.

These observations together make low-dimensional marginals an ideal statistic to measure with differential privacy.  We can readily invoke the Gaussian mechanism using standard numpy code, as follows:

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

# Private-PGM

Private-PGM is a post-processing tool that consumes a collection of noisy measurements, and it finds a probability distribution that best explains the measurements.  It expects the input to be a list of measurements defined over the data marginals.  In the example below, we simply add Gaussian noise to each marginal, although other types of measurements are also possible.  The code snippit below is an *end-to-end* mechanism for differentially private synthetic data on the adult dataset that only requires 25 lines of python code.

{% highlight python %}
import numpy as np
from scipy import sparse
from mbi import Dataset, FactoredInference

data = Dataset.load('../data/adult.csv', '../data/adult-domain.json')

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

# GENERATE synthetic data using Private-PGM 
engine = FactoredInference(data.domain, iters=2500)
model = engine.estimate(measurements)
synth = model.synthetic_data()
{% endhighlight %}

If you are interested in running or modifying the example above, you can do so on [Google Colab](https://colab.research.google.com/drive/1c8gT5m_GWfQoa_mx8eXh4sPD48Y0z3ML?usp=sharing){:target="\_blank"} in under 2 minutes!  The input is a Dataset object, and the output is also a Dataset object that shares the same domain.  Inspecting the (marital-status, sex) marginal reveals that the counts in the synthetic data approximately match the real counts, as desired.   

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

Some marginals are not preserved as well as others, however. For example, the relationship marginal is uniformly distributed in the synthetic data, but not in the real data.  This is not a failure of Private-PGM, but rather an artifact of what queries were selected: since none of the selected marginals include the relationship attribute, it is not surprising that the synthetic data does not preserve its distribution.  

{% highlight python %}
>>> data.project(['relationship']).datavector()
array([ 2331.,  7581., 19716., 12583.,  1506.,  5125.])

>>> synth.project(['relationship']).datavector()
array([8120., 8120., 8120., 8120., 8120., 8120.])
{% endhighlight %}

This example shows that the quality of the synthetic data crucially depends on the selected queries.  In the original [Private-PGM paper](https://arxiv.org/pdf/1901.09136.pdf), queries were selected by existing differentially private mechanisms (MWEM, PrivBayes, HDMM, and DualQuery), and in every case Private-PGM was shown to improve utility of those base mechanisms.  However, the true power of Private-PGM is that it enables simpler development of *new mechanisms* for synthetic data, a promising and under-explored area for future research.  Selecting a good set of queries to measure is an important open problem, and will be the topic of the next blog post.

# The Projection Problem

Underlying Private-PGM and several other approaches to generate synthetic data is the following optimization problem:
\\\[ \hat{P} \in \text{arg} \min\_{P \in \mathcal{S}} L(P, \tilde{Q}) \\\]
Here, \\\( \tilde{Q} \\\) are the noisy query answers, and \\\( \mathcal{S} \\\) is the set of all distributions over the data domain.  We seek to find a data distribution \\\( \hat{P} \\\) that *best explains* the noisy observations \\\( \tilde{Q} \\\) according to the loss function \\\( L \\\).  For the sake of concreteness, we can consider 
\\\( L(P) = \|\| Q(P) - \tilde{Q} \|\|\_2^2, \\\)
where \\\( Q(P) \\\) are the query answers under \\\( P \\\).  The above problem is known by many names: projection, consistency, maximum likelihood, least squares, etc.  

If the queries \\\( Q \\\) are linear, then this optimization problem is convex, but solving it directly is challenging on high-dimensional domains because the size of \\\( P \\\) is intractably large.  The key observation of Private-PGM is that when \\\( Q \\\) has special structure, so does \\\( \hat{P} \\\).  In particular, if \\\( Q \\\) only depends on \\\( P \\\) through it's low-dimensional marginals, then \\\( \hat{P} = P\_{\hat{\theta}} \\\), a graphical model parameterized by \\\( \hat{\theta} \\\).  The size of \\\( \hat{\theta} \\\) depends on \\\( Q \\\), and in the worst case is equal to the size of \\\( P \\\).  However, in many common cases of practical interest, the size of \\\( \hat{\theta} \\\) is exponentially smaller than \\\( P \\\), in which case we can efficiently solve the optimization problem above, finding \\\( \hat{\theta} \\\) along with a tractable representation of \\\( \hat{P} \\\).  Understanding the relationship between \\\( Q \\\) and the size of \\\( \hat{\theta} \\\) requires some expertise in graphical models.  However, the Private-PGM code allows you to quickly check how big the required graphical model is, based on which cliques were measured.  For example, the size of \\\( \theta \\\) in the code above would be 944, whereas the size of \\\( P \\\) would be \\\( 6.4 \times 10^{17} \\\).  

{% highlight python %}
>>> from mbi import GraphicalModel
>>> model = GraphicalModel(data.domain, cliques)
>>> model.size
944
>>> data.domain.size()
641263392000000000
{% endhighlight %}

The size of the parameter vector increases with the number and size of the selected cliques, but it also depends on the structure of the cliques as well.  This dependence is explained well in a recent paper, [PrivMRF](http://vldb.org/pvldb/vol14/p2190-cai.pdf){:target="\_blank"}.  A more detailed but still accessible overview of Private-PGM is given in another recent paper, [MST](https://arxiv.org/abs/2108.04978){:target="\_blank"}.  

# Alternatives to Private-PGM

Private-PGM is not the only method available for generating synthetic data from noisy measurements.  We briefly describe several other alternatives below, and include a table comparing the different options.  These methods differ primarily in the (implicit) assumptions they make, and the consequences of those assumptions.

### Public Data

One alternative was proposed in the recent [PMW<sup>Pub</sup>](https://arxiv.org/abs/2102.08598){:target="\_blank"} paper and refined by [one team](https://arxiv.org/abs/2106.05131){:target="\_blank"} in the recent NIST competition.  The basic idea is to restrict attention to distributions supported over the domain of some public data.  Distributions in this space can always be tractably represented, as the size of the optimization variable is upper bounded by the number of records in the public dataset.  While the size of the optimization variable does not depend on the structure of the selected queries, there is no guarantee that an optimizer of the original problem exists in this restricted space.  Indeed, the quality of the solution depends crucially on the expressive capacity of the public data domain, as well as how well it matches the true data domain.

### Mixture of Products

An alternative approach was proposed in the recent [RAP](https://arxiv.org/abs/2103.06641){:target="\_blank"} paper and refined in [RAP<sup>softmax</sup>](https://arxiv.org/abs/2106.07153){:target="\_blank"}.  Although not originally presented in this way, these methods essentially search over the space of distributions that can be represented as a *mixture of products*,  or a sum of distributions that factor over each attribute in the domain.  The number of mixture components is a tunable knob that can be set to trade off expressive capacity with computational efficiency.  With a sufficiently large knob size, the true minimizer can be expressed as a mixture of products, but there is no guarantee that gradient-based optimization will converge to it because this representation introduces non-convexity into the problem.  

### Generative Networks 

Another approach proposed in the recent [GEM](https://arxiv.org/abs/2106.07153){:target="\_blank"} paper is to search over the space of distributions representable as a generative network.  The distribution is implicitly encoded via the parameters of the network, which are learned through gradient-based optimization.  It can be seen as a compact parameterization of a mixture of products with an infinite number of mixture components.  Therefore, it shares some similarity with the previous approach, although optimization is done indirectly via the network parameters instead of directly over the probabilities in the mixture components.

### Local Consistency

Finally, [GUM](https://arxiv.org/abs/2106.07153) does not search over any space of distributions, but instead imposes *local consistency* constraints on the noisy measurements.  That is, it ensures that noisy marginals are internally consistent on common margins, but does not necessarily require there to be a distribution which realizes those marginals.  Synthetic data is obtained from a post-processing heuristic to translate these locally consistent pseudo-marginals into synthetic tabular data.  This approach was used by team DPSyn in both NIST competitions. 

### Summary

There are many alternatives to Private-PGM which make different assumptions to enable scalability.  While Private-PGM can be seen as an exact method, it's scalability can suffer when the number of marginals measured becomes too large, a drawback not suffered as severely by the other methods.  A comprehensive qualitative comparison between Private-PGM and the discussed alternatives is given below.  A direct quantitative comparison between the methods remains an unanswered question.

| | **Private-PGM** | **Public Data** | **Mixture of Products** | **Generative Networks** | **GUM** |
Solves original problem | <span style="color:green">Yes</span> | <span style="color:red">No</span> | <span style="color:red">No</span> | <span style="color:red">No</span> | <span style="color:red">No</span> | 
Convexity preserving | <span style="color:green">Yes</span> | <span style="color:green">Yes</span> | <span style="color:red">No</span> |  <span style="color:red">No</span> | <span style="color:green">Yes</span> |
Open source with simple interface [^3] | <span style="color:green">Yes</span> | <span style="color:red">No</span> | <span style="color:red">No</span> | <span style="color:red">No</span> | <span style="color:red">No</span> |
Requires public data | <span style="color:green">No</span> | <span style="color:red">Yes</span> | <span style="color:green">No</span> | <span style="color:green">No</span> | <span style="color:green">No</span> |
Dependence on number of measurements | <span style="color:red">Exponential (worst case)</span> | <span style="color:green">Polynomial</span> | <span style="color:green">Polynomial</span> | <span style="color:green">Polynomial</span> | <span style="color:green">Polynomial</span> |
Scales by | Exploiting structure | Restricting search space | Restricting search space | Restricting search space | Local consistency |
Method for breaking ties | Maximum entropy | None | None | None | None | 
Supported inputs | <span style="color:green">Differentiable loss function of marginals</span> | <span style="color:green">Differentiable loss function of marginals</span> | <span style="color:green">Differentiable loss function of marginals</span> | <span style="color:green">Differentiable loss function of marginals</span> | <span style="color:red">Noisy marginals</span> | 


[^3]: Some of these techniques have open source implementations, but the portion of the code corresponding to the generate step is not treated as a standalone entity, but rather a single component of one proposed mechanism.  Combining it with new mechanisms may require modifying the code and interface.

# Coming up Next

In this blog post, we focused on the **Generate** step of the select-measure-generate paradigm.  In the next blog post, we will focus on state-of-the-art approaches to the **Select** sub-problem.  If you have any comments, questions, or remarks, please feel free to share them in the comments section below.  If you would like to try generating synthetic data with Private-PGM, check out this [jupyter notebook](https://colab.research.google.com/drive/1c8gT5m_GWfQoa_mx8eXh4sPD48Y0z3ML?usp=sharing) on Google Colab!  

---