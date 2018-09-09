---
title: '[Reading notes] Entity Matching with Active Monotone Classification'
date: 2018-09-09 16:45:01
tags: [machine learning]
category: [学习一个]
---

[This](https://dl.acm.org/citation.cfm?id=3196984) is a single-author work by Prof. [Tao Yufei](http://www.cse.cuhk.edu.hk/~taoyf/), awarded with PODS 2018 best paper. Prof. Tao gave us a seminar of his outstanding work on this Friday. In this seminar, he introduced (1) what the entity matching problem is; (2) how he formulates entity matching to a multidimensional classification problem; (3) his "small-and-sweet" algorithm to solve this problem and (4) some hints about his theoretical analysis. In this post, we focus on the first 3 points.
<!--more-->

## The entity matching problem
Suppose Amazon and ebay place a set of advertisements on their website respectively, we denote this two sets with $A$ and $E$. each advertisement has attributes like *prod-name*, *prod-discription*, *year*, *price*, and so on. Now we want wo know whether advertisements $x$ and $y$ are about the same product, for all $(x, y) \in  A \times E$.

## Convert entity mathching to a multidimensional classification problem

## A "small-and-sweet" algorithm to solve the formulated problem


## My thoughts
1. The process of problem formulation introduced in this paper is quite instructive. The framework to deal with matching problem can be transferred to many other problems...

2. As the title of the paper implies, it's an active learning algorithm, which require less labeling job for training set than supervised leanrning, and can be done interactively. However, it's a monotone classifier, which means there is a trade-off: the algorithm will perform perfectly (zero error) if the data set is strictly monotone, but if the data set has undesired pattern, the classification result can be terrible. An extreme case can be shown as the following picture. In short, it has strong assumption for data, and not stable. If we have enough labeled data, or the labeling job is cheap, we should still choose descision tree or some other classifier.

(a naughty spy point dive deeply into its opposite group, and unfortunately be chosen by the algorithm)

(give a table of comparison with decision tree and SVM)

3. how to solve the problem in 2? Careful feature engineering? Run several times and do ensemble? or?