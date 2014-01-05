---
layout: default
title: "PLoS Biology Bigrams"
date: 2014-01-05
tags: plos nltk
---

# PLoS Biology Bigrams

Here I will use the [Natural Language Toolkit](http://nltk.org/) and a recipe
from
[Python Text Processing with NLTK 2.0 Cookbook](http://www.amazon.com/Python-
Text-Processing-NLTK-Cookbook/dp/1849513600)
to work out the most frequent
[bigrams](https://en.wikipedia.org/wiki/Bigram) in the PLoS Biology articles
that
I downloaded last year and have described in previous posts
[here](http://georg.io/2013/12/Most_Similar_PLoS_Biology/) and
[here](http://georg.io/2013/10/PLoS_Time_to_Publication/).

The amusing twist in this blog post is that the most frequent bigram,
after filtering out [stopwords](https://en.wikipedia.org/wiki/Stop_words),
is **unpublished data**.

As [before](http://georg.io/2013/12/Most_Similar_PLoS_Biology/) I will use a
small [helper library](https://github.com/waltherg/PLoSPy) that I started
putting
together:


    import plospy
    import os

    all_names = [name for name in os.listdir('../plos/plos_biology/plos_biology_data') if '.dat' in name]

    all_names[0:10]

    ['plos_biology_0004.dat',
     'plos_biology_0002.dat',
     'plos_biology_0001.dat',
     'plos_biology_0013.dat',
     'plos_biology_0008.dat',
     'plos_biology_0016.dat',
     'plos_biology_0010.dat',
     'plos_biology_0009.dat',
     'plos_biology_0000.dat',
     'plos_biology_0005.dat']

    article_bodies = []
    
    for name_i, name in enumerate(all_names):
        docs = plospy.PlosXml('../plos/plos_biology/plos_biology_data/'+name)
        for article in docs.docs:
            article_bodies.append(article['body'])

The number of PLoS Biology articles in my dataset is 1,754:


    len(article_bodies)

    1754

Following the recipe in the aforementioned book, I use the following
sentence tokenizer shipped with NLTK to break each article body up into
its individual sentences:

    from nltk.tokenize import sent_tokenize

    sentences = []
    for body in article_bodies:
        tokens = sent_tokenize(body)
        for sentence in tokens:
            sentences.append(sentence)

Separating article bodies into individual sentences is not perfect as can be
seen in
the following extracted sentence where the leading section heading becomes part
of
the sentence:

    sentences[0]

    u' Introduction  During the 1980s and 1990s methods of molecular genetics were used to determine the contributions of individual genes to different developmental processes, such as the segmentation of the Drosophila embryo [1] .'

Let us now break all sentences up into their individual words -- preserving the
order
they appear in:

    from nltk.tokenize import word_tokenize

    word_tokenize(sentences[0])
    
    [u'Introduction',
     u'During',
     u'the',
     u'1980s',
     u'and',
     u'1990s',
     u'methods',
     u'of',
     u'molecular',
     u'genetics',
     u'were',
     u'used',
     u'to',
     u'determine',
     u'the',
     u'contributions',
     u'of',
     u'individual',
     u'genes',
     u'to',
     u'different',
     u'developmental',
     u'processes',
     u',',
     u'such',
     u'as',
     u'the',
     u'segmentation',
     u'of',
     u'the',
     u'Drosophila',
     u'embryo',
     u'[',
     u'1',
     u']',
     u'.']

    words = []
    for sentence in sentences:
        for word in word_tokenize(sentence):
            words.append(word.lower())

And this is the number of words we now have in our data structure:

    len(words)

    16851150

    words[:10]

    [u'introduction',
     u'during',
     u'the',
     u'1980s',
     u'and',
     u'1990s',
     u'methods',
     u'of',
     u'molecular',
     u'genetics']
     
Let us now use the `BigramCollocationFinder` class in NLTK to detect all
bigrams:

    from nltk.collocations import BigramCollocationFinder
    from nltk.metrics import BigramAssocMeasures

    bcf = BigramCollocationFinder.from_words(words)

Without filtering out stopwords (short words and words that convey no important
meaning) the six most frequent bigrams are the following (in order of their
frequency):

    bcf.nbest(BigramAssocMeasures.likelihood_ratio, 6)

    [(u')', u'.'),
     (u'(', u'figure'),
     (u']', u'.'),
     (u'of', u'the'),
     (u'et', u'al'),
     (u'in', u'the')]

We detect readily the common forms of citing sources in scientific articles:
conventionally articles are cited as "... [1]." or "... (1)." - representing two
of the six most frequent bigrams.
The latin phrase *et al* is also no surprise here.

With the following function we can filter stopwords out of our bigrams:

    from nltk.corpus import stopwords
    stopset = set(stopwords.words('english'))
    filter_stops = lambda w: len(w) < 3 or w in stopset

    bcf.apply_word_filter(filter_stops)

After filtering, the six most frequent bigrams are the following:

    bcf.nbest(BigramAssocMeasures.likelihood_ratio, 6)

    [(u'unpublished', u'data'),
     (u'united', u'states'),
     (u'wild', u'type'),
     (u'gene', u'expression'),
     (u'amino', u'acid'),
     (u'scale', u'bar')]

The most frequent bigram is listed first hence the most frequent bigram in the
PLoS Biology articles in my dataset is **unpublished data**.
