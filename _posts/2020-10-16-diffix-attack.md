---
layout: post
title: "Reconstruction Attacks in Practice"
comments: true
authors:
  - alonicohen
  - sashonikolov
  - zacharyschutzman
  - jonullman
timestamp: 00:00:00 -0400
categories: [Surveys]
---

This is the second of two posts describing the theory and practice of linear programming reconstruction attacks.  To read the first post, which covers the theoretical basis of such attacks, [[click here]](https://dp.org).

----

In the last post, we discussed how an attacker can use noisy answers to questions about a database to reconstruct private information in the database. The reconstruction attack framework was:

1. The attacker submits sufficiently random queries that link prior information (which the attacker already knows) to private data (which the attacker wants to learn).
2. The attacker receives noisy answers to these queries and writes them down as constraints for a linear program to solve for the private bits.
3. The attacker solves the linear program and rounds the result to recover most of the bits.

Our last post discussed some of this attack's nice theoretical guarantees, and this post matches that with real-world performance. More specifically, we'll cover two successful applications of this attack against a piece of anonymizing SQL software called Diffix.

### What is Diffix? ###
Diffix is a system designed by the startup Aircloak for answering statistical queries over a private database. It is described by its creators as an "anonymizing SQL interface [that] sits in front of your data and enables you to conduct ad hoc analytics --- fully privacy preserving and GDPR-compliant."[^1]  Aircloak's approach is to develop targeted defenses for known vulnerabilities, but to otherwise privilege utility over protecting against unknown vulnerabilities.  They combine this approach with a serious effort to actually find vulnerabilities in Diffix through periodic bug bounties that offer monetary prizes for participants who mount successful attacks.  While this post is critical of the design of Diffix itself, we commend Aircloak for their genuine openness to scrutiny. Indeed, the attacks described in this post were carried out as a part of these bug bounty programs and led to the discovery of several vulnerabilities in the software that have since been addressed.

The goal of Diffix is to answer SQL queries, such as:
```SQL
SELECT COUNT(*) FROM loans
WHERE loanStatus = 'C'
AND clientId BETWEEN 2000 and 3000
```
on a database while preventing the disclosure of record-level data.  
A challenge for a system like Diffix is to answer such counting queries while preventing an adversarial user---the attacker---from learning record-level information. As you might remember from the last post, such a system must not provide exact answers to arbitrary queries. Otherwise the attacker could mount a *differencing attack*. For example, an attacker who knows that Billy Joel's `clientID` is 2744 could learn the status of the singer's loan by comparing the answer to the previous query with the answer to:

```SQL
SELECT COUNT(*) FROM loans
WHERE loanStatus = 'C'
AND clientId BETWEEN 2000 and 3000
AND clientId != 2744
```

An intuitive defense is to add noise to the answer---say, Gaussian noise sampled from \\(N(0,10)\\).
Now the difference \\(\Delta\\) in the responses to the two queries is a random variable sampled from \\(N(1,20)\\) or \\(N(0,20)\\) depending on whether Joel's `loanStatus` is or isn't `C`.
With just one sample, the distributions are hard to distinguish.

Still, this scheme is easily thwarted by *averaging attacks*.
If the noise is sampled anew each time a query is made, then repeatedly making the same pair of queries generates many independent samples from \\(N(1,20)\\) or \\(N(0,20)\\), and enough queries would make it possible to distinguish these distributions easily.

As before, there is an intuitive defense: use the same noise for repeated queries.  This defense introduces its own new attacks by making many syntactically-distinct but semantically-equivalent queries.  Those attacks in turn suggest new defenses which suggest new attacks, and so on.  Diffix is, in a sense, the result of this hypothetical arms race.


From a technical perspective, Diffix consists of three components, which together are intended to thwart these attacks.  First, Diffix only accepts a limited subset of SQL and will categorically reject any query that does not fit this subset.  These restrictions---including tight restrictions on `JOIN`s and on the number of mathematical functions in a single expression---limit the ability of an adversary to use the full power of SQL to access the database.  The second component is a collection of data-dependent ad-hoc methods to prevent leaking information about individuals or very small subsets of users, including suppressing answers to queries about small numbers of users, flattening outliers.


The final component is Diffix's layered noise.  This noise is comprised of two individual noise terms added together: a *data-dependent* term whose variance is constant[^2] and a *query-dependent* term whose variance depends on the complexity of the query. The data-dependent noise prevents na&iuml;ve averaging attacks. It is a pseudorandom error where the seed of the pseudorandom function depends on individual data records that contribute to the query result. Semantically equivalent queries using different syntax will nonetheless share this error, so simply averaging the responses will not remove this noise.

The query-dependent noise prevents a na&iuml;ve Dinur--Nissim style reconstruction attacks. A noise term of magnitude \\(\Omega(1)\\) is generated deterministically for each condition in the `WHERE` or `HAVING` clause of the SQL query, and the terms are added together.  A Dinur--Nissim query is a random subset of the dataset that contains \\(\Omega(n)\\) records. The straightforward way of specifying such a query is to enumerate the subset record by record using \\(\Omega(n)\\) conditions:[^3]

```SQL
SELECT COUNT(*) FROM loans
WHERE loanStatus = 'C'
AND (clientId = 2007
OR clientId = 2018
...
OR clientId = 2991)

```

A query with \\(\Omega(n)\\) conditions is answered with noise with standard deviation \\(\Omega(\sqrt{n})\\), enough to thwart the reconstruction algorithms.


### Carrying out reconstruction ###
The additional noise per SQL condition  is the main obstacle to running a successful reconstruction attack on a database behind Diffix.  As described above, the noise prevents the naive implementation of the reconstruction algorithm from receiving accurate enough answers to reconstruct the database using a reasonable number of queries.  
A natural approach is to use very few SQL conditions---ideally, just one---to make random-enough queries, each identifying a subset of the records in the dataset.
So the challenge is to formulate a large family of such queries that are accepted by Diffix's restricted subset of SQL, using as few conditions as possible.

#### The Cohen--Nissim Attack ####

Instead of specifying each row with a separate condition, the Cohen--Nissim attack uses an ad hoc *hash function* to extract entropy from the data itself in order to systematically choose the needed subsets.  
Suppose we have a list of the values in the database's  `clientId` column, and we want to recover the `loanStatus` secret bit. Rather than explicitly enumerating the `clientId`s for a random subset of the rows to include in each query, we can write a boolean-valued function which evaluates to true on about half of the `clientIds and ask Diffix to include only the rows for which the condition is true.  In this way, instead of first choosing a subset of rows and then asking Diffix about those rows, we choose this function and use its evaluation to specify our random subset.

After some experimentation with the language restrictions, Cohen-Nissim settled on the following:
  ```SQL
...
WHERE FLOOR(100 * ((clientId * 2)^0.7))
   = FLOOR(100 * ((clientId * 2)^0.7) + 0.5)
  ```

Let's see what this does. Let \\(d=d_0.d_1 d_2 d_3 d_4 \dots \\) be the decimal representation of the value \\(d = (\mathtt{clientID}\cdot 2)^{0.7}\\), which appears on both sides of the equality.
The expression is true if and only if \\(d_3 < 5\\).  To see this, the left hand side evaluates to \\(d_2 = \lfloor 100d \rfloor\\); the right hand side evaluates to \\(d_2\\) or \\(d_2+1\\) depending on whether or not \\(d_3 < 5\\). Replacing 100 with other powers of 10 changes which digit in the decimal expansion is checked.

By varying the constants in the SQL query, this single expression yields a whole family of conditions, albeit a very ad-hoc one.  The hope was that, for different primes \\(q\\) and fractional exponents \\(p\\), the individual digits of the decimal representations of \\((\mathtt{clientID}*q)^p\\) would be random enough for reconstruction to work.
The complete attack queries looked like this:

```SQL

SELECT COUNT(clientId) FROM loans
WHERE FLOOR(100 * ((clientId * 2)^.7))  
    = FLOOR(100 * ((clientId * 2)^.7) + 0.5)
AND clientId BETWEEN 2000 and 3000
AND loanStatus = 'C'

```

The range condition at the end simply selects a subset of the data which is small enough for the attack to run quickly on a personal computer but large enough to satisfy the requirements of the Diffix bounty program.  This family of queries allows for a linear program to reconstruct the secret `loanStatus` bits with high accuracy.


In the course of verifying the attack for the Diffix bounty program, reconstruction was carried out on 4 different ranges of `clientId`s containing 455 records. For each record, the attack correctly determined whether or not the corresponding `loanStatus` was `C`.

Aircloak's response to this attack was to further restrict the query allowed by Diffix.  Columns like `clientId`, where most of the values correspond to a single user, are tagged as 'isolating', and mathematical functions can no longer be used on such columns.  The hope was that this modification would prevent the extraction of entropy from an identifying column via hashing.

#### The Dick--Joseph--Schutzman Attack ####

Without the ability to directly use a uniquely identifying column from the database itself, we need another way to single out rows of the database.  We can use an idea that's been around since the 1990s, when  Latanya Sweeney showed[^4] that almost 90 percent of Americans could be identified with only a date of birth, ZIP code, and gender, but of course each of these alone is nowhere near sufficient to isolate a single individual.  We can use this idea and try to evade the modification to Diffix by choosing multiple non-isolating columns which, when taken together, can isolate rows in the database.

This modified attack uses the `pickup_latitude` column in the `taxi` data set as the source of entropy, which is non-isolating, in part because there are a large number of rows where the value is recorded as zero.  We can combine this column with the `trip_distance` column and run queries of the following form:

```SQL

SELECT COUNT(*) FROM rides
WHERE FLOOR(pickup_latitude ^  8.789 + 0.5)
    = FLOOR(pickup_latitude ^  8.789)
AND trip_distance IN (0.87, 1.97, 2.75)
AND payment_type = 'CSH'

```

This example query is part of an attack to recover the `payment_type` column, which (for the purposes of this attack) is a binary column containing two values: `CRD` (for credit card payments) and `CSH` (for cash payments).  The `IN (0.87, 1.97, 2.75)` restricts to a subset of the data with about 450 rows, each with a distinct value for `pickup_latitude`.  However, because across the whole database, very few rows have a distinct value in this column, Diffix does not consider it 'isolating' and it can be used as Cohen--Nissim used `clientId`.  The values in `pickup_latitude` are recorded to six decimal places of precision and the least significant four of them are essentially random digits.  By choosing an appropriate range for the exponent and using the same trick as in the Cohen--Nissim attack, allows the construction of a Diffix-accepted query which includes around half of the rows in the targeted subset. Using different values for the exponent leads to a large family of queries which allow the attack to be carried out as before with similarly high accuracy of over 95 percent.

Dick--Joseph--Schutzman additionally extends this attack to recover *numerical* rather than just binary secret data.  By using queries of the form

  ```SQL
SELECT SUM(passenger_count) FROM rides ...

```
Diffix will return return noisy sums over the specified subset for a numeric column like `passenger_count`.  Then, a similar linear program can reconstruct estimates for these values with high accuracy.  For numeric columns like `passenger_count` which take on relatively few distinct values, the attack recovers the exact values with accuracy above 75 percent.  Due to limitations in the Diffix bounty program rules which require perfect reconstruction of a value to be considered 'accurate', we didn't evaluate the performance of the attack on numeric columns with richer values, such as `dropoff_latitude`.


Finally, this attack extends to one used to reconstruct string data character-by-character.  A U.S. social security number consists of a string formatted like `xxx-xx-xxxx` with ten unknown digits in three blocks separated by dashes.  There are potentially ten billion different strings that could appear in this column.  However, by exploiting the structure of the data, a separate attack can be run to recover each digit individually using the summation attack, since there are only ten different values each digit could take.  Queries of the form

```SQL

SELECT SUM(CAST(SUBSTRING(ssn, 3, 1) AS integer)) FROM rides ....

```

can be used to recover the 3rd digit from each row's social security number.  Running this attack for each digit then aggregating the individual guesses to construct a guess for each user's entire social security number allows the attack to achieve perfect reconstruction on about 90 percent of the values.  A similar attack worked on the datetime columns, with separate attacks on the value in the seconds position, the minutes position, and so on, and finally piecing these together to correctly reconstruct about 85 percent of the values.

Again, Aircloak's response was to restrict the query language.  Both of the successful attacks relied on the use of some arithmetic inside of  a `FLOOR` function to check whether or not a row is included in a particular query. Diffix now forbids the use of arithmetic with *bucketing functions* such as `FLOOR`, `CEIL`, `ROUND`, etc.  This defeats strategies which choose random-ish subsets via hashing, but does not necessarily preclude the extraction of entropy from the data in other ways.






#### What's Next? ####


We'd again like to thank Aircloak for opening their system to attacks and critiques through the Diffix bounty program.  By being so willing to expose their product in this way, they have provided a test bed for us to bridge the gap between theory and application and demonstrate how a linear reconstruction attack might work in practice.  Vulnerability to these and other attacks are a potential threat to any data privacy system which does not account for the cumulative threat to privacy of may result from many seemingly-innocuous queries, including Diffix. The attacks we describe here only require the attacker have access to some subset of the data with a sufficient amount of entropy, and while more entropy allows for more complete reconstruction, it may be possible to use something potentially very accessible like a list of users' email addresses in this kind of attack to reconstruct a non-trivial amount of the secret data using queries against a system that adds independent noise to each query.  Systems like this fall into the trap of the classic arms race, where a designer builds a system to protect against certain attacks, then a clever and determined adversary defeats the system, and the designer is forced to make revisions.





[^1]: Descriptions of Diffix and Aircloak are based on [https://www.aircloak.com](https://www.aircloak.com), [https://arxiv.org/pdf/1806.02075.pdf](https://arxiv.org/pdf/1806.02075.pdf), [https://demo.aircloak.com/docs/](https://demo.aircloak.com/docs/), and the authors' participation in the Aircloak bounty program.

[^2]: The variance is proportional to the largest effect any single user has on the output. For `COUNT` queries, this largest contribution is 1, and for `SUM` queries, it's roughly the magnitude of the largest value in the column.

[^3]:Note that Diffix's syntax restrictions don't allow disjunctions (using `OR`s). An equivalent way of writing this that is allowed by Diffix would use `...WHERE ... AND clientId IN (2007, 2018,...)`. For such conditions, Diffix adds a noise layer for each element of the `IN` condition.

[^4]: Sweeney, Latanya. "Simple demographics often identify people uniquely." Health (San Francisco) 671.2000 (2000): 1-34.
