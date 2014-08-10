---
layout: default
title: "S2DS Introduction to Statistics"
date: 2014-08-10
tags: s2ds
---

# Introduction to Statistics

[Paul Hewson](https://www.plymouth.ac.uk/staff/phewson) of Plymouth University
gave us an introduction to statistics and R.

The following notes are based on this session.

## Introduction

- Data and summaries thereof are not considered "statistics"
- "Statistics" are rather mathematical models of what generated the observed
  data (explanation) and which can be used to predict more data points
- Generally, regard observed data as a sample from a population that we try
  and make a statement about
- Models are hardly every correct exactly and are usually just a starting point

## Bayesian Inference

- Bayes theorem is provable and solid
- Where it gets interesting is the choice of the prior distribution

## Exercises

- [Exercises in R by Paul Hewson](https://github.com/phewson/RIntro)

## Random Notes

- `logit` is a [link function](https://en.wikipedia.org/wiki/Link_function#Link_function)
- Least squares parameters describe a hyperplane that minimizes the
  $L_2$ error
- Normal QQ plots
- Leverage:
    * is a property of each observed data point
    * is the change in the predicted value for a given observation of
      the independent value caused by moving the corresponding observed
      dependent value up or down
    * [lecture notes on this](http://pages.stern.nyu.edu/~churvich/Undergrad/Handouts2/31-Reg6.pdf)
- Residuals
- p-value is a measure of the evidence against the Null hypothesis
- Statistical testing has not been confirmed well for large `N`
  (i.e. a great number of data points)
    * use a computer to generate data from the assumed model and compare
      model data with your sample to decide whether the model makes sense

