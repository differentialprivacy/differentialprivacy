---
layout: post
title: The Pitfalls of Average-Case Differential Privacy
comments: true
authors: 
    - thomassteinke 
    - jonullman
categories: [Definitions]
timestamp: 10:00:00 -0400
---

Differential privacy protects against extremely strong adversaries---even ones who know the entire dataset except for one bit of information about one individual.  Since its inception, people have considered ways to relax the definition to assume a more realistic adversary.  A natural way to do so is to incorporate some distributional assumptions. That is, rather than considering a worst-case dataset, assume the dataset is drawn from some distribution and provide some form of "average-case" or "Bayesian" privacy guarantee with respect to this distribution. This is especially tempting as it is common for statistical analysis to work under distributional assumptions.

In this post and in a planned follow-up post, we will discuss some pitfalls of average-case or Bayesian versions of differential privacy.  To avoid keeping you in suspense:

* The average-case assumptions in relaxations of differential privacy are qualitatively different to and much more brittle than the typical assumptions made about how the data is generated.
* Average-case relaxations do not satisfy the strong composition properties that have made differential privacy so successful.
* It is safer to use distributional assumptions in the accuracy analysis instead of the privacy analysis. That is, we can provide average-case utility and worst-case privacy. Recent work has shown that this model can capture most of the advantages of distributional assumptions.

We will show some illustrative examples for each of these points, but we will be purposefully vague as to exactly which alternative definition we are considering, as these issues arise in a wide variety of definitions.  Our hope is not to shut down discussion of these relaxations, nor to single out specific definitions as flawed. Our goal is simply to highlight some issues that, in our opinion, aren't adequately discussed when considering these relaxations.

### Assumptions about nature vs. assumptions about the adversary?

In any reasonable definition of privacy, we have to think about whom we are hiding sensitive information from.  This person---"the adversary"---could be a stranger, a close friend, a relative, a corporation we do business with, or the government, and who they are affects what information they have access to and what defenses are appropriate.   How the adversary can access the private system defines the [trust model](\trustmodels). Distributional assumptions correspond to the adversary's side information.  Our key point is:

>Assumptions incorporated into the definition of privacy are assumptions about the adversary and these are qualitatively different from assumptions about "nature," which is the process that generates the data.

For example, suppose an employer learns that two of its employees have expensive medical conditions. On its own, this information does not identify those employees and this privacy intuition could be formalized via distributional assumptions. But these distributional assumptions will break if the employer later receives some side information. For example, the other healthy employees may voluntarily disclose their medical status or the employer may find out that, before you were hired, that number was only one. (Incidentally, this is an example of a failure of composition, which we will discuss in another post.)

This example illustrates how assumptions about the adversary that might seem reasonable in a vacuum can be invalidated by context. Plus, assumptions about the adversary can be invalidated by *future* side information, and you can't retract a privacy leak once it happens the way you can a medical study. So assumptions about the adversary are much less future-proof than assumptions about nature.

### All models are wrong, but some are useful

One justification for incorporating distributional assumptions into the privacy definition is that the person using the data is often making these assumptions anyway---for example, that the data is i.i.d. Gaussian, or that two variables have some underlying linear relationship to be discovered.  So, if the assumption were false, wouldn't we already be in trouble?  Not really.

>It's important to remember the old saw "all models are wrong, but some are useful."  Some models have proven themselves useful for statistical purposes, but that does not mean they are useful as a basis for privacy.

For example, our methods may be robust to the relatively friendly ways that nature deviates from the model, but we can't trust adversaries to be as friendly.

For a toy example, suppose we model our data as coming from a normal distribution \\( N(\mu,\sigma^2) \\), but actually the data is collected at two different testing centers, one of which rounds its measurements to the nearest integer and the other of which provides two decimal places of precision.  This rounding makes the model wrong, but won't significantly affect our estimate of the mean.  However, just looking at the estimate of the mean might reveal that someone in the dataset went to the second testing center, potentially compromising that person's privacy.

A more natural setting where this issue arises is in dealing with *outliers* or other extreme examples, which we will discuss in the next section.

### Privacy for outliers

The usual worst-case definition of differential privacy provides privacy for everyone, including outliers.  Although there are lots of ways to achieve differential privacy, in order to compare definitions, it will help to restrict attention to the basic approach based on calibrating noise to sensitivity:

Suppose we have a private dataset \\( x \in \mathcal{X}^n \\) containing the data of \\( n \\) individuals, and some real-valued query \\( q : \mathcal{X}^n \to \mathbb{R} \\).  The standard way to release an estimate of \\( q(x) \\) is to compute
\\[
M(x) = q(x) + Z \cdot \sup_{\textrm{neighboring}~x',x"} |q(x') - q(x")|
\\]
where 
\\(
\sup_{\textrm{neighboring}~x',x"} |q(x') - q(x")|
\\)
is called the "worst-case sensitivity" of \\( q \\) and \\( Z \\) is some noise, commonly drawn from a Laplace or Gaussian distribution.

Unfortunately, the worst-case sensitivity may be large or even infinite for basic statistics of interest, such as the mean \\( q(x) = \frac{1}{n} \sum_{i} x_i \\) of unbounded real values.  There are a variety of differentially private algorithms for addressing this problem,[^1] but that is not what this post is about.  It's tempting to, instead, try to scale the noise to some notion of "average-case sensitivity," with the goal of satisfying some average-case version of differential privacy.  For example, suppose the data is drawn from some normal distribution \\( N(\mu,\sigma^2) \\) and the neighboring datasets \\( x\', x\'\' \\) are each \\( n \\) i.i.d. samples from this distribution, but differing on exactly one random sample.  Then the worst-case sensitivity of the mean is infinite:
\\[
\sup_{\textrm{neighboring}~x',x"} |q(x') - q(x")| = \infty,
\\]
but the average-sensitivity is proportional to \\(1/n\\):
\\[
\mathbb{E}_{\textrm{neighboring}~x', x"}(|q(x') - q(x")|) \approx \frac{\sigma}{n}.
\\]
Thus, under an average-case privacy guarantee, we can estimate the mean with very little noise.

But what happens to privacy if this assumption fails, perhaps because of outliers?  Imagine computing the average wealth of a subset of one hundred Amazon employees who test positive for COVID-19, and discovering that it's over one billion dollars.  Maybe Jeff Bezos isn't feeling well?[^2]  

Yes, this example is a little contrived, since you probably shouldn't have computed the empirical mean of such skewed data anyway. But, if this fact leaks out, you can't just go back in time and truncate the data or compute the median instead. Privacy tends to be high-stakes both because of the potential consequences of a breach and the inability to retract or correct a privacy violation after it is discovered. 

In the next section we'll see a slightly more complex example where average-case privacy / average-case sensitivity fails to protect privacy even when the distributional assumptions hold.

### Example: pairwise correlations and linear regression

Suppose our dataset \\( X \\) is a matrix \\( \\{-1,+1\\}^{n \times (d+1)} \\) where each row \\( X_i \\) corresponds to one person's data and each column corresponds to one feature.  For simplicity, let's suppose our distributional assumption is that the dataset is completely uniform---each bit is sampled independently and uniformly from \\( \\{-1,+1\\} \\).  We'll think of the first \\( d \\) columns as "features" and the last column as a "secret label."

First, consider the set of pairwise correlations between each feature and the secret label:
\\[
q_j(X) = \sum_{i = 1}^{n} X_{i,j} X_{i,d+1}
\\]
for \\( j = 1,\dots,d \\).  Note that \\( q_j(X) \\) has mean 0 and variance \\( n \\) under our distributional model of the data.  

Now, suppose we have a weight vector \\( w \in \mathbb{R}^{d} \\) and want to estimate the weighted average of correlations
\\[
q(X) = \sum_{j = 1}^{d} w_j q_j(X) = \sum_{i=1}^{n} \sum_{j=1}^{d} w_{j} X_{i,j} X_{i,d+1}
\\]
This statistic may look a little odd, but it's pretty close to computing the average squared error of the linear predictor \\( \hat X_{i,d+1} = \sum_{j=1}^{d} w_j X_{i,j} \\) given by the weight vector \\( w \\), which is a natural thing to estimate. 

The worst-case sensitivity of \\(q\\) is proportional to \\( \\\|w\\\|_1 \\).  
However, it's not too hard to show that, under our distributional model, the average-case sensitivity is much lower; it is proportional to \\( \\\| w \\\|_2 \\).  Thus, using average-case privacy may allow us to add significantly less noise.

What could go wrong here?  Well, we've implicitly assumed that the weights \\( w \\) are independent of the data \\( X \\).  That is, the person specifying the weights has no knowledge of the data itself, only its distribution.  Suppose the weights are specified by an adversary who has learned the \\( d \\) features of the first individual (although there is nothing special about considering the first individual), who sets the weights to \\(w = (X_{1,1},\dots,X_{1,d}) \\).  Another calculation shows that, in this case, _even when our model of the data is exactly correct_, the query \\( q(X) \\) has mean \\( d \cdot X_{1,d+1} \\) and standard deviation approximately \\( \sqrt{nd} \\).  Thus, if \\( d \gg n \\) we can confidently determine the secret label \\( X_{1,d+1} \\)  of the first individual from the value \\( q(X) \\).  Moreover, adding noise of standard deviation \\( \ll d \\) will not significantly affect the adversary's ability to learn the secret label.  But, earlier, we argued that average-case sensitivity is proportional to \\( \\|w \\|_2 = \sqrt{d} \\), so this form of average-case privacy fails to protect a user's data in this scenario!  Note that adding noise proportional to \\( \\|w\\|_1 = d \\) would satisfy (worst-case) differential privacy and would thwart this adversary.

>What went wrong is that the data satisfied our assumptions, but the adversary's beliefs about the data did not!

The set of reasonable distributions to consider for the adversary's beliefs may look very different from the set of reasonable distributions to consider for your analysis of the data.  You may think that it's not reasonable for the attacker to choose this weight vector \\( w \\) containing a lot of prior information about an individual, but assuming that the attacker cannot obtain or specify such a vector is very different from assuming that the data is uniform, and requires its own justification.

Before wrapping up, let's just make a couple more observations about this example:

* This attack is pretty robust. The assumption that the data is uniform with independent features can be relaxed significantly.  It's also not necessary for the adversary to exactly know all the features of the first user, all we need is for the weights to have correlation \\( \gg \sqrt{nd} \\) with the features.  For example, if the dataset is genomic data, having the data of a relative might suffice.
* This problem isn't specific to high-dimensional data with \\( d \gg n \\).  If we allow more general types of "queries", then a similar attack is possible when there are only \\( d \approx \log n \\) features.
* To make this example as crisp as possible, we allowed an adversarial data analyst to specify the weight vector \\( w \\).  You might think examples like this can't arise if the algorithm designer specifies all of the queries internally, but ensuring that requires great care (as we'll see in our upcoming post about composition).
    
### Conclusion
As we have discussed, the main issue that arises in average-case or Bayesian versions of differential privacy is that we must make strong assumptions about the adversary. A simple distributional assumption about the data, which may be entirely reasonable for statistical analysis, entails assuming a naïve adversary with essentially no side information, which is not reasonable from a privacy perspective.

In a future post, we will discuss *composition*, which is a key robustness property and really the secret to differential privacy's success.  As we'll see, average-case versions of differential privacy do not enjoy strong composition properties the way worst-case differential privacy does, which makes them much harder to deploy.

So, is there are role for distributional assumptions in differential privacy? Yes! Although we've discussed the pitfalls of making the _privacy guarantee_ contingent on distributional assumptions, none of these pitfalls apply to making the *utility guarantee* contingent on distributional assumptions, as is normally done in statistical analysis.  In recent years, this combination---worst-case privacy, average-case utility---has been fruitful, and seems to allow many of the benefits that average-case privacy definitions seek to capture.  For example, recent work has shown that worst-case differential privacy permits accurate mean and covariance estimation of unbounded data under natural modeling assumptions [**[KV18]**](https://arxiv.org/abs/1711.03908 "Vishesh Karwa, Salil Vadhan. Finite Sample Differentially Private Confidence Intervals. ITCS 2018."), [**[KLSU19]**](https://arxiv.org/abs/1805.00216 "Gautam Kamath, Jerry Li, Vikrant Singhal, Jonathan Ullman. Privately Learning High-Dimensional Distributions. COLT 2019."), [**[BS19]**](https://arxiv.org/abs/1906.02830 "Mark Bun, Thomas Steinke. Average-Case Averages: Private Algorithms for Smooth Sensitivity and Mean Estimation. NeurIPS 2019."), [**[DFMBG20]**](https://arxiv.org/abs/2001.02285 "Wenxin Du, Canyon Foot, Monica Moniot, Andrew Bray, Adam Groce. Differentially Private Confidence Intervals. 2020."), [**[KSU20]**](https://arxiv.org/abs/2002.09464 "Gautam Kamath, Vikrant Singhal, Jonathan Ullman.  Private Mean Estimation of Heavy-Tailed Distributions. COLT 2020."), [**[BDKU20]**](https://arxiv.org/abs/2006.06618 "Sourav Biswas, Yihe Dong, Gautam Kamath, Jonathan Ullman. CoinPress: Practical Private Mean and Covariance Estimation. 2020."), but this remains an active area of research.

---

[^1]: For example, there are approaches based on various paradigms like Smooth Sensitivity [**[NRS07]**](http://www.cse.psu.edu/~ads22/pubs/NRS07/NRS07-full-draft-v1.pdf "Kobbi Nissim, Sofya Raskhodnikova, Adam Smith. Smooth Sensitivity and Sampling in Private Data Analysis. STOC 2007.") [**[BS19]**](https://arxiv.org/abs/1906.02830 "Mark Bun, Thomas Steinke. Average-Case Averages: Private Algorithms for Smooth Sensitivity and Mean Estimation. NeurIPS 2019."), Propose-Test-Release [**[DL09]**](http://www.stat.cmu.edu/~jinglei/dl09.pdf "Cynthia Dwork, Jing Lei. Differential Privacy and Robust Statistics. STOC 2009."), or Truncation/Winsorization [**[S11]**](http://www.cse.psu.edu/~ads22/pubs/2011/stoc194-smith.pdf "Adam Smith. Privacy-preserving Statistical Estimation with Optimal Convergence Rates. STOC 2011.") [**[KV18]**](https://arxiv.org/abs/1711.03908 "Vishesh Karwa, Salil Vadhan. Finite Sample Differentially Private Confidence Intervals. ITCS 2018.") [**[KLSU19]**](https://arxiv.org/abs/1805.00216 "Gautam Kamath, Jerry Li, Vikrant Singhal, Jonathan Ullman. Privately Learning High-Dimensional Distributions. COLT 2019.") [**[KSU20]**](https://arxiv.org/abs/2002.09464 "Gautam Kamath, Vikrant Singhal, Jonathan Ullman.  Private Mean Estimation of Heavy-Tailed Distributions. COLT 2020.") to name a few.

[^2]: If you are confident that Jeff Bezos or other extremely high-wealth individuals are not in the sample, then you could *truncate* each sample and compute the mean of the truncated samples.  This would give worst-case privacy, and, if you are correct in your assumption, would not affect the mean.
