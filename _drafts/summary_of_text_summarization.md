---
layout: default
title: "A Summary of Text Summarization"
date: 2014-03-18
tags: summarization ML
published: False
---

# A Summary of Text Summarization

## Introduction

A summary of one or multiple texts conveys the important information in these
texts while being far shorter than the original texts.

## Categories of Summarization Techniques

### Extractive Summarization

Decide what the summary content should be.
Rely solely on extraction of sentences.

### Abstractive Summarization

Produce a grammatical summary.

### Topic-Driven Summarization

?

## Single-Document Summarization

### Luhn 1958

Word and phrase frequency.
Stem words and remove stop words.
Compile content words sorted by decreasing frequency.
For each sentence, derive significance factor that counts how often
significant words occur in a sentence and rank sentences based on this.
Return top sentences as summary.

### Baxendale 1958

Position in the text.
In majority of paragraphs, the topic sentence comes first.
To a lesser extent topic sentences are the last sentence of a paragraph.
Return either first or last sentence to summarize the entire paragraph.

### Edmundson 1969

Key phrases.
Mainly described the typical structure for extractive summarization.
Used the features of Luhn and Baxendale: word frequency and positional
information.
Further introduced cue words (*significant*, *hardly*, *We*) and the
skeleton of the document (sentence is title or heading).

### Kupiec *et al.* 1995

Naive-Bayes classifier to categorize each sentence as worthy or unworthy of
extraction.
Features equivalent to those introduced by Edmundson and additionally included
sentence length and the presence of uppercase words.
Score each sentence based on presence of features and extract most highly
scored sentences.

### Aone *et al.* 1999

Naive-Bayes classifier, just as Kupiec *et al.* but with richer features.
Describe a system called DimSum.
Features include: term frequency (tf) and inverse document frequency (idf) to
derive signature words.
Idf computed from corpus of the same domain.

### Lin and Hovy 1997

Only used sentence position as feature.
Idea: texts generally follow predictable structure.
Sentences with greater topic centrality occur in abstract, titles, etc.
Discourse structure of documents depends heavily on domain so this method
needs to be domain-tailored.

### Lin 1999

Naive-Bayes classifier approaches assume independence of features.
Here, Lin used decision trees to drop this assumption.
Work focused more on information retrieval so assumed some available query.
Features include query signature (score depending presence of query words),
IR signature (most prominent words in corpus),
numerical data (sentence contains a number),
proper name (sentence contains a proper name),
pronoun or adjective (contains pronoun or adjective),
quotation (...).

### Conroy and O'Leary 2001

Use a hidden Markov model (HMM).
This is a sequential model that can account for local dependencies between
sentences.
Three features used: position of the sentence in the document,
number of terms in the sentence, and likeliness of sentence terms given the
document terms.
Suppose a document is $s$ sentences long then this HMM contains
$s$ summary states and $s+1$ nonsummary states.

### Osborne 2002

Log-linear classifier as opposed to naive-Bayes classifier.
Does not need to assume independence of features as is necessary for
naive-Bayes approaches.
Features include word pairs, sentence length, sentence position,
inside introduction, inside conclusion.

### Svore *et al.* 2007

Approach based on neural nets.
Used a training set of 1365 CNN.com stories consisting of the article itself
and three human generated story highlights.

### Barzilay and Elhadad 1997

Define lexical chains that consist of related words in a text and span
short distances (adjacent words or sentences) or long distances
(the entire document).
Identify lexical chains and extract strong lexical chains.
Approach relies on lexical cohesion, i.e. words that refer to the same
topic across different sentences.
Using Wordnet, select a set of candidate words, find appropriate chain using
Wordnet distance as a measure of relatedness, if a chain is discovered
add the candidate to this chain.

### Marcu 1998

Used elaborate discourse theory known as Rhetorical Structure Theory (RST).
Non-overlapping pieces of text spans can be divided into the nucleus
(that holds the essence of a sentence) and the satellite.
Create a discourse tree, labeling each node either a nucleus or a satellite.
Return only top-most nodes of this tree.

## Multi-Document Summarization

### McKeown and Radev 1995

Summarization system SUMMONS.

## Evaluation

### TREC

### DUC

### MUC

### TIPSTER-SUMMAC

Stems from Lin 1999 work.

## References

- http://www.summarization.com/
- http://libots.sourceforge.net/
- http://www.lemurproject.org/
- http://pythonwise.blogspot.co.uk/2008/01/simple-text-summarizer.html
- https://github.com/Rotten194/summarize.py
- https://github.com/miso-belica/sumy
