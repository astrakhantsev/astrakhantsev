---
title: 'Underrated Positive-Unlabeled learning: specifics, use-cases and tools'
excerpt: "PU learning relates to semi-supervised learning, outlier detection, ordinary supervised learning, but has its own specifics, which I try to describe here."
permalink: /pu-learning/
tags:
  - ML
  - Tools
---

Positive-Unlabeled learning (PU) is a learning of ordinary binary classifier from only positive and unlabeled instances.
It relates to outlier/anomaly detection, semi-supervised learning (SSL) as well as oridnary supervised learning, but has important specifics, which I describe below.

## PU learning vs semi-supervised learning and outlier detection

Similarity stems from the fact that in all cases we try to build binary classifier from insufficient data: 
in outlier detection we don't have enough positive instances, while in SSL setting we simply don't have enough any labeled data.

However, in SSL we use unlabeled only for improving classifier --
however, it is rarely used in practice;
at least in my experience addition of unlabeled results didn't improve results, even for very small labeled data.

In anomaly detection (_how handy it is to have perfect synonyms -- the simplest way to cope with tautology!_),
outliers are usually not similar to each other, while non-outliers, i.e. negatives, are similar (e.g. they are assumed to be from the same distribution);
in other words, we can't say that 2 outliers are more similar to each other than outlier -- to non-outlier.

Contrariwise, in PU we have positives similar to each other and we know (hope) that they are not similar to negatives.
(And we can't simply declare all unlabeled instances as negatives,
because these unlabeleds may contains a lot of positives, not less than we have explicitly labeled as positives.)

Good example is a websites classification: if we want to separate restaurant sites from all others,
we can easily crawl dozens of positives, but it'll be hard to crawl representative negatives,
since we don't know a priori enough information about representativeness itself.

Note that it can be applied in other fields, e.g. image classification, in particular --
[land cover classification](https://papers.nips.cc/paper/5509-analysis-of-learning-from-positive-and-unlabeled-data.pdf).

## PU learning vs supervised learning

What is rarely mentioned is that ordinary supervised (binary) learning can sometimes be more correctly modeled by PU.
In the example above, one who didn't hear about PU would somehow sample negatives from all sites and apply good old SVM or ~~random forest~~ xgboost.

And maybe s/he'll get lucky and -- sufficiently good results.
This strategy, which is called negative sampling, has definitely a right to exist
and even to be included into the name of the most popular way of word2vec training,
namely skip-gram model with negative sampling (SGNS).

The only thing I'd like to warn is that you should use such negative sampling consciously, 
otherwise you risk to have kind of sample selection bias, i.e. obtain good quality on cross-validation, but fail in production.

One more example when shift from supervised learning to PU can help is word sense disambiguation --
choosing correct meaning for a term (word or collocation) depending on the context.

Assume that you have data labeled with non-perfect guide,
e.g. without formulation of boundary cases or needed level of sense-granularity,
and by non-perfect annotators, e.g. CS-students.

Let's consider the example sentence: `The army entered the city`.
Here term `army` may be disambiguated to top-level [Army](https://en.wikipedia.org/wiki/Army), or to specific [Ground forces](https://en.wikipedia.org/wiki/Army), or to known-from-context [US army](https://en.wikipedia.org/wiki/United_States_Army).
But other concepts like films or newspapers (see 
[wiki disambiguation page](https://en.wikipedia.org/wiki/Army_(disambiguation))) are obviously wrong.

In this case, any of the 3 correct meanings can be chosen by an annotator, but only one can be,
so non-chosen meanings would include both negative and positive instances,
so these non-chosen meanings are actually unlabeled instances -- hello, PU!

Of course, it is better to fix the source of the problem, i. e. 2 assumptions mentioned above, but it is not always feasible.

## PU learning as unsupervised learning

Another useful pipeline that was tested a couple of times (including my PhD work): 
from all unlabeled examples somehow extract positives, then learn PU on that.
Note that such positives should be precise, but heterogene/representative at the same time, so prefer simple heuristic.

Thus you'll have fully unsupervised approach that may learn from data in some sense --
of course, if your task lets invention of appropriate heuristic for seed positives extraction.

## PU tools

Despite its usefulness in practice, PU learning seems underrated:
Among possible implementations all I found is a 10-years old
[C-program](https://www.cs.uic.edu/~liub/LPU/LPU-download.html) (btw, this site lists many good papers on subject)
plus several abandoned python libs, e.g. [this](https://github.com/aldro61/pu-learning)
and [that](https://github.com/jperla/pulearning).

I don't fully understand the reasons.
Maybe, people just apply negative sampling or reinvent simple PU algorithms not naming this as PU.

In last project where I needed PU, I used Scala and Apache Spark MLlib, therefore I implemented a pair of simplest algorithms:
[Traditional](https://www.cs.uic.edu/~liub/S-EM/unlabelled.pdf) and
[GradualReduction aka PU-LEA](http://www.sciencedirect.com/science/article/pii/S0306457314001095).

In a nutshell, they wrap probabilistic classifiers like Logistic regression or Random forest by the following strategy:
consider all unlabeled as negatives;
learn probabilistic classifier on that;
extract so-called reliable negatives, i.e. those instances classified with too low confidence;
learn classifier on positives and those reliable negatives;
repeat.

[Spy-EM](https://www.cs.uic.edu/~liub/NSF/PSC-IIS-0307239.html) should be implemented as well,
but in my case simplest algorithms worked good enough
-- and notably better than trivial 1-step approach
(when you consider unlabeled instances as negatives and get SL classifier right away).

Ranking-based SVM and other sophisticated algorithms seem to be more sensitive to noise in data,
so they should be used only for pure PU setting, not the unsupervised pipeline described above.

Anyway, [pu4spark](https://github.com/ispras/pu4spark) is now ready for use;
it is also available at jcenter and mavenCentral
(uploading it there turned out to be non-trivial and I'm going to describe this soon).
