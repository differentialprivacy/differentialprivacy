---
layout: post
title: Trust models, and notions of privacy 
comments: true
authors: 
    - clementcanonne
categories: Definitions
timestamp: 19:00:00 -0400
---

There exist various notions of differential privacy which, while sharing a common core, differ in some key specific aspects. Broadly speaking, vary among a few main axes, such as the type of guarantee they provide, the specific similarity between data they consider, and the trust model they aim to address. This last point will be the focus of this post: *which notion of privacy is best suited to the specific scenario at hand?*

We will cover 4 of these notions. 
- (central) differential privacy (DP)
- local differential privacy (LDP)
- pan-privacy
- shuffle privacy

Typically, the world can be divided in a few categories: (i) the users, who hold the data; (ii) the "server," who runs the algorithm; and (iii) the rest of the world, which does what the rest of the world does. As the name indicates, the _trust model_ boils down to the following simple question: as a user, **who do you trust** with your sensitive data? 

In the *DP model* **\[DMN06\]**, the answer is essentially "the server, and nobody else." Users are happy to provide their data to the server, which runs the algorithm on the resulting dataset; however, the _output_ of that algorithm, which is released to the (untrusted) world, needs to be private, and not reveal sensitive information about any single user.

In the *LDP model* **\[EGS03,KLNRS08\]**, the server itself is untrusted, and the answer is "nobody." Any data communicated by the users must already be private, and even a prying server cannot learn much about any single user. Of course, this is a strictly more stringent privacy model than the central DP one, and this comes at a price: the utility one can obtain from the same amount of data is typically smaller than in the DP model.

The *pan-privacy model* **\[DNPRY10\]** introduces the notion of time. Each user contributes their data to the server sequentially, one after the other; once the server is done receiving and processing this data, the output is revealed to the world. The answer to the question then is that users trust the server _at the time they send it their data_, but maybe not in the future (and they _definitely_ don't trust the outside world). Put differently, this captures settings where a server can be compromised: at the time a  user sends their data, they trust the server; if the server is compromised at any point in the future, then the data already in the server _stays_ private (but, of course, sending any more data after the server has already been attacked is a bad idea). 

Finally, the recent *shuffle model* of privacy **\[CSUZZ19,EFMRTT19\]** is in some sense intermediate between the central and local models of DP: users do not trust the server (and, god forbid, they still don't trust the outside world!); however, they do trust some small blackbox in the middle, whose role is to randomly, well, _shuffle_ the data. That is, when all users send their data to the untrusted server, this box-in-the-middle randomly permutes all the data points, so that the server had no idea who sent which part of the data. This simple-yet-helpful trusted backbox, in turn, can be implemented using e.g., cryptographic primitives; and the goal is to try and provide stronger privacy than in the DP model, while suffering a smaller utility loss than in the stringent LDP model.

It is important to note that _there is no right or wrong model_ of privacy here, and one cannot say that any of the above notion is "better" than the others with regard to both privacy and accuracy. They all aim at modeling different scenarios, and provide incomparable guarantees: depending on your situation, pick the one that fits best.

---

**\[CSUZZ19\]** Albert Cheu, Adam D. Smith, Jonathan Ullman, David Zeber, Maxim Zhilyaev:
_Distributed Differential Privacy via Shuffling._ EUROCRYPT (1) 2019: 375-403

**\[DMN06\]** Cynthia Dwork, Frank McSherry, Kobbi Nissim, Adam D. Smith:
_Calibrating Noise to Sensitivity in Private Data Analysis._ TCC 2006: 265-284

**\[DNPRY10\]** Cynthia Dwork, Moni Naor, Toniann Pitassi, Guy N. Rothblum, Sergey Yekhanin:
_Pan-Private Streaming Algorithms._ ICS 2010: 66-80

**\[EFMRTT19\]** Úlfar Erlingsson, Vitaly Feldman, Ilya Mironov, Ananth Raghunathan, Kunal Talwar, Abhradeep Thakurta:
_Amplification by Shuffling: From Local to Central Differential Privacy via Anonymity._ SODA 2019: 2468-2479

**\[EGS03\]** Alexandre V. Evfimievski, Johannes Gehrke, Ramakrishnan Srikant:
_Limiting privacy breaches in privacy preserving data mining._ PODS 2003: 211-222

**\[KLNRS08\]** Shiva Prasad Kasiviswanathan, Homin K. Lee, Kobbi Nissim, Sofya Raskhodnikova, Adam D. Smith:
_What Can We Learn Privately?_ FOCS 2008: 531-540
