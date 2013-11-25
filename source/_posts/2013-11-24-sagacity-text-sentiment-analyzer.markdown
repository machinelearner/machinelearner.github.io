---
layout: post
title: "Sagacity - Text Sentiment Analyzer"
date: 2013-11-24 23:41:31 +0530
comments: true
sharing: true
footer: true
categories:  [Mongo, Rails, Sentiment Analysis, Support Vector Machine]
---

Hello Hive-Mind,

***This following is a piece of work which i tried out in my free time a while ago.
The emphasis was on extracting sentiment from a piece of text, preferably a tweet(and all the twittery characters follow!!). I've have tried solving this as a text classification problem. The system is trained using nearly a million positively and negatively labelled tweets using the LibLinear version of SVM. I've tried to summarize the whole process in the following sections.***

<!--more-->

**Support Vector Machine: An Introduction**
===========================================

Visual and detailed explanation with different examples can be found in this blog article and in this paper/article. But I've given it a try, explaining Basic working of SVM in this section.

Basic SVM algorithm is an efficient binary classifier. The idea behind SVM approach to text classification is that the text(terms/grams in text) is mapped to a feature space in N dimension. This feature space is the basis for the SVM algorithm, which determines a linear decision surface (hyperplane) using the set of labeled data within it. This surface is then used to classify future instances of data. Data is classified based upon which side of the decision surfaces it falls. SVM is applicable to both linearly separable and non-linearly separable patterns. Patterns not linearly separable are transformed using kernel functions- a mapping function, into linearly separable ones. It can be formulated as follows,
The optimal hyperplane separating the two classes can be represented as:
        ω.X +β = 0     (1)

where,

X – sample input vectors defined as

{(x1,y1),(x2,y2)............(xk,yk)} xk ∈ Rn yi ∈ {1,-1}

ω,β - non zero constants ω indicating the weight component and β indicating the bias component


The ordered pair x,y is the representation of each input used to form hyperplane which are N dimensional vectors labeled with corresponding y.

        ω.X +β ≥ 1 if yi =1     (2)
        ω.X +β ≤ -1 if yi=-1    (3)

These can be combined into one set of inequalities:

        yi(xi. ω + β) ≥ 1 ∀i      (4)

The above inequalities hold for all input samples (linearly separable and suffice the optimal hyperplane equation). The optimal hyperplane is the unique one, which separates the training data with a maximal margin. The figure 1 depicts the above mathematical representation.
![multi class svm](http://www.jvrb.org/past-issues/3.2006/760/svm1.jpg "SVM and Hyperplanes")

        Figure 1: Classification in Linearly separable two class data

In our experiments and the proposed system, LIBLINEAR - A Library for Large Linear Classification is used as the library providing the SVM algorithm.

**Feature Space**
=================

There are various ways of handling text-processing problems in a vector space model. The system considers a bag of words, Unigram scenario where in every word in the corpus is represented as a dimension, which make up to the feature space. There are plenty of ways to weight such features:

*Presence*, where in the term is accounted for in a binary fashion depending on whether or not the word is present in the document or not.

*TFIDF* – Term Frequency & Inverse document frequency, where in the belonging and importance of a particular word to a particular category is accounted. TFIDF is a numerical statistic, which reflects how important a word is to a document corpus

        TFIDF = norm(TF) * IDF

Where,

TF = number of times a term occurred in the set

IDF = log(|D|/||Dt|)

*Delta-TFIDF* - This measure works best for binary classification and especially well in a sentiment analysis scenario. This kind of term frequency transformation boosts the importance of words that are unevenly distributed between the positive and negative classes and discounts evenly distributed words. This better represents their true importance within the document for sentiment classiﬁcation. The value of an evenly distributed feature is zero. The more uneven the distribution the more important a feature should be. Features that are more prominent in the negative training set than the positive training set will have a positive score, and features that are more prominent in the positive training set than the negative training set will have a negative score. This makes a clean linear division between positive sentiment features and negative sentiment features(the reason why I have chosen to use this weighting mechanism).
Table 1 summarizes the top 10 words representing the positive and negative sentiment.

                                ΔTFIDF = TFIDF+ve – TFIDF-ve

|Positive Sentiment Words  |Negative Sentiment Words 
|--------------------------|-------------------------
|thanks                    |sucks                    
|love                      |want                     
|good                      |sorry                    
|great                     |sick                     
|happy                     |hate                     
|you                       |wish                     
|awesome                   |bad                      
|haha                      |work                     
|nice                      |miss                     
|lol                       |sad                      
|----------------------------------------------------
	

        Table 1: Top words occurring in positive and negative sentiments.

The system can be divided into the following components:

*Pre-Processor*

This stage includes data cleaning processes required for text analysis. Cleansing is essential in any for of text analysis. The reason being the dimensionality factor. The more the number of unigrams/terms imply the dimensionality increase, which in turn result in complex computational needs and in the end a complex model. In order to avoid such a scenario, pre-processing techniques can help reduce the size of the feature space. There are other more sophisticated dimensionality reduction techniques, which can be employed, but eliminating few of the obvious things and improving unigram indicators is the easiest and the primary step. In this regard, the pre-processing comprises of removal of stop words, which are usually articles, pronouns, helping verbs etc., which are not indicative of any category. Spell-correction reduces the chances of same word being accounted twice in because they were spelt differently. This process also deals with accounting over expressiveness prior to correction. As part of pre-processing, the document/tweet is modeled into vectors and into a format such that the LibLinear classifier can easily consume these vectors.

*Training*

Training process consists of first generating the delta-tfidf dictionary which can then be used to weight vectors. The liblinear train involves choosing the right dataset and optimal training parameters for model generation. The system uses L1R_L2LOSS_SVC solver type(Tiny peek into what is L1R_L2LOSS_SVC). The output of the training process is a model, which represents the trained knowledge and is used for classification of new instances during the prediction phase.

*Classification* 

This is the stage where, given a sentence, transform it to a vector and predict against the model generated during the training stage.


The code for all the above process can be found at Sagacity. Will try and make the app accessible to all of you. The tech stack included the following,
Ruby back end(training process)
Rails front end(very simple)
Ruby version of LibLinear for training and classification
Mongo

Analysis indicated a cross-validated(10 fold) accuracy of 78.4667% and average accuracy of ~83% with similar F-Measure values across various datasets. Lot of scope for improvement. Hopefully I'll be able to take it forward.

This is the first of many article to come. Until Next Time, Live Long and Prosper!


[Originally Posted on Blogspot]
