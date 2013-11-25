---
layout: post
title: "The Bubble around NLP and Article Summarization. A text summarization engine!!"
date: 2013-11-25 10:31:21 +0530
comments: true
sharing: true
footer: true
categories: [ NLP, Text Processing, Text Summarization ] 
---

The development of the following piece was mainly during the same time as the acquisition of Summly by Yahoo!. The main intention in developing such a solution was to generate a stream summarization where a part of the article is edited by many users and as a article owner one would want to maintain a master version which would consist of the best of all the edits. The description of the thought-process while solving such a problem is the main content of this article.

*Questions asked*

Can the subject document be summarized to highlight the most relevant points?

Can the comments/edits to particular sections be summarized to capture the essence of all the edits to that section?

**How was this solved?**

Text summarization, typical text-processing problem, is extensively used in search engine optimization (but for a different purpose). I've created, structured and tailored this to summarize documents to improve user experience and summarize comments/edits as a mechanism for information enhancement.

**Article Summarization**

2 broad ways of solving this: 1) Extract 2) Abstract
Extract is the process of extracting sentences from the main document which can capture the essence of the document without modifying the sentence structure or creating new text in coming up with a summary.
Abstract is the process of generating document summary by synthesizing text by capturing the intent, emotion etc. etc. of the author in a particular context (Slightly harder problem to crack).

This system goes the extract way to summarize documents/comments/edits. It is a simple weighting and relevance based extraction, which identifies sentences that are important in the document context and includes them to be part of the summary.

*Two parts for extraction*

*Weighting Measure*

The idea is to weight sentences based of things like where is the sentences positioned in the document, what are the kinds of words which are present in the document, are there named entities in the sentences E.g. Name of a person, organization, location etc.

*Theory behind this*

Sentences weight based on position is because of the general notion of providing information upfront and slightly fading away towards the end(Writers call it the inverted pyramid style of writing). The measure again is to avoid not so informative sentences, which tend to fall in the middle of the argument. But this does not imply these sentences are not included at all. It depends on the other two weighting criteria as well.

The second parameter is to identify sentences, which consists of words that are important to a particular collection of documents/document E.g. If the document set is world politics, then the words belonging to world-genre and politics would have more weightage than the words belonging to lets say music. One such measure is TFIDF – Term Frequency Inverse Document Frequency, which is calculated for a category of documents, which represents the importance of a word/gram to the given category.

Nouns, proper nouns in particular give out more information individually than other kind of words. The third parameter is used to capture this and include this as part of the weighting measure. Name Entity Recognition is a NLP process, which allows one to extract named entities in a give piece of text.

        Weight of Sentence i = alpha * X(i) + beta * Y(i) + gamma * Z(i)

Where,

Alpha, beta and gamma are bias terms (co-efficients) to keep the weighting measure tunable

X(i) = position based weight for sentence i

Y(i) = sigma k=0 to K( normalized tfidf(k))/K

K-number of terms in the sentence

Z(i) = normalized NER weight


*Cosine Similarity Based Relevance Measure*

Cosine Similarity is a measure of closeness of any two given vector in N dimensional space. The value is the cosine of the angle between vectors.

The idea is to represent the document and the sentences within the document as vectors mapping to the feature space of terms/grams present in the sentence.
Algorithm

The following steps are performed to find the summary sentences combining the above-described measures:

Feature space = all terms/grams in the document (after preprocessing)

    Step 0:find weights for all the sentences,
    Step 1:represent the document and sentences as vectors in the given Feature space
    Step 2:find the cosine similarity of sentences with the document
    Step 3:select the sentence x with highest effective weight and add it to summary list where

Effective weight = cos-measure of sentence i from step 2 + corresponding weight of the sentence i from step 0

    Step 4: remove all the terms/grams from the feature space which were part of the chosen sentence x
    Step 5: check if the number of sentences in the summary list meets the compression ratio. If no then goto Step 1


*Why combination of two techniques?*

Initially when I started, it was just the weighting measure. Then I tried the cosine similarity measure which measure the relevance better than the weighting measure. But in the end the summary should consist of both the relevance measure and also the informative measure where we are interested in what the sentences themselves have to say.(sort of incremental addition)

*Why step 4 in the algorithm?*

At the end of Step 3, the sentence obtained is the one that relates to the document the most. Selecting sentences based on above measure ensures that the summary covers the major topics and the topics, which are informative. Whereas eliminating all the terms/grams contained in x from the document in Step 4 ensures that the subsequent sentence selection will pick the sentences with a minimum overlap with x. This leads to the creation of a summary that contains little redundancy.

{% img /images/sample_summary_document.png [Sample Article [Sample Article used to summarization]] %}

{% img /images/sample_summary.png [Sample Summary [Sample Summary of the above article]] %}
**Edit Stream Summarization**

- *Defining this problem Space*

*Why summarize edit stream?*

- Various applications

News websites, websites with text content and lot of people talking on a particular topic/document/article/ page (100s and thousands of comments) usually, face this scenario of having to go through all comments to understand user intent. It is very likely that people are talking about the given topic/article and specifically to some part of the topic. The idea is to go through this comment stream, identify the sub-topic being discussed and group comments based on such sub-topics and give an essence what the crowd is talking about and what is the gist of what they are talking about.

Information enhancement, where in the information/content provider would like to benefit from the information in comments/edits to enhance the primary/base content (in other cases, probably more of a mechanism to aggregate feedback).

The capability to edit a particular document, to start with, is limited to at the paragraph level. The following are the steps involved in coming up with an edit stream summary:

For Every edited paragraph do:

- Step 1: Cluster similar sentences in the edits for a paragraph to capture the essence of themes of edit paragraphs. The feature space is something which could be tuned accordingly looking at the amount of data being handled (keeping in mind the dimensionality aspects, convergence etc. etc.
- Step 2: Pick dissimilar sentences within the cluster and summarize/find relevant sentences using the weights measure at the cluster level. The extent of dissimilarity can be found using simple measures like Jaccard Index, cosine similarity, pearson co-relation coefficient etc.
- Step 3: Combine to represent summary of an edited paragraph.

{% img /images/edits_and_edits_summary.png [Edits and edits summary ] %}

The edits and the corresponding edit summary for the first paragraph of the article shown in the previous section is as shown above.

Clustering mechanism employed is called EM Clustering. Basis: Expectation Maximization Algorithm

*Reason why clustering works?*

Modeling sentences into a feature space tends to follow normal distribution (Gaussian Distribution) where many of the points tend to collect together (Measures of central tendency follow). In the sentence collection scenario (global population), there exist many such gaussians simultaneously. They can hence be modeled as mixture of one or more gaussians (Hence the name mixture model). Its hard to keep the explanation purely verbose, shoot questions if you do not understand anything or you want more explanation, the following have useful information which might help

http://en.wikipedia.org/wiki/Mixture_model#Gaussian_mixture_model

http://www.quora.com/Machine-Learning/What-is-an-intuitive-explanation-of-Gaussian-Process-Models

http://stackoverflow.com/questions/11808074/what-is-an-intuitive-explanation-of-expectation-maximization-technique

Explanation for EM for Gaussian Mixture Models(one of the best n simple explanation i've found after a lot of searching)

The article summarization can be achieved using the Apache Lucene's snippet summary capability(and built in NLP capabilities) by indexing docs and generating term frequency list and querying sentences consisting top words in the generated list against a particular document. The retrieved list of sentences can be part of summary. But this does not give the capability to tune the summarization process to tailor different kinds of documents and writing style(Its always a relevance based summary).

Tech Stack Highlight:

- Django
- NLTK

*What more can be done?*

- Modeling the hypothesis of Rhetorical Structure theory
- Use of more sophisticated NLP techniques like use of sysnets, relationship extraction etc.

Until Next Time, Live Long and Prosper!

[Originally Posted on Blogspot]
