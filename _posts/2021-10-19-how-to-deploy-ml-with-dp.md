---
layout: post
title: "How to deploy machine learning with differential privacy?"
comments: true
authors:
  - npapernot
  - athakurta
timestamp: 10:00:00 -0700
categories: [Surveys]
---


In many applications of machine learning, such as machine learning for medical diagnosis, we would like to have machine learning algorithms that do not memorize sensitive information about the training set, such as the specific medical histories of individual patients. Differential privacy is a notion that allows quantifying the degree of privacy protection provided by an algorithm on the underlying (sensitive) data set it operates on. Through the lens of differential privacy, we can design machine learning algorithms that responsibly train models on private data.

## Why do we need private machine learning algorithms?

Machine learning algorithms work by studying a lot of data and updating their parameters to encode the relationships in that data. Ideally, we would like the parameters of these machine learning models to encode general patterns (e.g., ‘‘patients who smoke are more likely to have heart disease’’) rather than facts about specific training examples (e.g., “Jane Smith has heart disease”). Unfortunately, machine learning algorithms do not learn to ignore these specifics by default. If we want to use machine learning to solve an important task, like making a cancer diagnosis model, then when we publish that machine learning model (for example, by making an open source cancer diagnosis model for doctors all over the world to use) we might also inadvertently reveal information about the training set. A malicious attacker might be able to inspect the published model’s predictions and learn private information about Jane Smith. For instance, the adversary could mount a membership inference attack to know whether or not Jane Smith contributed her data to the model’s training set [SSS17]. The adversary could also build on membership inference attacks to extract training data by repeatedly guessing possible training points until they result in a sufficiently strong membership signal from the model’s prediction [CTW20]. In many instances, the model itself may be represented by a few of the data samples (e.g., Support Vector Machine in its dual form). 

A common misconception is that if a model generalizes (i.e., performs well on the test examples), then it preserves privacy. As mentioned earlier, this is far from being true. One of the main reasons being that generalization is an average case behavior of a model (over the distribution of data samples), whereas privacy must be provided for everyone, including outliers (which may deviate from our distributional assumptions).

Over the years, researchers have proposed various approaches towards protecting privacy in learning algorithms (k-anonymity [SS98], l-diversity [MKG07], m-invariance [XT07], t-closeness [LLV07] etc.). Unfortunately, [GKS08] all these approaches are vulnerable to what are called composition attacks, that use auxiliary information to violate the privacy protection. Famously, this strategy allowed researchers to de-anonymize part of a movie ratings dataset released to participants of the Netflix Prize when the individuals had also shared their movie ratings publicly on the Internet Movie Database (IMDb) [NS08]. If Jane Smith had assigned the same ratings to movies A, B and C in the Netflix Prize dataset and publicly on IMDb at similar times, then researchers could link data corresponding to Jane across both datasets. This would in turn give them the means to recover ratings that were included in the Netflix Prize but not on IMDb. This example shows how difficult it is to define and guarantee privacy because it is hard to estimate the scope of knowledge—about individuals—available to adversaries. While the dataset released by Netflix has since been taken down, it is difficult to ensure that all of its copies have been deleted. In recent years, data sample instance encoding based methods like InstaHide [HSL20], and NeuraCrypt [YEO21] have been demonstrated to be vulnerable to such composition attacks as well.

As a result, the research community has converged on differential privacy [DMNS06], which provides the following semantic guarantee, as opposed to ad-hoc approaches: An adversary learns almost the same information about an individual whether or not they are present in or absent from the training data set. In particular, it provides a condition on the algorithm, independent from who might be attacking it, or the specifics of the data set instantiation.  Put another way, differential privacy is a framework for evaluating the guarantees provided by a system that was designed to protect privacy. Such systems can be applied directly to “raw” data which potentially still contains sensitive information, altogether removing the need for procedures that sanitize or anonymize data and are prone to the failures described previously. That said, minimizing data collection in the first place remains a good practice to limit other forms of privacy risk.

## Designing Private Machine Learning Algorithms via Differential Privacy

Differential privacy [DMNS06] is a semantic notion of privacy that addresses a lot of the limitations of previous approaches like k-anonymity. The basic idea is to randomize part of the mechanism’s behavior to provide privacy. In our case, the mechanism considered is a learning algorithm, but the differential privacy framework can be applied to study any algorithm. 

The intuition for why we introduce randomness into the learning algorithm is that it obscures the contribution of an individual, but does not obscure important statistical patterns. Without randomness, we would be able to ask questions like: “What parameters does the learning algorithm choose when we train it on this specific dataset?” With randomness in the learning algorithm, we instead ask questions like: “What is the probability that the learning algorithm will choose parameters in this set of possible parameters, when we train it on this specific dataset?”

We use a version of differential privacy which requires (in our use case of machine learning) that the probability of learning any particular set of parameters stays roughly the same if we change a single data record in the training set. A data record can be a single training example from an individual, or the collection of all the training examples provided by an individual. The former is often referred to as example level/item level privacy, and the latter is referred to as user level differential privacy. While user level privacy provides stronger semantics, it may be harder to achieve. For a more thorough discussion about the taxonomy of these notions, see [DNPR10, JTT18, HR12, HR13]. In this document, for the ease of exposition of the technical results, we focus on the example level notion. This could mean to add a training example, remove a training example, or change the values within one training example. The intuition is that if a single patient (Jane Smith) does not affect the outcome of learning much, then that patient’s records cannot be memorized and her privacy is respected. In the rest of this post, how much a single record can affect the outcome of learning is called the sensitivity of an algorithm. 

The guarantee of differential privacy is that the adversary is not able to distinguish the answers produced by the randomized algorithm based on the data of two of the three users from the answers returned by the same algorithm based on the data of all three users. We also refer to the degree of indistinguishability as the privacy loss. Smaller privacy loss corresponds to a stronger privacy guarantee.

It is often thought that privacy is a fundamental bottleneck in obtaining good prediction accuracy/generalization for machine learning algorithms. In fact, recent research has shown that in many instances it actually helps in designing algorithms with strong generalization ability. Some of the examples where DP has resulted in designing better learning algorithms are Online linear predictions [KV05] and online PCA [DTTZ13]. Notably, [DFH15] formally showed that generalization for any DP learning algorithm comes for free. More concretely, if a DP learning algorithm has good training accuracy, it is guaranteed to have good test accuracy.
This is true because differential privacy itself acts as a very strong form of regularization. 

One might argue that the generalization guarantee which a DP algorithm can achieve may be sub-par to that of its non-private baselines. For a large class of learning tasks, one can show that asymptotically DP does not introduce any further error beyond the inherent statistical error [SSTT21]. [ACG16,BFTT19] highlights that in the presence of enough data, a DP algorithm can get arbitrarily close to the inherent statistical error, even under strong privacy parameters.

## Private Empirical Risk Minimization

Before we go into the design of specific differentially private learning algorithms, we first formalize the problem setup, and standardize some notation. Consider a training data set \\\(D={(x_1,y_1),...,(x_n,y_n)}\\\) drawn i.i.d. from some fixed (unknown) distribution \\\(\Pi\\\), with the feature vector being \\\(x_i\\\) and label/response being \\\(y_i\\\). We define the training loss at any model \\\(\\theta\\\) as \\\(L_{train} (\\theta, D) = \\frac{1}{n} \\sum_{i=1}^{n} l(\\theta; (x_i, y_i))\\\), and the corresponding test loss as \\\(L_{test} (\\theta) = E_{(x,y) \\sim \Pi} l(\\theta;(x,y)) \\\).
We will design DP algorithms to output models that approximately minimize the test loss while having access only to the training loss.


In the literature, there are a variety of approaches towards designing these DP learning algorithms [CMS11, KST12, BST14, PAE16, BTT18]. One can categorize them broadly as: i) algorithms that assume that the individual loss function \\\(l(\\theta;\cdot) \\\) is convex in the model parameter  to ensure differential privacy, ii) algorithms that are differentially private even when the loss function is non-convex in nature (e.g., deep learning models), and iii) model agnostic algorithms, that do not require any information about the representation of the model \\\(\\theta\\\), or the loss function \\\(l(\\theta;\cdot) \\\). In our current discussion, we will only focus on designing algorithms for (ii), and (iii). This is because it turns out that the best known algorithms for (ii) are already competitive to algorithms that are specific for (i) [INS19].

## Private Algorithms for Training Deep Learning Models


The first approach, due to SCS13, BST14, and ACG16, is named differentially private stochastic gradient descent (DP-SGD). It proposes to modify the model updates computed by the most common optimizer used in deep learning: stochastic gradient descent (SGD). Typically, stochastic gradient descent trains iteratively. At each iteration, a small number of training examples (a “minibatch”) are sampled from the training set. The optimizer computes the average model error on these examples, and then differentiates this average error with respect to each of the model parameters to obtain a gradient vector. Finally, the model parameters (\\\(\\theta_t\\\)) are updated by subtracting this gradient (\\\(\\nabla_t\\\)) multiplied by a small constant \\\(\\eta\\\) (the learning rate controls how quickly the optimizer updates the model’s parameters). At a high level, two modifications are made by DP-SGD to obtain differential privacy: gradients, which are computed on a per-example basis (rather than averaged over multiple examples), are first clipped to control their sensitivity, and, second, spherical Gaussian noise \\\(b_t\\\) is added to their sum to obtain the indistinguishability needed for DP. Succinctly, the update step can be written as follows: \\\(\\theta_{t+1} \\leftarrow \\theta_t - \\eta \cdot (\\nabla_t + b_t)\\\).

Let us take the example of a hospital training a model to predict whether patients will be readmitted after being released. To train the model, the hospital uses information from patient records, such as demographic variables and admission variables (e.g., age, ethnicity, insurance type, type of Intensive Care Unit admitted to) but also time-varying vitals and labs (e.g., heart rate, blood pressure, white blood cell counts) [JPS16]. The modifications made by DP-SGD ensure that if (1) Jane Smith’s individual patient record contained unusual features, e.g., her insurance provider was uncommon for people of her age or her heart rate followed an unusual pattern, the resulting signal will have a bounded impact on our model updates, and (2) the model’s final parameters would be essentially identical should Jane Smith have chosen to not contribute (i.e., opt-out) her patient record to the training set. Stronger differential privacy is achieved when one is able to introduce more noise (i.e., sample noise with larger standard deviation) and train for as few iterations as possible. 

Two main components in the above DP-SGD algorithm that distinguishes itself from traditional SGD are: i) per-example clipping and ii) Gaussian noise addition. In addition, for the analysis to hold, DP-SGD requires that sub-sampling of mini batches is uniform at random from the training data set. While this is not a requirement of DP-SGD per se, in practice many implementations of SGD do not satisfy this requirement and instead analyze different permutations of the data at each epoch of training.

While gradient clipping is common in deep learning, often used as a form of regularization, it differs from that in DP-SGD as follows: The average gradient over the minibatch is clipped, as opposed to clipping the gradient of individual examples (i.e., \\\(l(\\theta_t;(x,y)) \\\) before averaging. It is an ongoing research direction to both understand the effect of per-example clipping in DP-SGD in model training [SSTT21], and also effective ways to mitigate its impact both in terms of accuracy [PTS21], and training time [ZHS19]. 

In standard stochastic gradient descent, subsampling is usually used either as a way to speed up the training process [CAR16], or as a a form of regularization [RCR15]. In DP-SGD, the randomness in the subsampling of the minibatch is used to guarantee DP. The technical component for this sort of privacy analysis is called privacy amplification by subsampling [KLNRS08,BBG18]. Since the sampling randomness is used to guarantee DP, it is crucial that the uniformity in the sampling step is of cryptographic strength. Another, (possibly) counterintuitive feature of DP-SGD is that for best privacy/utility trade-off it is in general better to have larger batch sizes. In fact, full-batch DP-gradient descent may provide the best privacy/utility trade-offs, albeit at the expense of computational feasibility.

For a fixed DP guarantee, the magnitude of the Gaussian noise that gets added to the gradient updates in each step in DP-SGD is proportional to \\\(\sqrt{the\\ number\\ of\\ steps}\\\) the model is trained for. As a result, it is important to tune the number of training steps for best privacy/utility trade-offs.

In the [following tutorial](https://github.com/tensorflow/privacy/blob/master/tutorials/mnist_dpsgd_tutorial.py), we provide a small code snippet to train a model with DP-SGD.

## Model Agnostic Private Learning

The Sample and Aggregate framework [NRS07] is a generic method to add differential privacy to a non-private algorithm without caring about the internal workings of it, a.k.a. model agnostic. In the context of machine learning, one can state the main idea as follows: Consider a multi-class classification problem. Take the training data, and split into k disjoint subsets of equal size. Train independent models \\\(\\theta_1, \\theta_2, ..., \\theta_k \\\) on the disjoint subsets. In order to predict on an test example x, first, compute a private histogram over the set of k predictions \\\(\\theta_1(x), \\theta_2(x), ..., \\theta_k(x) \\\). Then, select and output the bin in the histogram based on the highest count, after adding a small amount of Laplace/Gaussian noise to the counts. In the context of DP learning, this particular approach was used in two different lines of work: i) PATE [PAE16], and ii) Model agnostic private learning [BTT18]. While the latter focussed on obtaining theoretical privacy/utility trade-offs for a class of learning tasks (e.g., agnostic PAC learning), the PATE approach focuses on practical deployment. Both these lines of work make one common observation. If the predictions from \\\(\\theta_1(x), \\theta_2(x), ..., \\theta_k(x) \\\) are fairly consistent, then the privacy cost in terms of DP is very small. Hence, one can run a large number of prediction queries, without violating DP constraints. In the following, we describe the PATE approach in detail.

The private aggregation of teacher ensembles (PATE) demonstrated in particular that this approach allows one to learn deep neural networks with differential privacy. It proposes to have an ensemble of models trained without privacy predict with differential privacy by having these models predict in aggregate rather than revealing their individual predictions. In PATE, we start by partitioning the private dataset into smaller subsets of data. These subsets are partitions, so there is no overlap between the data included in any pair of partitions. If Jane Smith’s record was in our private dataset, then it is included in one of the partitions only. That is, only one of the teachers has analyzed Jane Smith’s record during training. We train a ML model, called a teacher, on each of these partitions. We now have an ensemble of teacher models that were trained independently, but without any guarantees of privacy. How do we use this ensemble to make predictions that respect privacy? In PATE, we add noise while aggregating the predictions made individually by each teacher to form a single common prediction. We count the number of teachers who voted for each class, and then perturb that count by adding random noise sampled from the Laplace or Gaussian distribution. Each label predicted by the noisy aggregation mechanism comes with rigorous differential privacy guarantees that bound the privacy budget spent to label that input. Again, stronger differential privacy is achieved when we are able to introduce more noise in the aggregation and are able to answer as few queries as possible. Let us now come back to our running example. Imagine that we’d like to use the output of PATE to know if Jane likes a particular movie. The only teacher trained on the partition containing Jane Smith’s data—has now learned that a record similar to Jane’s is characteristic of an individual who likes similar movies, and as a consequence changes its prediction on a test input which is similar to Jane’s to predict the movie rating assigned by Jane. However, because the teacher only contributes a single vote to the aggregation, and that the aggregation injects noise, we won’t be able to know whether the teacher changed its prediction to the movie rating assigned by Jane because the teacher indeed trained on Jane’s data or because the noise injected during the aggregation “flipped” that teacher’s vote. The random noise added to vote counts prevents the outcome of aggregation from reflecting the votes of any individual teachers to protect privacy.

## Practically deploying differential privacy in machine learning

The two approaches we introduced have the advantage of being conceptually simple to understand. Fortunately, there also exist several open-source implementations of these approaches. For instance, DP-SGD is implemented in TensorFlow Privacy, Objax, and Opacus. This means that one is able to take an existing TensorFlow, JAX, or PyTorch pipeline for training a machine learning model and replace a non-private optimizer with DP-SGD. An example implementation of PATE is also available in TensorFlow Privacy. So what are the concrete potential obstacles to deploying machine learning with differential privacy? 

The first obstacle is the accuracy of privacy-preserving models. Datasets are often sampled from distribution with heavy tails. For instance, in a medical application, there are typically (and fortunately) fewer patients with a given medical condition than patients without that condition. This means that there are fewer training examples for patients with each medical condition to learn from. Because differential privacy prevents us from learning patterns which are not found generally across the training data, it limits our ability to learn from these patients for which we have very few examples of [SPG]. More generally, there is often a trade-off between the accuracy of a model and the strength of the differential privacy guarantee it was trained with: the smaller the privacy budget is, the larger the impact on accuracy typically is. That said, this tension is not always inevitable and there are instances where privacy and accuracy are synergical because differential privacy implies generalization [DFH15] (but not vice versa). 

The second obstacle to deploying differentially private machine learning can be the computational overhead. For instance, in DP-SGD one must compute per-example gradients rather than average gradients. This often means that optimizations implemented in machine learning frameworks to exploit matrix algebra supported by underlying hardware accelerators (e.g., GPUs) are harder to take advantage of. In another example, PATE requires that one train multiple models (the teachers) rather than a single model so this can also introduce overhead in the training procedure. Fortunately, this cost is mostly mitigated in recent implementations of private learning algorithms, in particular in Objax and Opacus. 

The third obstacle to deploying differential privacy, in machine learning but more generally in any form of data analysis, is the choice of privacy budget. The smaller the budget, the stronger the guarantee is. This means one can compare two analyses and say which one is “more private”. However, this also means that it is unclear what is “small enough” of a privacy budget. This is particularly problematic given that applications of differential privacy to machine learning often require a privacy budget that provides little theoretical guarantees in order to train a model whose accuracy is large enough to warrant a useful deployment. Thus, it may be interesting for practitioners to evaluate the privacy of their machine learning algorithm by attacking it themselves. Whereas the theoretical analysis of an algorithm’s differential privacy guarantees provides a worst-case guarantee limiting how much private information the algorithm can leak against any adversary, implementing a specific attack can be useful to know how successful a particular adversary or class of adversaries would be. This helps interpret the theoretical guarantee but may not be treated as a direct substitute for it. Open-source implementations of such attacks are increasingly available: e.g., for membership inference [here](https://github.com/tensorflow/privacy/tree/master/tensorflow_privacy/privacy/privacy_tests/membership_inference_attack) and [here](https://github.com/cchoquette/membership-inference ). 

## Conclusion

In the above, we discussed some of the algorithmic approaches towards differentially private model training which have been effective both in theoretical and practical settings. Since it is a rapidly growing field, we could not cover all the important aspects of the research space. Some prominent ones include: i) Choice of the best hyperparameters in the training of DP models.In order to ensure that the overall algorithm preserves differential privacy, one needs to ensure that the choice of hyperparameters itself preserves DP. Recent research has provided algorithms for selecting the best hyperparameters in a differentially private fashion [LT19]. ii) Choice of network architecture: it is not always true that the best known model architectures for non-private model training are indeed the best for training with differential privacy. In particular, we know that the number of model parameters may have adverse effects on the privacy/utility trade-offs [BST14]. Hence, choosing the right model architecture is important for providing a good privacy/utility trade-off [PTS21].(iii) Training in the federated/distributed setting: in the above exposition, we assumed that the training data lies in a single centralized location. However, in settings like Federated Learning (FL) [MMRHA17], the data records can be highly distributed, e.g., across various mobile devices. Running DP-SGD in the FL setting, which is required for FL to provide privacy guarantees for the training data, raises a series of challenges [KMA19] which are often facilitated by distributed private learning algorithms designed specifically for FL settings [BKMTT20, KMSTTZ21]. Some of the specific challenges in the context of FL include, limited and non-uniform availability of clients (holding individual data records) and unknown (and variable) size of the training data [BKMTT18]. On the other hand, PATE style algorithms lend themselves naturally to the distributed setting once combined with existing cryptographic primitives, as demonstrated by the CaPC protocol [CDD21]. It is an active area of research to address these above challenges.


## Acknowledgements

The authors would like to thank Thomas Steinke and Andreas Terzis for detailed feedback and edit suggestions. Parts of this blog post previously appeared on [www.cleverhans.io](www.cleverhans.io).

## Citations

[ACG16] Abadi, M., Chu, A., Goodfellow, I., McMahan, H. B., Mironov, I., Talwar, K., & Zhang, L. (2016, October). Deep learning with differential privacy. In Proceedings of the 2016 ACM SIGSAC Conference on Computer and Communications Security (pp. 308-318). ACM.

[BBG18] Balle, B., Barthe, G., & Gaboardi, M. (2018). Privacy amplification by subsampling: Tight analyses via couplings and divergences. arXiv preprint arXiv:1807.01647.

[BKMTT18] Balle, B., Kairouz P., McMahan M., Thakkar O. & Thakurta A. (2020). Privacy amplification via random check-ins. In NeurIPS.

[MMRHA17] McMahan, B., Moore, E., Ramage, D., Hampson, S., & y Arcas, B. A. (2017, April). Communication-efficient learning of deep networks from decentralized data. In Artificial intelligence and statistics (pp. 1273-1282). PMLR.

[KMSTTZ18] Kairouz P., McMahan M., Song S., Thakkar O., Thakurta A., & Xu Z. (2021). Practical and Private (Deep) Learning without Sampling or Shuffling. In ICML.

[BFTT19] Bassily, R., Feldman, V., Talwar, K., & Thakurta, A. Private Stochastic Convex Optimization with Optimal Rates. In NeurIPS 2019.

[BST14] Raef Bassily, Adam Smith, and Abhradeep Thakurta. Private empirical risk minimization: Efficient algorithms and tight error bounds. In Proceedings of the 55th Annual IEEE Symposium on Foundations of Computer Science.

[BTT18] Bassily, R., Thakurta, A. G., & Thakkar, O. D. (2018). Model-agnostic private learning. Advances in Neural Information Processing Systems.

[CDD21] Choquette-Choo, C. A., Dullerud, N., Dziedzic, A., Zhang, Y., Jha, S., Papernot, N., & Wang, X. (2021). CaPC Learning: Confidential and Private Collaborative Learning. arXiv preprint arXiv:2102.05188.

[CMS11] Chaudhuri, K., Monteleoni, C., & Sarwate, A. D. (2011). Differentially private empirical risk minimization. Journal of Machine Learning Research, 12(3).

[CTW20] Carlini, N., Tramer, F., Wallace, E., Jagielski, M., Herbert-Voss, A., Lee, K., ... & Raffel, C. (2020). Extracting training data from large language models. arXiv preprint arXiv:2012.07805.

[DFH15] Dwork, C., Feldman, V., Hardt, M., Pitassi, T., Reingold, O., & Roth, A. (2015). Generalization in adaptive data analysis and holdout reuse. arXiv preprint arXiv:1506.02629.

[DMNS06] Dwork, C., McSherry, F., Nissim, K., & Smith, A. (2006, March). Calibrating noise to sensitivity in private data analysis. In Theory of Cryptography Conference (pp. 265-284). Springer, Berlin, Heidelberg.

[DNPR10] Dwork, C., Naor, M., Pitassi, T., & Rothblum, G. N. (2010, June). Differential privacy under continual observation. In Proceedings of the forty-second ACM symposium on Theory of computing (pp. 715-724).

[DTTZ14] Dwork, C., Talwar, K., Thakurta, A., & Zhang, L. (2014, May). Analyze gauss: optimal bounds for privacy-preserving principal component analysis. In Proceedings of the forty-sixth annual ACM symposium on Theory of computing (pp. 11-20).


[HSL20] Huang, Y., Song, Z., Li, K., & Arora, S. (2020, November). Instahide: Instance-hiding schemes for private distributed learning. In International Conference on Machine Learning (pp. 4507-4518). PMLR.

[HR12] Hardt, M., & Roth, A. (2012, May). Beating randomized response on incoherent matrices. In Proceedings of the forty-fourth annual ACM symposium on Theory of computing (pp. 1255-1268).

[HR13] Hardt, M., & Roth, A. (2013, June). Beyond worst-case analysis in private singular vector computation. In Proceedings of the forty-fifth annual ACM symposium on Theory of computing (pp. 331-340).

[JPS16] Johnson, A., Pollard, T., Shen, L. et al. MIMIC-III, a freely accessible critical care database. Sci Data 3, 160035 (2016). https://doi.org/10.1038/sdata.2016.35

[JTT18] Jain, P., Thakkar, O. D., & Thakurta, A. (2018, July). Differentially private matrix completion revisited. In International Conference on Machine Learning (pp. 2215-2224). PMLR.

[INS19] Iyengar, R., Near, J. P., Song, D., Thakkar, O., Thakurta, A., & Wang, L. (2019, May). Towards practical differentially private convex optimization. In 2019 IEEE Symposium on Security and Privacy (SP) (pp. 299-316). IEEE.


[KST12] Kifer, D., Smith, A., & Thakurta, A. (2012, June). Private convex empirical risk minimization and high-dimensional regression. In Conference on Learning Theory (pp. 25-1). JMLR Workshop and Conference Proceedings.

[KMA19] Kairouz, P., McMahan, H. B., Avent, B., Bellet, A., Bennis, M., Bhagoji, A. N., ... & Zhao, S. (2019). Advances and open problems in federated learning. arXiv preprint arXiv:1912.04977.

[KV05] Kalai, Adam, and Santosh Vempala. "Efficient algorithms for online decision problems." Journal of Computer and System Sciences 71.3 (2005): 291-307.

[KLNRS08] Raskhodnikova, S., Smith, A., Lee, H. K., Nissim, K., & Kasiviswanathan, S. P. (2008). What can we learn privately. In Proceedings of the 54th Annual Symposium on Foundations of Computer Science (pp. 531-540).

[LLV07] Li, N., Li, T., & Venkatasubramanian, S. (2007, April). t-closeness: Privacy beyond k-anonymity and l-diversity. In 2007 IEEE 23rd International Conference on Data Engineering (pp. 106-115). IEEE.

[LT19] Liu, J., & Talwar, K. (2019, June). Private selection from private candidates. In Proceedings of the 51st Annual ACM SIGACT Symposium on Theory of Computing (pp. 298-309).

[M17] Mironov, I. (2017, August). Renyi differential privacy. In Computer Security Foundations Symposium (CSF), 2017 IEEE 30th (pp. 263-275). IEEE.

[MKG07] Machanavajjhala, Ashwin; Kifer, Daniel; Gehrke, Johannes; Venkitasubramaniam, Muthuramakrishnan (March 2007). "L-diversity: Privacy Beyond K-anonymity". ACM Transactions on Knowledge Discovery from Data.

[NRS07] Nissim, K., Raskhodnikova, S., & Smith, A. (2007, June). Smooth sensitivity and sampling in private data analysis. In Proceedings of the thirty-ninth annual ACM symposium on Theory of computing (pp. 75-84).

[NS08] Narayanan, A., & Shmatikov, V. (2008, May). Robust de-anonymization of large sparse datasets. In Security and Privacy, 2008. SP 2008. IEEE Symposium on (pp. 111-125). IEEE.

[PAE16] Papernot, N., Abadi, M., Erlingsson, U., Goodfellow, I., & Talwar, K. (2016). Semi-supervised knowledge transfer for deep learning from private training data. ICLR 2017.

[PTS21] Papernot, N., Thakurta, A., Song, S., Chien, S., & Erlingsson, U. (2020). Tempered sigmoid activations for deep learning with differential privacy. AAAI 2021.

[RCR15] Rudi, A., Camoriano, R., & Rosasco, L. (2015, December). Less is More: Nyström Computational Regularization. In NIPS (pp. 1657-1665).

[SCS13] Shuang Song, Kamalika Chaudhuri, and Anand D Sarwate. Stochastic gradient descent with differentially private updates. In Proceedings of the 2013 IEEE Global Conference on Signal and Information Processing, GlobalSIP ’13, pages 245–248, Washington, DC, USA, 2013. IEEE Computer Society.

[SPG] Chasing Your Long Tails: Differentially Private Prediction in Health Care Settings. Vinith Suriyakumar, Nicolas Papernot, Anna Goldenberg, Marzyeh Ghassemi. Proceedings of the 2021 ACM Conference on Fairness, Accountability, and Transparency. 

[SS98] Samarati, Pierangela; Sweeney, Latanya (1998). "Protecting privacy when disclosing information: k-anonymity and its enforcement through generalization and suppression" (PDF). Harvard Data Privacy Lab. Retrieved April 12, 2017

[SSS17] Shokri, R., Stronati, M., Song, C., & Shmatikov, V. (2017, May). Membership inference attacks against machine learning models. In Security and Privacy (SP), 2017 IEEE Symposium on (pp. 3-18). IEEE.

[SSTT21] Song, S., Thakkar, O., & Thakurta, A. (2020). Evading the Curse of Dimensionality in Unconstrained Private GLMs. In AISTATS 2021.

[XT07] Xiao X, Tao Y (2007) M-invariance: towards privacy preserving re-publication of dynamic datasets. In: SIGMOD conference, Beijing, China, pp 689–700

[YEO21] Yala, A., Esfahanizadeh, H., Oliveira, R. G. D., Duffy, K. R., Ghobadi, M., Jaakkola, T. S., ... & Medard, M. (2021). NeuraCrypt: Hiding Private Health Data via Random Neural Networks for Public Training. arXiv preprint arXiv:2106.02484.

[ZHS19] Jingzhao Zhang, Tianxing He, Suvrit Sra, and Ali Jadbabaie. Why gradient clipping accelerates training: A theoretical justification for adaptivity. In International Conference on Learning Representations, 2019.


