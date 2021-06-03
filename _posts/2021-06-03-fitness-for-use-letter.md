---
layout: post
title: Statistical Inference is Not a Privacy Violation
comments: true
authors: 
    - jonullman
categories: Outreach Definitions
timestamp: 16:00:00 -0400
---

On April 28, 2021, the US Census Bureau [released](https://www.census.gov/programs-surveys/decennial-census/decade/2020/planning-management/process/disclosure-avoidance/2020-das-updates.html) a new demonstration of its differentially private Disclosure Avoidance System (DAS) for the 2020 US Census. The public were given a month to submit feedback before the system is finalized.
This demonstration data and the feedback has generated a lot of discussion, including media coverage on [National Public Radio](https://www.npr.org/2021/05/19/993247101/for-the-u-s-census-keeping-your-data-anonymous-and-useful-is-a-tricky-balance), in [the Washington Post](https://www.washingtonpost.com/local/social-issues/2020-census-differential-privacy-ipums/2021/06/01/6c94b46e-c30d-11eb-93f5-ee9558eecf4b_story.html), and via [the Associated Press](https://apnews.com/article/business-census-2020-technology-e701e313e841674be6396321343b7e49). The DAS is also the subject of an [ongoing lawsuit](https://www.courtlistener.com/docket/59728874/state-v-united-states-department-of-commerce/).


The following is a response from experts on differential privacy and cryptography to the [working paper of Kenny et al.](https://alarm-redist.github.io/posts/2021-05-28-census-das/Harvard-DAS-Evaluation.pdf) on the impact of the 2020 U.S. Census Disclosure Avoidance System (DAS) on redistricting.

This paper makes a [common but serious mistake](https://github.com/frankmcsherry/blog/blob/master/posts/2016-06-14.md), from which the authors wrongfully conclude the Census Bureau should not modernize its privacy-protection technology.  Not only do the results not support this conclusion, but they instead show the power of the methodology, known as differential privacy, adopted by the Bureau, precisely the opposite of the authors’ erroneous conclusions. 

Trust is essential; once destroyed it can be nearly impossible to rebuild, and getting privacy wrong in this Census will have an impact on all future government surveys.  The Census Bureau has shown that their [2010 (DAS) does not survive modern privacy threats](https://desfontain.es/privacy/index.html), and in fact was roughly equivalent to publishing nearly three quarters of the responses.  The Census Bureau’s decision to modernize its Disclosure Avoidance System (DAS) for the 2020 Decennial Census to be differentially private is the correct response to decades of theoretical and empirical work on the privacy risks inherent in releasing large numbers of statistics derived from a dataset.  

The importance of the Census, and the reality that no technology competing with differential privacy exists for meeting their confidentiality obligations, makes it very important that the public and policy makers have accurate information. We imagine you will be reporting on this topic in the future.  Others have [addressed flaws](https://gerrymander.princeton.edu/DAS-evaluation-Kenny-response) in the paper regarding implications for redistricting; we want to provide you with an understanding of the privacy mistake in the study.  

To understand the flaw in the paper’s argument, consider the role of smoking in determining cancer risk.  Statistical study of medical data has taught us that smoking causes cancer.   Armed with this knowledge, if we are told that 40 year old Mr. S is a smoker, we can conclude that he has an elevated cancer risk.  The statistical inference of elevated cancer risk---made before Mr. S was born---did not violate Mr. S’s privacy. To conclude otherwise is to define science to be a privacy attack.  This is the mistake made in the paper. 

This is basically what Kenny et al. found.

The authors looked at three different predictors: one built directly from (swapped) 2010 Census data and the other two built using differential privacy applied to (swapped) 2010 Census data, and evaluated all three “on approximately 5.8 million registered voters included in the North Carolina February 2021 voter file.”  What did they find?

>“Our analysis shows that across three main racial and ethnic groups, the predictions based on the [differential privacy based] DAS data appear to be as accurate as those based on the 2010 Census data.” 

This makes perfect sense. Bayesian Improved Surname Geocoding, or BISG, is a statistical method of building a predictor inferring ethnicity (or race) from name and geography.  Here, name and geography play the role of the information as to whether or not one smokes, and the prediction of ethnicity corresponds to the cancer risk prediction.  The predictor is constructed from census data on the ethnic makeup of individual census blocks and statistical information about the popularity of individual surnames within different ethnic groups.  With such a predictor, moving across the country can change the outcome, as can changing one’s name.  But a BISG prediction is not about the individual, it is about the statistical---population-level---relationship between name, geography, and ethnicity.

The differentially private DAS enabled learning to make statistical inferences about ethnicity from name and geography, without compromising the privacy of any Census respondent, exactly as it was intended to do.  In other words, the paper establishes fitness-for-use of the DAS data for the BISG statistical method!  Because differential privacy permits learning statistical patterns without compromising the privacy of individual members of the dataset, it should not interfere with learning the predictor, which is exactly what the authors found. Returning to our “smoking causes cancer” example, the researchers found that it was just as easy to detect this statistical pattern with a modern disclosure avoidance system in place as it was with the older, less protective system. 

The authors’ conclusions --“ the DAS data may not provide universal privacy protection” -- are simply not supported by their findings.

They have confused learning that smoking causes cancer---and applying this predictor to an individual smoker---with learning medical details of individual patients in the dataset. Change the input to the predictor---replace “smoker” with “non-smoker” or move across the country, for example---and the prediction changes.  

The BISG prediction is not about the individual, it does not accompany her as she relocates from one neighborhood to another, it is a statistical relationship between name, geography, and ethnicity.  It is not a privacy compromise, it is science.

Signed:
* Mark Bun, Assistant Professor of Computer Science, Boston University
* Damien Desfontaines, Privacy Engineer, Google
* Cynthia Dwork, Professor of Computer Science, Harvard University 
* Moni Naor, Professor of Computer Science, The Weizmann Institute of Science 
* Kobbi Nissim, Professor of Computer Science, Georgetown University
* Aaron Roth, Professor of Computer and Information Science, University of Pennsylvania
* Adam Smith, Professor of Computer Science, Boston University
* Thomas Steinke, Research Scientist, Google 
* Jonathan Ullman, Assistant Professor of Computer Science, Northeastern University
* Salil Vadhan, Professor of Computer Science and Applied Mathematics, Harvard University



Please contact Cynthia Dwork for contact information for authors happy to speak about this on the record.

