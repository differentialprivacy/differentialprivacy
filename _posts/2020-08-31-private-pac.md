---
layout: post
title: "Differentially Private PAC Learning"
comments: true
authors: matthewjoseph
timestamp: 14:00:00 -0400
categories: []
---

The study of PAC learning under differential privacy runs all the way
from its introduction in 2008 **[KLNRS08]** to a best paper award at the
Symposium on Foundations of Computer Science (FOCS) this year **[BLM20]**.
In this post, we'll look at the main results in this line of work,
aiming for enough detail for a rough understanding and no more. The
ideal reader has some familiarity with PAC learning and differential
privacy, but significant knowledge of either should not be necessary.

Let us start with the underlying, fundamental question this line of work
is concerned about: *does privacy in machine learning come at a price?*
Machine learning has been, over the past decades, immensely successful;
and privacy concerns are now front and center in many critical
applications. Is there a trade-off there -- and if so, how does it
manifest itself?

To even start to address this question, one needs a formalization of
machine learning, one that allows us to reason in a clean and rigorous
way about these things. Statistical learning theory, and its
computational formalization, the so-called "Probably Approximately
Correct" framework of learning, provide such a clean model, and one
whose insights we can then hope to carry to other notions such as deep
learning.

### PAC Learning

We start with a brief overview of PAC learning absent any privacy
restrictions. Readers familiar with PAC learning can probably skip this
section while noting that

1.  Occam's razor is a baseline learner using \\(O(\log|\mathcal{H}|)\\)
    samples,

2.  VC dimension characterizes non-private PAC learning,

3.  we'll focus on realizable PAC learning, and

4.  we'll usually omit dependencies on accuracy and success probability
    parameters.

For readers needing a refresher on PAC learning, the basic element of
the "probably approximately correct" (PAC) framework [@V84] is a
*hypothesis*. Each hypothesis is a function
\\(h \colon \mathcal{X}\to \{-1,1\}\\) mapping *examples* from some space
\\(\mathcal{X}\\) to binary labels. A collection of hypotheses is a
*hypothesis class* \\(\mathcal{H}\\), e.g., thresholds (a.k.a. perceptrons),
rectangles, conjunctions, and so on. In the *realizable* setting, a
learner receives examples drawn from some unknown distribution and
labeled by an unknown \\(h^* \in \mathcal{H}\\). The learner's goal is to
with high probability ("probably") output a hypothesis that gets low
error labeling future examples from the unknown example distribution
("approximately correct"). In the *agnostic* setting, examples are not
necessarily labeled by any \\(h \in \mathcal{H}\\), and the goal is only to
output a hypothesis that approximates the best error of any hypothesis
from \\(\mathcal{H}\\). As mentioned above, we focus on the realizable
setting unless otherwise specified. In the *proper* setting, the learner
must output a hypothesis from \\(\mathcal{H}\\) itself. In the *improper*
setting, this requirement is removed.

In general, we say an algorithm \\((\alpha,\beta)\\)-PAC learns
\\(\mathcal{H}\\) with sample complexity \\(n\\) if \\(n\\) samples are sufficient
to obtain error at most \\(\alpha\\) over new examples from the distribution
with probability at least \\(1-\beta\\). For the purposes of this post, we
generally omit these dependencies on \\(\alpha\\) and \\(\beta\\), as they
typically vary little or not at all when switching between non-private
and private PAC learning.

We always have a simple baseline learner, often called "Occam's razor":
iterate over all hypotheses \\(h \in \mathcal{H}\\), check how many of the
labeled examples \\(h\\) mislabels, and output a hypothesis that mislabels
the fewest examples. With Occam's razor, \\(O(\log|\mathcal{H}|)\\) samples
suffice to PAC learn \\(\mathcal{H}\\).

At the same time, \\(|\mathcal{H}|\\) is a pretty coarse measure of
hypothesis class complexity, as it would immediately rule out learning
any infinite hypothesis class (of which there are many). Thus, as you
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
"correct" measure of how hard it is to PAC learn \\(\mathcal{H}\\). As a
motivating example that will re-appear later, note that for the
hypothesis class of 1-dimensional thresholds over \\(T\\) points,
\\(\log |\mathcal{H}| = \log T\\), while
\\(\mathsf{VCD}\left(\mathcal{H}\right)\\) is only 1.

![image](thresh)

### A Simple Private PAC Learner

It is straightforward to add a differential privacy constraint to the
PAC framework: the hypothesis output by the learner must be a
differentially private function of the labeled examples
\((x_1, y_1), \ldots, (x_n, y_n)\). That is, we must be able to change any
of the examples --- even to one with an inconsistent label --- and still
have a similar distribution over hypotheses output by the learner.

Since we haven't talked about any other PAC learner, we may as well
start with Occam's razor. A private version becomes easy if we view
Occam's razor in the right light. All Occam's razor is doing is
assigning a score to each possible output (the hypothesis' empirical
error) and outputting one with the "best" (lowest) score. This makes
Occam's razor a candidate for privatization by the *exponential
mechanism*.

Recall that the exponential mechanism uses a scoring function over
outputs to release better outputs with higher probability, subject to
the privacy constraint. More formally, the exponential mechanism
requires a scoring function \\(u(X,h)\\) mapping (database, output) pairs to
real-valued scores and then selects a given output \\(h\\) with probability
proportional to
\\(\exp\left(\tfrac{\varepsilon u(X,h)}{2\Delta(u)}\right)\\), where
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
Nissim, Raskhodnikova, and Smith [@KLNRS08] did when they introduced
differentially private PAC learning in 2008. The resulting sample
complexity bounds differ from the generic Occam's razor only by an
\\(\varepsilon\\) factor in the denominator, and
\\(O(\log|\mathcal{H}|/\varepsilon)\\) samples suffice to privately PAC
learn \\(\mathcal{H}\\).

Of course, our experience with non-private PAC learning suggests that we
shouldn't be satisfied with this \\(\log |\mathcal{H}|\\) dependence. Maybe
VC dimension characterizes private PAC learning, too?

### Characterizing Pure Private PAC Learning

The first work on this question considered private learners with two
restrictions. First, we'll restrict ourselves to learners that satisfy
*pure* privacy. Recall that pure \\((\varepsilon,0)\\(-differential privacy
forces output distributions that may only differ by a certain
\\(e^\varepsilon\\) multiplicative factor (like the exponential mechanism
above). The strictly weaker notion of approximate
\\((\varepsilon,\delta)\\)-differential privacy also allows a small additive
\\(\delta\\) factor. Second, we restrict ourselves to *proper* learners,
which may only output hypotheses from the learned class \\(\mathcal{H}\\).

With these assumptions in place, in 2010, Beimel, Kasiviswanathan, and
Nissim [@BKN10] showed that the hypothesis class \\(\mathsf{Point_d}\\) ---
\\(2^d\\) hypotheses, each associated with a unique binary vector
\\(\{-1,1\}^d\\), labeling only its associated binary vector as 1 ---
requires \\(\Omega(d)\\) samples for proper pure private PAC learning. Their
proof used the classic "packing" lower bound method, which powers many
lower bounds for pure differential privacy. The general packing method
is to first construct a large collection of databases which are all
"close enough" to each other but nonetheless all have different "good"
outputs. Once we have such a collection, we use *group privacy*: as
databases that differ in one element must have \\(\varepsilon\\)-close
output distributions under pure privacy, databases that differ in \\(k\\)
elements must have \\(k\varepsilon\\)-close output distributions. With group
privacy, since the databases are all close together, the output
distributions for any two databases in the collection cannot be that
different. This creates a tension: utility forces the algorithm to
produce different output distributions for different databases, but
privacy forces similarity. The packing argument comes down to arguing
that, unless the databases are large, privacy wins out, and there is
some database where the algorithm probably produces a bad output. In the
specific case of \\(\mathsf{Point_d}\\), the packing argument says that
databases of size \\(o(d)\\) are close enough together, and we have one
database for each hypothesis in \\(\mathsf{Point_d}\\).

Most importantly, \\(\mathsf{VCD}\left(\mathsf{Point_d}\right) = 1\\), so
the \\(\Omega(d)\\) lower bound shows us that VC dimension does not
characterize proper pure private PAC learning.

@BKN10 then contrasted this result with *improper* pure private PAC
learning. They proved that an application of the exponential mechanism
to a class \\(\mathsf{Point_d}'\\) of hypotheses derived from
\\(\mathsf{Point_d}\\) --- but *not* necessarily a subset of
\\(\mathsf{Point_d}\\) --- gives an improper pure private PAC learner with
sample complexity \\(O(\log d)\\). Moreover, they gave a still more involved
improper pure private PAC learner requiring only \\(O(1)\\) samples. This
separates proper pure private PAC learning from improper pure private
PAC learning. In contrast, the sample complexities of proper and
improper PAC learning absent privacy are the same up to logarithmic
factors in \\(\alpha\\) and \\(\beta\\).

In 2013, Beimel, Nissim, and Stemmer [@BNS13] proved a more general
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

With the notion of representation dimension in hand, @BNS13 gave the
following result:

[\[thm:rep\]]{#thm:rep label="thm:rep"} The sample complexity to pure
private PAC learn \\(\mathcal{H}\\) is
\\(\Theta(\mathsf{REPD}\left(\mathcal{H}\right))\\).

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

![image](private_pac_1){width="7cm" height="7cm"}

Note the dotted line, since we don't yet have any evidence separating
finite representation dimension and finite VC dimension.

### Separating Pure and Approximate Private PAC Learning

A pair of interconnected papers give the first results for approximate
private PAC learning. Among other things, Feldman and Xiao [@FX14]
introduced *Littlestone dimension* to private PAC learning. Littlestone
dimension \\(\mathsf{LD}\left(\mathcal{H}\right)\\) is, roughly, the maximum
number of mistakes an adversary can force an *online* PAC-learning
algorithm to make [@L88]. In general
\\(\mathsf{VCD}\left(\mathcal{H}\right) \leq \mathsf{LD}\left(\mathcal{H}\right) \leq \log|\mathcal{H}|\\).
Feldman and Xiao first connected the representation dimension of a
hypothesis class to the randomized one-way communication complexity of a
problem derived from that hypothesis class, and then connected
randomized one-way communication complexity to Littlestone dimension.
The overall result follows:

[\[thm:pure\_little\]]{#thm:pure_little label="thm:pure_little"} The
sample complexity to pure private PAC learn \\(\mathcal{H}\\) is
\\(\Omega(\mathsf{LD}\left(\mathcal{H}\right))\\).

At first glance, it's unclear what
Theorem [\[thm:pure\_little\]](#thm:pure_little){reference-type="ref"
reference="thm:pure_little"} adds over
Theorem [\[thm:rep\]](#thm:rep){reference-type="ref"
reference="thm:rep"}. After all,
Theorem [\[thm:rep\]](#thm:rep){reference-type="ref"
reference="thm:rep"} gives an equivalence, not just a lower bound. One
advantage of
Theorem [\[thm:pure\_little\]](#thm:pure_little){reference-type="ref"
reference="thm:pure_little"} is that Littlestone dimension is a known
quantity that has already been studied in its own right. For example,
this connection to Littlestone dimension gives an \\(\Omega(\log T)\\) lower
bound for learning \\(\mathsf{Thresh_T}\\), the class of thresholds over
\\(\{1, 2, \ldots, T\}\\), since an adversary can force \\(\Theta(\log T)\\(
wrong answers from an online learner binary searching over
\\(\{1,2, \ldots, T\}\\). A second advantage is that Littlestone dimension
conceptually connects private learning and online learning: we now know
that pure private PAC learning is no easier than online PAC learning.

A second paper by Beimel, Nissim, and Stemmer [@BNS13b] contrasted this
\\(\Omega(\log T)\\) lower bound with a \\(2^{O(\log^*T)}\\) upper bound for
*approximate* private PAC learning \\(\mathsf{Thresh_T}\\). Here \\(\log^*\\)
denotes the very slow-growing iterated logarithm, the number of times
the logarithm of the argument must be taken to bring it \\(\leq 1\\). (We're
not kidding about "very slow-growing" either:
\\(\log^*(\text{number of atoms in universe}) \approx 4\\).) With Feldman
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
pure and approximate private PAC learning given by @BNS13b.

Let's pause to recap what's happened so far. We know that representation
dimension characterizes pure private PAC learning [@BNS13]. However,
\\(\mathsf{Thresh_T}\\) shows that representation dimension does not
characterize approximate private PAC learning [@FX14; @BNS13b].
Littlestone dimension also gives lower bounds for pure private PAC
learning but, as shown by \\(\mathsf{Line_p}\\), these bounds are sometimes
quite loose [@FX14]. The picture now looks like this:

![image](private_pac_2){width="7cm" height="7cm"}

### Lower Bounds for Approximate Private PAC Learning

In 2015, Bun, Nissim, Stemmer, and Vadhan [@BNSV15] improved this
situation by showing that learning \\(\mathsf{Thresh_T}\\) has *proper*
approximate private sample complexity \\(\Omega(\log^*(T))\\) and
\\(O(2^{\log^*(T)})\\). The inclusion of \\(\log^*\\) means this is an extremely
mild lower bound, but it was still the first definitive evidence that
proper approximate privacy introduces a cost over non-private PAC
learning. (Importantly, quantifying the price of approximate
differential privacy is a much more daunting task than of pure DP, as
the latter is a significantly more restrictive model.)

We'll try to give some intuition for the presence of \\(\log^*\\).
Informally, the lower bound relies on an inductive construction of a
sequence of hard problems for databases of size \\(n=1, 2, \ldots\\). The
\\(n^{th}\\) hard problem relies on a distribution over databases of size
\\(n\\) whose data universe is of of size exponential in the size of the
data universe for the \\((n-1)^{th}\\) distribution. The base case is the
uniform distribution over the two singleton databases \\(\{0\}\\) and
\\(\{1\}\\), and they show how to inductively construct successive problems
such that a solution for the \\(n^{th}\\) problem implies a solution for the
\\((n-1)^{th}\\) problem. Unraveling the recursive relationship between the
problem domain sizes implies a general lower bound of roughly
\\(\log^*|X|\\) for domain \\(X\\).

Even with this result, no lower bounds were known for *improper*
approximate private PAC learning (beyond the generic non-private ones).
This situation persisted for a few years. Then, in 2018, Alon, Livni,
Malliaris, and Moran [@ALMM19] extended the \\(\Omega(\log^*T)\\) lower
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
\\(\log^*(\log(\mathsf{LD}\left(\mathcal{H}\right))) = \Omega(\log^*(\mathsf{LD}\left(\mathcal{H}\right)))\\),
combining these two results puts the following limit on private PAC
learning:

[\[thm:almm\]]{#thm:almm label="thm:almm"} The sample complexity to
approximate private PAC learn \\(\mathcal{H}\\) is
\\(\Omega(\log^*(\mathsf{LD}\left(\mathcal{H}\right)))\\).

Since Littlestone dimension characterizes online PAC learning, we now
know that private PAC learnability is tied to online PAC learnability
(albeit mildly, due to \\(\log^*\\)). This partially resolves the question
of private PAC learning: finite Littlestone dimension is necessary.
Sufficiency, however, remains an open question.

![image](private_pac_3){width="7cm" height="7cm"}

### Characterizing Approximate Private PAC Learning

Spurred by this question, several advances in private PAC learning have
appeared in the last year. First, Gonen, Hazan, and Moran strengthened
Theorem [\[thm:almm\]](#thm:almm){reference-type="ref"
reference="thm:almm"} by giving a constructive method for converting
*pure* private learners to online learners [@GHM19]. Their result
reaches back to the 2013 characterization of pure private learning in
terms of representation dimension by using the covering distribution to
generate a collection of "experts" for online learning. Again revisiting
\\(\mathsf{Thresh_T}\\), Kaplan, Ligett, Mansour, Naor, and
Stemmer [@KLMNS20] significantly reduced the \\(O(2^{\log^*(T)})\\) upper
bound of @BNSV15 to just \\(O((\log^*(T))^{1.5})\\). Finally, Alon, Beimel,
Moran, and Stemmer [@ABMS20] justified our focus on realizable private
PAC learning: they gave a transformation from a realizable approximate
private PAC learner to an agnostic one at the cost of slightly larger
privacy parameters and an increase in sample complexity that depends on
the VC dimension. This builds on an earlier transformation that only
applied to *proper* learners [@BNS15].

Finally, Bun, Livni, and Moran [@BLM20] answered the open question posed
by @ALMM19:

[\[thm:equivalence\]]{#thm:equivalence label="thm:equivalence"} The
sample complexity to approximate private PAC learn \\(\mathcal{H}\\) is
\\(2^{O({\mathsf{LD}\left(\mathcal{H}\right)})}\\).

With the result of @ALMM19, the sample complexity of private PAC
learning any \\(\mathcal{H}\\) is at least
\\(\Omega(\log^*(\mathsf{LD}\left(\mathcal{H}\right)))\\) and at most
\\(2^{O({\mathsf{LD}\left(\mathcal{H}\right)})}\\). In this sense,
Littlestone dimension --- and not VC dimension, or even representation
dimension --- characterizes private learnability.

![image](private_pac_4){width="7cm" height="7cm"}

Closing this gap is in part an open question, but note that we cannot
hope to close it completely. For the lower bound, the current
\\(\mathsf{Thresh_T}\\) upper bound implies that no general lower bound can
be stronger than
\\(\Omega((\log^*(\mathsf{LD}\left(\mathcal{H}\right)))^{1.5})\\). For the
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
paper award --- and to all who paved the way before them!
