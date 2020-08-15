---
layout: post
title: "The Pitfalls of Average-Case Differential Privacy: Composition"
comments: true
authors: 
    - thomassteinke 
    - jonullman
categories: [Definitions]
timestamp: 10:00:00 -0400
---

We're back!  In our last [post](\average-case-dp) we discussed some of the subtle pitfalls of formulating the assumptions underlying average-case relaxations of differential privacy.  This time we're going to look at the composition property of differential privacy&mdash;that is, running two independent differentially private algorithms on your data and combining their outputs is still differentially private. This is a key property of differential privacy and is actually closely related to the worst-case nature of differential privacy.

Composition is really the key property that has made differential privacy successful. Data analysis doesn't happen in a vacuum, and the greatest threat to privacy comes from combining multiple pieces of information. These pieces of information can come from a single source that releases detailed statistics, or they could come from separate sources. So it's critical to understand how the composition of multiple pieces of information can affect privacy.

In this post we'll give some examples to illustrate why we need composition, and why composition is challenging for average-case relaxations of differential privacy.  Composition is what allows you to design sophisticated differentially private algorithms out of simple building blocks, and it's what allows one organization to release differentially private statistics without having to understand the entire ecosystem of related information that has been released.  As we'll see, the challenges of composing average-case privacy guarantees are also very closely related to the subtleties that arise in thinking about the adversary's beliefs.

### Differencing Attacks

Let's start with a simple example of composition that was alluded to in our last post. 

You've just started a new job and signed up for the health insurance provided by your employer. Thus, your employer is able to obtain aggregated data from the insurance provider. In particular, your employer can ask "How many of our employees have submitted claims for condition X?"  However, your employer should not be able to find out whether or not *you* have condition X. 
For concreteness, condition X could be a mental health condition, drug addiction, being pregnant, terminal cancer, or an expensive chronic illness. Each of these could result in some kind of employment discrimination. 

The employer may find out that 417 employees have condition X.  That's OK, on its own, this number reveals very little about whether or not *you* have condition X, as long as your employer is uncertainty about how many employees *other than you* have ccondition X.  We can formalize this as some kind of average-case or Bayesian privacy guarantee. Thus the health-insurance company is comfortable releasing this number exactly.  But, yesterday, before you started your job, it also seemed reasonable to allow your employer to ask the exact same question, and yesterday the answer was 416. Thus your employer concludes that you have condition X. 

In this example, we see how two pieces of information&mdash;the count before you started and the count after you started&mdash;each of which seems innocuous on its own can be combined to reveal private information. This is a simple example of a *differencing attack* and composition is important in part because it prevents these attacks.

This example involves only two pieces of information. However, an attack could combine many pieces of information. For example, the counts could be broken down by sex, race/ethnicity, age, location, and tobacco use.[^1] Additional data may also be obtained from other sources, such as public records, social media, voluntary disclosures, healthcare providers, financial records, employment records, or even illicit sources. The possibilities for attacks grow rapidly as more information is made available. And an employer is only one example of a potential privacy adversary.

The point of this example is that it's easy to argue that one piece of information is harmless to privacy by making plausible-looking assumptions about the adversary. But this intuition rapidly breaks down once you consider the bigger picture where there are many pieces of information that can complete the puzzle. That's why we need rigorous methods for understanding privacy and its composition.

### Quantifying Composition

How does differential privacy prevent a differencing attack like the one we just discussed? The simplest way is to add a little bit of random noise to each answer. On the first day, instead of releasing the exact count 416, we could release a noisy count, say, 420. Then on the second day, instead of releasing the true count 417, we release another noisy count, say, 415. More precisely, it is common to add noise to counts drawn from a Laplace or Gaussian distribution.  These figures are still close enough to the true values to be useful, but the difference of 1 is now obscured by the noise, so your privacy is protected.

Since the noise is unknown to *any* potential adversary, it introduces uncertainty that protects the contribution that an individual makes to the count. Taking the difference of two independent noisy counts results in something that is still noisy. However, we must be careful to quantify this privacy guarantee, particularly when it comes to composition. 

So, how much noise do we need to add? Let's go back to the example and suppose the insurance company provides noisy answers where the noise has mean zero and some fixed variance. Your employer could simply ask the same question again and again and each time receive a different noisy answer. Averaging these noisy answers will effectively reduce the variance of the added noise and allow the true answer to be discerned. That leaves us back where we started.

The moral of this revised example is that the scale of the noise must increase if we allow more access to the data, so more questions means more noise in each answer.[^2] 
Asking the same question again and again may seem silly. There are easy ways to defend against this and other attacks. But, unfortunately, the underlying phenomenon cannot be circumvented. One of the seminal works that led to differential privacy [**[DN03]**](https://dl.acm.org/doi/10.1145/773153.773173 "Irit Dinur, Kobbi Nissim. Revealing Information While Preserving Privacy. PODS 2003") showed that there is an inherent tradeoff between the number of questions to be answered and the amount of noise that needs to be added to protect privacy. The general attack is simple: Instead of asking the same query again and again, the attacker asks "random" queries. This attack only requires basic linear algebra and has been demonstrated on real systems [**[CN18]**](https://arxiv.org/abs/1810.05692 "Aloni Cohen, Kobbi Nissim. Linear Programming Reconstruction in Practice.").

### Adaptive Composition

There are actually two kinds of composition to consider. There is non-adaptive composition where the questions to be asked are pre-specified and thus independent of the data, and there is adaptive composition, where the questions may themselves depend on the results of prior access to the data. So far, we have really only considered non-adaptive composition. 

Any interactive system must take adaptive composition into account.  A natural algorithm which asks adaptive questions is stochastic gradient descent for minimizing a function that is determined by private data (e.g., for regression on medical records). At each step, the algorithm asks for a gradient of the function, which depends on the private data, at the current point. Then the point is updated according to the reported gradient and the process repeats. Since the updated point depends on the previous answer, the next gradient computation is adaptive.

The good news is that differential privacy can handle adaptive composition just fine.  However, to handle adaptive composition, it's really important that you have a worst-case privacy definition like differential privacy. Average-case variants of differential privacy cannot handle adaptive composition. Intuitively, the problem is that whatever distributional assumption you might make about the data or query a priori is unlikely to hold when you condition on past interactions with the same data or related data.

Here's a technical example that shows the difficulty of adaptive composition. Our data \\(x \in \\{-1,+1\\}^n\\) is a vector of \\(n\\) bits, one bit per person.  Because we're considering average-case differential privacy, we'll model this vector as uniformly random.  Consider the following slightly odd algorithm \\(M_2(x,v)\\)---it takes a vector \\(v \in \\{-1,+1\\}^n\\) from the user, and if the correlation \\(\langle x, v \rangle / n\\) between \\(v\\) and \\(x\\) is smaller than \\(\varepsilon/2\\), the query returns \\(0\\), but if the correlation between \\(v\\) and \\(x\\) is larger than \\(\varepsilon/2\\), the query returns the dataset \\(x\\).  This algorithm may seem a little odd, but in isolation it satisfies an average-case version of differential privacy, because if \\(n\\) is large enough and $x$ is uniformly random, then it's very unlikely that the user can guess a vector $v$ that causes this algorithm to output anything other than \\(0\\).  

Now, consider another, more familiar differentially private algorithm called randomized response [**[War65]**](https://www.jstor.org/stable/2283137?seq=1 "Stanley Warner. Randomized Response: A Survey Technique for Eliminating Evasive Answer Bias. Journal of the American Statistical Association 1965.").  For those not familiar, this algorithm \\(M_1(x)\\) outputs a vector \\(y \in \{0,1\}^n\\), where \\(y_i\\) is slightly more likely to be \\(x_i\\) than \\(-x_i\\).  Specifically, we set \\(y_i = x_i\\) with probability \\((1+\varepsilon)/2\\) and \\(y_i = - x_i\\) otherwise. This satisfies \\(\log(\frac{1+\varepsilon}{1-\varepsilon})\\)-differential privacy or, roughly, \\(2\varepsilon\\)-differential privacy. The upshot is that we obtain a vector \\(y\\) where the correlation between \\(x\\) and \\(y\\) is about \\(\varepsilon\\), i.e. \\(\langle x , y \rangle / n \approx \varepsilon\\)

OK, so \\(M_1\\) and \\(M_2\\) both satisfy strong average-case versions of differential privacy when the data is uniform, but what about their composition?  Well, the bad news is that running \\(y = M_1(x)\\) followed by \\(M_2(x,y)\\) is going to return the dataset \\(x\\) with probability approaching 100%!  That's because \\(y\\) was designed precisely to be a vector with correlation about \\(\varepsilon\\) with \\(x\\), and this is exactly the key that gets \\(M_2\\) to unlock the dataset.  What went wrong here is that even if \\(x\\) really is uniformly random, it's very far from it when conditioned on the output \\(M_1(x)\\). To analyze \\(M_2(x,y)\\) we must look at the distribution of \\(x\\) conditioned on \\(y\\). This distribution is going to be messy and may as well be a worst-case distribution, which means we must leave the realm of average-case privacy.

### Conclusion
Composition is what allows us to design sophisticated differentially private algorithms out of simple building blocks -- as long as each part is differentially private, the overall system is too. And it's what allows one organization to release differentially private statistics without having to understand the entire ecosystem of related information that has been or will be released. In short, it is what allows differential privacy to deal with the complexities of the real world.

It is unlikely that differential privacy would have taken off as a field of research without this composition property. Any proposal for an alternative approach to privacy-preserving data analysis should first be evaluated in terms of how it handles composition.

This post only scratches the surface. We will leave you with some pointers to further reading on the topic:

* [**[DSSU17]**](https://privacytools.seas.harvard.edu/publications/exposed-survey-attacks-private-data "Cynthia Dwork, Adam Smith, Thomas Steinke, Jonathan Ullman. Exposed! A Survey of Attacks on Private Data. Annual Review of Statistics and its Applications 2017.") This is a survey of attacks which explains quantitatively the relationship between noise, number of questions, and privacy risks.
* [**[KOV15]**](https://arxiv.org/abs/1311.0776 "Peter Kairouz, Sewoong Oh, Pramod Viswanath. The Composition Theorem for Differential Privacy. ICML 2015.") [**[MV15]**](https://arxiv.org/abs/1507.03113 "Jack Murtagh, Salil Vadhan. The Complexity of Computing the Optimal Composition of Differential Privacy. TCC 2016") [**[DR16]**](https://arxiv.org/abs/1603.01887 "Cynthia Dwork, Guy Rothbum. Concentrated Differential Privacy.") [**[BS16]**](https://arxiv.org/abs/1605.02065 "Mark Bun, Thomas Steinke. Concentrated Differential Privacy: Simplifications, Extensions, and Lower Bounds. TCC 2016.") [**[M17]**](https://arxiv.org/abs/1702.07476 "Ilya Mironov. Renyi Differential Privacy. CSF 2017.") [**[DRS19]**](https://arxiv.org/abs/1905.02383 "Jinshuo Dong, Aaron Roth, Weijie Su. Gaussian Differential Privacy. Journal of the Royal Statistical Society: Series B.") On the positive side, these papers analyze how differential privacy composes, yielding sharp quantitative bounds.

---

[^1]: A good rule of thumb is that, if the number of released values is much larger than the number of people, then a privacy attack is probably possible. This is analogous to the rule from algebra that, if the number of constraints (released values) is greater than the number of unknown variables (people's data), then the unknowns can be worked out.

[^2]: Exactly quantifying how much noise is needed as the number of questions grows leads to the concept of a "privacy budget." That is, we must precisely quantify how differential privacy degrades under composition. This is a very deep topic and is something we hope to discuss in future posts.
