---
layout: default
title: "Good Reads January 2014"
date: 2014-01-03
tags: links reads
---

# Good Reads January 2014

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
