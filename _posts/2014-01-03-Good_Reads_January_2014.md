---
layout: default
title: "Good Reads January 2014"
date: 2014-01-03
tags: links reads
---

# Good Reads January 2014

## Recursive Neural Networks

[Socher *et al.*](http://nlp.stanford.edu/pubs/SocherLinNgManning_ICML2011.pdf)

[Yoshua Bengio](http://nlp.stanford.edu/pubs/SocherLinNgManning_ICML2011.pdf)

* Recursive structure in scenes and natural language
  (Principle of Compositionality)

    * The 'whole' (sentence, image) can be split into hierarchical regions
      (noun phrases, words, objects) that may occur in different 'whole'
      objects
    * Meaning of a sentence is given by its words and the rules that combine
      these words.
      
* Composite vector representation

    * Representation of a sentence in a vector space that represents
      the vocabulary.
      
* Recursive neural networks jointly learn compositional vector representations
  and parse trees

* Generate a parse tree of the sentence using a standard tokenizer,
  the tokens are the leaves of the parse tree
* Iteratively, choose two child nodes and decide if you want to combine them
  into their parent node which is the semantic representation of the child
  nodes - also produce a score of how plausible the new parent node is
  
## Unsupervised Feature Learning - Amgad Muhammed

[http://www.slideshare.net/AmgadMuhammad/unsupervised-feature-learning](http://www.slideshare.net/AmgadMuhammad/unsupervised-feature-learning)

- Autoencoders

    - These are neural networks whose weights are trained (using backpropagation)
      to reproduce the input in the output (the weights are trained so that the
      final network is a good representation of the input)

    - A sparsity condition, enforced with the Kullback-Leibler divergence is
      used to constrain the activity of the hidden neurons

- Principal Component Analysis

- Whitening

    - Scale features so that they all have unit variance

- Self-Taught Learning and Unsupervised Feature Learning

    - Train autoencoder on unlabeled data to get condensed representation of
      the data

    - Condensed because the number of hidden nodes is chosen to be less than
      the number of input (and equally output) nodes?

    - Feed labeled data to trained autoencoder and replace input with
      activation of hidden nodes - this condenses our 
      representation of the input
      to the (fewer) hidden neurons

