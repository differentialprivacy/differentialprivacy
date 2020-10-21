---
layout: post
title: "The Theory of Reconstruction Attacks"
comments: true
authors: 
  - alonicohen
  - sashonikolov
  - zacharyschutzman
  - jonullman
timestamp: 12:30:00 -0400
categories: [Surveys]
---

We often see people asking whether or not differential privacy might be overkill.  Why do we need strong privacy protections like differential privacy when we're only releasing approximate, aggregate statistical information about a dataset?  Is it really possible to extract information about specific users from releasing these statistics?  The answer turns out to be a resounding yes!  The textbook by Dwork and Roth [[DR14]](https://www.cis.upenn.edu/~aaroth/privacybook.html) calls this phenomenon the Fundamental Law of Information Recovery:
 
>Giving overly accurate answers to too many questions will inevitably destroy privacy.

So what exactly does this fundamental law mean precisely, and how can we prove it?  We can formalize and prove the law via *reconstruction attacks*, where an attacker can recover secret information from nearly every user in the dataset, simply by observing noisy answers to a modestly large number of (surprisingly simple) queries on the dataset. Reconstruction attacks were introduced in a seminal paper by Dinur and Nissim in 2003 [[DN03]](https://dl.acm.org/doi/10.1145/773153.773173).  Although this paper predates differential privacy by a few years, the discovery of reconstruction attacks directly led to the definition of differential privacy, and shaped a lot of the early research on the topic. We now know that differentially private algorithms can, in some cases, match the limitations on accuracy implied by reconstruction attacks. When this is the case, we have a remarkably sharp transition from a blatant privacy violation when the accuracy is high enough to enable a reconstruction attack, to the strong protection given by differential privacy at the cost of only slightly lower accuracy.

Aside from the theoretical importance of reconstruction attacks, one may wonder if they can be carried out in practice, or if the attack model is unrealistic and can be avoided with some simple workarounds?  In this series of posts, we argue that reconstruction attacks can be quite practical.  In particular, we describe successful attacks by some of this post's authors on a family of systems called *Diffix*, that attempt to prevent reconstruction without introducing as much noise as the reconstruction attacks suggest is necessary. To the best of our knowledge, these attacks represent the first successful attempt to reconstruct data from a commercial statistical-database system that is specifically designed to protect the privacy of the underlying data.  A larger and much more significant demonstration of the practical power of reconstruction attacks was carried out by the US Census Bureau in 2018, motivating the Bureau's adoption of differential privacy for data products derived from the 2020 decennial census [[GAM18]](https://queue.acm.org/detail.cfm?ref=rss&id=3295691).

This series will come in two parts: In this post, we will review the theory of reconstruction attacks, and present a model for reconstruction attacks that corresponds more directly to real attacks than the one that is typically presented.  In the second post, we will describe attacks that were launched against various iterations of the *Diffix* system. \\\(
\newcommand{\uni}{\mathcal{X}} % The universe
\newcommand{\usize}{T} % Universe size
\newcommand{\elem}{x} % Generic universe element. 
\newcommand{\pbs}{z} %Non-secret bits
\newcommand{\pbsuni}{\mathcal{Z}}
\renewcommand{\sb}{b} % Secret bit
\newcommand{\pds}{Z} %non-secret part of the data set
\newcommand{\ddim}{d} % Data dimension
\newcommand{\queries}{Q} % A set/workload of queries
\newcommand{\qmat}{\mat{Q}} % Query matrix
\newcommand{\qent}{w} % Entry of the query matrix
\newcommand{\hist}{h} % Histogram vector
\newcommand{\mech}{\mathcal{M}} % Generic Mechanism
\newcommand{\query}{q}
\newcommand{\queryfunc}{\varphi}
\newcommand{\ans}{a} % query answer
\newcommand{\qsize}{k}
\newcommand{\ds}{X}
\newcommand{\dsrow}{\elem} % same as elem above
\newcommand{\dsize}{n}
\newcommand{\priv}{\eps}
\newcommand{\privd}{\delta}
\newcommand{\acc}{\alpha}
\newcommand{\from}{:}
\newcommand{\set}[1]{\left\{#1\right\}}
\newcommand{\R}{\mathbb{R}}
\newcommand{\N}{\mathbb{N}}
\newcommand{\Z}{\mathbb{Z}}
\newcommand{\E}{\mathbb{E}}
\newcommand{\var}{\mathrm{Var}}
\newcommand{\I}{\mathbb{I}}
\newcommand{\tr}{\mathrm{Tr}}
\newcommand{\eps}{\varepsilon}
\newcommand{\pmass}{\mathbbm{1}}
\newcommand{\zo}{\\\{0,1\\\}}
\newcommand{\mat}[1]{#1} % matrix notation: for now nothing
\\\)
### A Model of Reconstruction Attacks ###

This part presents the basic theory of reconstruction attacks.  We'll introduce a model of reconstruction attacks that is a little different from what you would see if you read the papers, and then describe the main results of Dinur and Nissim.  At the end we will briefly mention some variations that have been considered in the nearly two decades since.

Let us fix a dataset model, so that we can describe the attack precisely. (These attacks are very flexible and the ideas can usually be adapted to new models, as we'll see at the end of this part.) We take the dataset to be a collection of \\\(\dsize\\\) records \\\(\ds = \\\{\elem_1,\dots,\elem_n\\\}\\\), each corresponding to the data of a single person.  The attacker's goal is to learn some piece of secret information about as many individuals as possible, so we think of each record as having the form \\\(\elem_i = (\pbs_i,\sb_i)\\\) where \\\(\pbs_i\\\) is some identifying information, and \\\(\sb_i \in \zo\\\) is some secret. We assume that the secret is binary, although this aspect of the model can be generalized. We can visualize such a dataset as a matrix \\\([\pds \mid \sb]\\\) with two blocks as follows:
\\[ \left[ \begin{array}{c|c} \pbs_1 & \sb_1 \\\\ \vdots & \vdots \\\\ \pbs_n & \sb_n \end{array} \right] \\]
For a concrete example, suppose each element in the dataset contains \\(d\\) binary attributes, and the attacker's goal is to learn the last attribute of each user.  In this case we would write each element as a pair \\\((\pbs_i, \sb_i)\\\) where \\\(\pbs_i \in \zo^{d-1}\\\) and \\\(\sb_i \in \zo\\\).

Note that this distinction between \\\(\pbs_i\\\) and \\\(\sb_i\\\) is only in the mind of the attacker, who has some prior information about the users, but is trying to learn some specific secret information.  In order to make the attack simpler to describe, we will also assume that the attacker knows \\\(\pbs_1,\dots,\pbs_\dsize\\\), which is everything about the dataset except the secret bits, although this assumption can also be relaxed to a large extent. As a shorthand, we will refer to \\\(\pbs_1, \ldots, \pbs_\dsize\\\) as the prior information, and to \\\(\sb_1, \ldots,\sb_\dsize\\\) as the secret bits.

Our goal is to understand whether asking aggregate queries defined by the prior information can allow an attacker to learn non-trivial information about the secret bits.  Perhaps the most basic type of aggregate query we can ask is a *counting query*, which is a query that asks what number of the data points satisfy a given property. The Dinur-Nissim attacks assume that the attacker can get approximate answers to a type of counting queries that ask how many data points satisfy some property defined in terms of the prior information, and also have the sensitive bit set to \\\(1\\\).  Let us use the notation \\\(\pbsuni\\\) for the set of all possible values that the prior information can take. For the purposes of the attack, each query \\\(\query\\\) will be specified by a function \\\(\queryfunc \from \pbsuni \to \zo\\\) and have the specific form
\\[
\query(\ds) = \sum_{j=1}^{\dsize} \queryfunc(\pbs_j) \cdot \sb_j.
\\]
This is a good time to make one absolutely crucial point about this model, which is that 
>all the users are treated completely symmetrically by the queries, and the attacker cannot issue a query that targets a specific user \\\(x_i\\\) by name or a specific subset of users.  The different users are distinguished only by their data.  Nonetheless, we will see how to learn information about specific users from the answers to these queries.

Returning to our example with binary attributes, consider the very natural set of queries that asks for the inner product of the secret bits with each attribute in the prior information, which is a measure of the correlation between these two attributes.  Then each query takes the form \\\(\query_i(\ds) = \sum_{j=1}^{n} \pbs_{j,i} \cdot \sb_{j}\\\).

The nice thing about this type of query is that we can express the answers to a set of queries \\\(\{\query_1,\dots,\query_\qsize\}\\\) defined by \\\(\queryfunc_1, \ldots, \queryfunc_\qsize\\\) as the following matrix-vector product \\\(\qmat_{\pds}\cdot \mat{b}\\\):
\\[ \left[ \begin{array}{c}\query_1(\ds) \\\\ \vdots  \\\\ \query_\qsize(\ds) \end{array} \right] = \left[ \begin{array}{ccc} \queryfunc_1(\pbs_1) & \dots & \queryfunc_1(\pbs_\dsize) \\\\ \vdots &  \ddots  & \vdots \\\\ \queryfunc_\qsize(\pbs_1) &   \dots & \queryfunc_k(\pbs_\dsize)  \end{array} \right] \left[ \begin{array}{c} \sb_1 \\\\ \vdots  \\\\ \sb_n  \end{array} \right]
\\]
so we can study this model using tools from linear algebra.


### An Inefficient Attack ###

Exact answers to such queries are clearly revealing, because, the attacker can use the predicates \\[ \queryfunc_i(z) = \begin{cases} 1 & \textrm{if } \pbs = \pbs\_i  \\\\ 0 &  \textrm{otherwise} \end{cases} \\] to single out a specific user and receive their bit \\\(\sb_i\\\).  It is less obvious, however, that an attacker can learn a lot about the private bits even given noisy answers to the queries.

The first Dinur-Nissim attack shows that this is indeed possible---if the attacker can ask an unbounded number of counting queries, and each query is answered with, for example, 5% error, then the attacker can reconstruct 80% of the secret bits.  This attack requires exponentially many queries to run, making it somewhat impractical, but it is a proof of concept that an attack can reconstruct a large amount of private information even from very noisy statistics. Later we will see how to scale down the attack to use fewer queries at the cost of requiring more accurate answers.

The attack itself is quite simple: 

* For simplicity, assume all the \\\(\pbs_1, \ldots, \pbs_\dsize\\\) are distinct so that each user is uniquely identified by the prior information.

* The attacker chooses the queries \\\(\query_1, \ldots, \query_\qsize\\\) so that the matrix \\\(\qmat_\pds\\\) has as its rows all of \\\(\zo^\dsize\\\). Namely, \\\(\qsize=2^\dsize\\\) and the functions \\\(\queryfunc_1, \ldots, \queryfunc_\qsize\\\) defining the queries take all possible values on \\\(\pbs_1, \ldots, \pbs_\dsize\\\).
	
* The attacker receives a vector \\\(\ans\\\) of noisy answers to the queries, where \\\( \|\query_{i}(\ds) - \ans_{i}\| < \acc \dsize \\\) for each query \\\( \query_i \\\).  In matrix notation, this means \\[ \max_{i = 1}^\qsize  |(\qmat\_\pds\cdot {\sb})\_i -\ans_i|= \\| \qmat\_\pds \cdot \sb -\ans\\|\_\infty  \leq \alpha \dsize. \\]
	Note that, for \\\(\\{0,1\\}\\\)-valued queries, the answers range from \\\(0\\\) to \\\(\dsize\\\), so answers with additive error \\\(\pm 5\%\\\) corresponds to \\\(\acc = 0.05\\\). 

* Finally, the attacker outputs any guess \\\(\hat{\sb} = (\hat{\sb}\_{1}, \ldots, \hat{\sb}\_{n})\\\) of the private bits vector that is consistent with the answers and the additive error bound \\\(\acc\\\). In other words, \\\(\hat{\sb}\\\) just needs to satisfy \\[\max_{i = 1}^\qsize |\ans_i - (\qmat_\pds\cdot \hat{\sb})_i|= \\| \qmat\_\pds \cdot \hat\sb - a \\|\_{\infty} \leq \alpha \dsize \\]
	Note that a solution always exists, since the true private bits \\\(\sb\\\) will do.  

Our claim is that any such guess \\\(\hat{b}\\\) in fact agrees with the true private bits \\\(b\\\) for all but \\\(4\acc \dsize\\\) of the users. The reason is that if \\\(\hat{\sb}\\\) disagreed with more than \\\(4\acc \dsize\\\) of the secret bits, then the answer to some query would have eliminated \\\(\hat{\sb}\\\) from contention.  To see this, fix some \\\(\hat{\sb}\in \zo^\dsize\\\), and let \\\[ S\_{01} = \\\{j: \hat{\sb}\_j = 0,  \sb\_j = 1\\\} \textrm{ and } S\_{10} = \\\{j: \hat{\sb}\_j = 1,  \sb\_j = 0\\\}\\\] 
If \\\(\hat{\sb}\\\) and \\\(\sb\\\) disagree on more than \\\(4\acc \dsize\\\) bits, then at least one of these two sets has size larger than \\\(2\acc \dsize\\\). Let us assume that this set is \\\(S_{01}\\\), and we'll deal with the other case by symmetry.  Suppose that the \\\(i\\\)-th row of \\\(\qmat_\pds\\\) is the indicator vector of \\\(S_{01}\\\), i.e., \\\[(\qmat\_\pds)\_{i,j} = 1 \iff j \in S\_{01}.\\\] We then have
\\\[
|(\qmat\_{\pds}\cdot {\sb})\_i - (\qmat\_{\pds}\cdot \hat{\sb})\_i|= |S\_{01}| > 2 \acc \dsize,
\\\]
but, at the same time, if \\\(\hat{\sb}\\\) were output by the attacker, we would have
\\\[
|(\qmat\_{\pds}\cdot {\sb})\_i - (\qmat\_{\pds}\cdot \hat{\sb})\_i| \le |\ans\_i - (\qmat\_\pds\cdot \hat{\sb})\_i| + |(\qmat\_\pds \cdot \sb)\_i - \ans\_{i}| \le 2\acc \dsize, \\\]
which is a contradiction. An important point to note is that the attacker does not need to know the set \\\(S_{10}\\\), or the corresponding \\\(i\\\)-th row of \\\(\qmat_\pds\\\) and query \\\(\query_i\\\). Since the attacker asks all possible queries determined by the prior information, we can be sure \\\(\query_i\\\) is one of these queries, and an accurate answer to it rules out this particular bad choice of \\\(\hat{\sb}\\\).  To give you something concrete to cherish, we can summarize this discussion in the following theorem.

>**Theorem [[DN03]](https://dl.acm.org/doi/10.1145/773153.773173):** There is a reconstruction attack that issues \\\(2^n\\\) queries to a dataset of \\\(n\\\) users, obtains answers with error \\\(\alpha n\\\), and reconstructs the secret bits of all but \\\(4 \alpha n\\\) users.

### An Efficient Attack ###

The exponential Dinur-Nissim attack is quite powerful, as it recovers 80% of the secret bits even from answers with 5% error, but it has the drawback that it requires asking \\\(2^\dsize\\\) queries to a dataset with \\\(\dsize\\\) users.  Note that this is inherent to some extent.  Suppose we randomly subsample 50% of the dataset and answer the queries using only this subset by rescaling appropriately.  Although this random subsampling does not guarantee any meaningful privacy, clearly no attacker can reconstruct 75% of the secret bits, since some of them are effectively deleted.  However, the guarantees of random sampling tell us that any set of \\\(\qsize\\\) queries will be answered with maximum error \\\( \acc n =  O(\sqrt{n \log \qsize})\\\), so we can answer \\\( 2^{\Omega(n)} \\\) queries with \\\(5\%\\\) error while provably preventing this sort of reconstruction.

However, Dinur and Nissim showed that if we obtain *highly accurate* answers---still noisy, but with error smaller than the sampling error---then we can reconstruct the dataset to high accuracy.  We can also make the reconstruction process computationally efficient by using linear programming to replace the exhaustive search over all \\\(2^\dsize\\\) possible vectors of secrets.  Specifically, we change the attack as follows:

* The attacker now chooses \\\(\qsize\\\) *randomly chosen* functions \\\( \varphi_i \from \pbsuni \to \\{0,1\\} \\\) for a much smaller \\\(\qsize = O(\dsize) \\\). 

* Upon receiving an answer vector \\\(\ans\\\), the attacker now searches for a *real-valued* \\\( \tilde{b} \in [0,1]^{\dsize} \\\) such that \\\( \\| \ans - \qmat_\pds \cdot \tilde{b} \\|_{\infty} \leq \acc n \\\).  Note that this vector can be found efficiently via linear programming.  The attacker then rounds each \\\( \tilde{b}\_{i} \\\) to the nearest \\\( \hat{b}\_{i} \in \\{0,1\\}\\\).

It's now much trickier to analyze this attack and show that it achieves low reconstruction error, and we won't go into details in this post.  However, the key idea is that, because the queries are chosen randomly, \\\( \qmat_\pds \\\) is a random matrix with entries in \\\( \\{0,1\\} \\\), and we can use the statistical properties of this random matrix to argue that, with high probability,
\\[
\\|\qmat\_\pds \cdot \sb - \qmat\_\pds \cdot \tilde{\sb}\\|\_\infty^2 \gtrsim |\{i: \sb\_i \neq \hat{\sb}\_i\}|.
\\]
By the way we chose \\\(\tilde{\sb}\\\), we have 
\\[
\\|\qmat\_\pds \cdot \sb - \qmat\_\pds \cdot \tilde{\sb}\\|\_\infty \le \\|\qmat\_\pds \cdot \sb - \ans\\|\_\infty + \\| \ans - \qmat\_\pds \cdot \tilde{b} \\|\_{\infty} \leq 2\acc n,
\\]
so, by combining the inequalities we get that the reconstruction error is about \\\( O(\alpha^2 n^2) \\\). Note that, in order to reconstruct 80% of the secret bits using this attack, we now need the error to be \\\( \alpha n  \ll \sqrt{n} \\\), but as long as this condition on the error is satisfied, we will have a highly accurate reconstruction.  Let's add this theorem to your goodie bags:

>**Theorem [[DN03]](https://dl.acm.org/doi/10.1145/773153.773173):** There is an efficient reconstruction attack that issues \\\(O(n)\\\) random queries to a dataset of \\\(n\\\) users, obtains answer with error \\\(\alpha n\\\), and, with high probability, reconstructs the secret bits of all but \\\( O(\alpha^2 n^2)\\\) users.

Although we modeled the queries, and thus the matrix \\\(\qmat_\pds\\\) as uniformly random, it's important to note that we really only relied on the fact that 
\\[
\\|\qmat_\pds \cdot \sb - \qmat_\pds \cdot \tilde{\sb}\\|_\infty^2 \gtrsim
|\\{i: \sb_i \neq \hat{\sb}_i\\}|,
\\]
and we can reconstruct while tolerating the same \\\(\Omega(\sqrt{n})\\\) error for any family of queries that gives rise to a matrix with this property.  Intuitively, any *random-enough* family of queries will have this property.  More specifically, the property is satisfied by any matrix with no small singular values [[DY08]](https://dl.acm.org/doi/10.1007/978-3-540-85174-5_26) or with large discrepancy [[MN12]](https://arxiv.org/abs/1203.5453).  There is a large body of work showing that many specific families of queries lead to reconstruction. For example, we can perform reconstruction using *conjunction queries* that ask for the marginal distribution of small subsets of the attributes [[KLSU10]](https://dl.acm.org/doi/abs/10.1145/1806689.1806795).  That is, queries of the form "count the number of people with blue eyes and brown hair and a birthday in August."  In fairness, there are also families of queries that do not satisfy the property, or only satisfy quantitatively weaker versions of it, such as histograms and threshold queries, and for these queries it is indeed possible to achieve differential privacy with \\\( \ll \sqrt{n} \\\) error.

### Conclusion ###

This is going to be the end of our technical discussion, but before signing off, let's mention some of the important extensions of this theorem that have been developed over the years:

* We can allow the secret information \\\(\sb\\\) to be integers or real numbers, rather than bits. The queries still return \\\(\qmat_\pds\cdot \sb\\\). The exponential attack then guarantees that, given answers with error \\\(\acc n\\\), the reconstruction \\\(\hat{\sb}\\\) satisfies \\\(\\|\hat{\sb}-\sb\\|_1 \le 4\acc n\\\). This means, for example, that the reconstructed secrets of all but \\\(4\alpha n\\\) users are within \\\(\pm 1\\\) of the true secrets. The efficient attack guarantees that \\\(\\|\hat{\sb}-\sb\\|_2^2 \le O(\acc^2 n^2)\\\), which means that the reconstructed secrets are within \\\(\pm 1\\\) for all but \\\(O(\acc^2 n^2)\\\) users.
	
* It's not crucial that *every* query be answered with error \\\( \ll \sqrt{n} \\\).  If we are willing to settle 
	for an inefficient attack, then we can reconstruct even if only 51% of the queries have small error.  If at least 75% have small error, then we can reconstruct efficiently [[DMT07]](https://dl.acm.org/doi/10.1145/1250790.1250804).
		
* The reconstruction attacks still apply to the seemingly more general data model in which the private 
	dataset \\\(\ds\\\) is a subset of some arbitrary (but public) data universe \\\(\uni\\\).  To see this, note that we can take \\\(\uni = \\{\pbs_1, \ldots, \pbs_\dsize\\}\\\), and we can interpret the secret bits \\\(\sb_i\\\) to indicate whether \\\(\pbs_i\\\) is an element of \\\(\ds\\\). Then the reconstruction attacks allow us to determine, up to some error, which elements of \\\(\uni\\\) are contained in \\\(\ds\\\). In the setting, the attack is sometimes called *membership inference*.  
	
* The fact that the efficient Dinur-Nissim reconstruction attack fails when the error is \\\( \gg \sqrt{n} \\\) 
	does not mean it's easy to achieve privacy with error of that magnitude.  As we mentioned earlier, we can achieve non-trivial error guarantees for a large number of queries simply by using a random subsample of half of the dataset, which is not a private algorithm in any reasonable sense of the word, as it can reveal everything about the chosen subset.  As this example shows,
	
	>preventing reconstruction attacks does not mean preserving privacy.
	
	In particular, there are membership-inference attacks that succeed in violating privacy even when the queries are answered with \\\( \gg \sqrt{n}\\\) error. We refer the reader to the survey [[DSSU17]](https://privacytools.seas.harvard.edu/publications/exposed-survey-attacks-private-data) for a somewhat more in-depth survey of reconstruction and membership-inference attacks.

Many types of queries give rise to the conditions under which reconstruction is possible.  Stay tuned for our next post, where we show how to generate those types of queries in practice against a family of systems known as *Diffix* that are specifically designed to thwart reconstruction.
