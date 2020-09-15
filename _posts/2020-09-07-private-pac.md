---
layout: post
title: "Differentially Private PAC Learning"
comments: true
authors: matthewjoseph
timestamp: 14:00:00 -0400
categories: [Surveys]
---

The study of PAC learning under differential privacy runs all the way
from its introduction in 2008 [**[KLNRS08]**](https://arxiv.org/abs/0803.0924 "Shiva Prasad Kasiviswanathan, Homin K. Lee, Kobbi Nissim, Sofya Raskhodnikova, and Adam Smith. What Can We Learn Privately? FOCS 2008") to a best paper award at the
Symposium on Foundations of Computer Science (FOCS) this year [**[BLM20]**](https://arxiv.org/abs/2003.00563 "Mark Bun, Roi Livni, and Shay Moran. An equivalence between private classification and online prediction. FOCS 2020").  Thanks to this recent work, we now have an answer to the question posted in the very first work on this topic: "What concept classes can we learn privately?"

In this post, we'll look at the main results in this line of work,
aiming for enough detail for a rough understanding and no more. The
ideal reader has some familiarity with PAC learning and differential
privacy, but significant knowledge of either should not be necessary.

Before we get to the "what" and "how" of private PAC learning, it's worth thinking about the "why." One motivation for this line of work is that it neatly captures a fundamental question: _does privacy in machine learning come at a price?_ Machine learning is now sufficiently successful and widespread for this question to have real import. But to even start to address this question, we need a formalization of machine learning that allows us to reason about possible trade-offs in a rigorous way. Statistical learning theory, and its computational formalization as PAC learning, provide one such clean and well-studied
 model. We can therefore use PAC learning as a testbed whose insights we might carry to other less idealized forms of learning.

With this motivation in mind, the rest of this post is structured as follows. The first section covers the basics of the PAC model, and subsequent sections gradually build up a chronology of results. When possible, we give short sketches of the accompanying techniques

### PAC Learning

We start with a brief overview of PAC learning absent any privacy
restrictions. Readers familiar with PAC learning can probably skip this
section while noting that

1.  (the cardinality version of) Occam's razor is a baseline learner using \\(O(\log|\mathcal{H}|)\\)
    samples,

2.  VC dimension characterizes non-private PAC learning,

3.  we'll focus on the sample complexity of realizable PAC learning, and

4.  we'll usually omit dependencies on accuracy and success probability
    parameters, and

5. we'll usually ignore computational efficiency.

For readers needing a refresher on PAC learning, the basic element of
the "probably approximately correct" (PAC) framework [**[Val84]**](https://dl.acm.org/doi/10.1145/1968.1972 "Leslie G Valiant. A theory of the learnable. Communications of the ACM, 1984") is a
*hypothesis*. Each hypothesis is a function
\\(h \colon \mathcal{X}\to \\{-1,1\\}\\) mapping *examples* from some space
\\(\mathcal{X}\\) to binary labels. A collection of hypotheses is a
*hypothesis class* \\(\mathcal{H}\\), e.g., thresholds (a.k.a. perceptrons),
rectangles, conjunctions, and so on. In the *realizable* setting, a
learner receives examples drawn from some unknown distribution and
labeled by an unknown \\(h^* \in \mathcal{H}\\). The learner's goal is to
with high probability ("probably") output a hypothesis that mostly matches the labels of \\(h^\ast\\) on future examples from the unknown example distribution
("approximately correct"). In the *agnostic* setting, examples are not
necessarily labeled by any \\(h \in \mathcal{H}\\), and the goal is only to
output a hypothesis that approximates the best error of any hypothesis
from \\(\mathcal{H}\\). As mentioned above, we focus on the realizable
setting unless otherwise specified. In the *proper* setting, the learner
must output a hypothesis from \\(\mathcal{H}\\) itself. In the *improper*
setting, this requirement is removed.

In general, we say an algorithm \\((\alpha,\beta)\\)-PAC learns
\\(\mathcal{H}\\) with sample complexity \\(n\\) if \\(n\\) samples are sufficient
to with probability at least \\(1-\beta\\) obtain error at most \\(\alpha\\) over new examples from the distribution. For the purposes of this post, we
generally omit these dependencies on \\(\alpha\\) and \\(\beta\\), as they
typically vary little or not at all when switching between non-private
and private PAC learning.

Fortunately, we always have a simple baseline learner based
on empirical risk minimization: given a set of labeled
examples, iterate over all hypotheses \\(h \in \mathcal{H}\\), check how
many of the labeled examples each $h$ mislabels, and output
a hypothesis that mislabels the fewest examples. Using
this learner, which is sometimes called "Occam's razor,"
\\(O(\log|\mathcal{H}|)\\) samples suffice to PAC learn \\(\mathcal{H}\\).


At the same time, \\(|\mathcal{H}|\\) is a pretty coarse measure of
hypothesis class complexity, as it would immediately rule out learning
any infinite hypothesis class, of which there are many. Thus, as you
might expect, we can do better. We do so using *VC dimension*.
\\(\mathsf{VCD}\left(\mathcal{H}\right)\\) is the size of the largest
possible collection of examples such that, for every labeling of the
examples, \\(\mathcal{H}\\) contains a hypothesis with that labeling. With
VC dimension, we can essentially swap \\(\log|\mathcal{H}|\\) with
\\(\mathsf{VCD}\left(\mathcal{H}\right)\\) in the Occam's razor bound and
PAC learn with \\(O(\mathsf{VCD}\left(\mathcal{H}\right))\\) samples. In
fact, the "Fundamental Theorem of Statistical Learning" says that PAC
learnability (realizable or agnostic) is equivalent to finite VC
dimension. In this sense, \\(\mathsf{VCD}\left(\mathcal{H}\right)\\) is a
good measure of how hard it is to PAC learn \\(\mathcal{H}\\). As a
motivating example that will re-appear later, note that for the
hypothesis class of 1-dimensional thresholds over \\(T\\) points,
\\(\log |\mathcal{H}| = \log T\\), while
\\(\mathsf{VCD}\left(\mathcal{H}\right)\\) is only 1.

<img src="/images/thresh.png" width="400" alt="Example: a one-dimensional threshold function" style="margin:auto;display: block;" />
<center>An illustration of 1-dimensional thresholds. A given threshold is determined by some point \\(x^* \in [T]\\): any example \\(x \leq x^*\\) receives label \\(-1\\), and any example \\(x > x^*\\) receives label 1.</center>

### A Simple Private PAC Learner

It is straightforward to add a differential privacy constraint to the
PAC framework: the hypothesis output by the learner must be a
differentially private function of the labeled examples
\\((x_1, y_1), \ldots, (x_n, y_n)\\). That is, changing any one of the examples &mdash; even to one with an inconsistent label &mdash; must not affect the distribution over hypotheses output by the learner by too much.

Since we haven't talked about any other PAC learner, we may
as well start with the empirical risk minimization-style
Occam's razor discussed in the previous section, which
simply selects a hypothesis that minimizes empirical
error. A private version becomes easy if we view
this algorithm in the right light. All it is doing is
assigning a score to each possible output (the hypothesis'
empirical error) and outputting one with the best (lowest)
score. This makes it a good candidate for privatization
by the *exponential mechanism*.

Recall that the exponential mechanism uses a scoring function over
outputs to release better outputs with higher probability, subject to
the privacy constraint. More formally, the exponential mechanism
requires a scoring function \\(u(X,h)\\) mapping (database, output) pairs to
real-valued scores and then selects a given output \\(h\\) with probability
proportional to
\\[\exp\left(\tfrac{\varepsilon u(X,h)}{2\Delta(u)}\right),\\] where
\\(\Delta(u) := \sup_h\sup_{X\sim X'} |u(X,h)-u(X',h)|\\) is the sensitivity
of the scoring function. Thus a lower \\(\varepsilon\\) (stricter privacy
requirement) and larger \\(\Delta(u)\\) (scoring function more sensitive to
changing the database) both lead to a more uniform (more private) output
distribution.

Fortunately for our PAC learning setting, empirical error is not a very
sensitive scoring function: changing one sample only changes empirical
error by 1. We can therefore use (negative) empirical error as our
scoring function \\(u(X,h)\\), apply the exponential mechanism, and get a
"private Occam's razor". This was exactly what Kasiviswanathan, Lee,
Nissim, Raskhodnikova, and Smith [**[KLNRS08]**](https://arxiv.org/abs/0803.0924 "Shiva Prasad Kasiviswanathan, Homin K. Lee, Kobbi Nissim, Sofya Raskhodnikova, and Adam Smith. What Can We Learn Privately? FOCS 2008") did when they introduced
differentially private PAC learning in 2008. The resulting sample
complexity bounds differ from the generic Occam's razor only by an
\\(\varepsilon\\) factor in the denominator, and
\\(O(\log|\mathcal{H}|/\varepsilon)\\) samples suffice to privately PAC
learn \\(\mathcal{H}\\).

Of course, our experience with non-private PAC learning suggests that we
shouldn't be satisfied with this \\(\log |\mathcal{H}|\\) dependence. Maybe
VC dimension characterizes private PAC learning, too?

### Characterizing Pure Private PAC Learning

As it turns out, answering this question will take some time. We
start with a partial negative answer. Specifically, we'll
see a class with VC dimension 1 and (a restricted form of)
private sample complexity arbitrarily larger than 1. We'll
also cover the first in a line of characterization results
for private PAC learning.
 
We first consider learners that satisfy *pure*
privacy. Recall that pure \\((\varepsilon,0)\\)-differential
privacy forces output distributions that may only differ
by a certain \\(e^\varepsilon\\) multiplicative factor (like the
exponential mechanism above). The strictly weaker notion
of approximate \\((\varepsilon,\delta)\\)-differential privacy
also allows a small additive \\(\delta\\) factor. Second,
we restrict ourselves to *proper* learners, which
may only output hypotheses from the learned class \\(\mathcal{H}\\).

With these assumptions in place, in 2010, Beimel, Kasiviswanathan, and
Nissim [**[BKN10]**](https://dl.acm.org/doi/10.1007/978-3-642-11799-2_26 "Amos Beimel, Shiva Prasad Kasiviswanathan, and Kobbi Nissim. Bounds on the sample complexity for private learning and private data release. TCC 2010") studied
a hypothesis class called \\(\mathsf{Point_d}\\). \\(\mathsf{Point_d}\\)
consists of \\(2^d\\) hypotheses, one for each vector
in \\(\{01\}^d\\). Taking the set of examples \\(\mathcal{X}\\) to
be \\(\{01\}^d\\) as well, we define each hypothesis in
\\(\mathsf{Point_d}\\) to label only its associated vector as 1, and
the remaining \\(2^d-1\\) examples as -1. [**[BKN10]**](https://dl.acm.org/doi/10.1007/978-3-642-11799-2_26 "Amos Beimel, Shiva Prasad Kasiviswanathan, and Kobbi Nissim. Bounds on the sample complexity for private learning and private data release. TCC 2010") showed
that the hypothesis class \\(\mathsf{Point_d}\\) requires \\(\Omega(d)\\)
samples for proper pure private PAC learning. In contrast,
\\(\mathsf{VCD}\left(\mathsf{Point_d}\right) = 1\\), so this \\(\Omega(d)\\) lower bound shows
us that VC dimension does *not* characterize proper
pure private PAC learning.

This result uses the classic "packing" lower
bound method, which powers many lower bounds for pure
differential privacy. The general packing method is to
first construct a large collection of databases which
are all "close enough" to each other but nonetheless
all have different "good" outputs. Once we have such a
collection, we use *group privacy*. Group privacy is a
corollary of differential privacy that requires databases
differing in \\(k\\) elements to have \\(k\varepsilon\\)-close output
distributions. Because of group privacy, if we start
with a collection of databases that are close together,
then the output distributions for any two databases in the
collection cannot be too different. This creates a tension:
utility forces the algorithm to produce different output
distributions for different databases, but privacy forces
similarity. The packing argument comes down to arguing
that, unless the databases are large, privacy wins out,
and when privacy wins out then there is some database
where the algorithm probably produces a bad output.

For \\(\mathsf{Point_d}\\), we sketch the resulting argument as
follows. Suppose we have an \\(\varepsilon\\)-private PAC learner
that uses $m$ samples. Then we can define a collection of
different databases of size \\(m\\), one for each hypothesis
in \\(\mathsf{Point_d}\\). By group privacy, the output distribution
for our private PAC learner changes by at most \\(e^{m\varepsilon}\\)
between any two of the databases in this collection. Thus
we can pick any \\(h\in \mathsf{Point_d}\\) and know that the
probability of outputting the wrong hypothesis is at
least roughly \\(2^d\cdot e^{-m\varepsilon}\\). Since we need
this probability to be small, rearranging implies \\(m=Omega(d/\varepsilon)\\).

[**[BKN10]**](https://dl.acm.org/doi/10.1007/978-3-642-11799-2_26 "Amos Beimel, Shiva Prasad Kasiviswanathan, and Kobbi Nissim. Bounds on the sample complexity for private learning and private data release. TCC 2010") then contrasted this result with an *improper* pure private PAC learner. This learner
applies the exponential mechanism to a class \\(\mathsf{Point'_d}\\)
of hypotheses derived from \\(\mathsf{Point_d}\\) --- but *not*
necessarily a subset of \\(\mathsf{Point_d}\\) --- gives an improper
pure private PAC learner with sample complexity \\(O(\log d)\\). Since this learner is improper, it circumvents the
"one database per hypothesis" step of the packing lower
bound. Moreover, [**[BKN10]**](https://dl.acm.org/doi/10.1007/978-3-642-11799-2_26 "Amos Beimel, Shiva Prasad Kasiviswanathan, and Kobbi Nissim. Bounds on the sample complexity for private learning and private data release. TCC 2010") gave a still more involved
improper pure private PAC learner requiring only \\(O(1)\\) samples. This
separates proper pure private PAC learning from improper pure private
PAC learning. In contrast, the sample complexities of proper and
improper PAC learning absent privacy are the same up to logarithmic
factors in \\(\alpha\\) and \\(\beta\\).

In 2013, Beimel, Nissim, and Stemmer [**[BNS13]**](https://arxiv.org/abs/1402.2224 "Amos Beimel, Kobbi Nissim, and Uri Stemmer. Characterizing the sample complexity of private learners. ITCS 2013") proved a more general
result. They gave the first characterization of pure (improper) private
PAC learning by defining a new hypothesis class measure called the
*representation dimension*, \\(\mathsf{REPD}\left(\mathcal{H}\right)\\).

Roughly, the representation dimension considers the collection of all
distributions \\(\mathcal{D}\\) over sets of hypotheses (not necessarily
from \\(\mathcal{H}\\)) that "cover" \\(\mathcal{H}\\) in the following sense:
for any \\(h \in \mathcal{H}\\), with high probability a set drawn from a
covering distribution \\(\mathcal{D}\\) includes a hypothesis that mostly
produces labels that agree with \\(h\\). With this collection of
distributions defined, \\(\mathsf{REPD}\left(\mathcal{H}\right)\\) is the
minimum over all such covering distributions of the logarithm of the
size of the largest set in its support. Thus a hypothesis class that can
be covered by a distribution over small sets of hypotheses will have a
small representation dimension.

With the notion of representation dimension in hand, [**[BNS13]**](https://arxiv.org/abs/1402.2224 "Amos Beimel, Kobbi Nissim, and Uri Stemmer. Characterizing the sample complexity of private learners. ITCS 2013") gave the
following result:

> **Theorem 1** ([**[BNS13]**](https://arxiv.org/abs/1402.2224 "Amos Beimel, Kobbi Nissim, and Uri Stemmer. Characterizing the sample complexity of private learners. ITCS 2013")). The sample complexity to pure private PAC learn \\(\mathcal{H}\\) is \\(\Theta(\mathsf{REPD}\left(\mathcal{H}\right))\\).

Here is a rough sketch of the proof: to go from
\\(\mathsf{REPD}\left(\mathcal{H}\right)\\) to pure private sample
complexity, consider the distribution that gives the representation
dimension upper bound. Now construct the learner to draw a hypothesis
set from that distribution and then uses the exponential mechanism on
that set. This logic works in the other direction as well. To go from a
pure private PAC learner to a bound on
\\(\mathsf{REPD}\left(\mathcal{H}\right)\\), we can use the group privacy
property from the packing result above. Group privacy implies that we
can repeatedly call the private learner on a fixed (and by the sample
complexity bound, not too large) dummy dataset and use the resulting
distribution to cover \\(\mathcal{H}\\).

To recap, we now know that proper pure private PAC learning is strictly
harder than improper pure private PAC learning, which is characterized
by representation dimension. A picture sums it up.

<img src="/images/private_pac_1.png" width="400" alt="Landscape of Private PAC, take 1" style="margin:auto;display: block;" />

Note the dotted line, since we don't yet have any evidence separating
finite representation dimension and finite VC dimension.

### Separating Pure and Approximate Private PAC Learning

A pair of interconnected papers give the first results for approximate
private PAC learning. Among other things, Feldman and Xiao [**[FX14]**](https://arxiv.org/abs/1402.6278 "Vitaly Feldman and David Xiao. Sample complexity bounds on differentially private learning via communication complexity. COLT 2014")
introduced *Littlestone dimension* to private PAC learning. Littlestone
dimension \\(\mathsf{LD}\left(\mathcal{H}\right)\\) is, roughly, the maximum
number of mistakes an adversary can force an *online* PAC-learning
algorithm to make [**[Lit88]**](https://link.springer.com/article/10.1023/A:1022869011914 "Nick Littlestone. Learning quickly when irrelevant attributes abound: A new linear-threshold algorithm. Machine learning, 1988"). In general
\\(\mathsf{VCD}\left(\mathcal{H}\right) \leq \mathsf{LD}\left(\mathcal{H}\right) \leq \log|\mathcal{H}|\\).
Feldman and Xiao first connected the representation dimension of a
hypothesis class to the randomized one-way communication complexity of a
problem derived from that hypothesis class, and then connected
randomized one-way communication complexity to Littlestone dimension.
The overall result follows:

> **Theorem 2** ([**[FX14]**](https://arxiv.org/abs/1402.6278 "Vitaly Feldman and David Xiao. Sample complexity bounds on differentially private learning via communication complexity. COLT 2014")). The sample complexity to pure private PAC learn \\(\mathcal{H}\\) is \\(\Omega(\mathsf{LD}\left(\mathcal{H}\right))\\).

At first glance, it's unclear what
Theorem 2 adds over Theorem 1. After all, Theorem 1 gives an equivalence, not just a lower bound. One
advantage of Theorem 2 is that Littlestone dimension is a known
quantity that has already been studied in its own right. For example,
this connection to Littlestone dimension gives an \\(\Omega(\log T)\\) lower
bound for learning \\(\mathsf{Thresh_T}\\), the class of thresholds over
\\(\\{1, 2, \ldots, T\\}\\), since an adversary can force \\(\Theta(\log T)\\)
wrong answers from an online learner binary searching over
\\(\\{1,2, \ldots, T\\}\\). A second advantage is that Littlestone dimension
conceptually connects private learning and online learning: we now know
that pure private PAC learning is no easier than online PAC learning.

A second paper by Beimel, Nissim, and Stemmer [**[BNS13b]**](https://arxiv.org/abs/1407.2674 "Amos Beimel, Kobbi Nissim, and Uri Stemmer. Private learning and sanitization: Pure vs. approximate differential privacy. APPROX-RANDOM 2013") contrasted this
\\(\Omega(\log T)\\) lower bound with a \\(2^{O(\log^\ast T)}\\) upper bound for
*approximate* private PAC learning \\(\mathsf{Thresh_T}\\). Here \\(\log^\ast\\)
denotes the very slow-growing iterated logarithm, the number of times
the logarithm of the argument must be taken to bring it \\(\leq 1\\). (We're
not kidding about "very slow-growing" either:
\\(\log^\ast(\text{number of atoms in universe}) \approx 4\\).) With Feldman
and Xiao's result, this separates pure private PAC learning from
approximate private PAC learning. It also shows that representation
dimension does *not* characterize approximate private PAC learning.

At the same time, Feldman and Xiao observed that the connection between
private PAC learning and Littlestone dimension has its drawbacks as
well. Again borrowing results from communication complexity, they
observed that the hypothesis class \\(\mathsf{Line_p}\\) (which we won't
define here) has \\(\mathsf{LD}\left(\mathsf{Line_p}\right) = 2\\) but
\\(\mathsf{REPD}\left(\mathsf{Line_p}\right) = \Theta(\log(p))\\). In
contrast, they showed that an *approximate* private PAC learner can
learn \\(\mathsf{Line_p}\\) using
\\(O\left(\tfrac{\log(1/\beta)}{\alpha}\right)\\) samples. Since this
entails no dependence on \\(p\\) at all, it improves the separation between
pure and approximate private PAC learning given by [**[BNS13b]**](https://arxiv.org/abs/1407.2674 "Amos Beimel, Kobbi Nissim, and Uri Stemmer. Private learning and sanitization: Pure vs. approximate differential privacy. APPROX-RANDOM 2013").

Let's pause to recap what's happened so far. We know that representation
dimension characterizes pure private PAC learning [**[BNS13]**](https://arxiv.org/abs/1402.2224 "Amos Beimel, Kobbi Nissim, and Uri Stemmer. Characterizing the sample complexity of private learners. ITCS 2013"). However,
\\(\mathsf{Thresh_T}\\) shows that representation dimension does not
characterize approximate private PAC learning **[[FX14](https://arxiv.org/abs/1402.6278 "Vitaly Feldman and David Xiao. Sample complexity bounds on differentially private learning via communication complexity. COLT 2014")
; [BNS13b](https://arxiv.org/abs/1407.2674 "Amos Beimel, Kobbi Nissim, and Uri Stemmer. Private learning and sanitization: Pure vs. approximate differential privacy. APPROX-RANDOM 2013")]**.
Littlestone dimension also gives lower bounds for pure private PAC
learning but, as shown by \\(\mathsf{Line_p}\\), these bounds are sometimes
quite loose [**[FX14]**](https://arxiv.org/abs/1402.6278 "Vitaly Feldman and David Xiao. Sample complexity bounds on differentially private learning via communication complexity. COLT 2014"). The picture now looks like this:

<img src="/images/private_pac_2.png" width="400" alt="Landscape of Private PAC, take 2" style="margin:auto;display: block;" />

### Lower Bounds for Approximate Private PAC Learning

In 2015, Bun, Nissim, Stemmer, and Vadhan [**[BNSV15]**](https://arxiv.org/abs/1504.07553 "Mark Bun, Kobbi Nissim, Uri Stemmer, and Salil Vadhan. Differentially private release and learning of threshold functions. FOCS 2015") improved this
situation by showing that learning \\(\mathsf{Thresh_T}\\) has *proper*
approximate private sample complexity \\(\Omega(\log^\ast(T))\\) and
\\(O(2^{\log^\ast(T)})\\). The inclusion of \\(\log^\ast\\) means this is an extremely
mild lower bound, but it was still the first definitive evidence that
proper approximate privacy introduces a cost over non-private PAC
learning. (Importantly, quantifying the price of approximate
differential privacy is a much more daunting task than of pure DP, as
the latter is a significantly more restrictive model.)

We'll try to give some intuition for the presence of \\(\log^\ast\\).
Informally, the lower bound relies on an inductive construction of a
sequence of hard problems for databases of size \\(n=1, 2, \ldots\\). The
\\(n^{th}\\) hard problem relies on a distribution over databases of size
\\(n\\) whose data universe is of of size exponential in the size of the
data universe for the \\((n-1)^{th}\\) distribution. The base case is the
uniform distribution over the two singleton databases \\(\\{0\\}\\) and
\\(\\{1\\}\\), and they show how to inductively construct successive problems
such that a solution for the \\(n^{th}\\) problem implies a solution for the
\\((n-1)^{th}\\) problem. Unraveling the recursive relationship between the
problem domain sizes implies a general lower bound of roughly
\\(\log^\ast|X|\\) for domain \\(X\\).

Even with this result, no lower bounds were known for *improper*
approximate private PAC learning (beyond the generic non-private ones).
This situation persisted for a few years. Then, in 2018, Alon, Livni,
Malliaris, and Moran [**[ALMM19]**](https://arxiv.org/abs/1806.00949 "Noga Alon, Roi Livni, Maryanthe Malliaris, and Shay Moran. Private PAC learning implies finite Littlestone dimension. STOC 2019") extended the \\(\Omega(\log^\ast T)\\) lower
bound for \\(\mathsf{Thresh_T}\\) to *improper* approximate privacy. More
generally, they gave concrete evidence for the importance of thresholds
by relating a class' Littlestone dimension to its ability to "contain"
thresholds. Here, we say \\(\mathcal{H}\\) "contains" \\(m\\) thresholds if
there exist \\(m\\) (unlabeled) examples and \\(m\\) hypotheses in \\(\mathcal{H}\\)
such that the hypotheses "behave like" thresholds on the \\(m\\) examples,
i.e. \\(h_i(x_j) = 1 \Leftrightarrow i \leq j\\). With this language, they
imported a result from model theory to show that any hypothesis class
\\(\mathcal{H}\\) contains \\(\log(\mathsf{LD}\left(\mathcal{H}\right))\\)
thresholds. This implies that learning \\(\mathcal{H}\\) is at least as hard
as learning \\(\mathsf{Thresh_T}\\) with
\\(T = \log(\mathsf{LD}\left(\mathcal{H}\right))\\). Since
\\(\log^\ast(\log(\mathsf{LD}\left(\mathcal{H}\right))) = \Omega(\log^\ast(\mathsf{LD}\left(\mathcal{H}\right)))\\),
combining these two results puts the following limit on private PAC
learning:

> **Theorem 3** ([**[ALMM19]**](https://arxiv.org/abs/1806.00949 "Noga Alon, Roi Livni, Maryanthe Malliaris, and Shay Moran. Private PAC learning implies finite Littlestone dimension. STOC 2019")). The sample complexity to approximate private PAC learn \\(\mathcal{H}\\) is \\(\Omega(\log^\ast(\mathsf{LD}\left(\mathcal{H}\right)))\\).

Since Littlestone dimension characterizes online PAC learning, we now
know that private PAC learnability is tied to online PAC learnability
(albeit mildly, due to \\(\log^\ast\\)). This partially resolves the question
of private PAC learning: finite Littlestone dimension is necessary.
Sufficiency, however, remains an open question.

<img src="/images/private_pac_3.png" width="400" alt="Landscape of Private PAC, take 3" style="margin:auto;display: block;" />

### Characterizing Approximate Private PAC Learning

Spurred by this question, several advances in private PAC learning have
appeared in the last year. First, Gonen, Hazan, and Moran strengthened
Theorem 3 by giving a constructive method for converting
*pure* private learners to online learners [**[GHM19]**](https://arxiv.org/abs/1905.11311 "Alon Gonen, Elad Hazan, and Shay Moran. Private learning implies online learning: An efficient reduction. NeurIPS 2019"). Their result
reaches back to the 2013 characterization of pure private learning in
terms of representation dimension by using the covering distribution to
generate a collection of "experts" for online learning. Again revisiting
\\(\mathsf{Thresh_T}\\), Kaplan, Ligett, Mansour, Naor, and
Stemmer [**[KLMNS20]**](https://arxiv.org/abs/1911.10137 "Haim Kaplan, Katrina Ligett, Yishay Mansour, Moni Naor, and Uri Stemmer. Privately learning thresholds: Closing the exponential gap. COLT 2020") significantly reduced the \\(O(2^{\log^\ast(T)})\\) upper
bound of [**[BNSV15]**](https://arxiv.org/abs/1504.07553 "Mark Bun, Kobbi Nissim, Uri Stemmer, and Salil Vadhan. Differentially private release and learning of threshold functions. FOCS 2015") to just \\(O((\log^\ast(T))^{1.5})\\). Finally, Alon, Beimel,
Moran, and Stemmer [**[ABMS20]**](https://arxiv.org/abs/2003.04509 "Noga Alon, Amos Beimel, Shay Moran, and Uri Stemmer. Closure properties for private classification and online prediction. COLT 2020") justified our focus on realizable private
PAC learning: they gave a transformation from a realizable approximate
private PAC learner to an agnostic one at the cost of slightly larger
privacy parameters and an increase in sample complexity that depends on
the VC dimension. This builds on an earlier transformation that only
applied to *proper* learners [**[BNS15]**](https://arxiv.org/abs/1407.2662 "Amos Beimel, Kobbi Nissim, and Uri Stemmer. Learning privately with labeled and unlabeled examples. SODA 2015").

Finally, Bun, Livni, and Moran [**[BLM20]**](https://arxiv.org/abs/2003.00563 "Mark Bun, Roi Livni, and Shay Moran. An equivalence between private classification and online prediction. FOCS 2020") answered the open question posed
by [**[ALMM19]**](https://arxiv.org/abs/1806.00949 "Noga Alon, Roi Livni, Maryanthe Malliaris, and Shay Moran. Private PAC learning implies finite Littlestone dimension. STOC 2019"):

> **Theorem 4** ([**[BLM20]**](https://arxiv.org/abs/2003.00563 "Mark Bun, Roi Livni, and Shay Moran. An equivalence between private classification and online prediction. FOCS 2020")). The sample complexity to approximate private PAC learn \\(\mathcal{H}\\) is \\(2^{O({\mathsf{LD}\left(\mathcal{H}\right)})}\\).

With the result of [**[ALMM19]**](https://arxiv.org/abs/1806.00949 "Noga Alon, Roi Livni, Maryanthe Malliaris, and Shay Moran. Private PAC learning implies finite Littlestone dimension. STOC 2019"), the sample complexity of private PAC
learning any \\(\mathcal{H}\\) is at least
\\(\Omega(\log^\ast(\mathsf{LD}\left(\mathcal{H}\right)))\\) and at most
\\(2^{O({\mathsf{LD}\left(\mathcal{H}\right)})}\\). In this sense,
Littlestone dimension &mdash; and not VC dimension, or even representation
dimension &mdash; characterizes private learnability.

<img src="/images/private_pac_4.png" width="400" alt="Landscape of Private PAC, final take" style="margin:auto;display: block;" />

Closing this gap is in part an open question, but note that we cannot
hope to close it completely. For the lower bound, the current
\\(\mathsf{Thresh_T}\\) upper bound implies that no general lower bound can
be stronger than
\\(\Omega((\log^\ast(\mathsf{LD}\left(\mathcal{H}\right)))^{1.5})\\). For the
upper bound, there exist hypotheses classes \\(\mathcal{H}\\) with
\\(\mathsf{VCD}\left(\mathcal{H}\right) = \mathsf{LD}\left(\mathcal{H}\right)\\)
(e.g.,
\\(\mathsf{VCD}\left(\mathsf{Point_d}\right) = \mathsf{LD}\left(\mathsf{Point_d}\right) = 1\\)),
so since non-private PAC learning requires
\\(\Omega(\mathsf{VCD}\left(\mathcal{H}\right))\\) samples, the best
possible private PAC learning upper bound is
\\(\mathsf{LD}\left(\mathcal{H}\right)\\). Nevertheless, certifying either
bound remains open.

### Conclusion

This concludes this blog post, and with it our discussion of this
fundamental question: the *price of privacy* in machine learning. As
discussed in the introduction, we focused on the clean, yet widely
successful and influential model of *PAC learning*. Having characterized
how privacy enters the picture in PAC learning, we can hopefully convey
this understanding to other models of learning, and now approach these
questions from a rigorous and grounded point of view.

Congratulations to Mark Bun, Roi Livni, and Shay Moran on their best
paper award &mdash; and to all who paved the way before them!
