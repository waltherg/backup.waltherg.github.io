---
layout: default
title: "Citation Counts in PLoS Biology Articles"
date: 2014-01-19
tags: plos nltk
---

# Citation Counts in PLoS Biology Articles

As in a number of earlier blog posts, I will play around with just over 1,700
PLoS Biology articles that I downloaded last year.

In this blog post I will attempt to tackle a question that I have been curious
about for a long time now:
How often are references cited in the main text of scientific articles? That is,
what is the citation count of
each reference listed in the bibliography of peer-reviewed articles?

This question touches upon
[the observation](http://lemire.me/blog/archives/2013/11/18/not-all-citations-are-equal-identifying-key-citations-automatically/)
that many references cited in a given article are rather shallow, i.e. the
citing the author
may not cite certain references for their ingenious influence on their study but
rather for any number of alternate reasons.

My current approach to this question is to use [regular expressions](https://en.wikipedia.org/wiki/Regular_expression) in an
attempt to detect as many citations in a given article as possible.

As I am rather unfamiliar with regular expressions and citation typography is
not uniform across all articles in my
data set I am certain that the citation counts I compute are imperfect.

Therefore, please remember that the results graphed at the bottom should be
interpreted as approximations.

**I would be extremely grateful for any improvements to my code that you may be
able to contribute for better data collection.**
(Just get in touch if you would like to contribute)

To read the articles I have downloaded, I will use my own [helper library](https://github.com/waltherg/PLoSPy).

    import plospy
    import os
    
    from nltk.tokenize import sent_tokenize
    
    from matplotlib import pyplot
    %matplotlib inline
    
    from nltk.tag import RegexpTagger, BigramTagger, UnigramTagger
    from nltk.corpus import brown
    import nltk
    
    import re
    from nltk import RegexpTokenizer
    
    from collections import Counter
    
    import numpy


    all_names = [name for name in os.listdir('../plos/plos_biology/plos_biology_data') if '.dat' in name]


    article_bodies = {}
    article_titles = {}
    
    for name_i, name in enumerate(all_names):
        docs = plospy.PlosXml('../plos/plos_biology/plos_biology_data/'+name)
        for article in docs.docs:
            article_bodies[article['id']] = article['body']
            article_titles[article['id']] = article['title']

Variable `article_bodies` now contains the main texts of the articles in our
data set.

Below, I will use `RegexpTokenizer` instances to capture different styles of
citations
(in hindsight, I probably could have gotten away with just using `re.findall`
instead of `RegexpTokenizer`).

The logic in the code below is that some articles use numeric citations such as
*[1]* and some others use named citations
such as *Name et al. 2014*.
There, of course, variations of these such as ranges and series in the numeric
style, e.g. *[1 - 10]* and *[1,2,3,4]* respectively,
and single authors and author pairs in the named style, e.g. *Name 2014* and
*Name and Nombre 2014* respectively.

The code snippet below attempts to take care of these variations to the best of
my current regex mastery and available spare time.


    tokenizers = [('single', RegexpTokenizer(r'\[\s*\d+\s*\]')),
                  ('range', RegexpTokenizer(r'\[\s*(\d+)\s*(–|-|\u2013)\s*(\d+)\s*\]')),
                  ('name-et-al-year', RegexpTokenizer(r'[a-zA-Z]+\set al.\s19\d{2}|[a-zA-Z]+\set al.\s20\d{2}')),
                  ('name-year', RegexpTokenizer(r'[a-zA-Z]+\s19\d{2}|[a-zA-Z]+\s20\d{2}')),
                  ('name-and-name-year', RegexpTokenizer(r'[a-zA-Z]+\sand\s[a-zA-Z]+\s19\d{2}|[a-zA-Z]+\sand\s[a-zA-Z]+\s20\d{2}')),
                  ('series', RegexpTokenizer(r'\[\s*\d+\s*(,\s*\d+\s*)+\]'))]
    
    citations = {}
    
    for doi_i, doi in enumerate(article_bodies.keys()):
        body = article_bodies[doi]
        citations[doi] = {}
        citations[doi]['named'] = []
        citations[doi]['indexed'] = []
        for (name, tokenizer) in tokenizers:
                tokens = tokenizer.tokenize(body.encode('utf8'))
                for token in tokens:
                    if name == 'single':
                        [index] = re.findall(r'\d+', token)
                        index = int(index)
                        if index < 1:
                            continue
                        citations[doi]['indexed'].append(index)
                    elif name == 'name-year':
                        if 'al.' in token:
                            continue
                        elif token[0] == ',':
                            continue
                        elif token[:2].lower() == 'in':
                            continue
                        elif token[0].lower() == 'a':
                            continue
                        else:
                            citations[doi]['named'].append(token)
                    elif name == 'name-et-al-year' or name == 'name-and-name-year':
                        citations[doi]['named'].append(token)
                    elif name == 'range':
                        try:
                            dummy = re.findall(r'\d+', token)
                            start = dummy[0]
                            end = dummy[-1]
                        except:
                            print 'doi_i', doi_i
                            print token
                            raise('Error')
                        start = int(start)
                        end = int(end)
                        citations[doi]['indexed'] += [i for i in range(start, end+1)]
                    elif name == 'series':
                        if token.strip() == '[0,1]':
                            continue
                        indeces = re.findall(r'\d+', token)
                        citations[doi]['indexed'] += [int(index) for index in indeces]


    for doi in citations:
        if len(citations[doi]['indexed']) > len(citations[doi]['named']):
            citations[doi]['list'] = citations[doi]['indexed']
            citations[doi]['type'] = 'indexed'
            start = min(citations[doi]['indexed'])
            end = max(citations[doi]['indexed'])
            citations[doi]['counter'] = Counter({i: 0 for i in range(start, end+1)})
            citations[doi]['counter'].update(citations[doi]['indexed'])
            
            if start != 1 or len(citations[doi]['counter'].keys()) > 200:
                citations[doi]['ignore'] = True
            else:
                citations[doi]['ignore'] = False
    
        else:
            citations[doi]['list'] = citations[doi]['named']
            citations[doi]['type'] = 'named'
            citations[doi]['counter'] = Counter(citations[doi]['named'])
            
            if len(citations[doi]['counter'].keys()) > 200:
                citations[doi]['ignore'] = True
            else:
                citations[doi]['ignore'] = False

Some citations are rather hard to parse with regular expressions and so we will
ignore those articles whose
citation style appears to use numbered references and we were unable to detect
reference *1*.
We will further ignore those articles that we think contain a ridiculous number
of references (here, we will use a cutoff of 200
items in the bibliography).
We should likely think about other ways that our parsing can fail.

Hence, we will ignore this many articles out of our total set of articles:

    sum([1 for doi in citations if citations[doi]['ignore']])

    25

Let us take a look at how many unique references (i.e. the number of items in
the bibliography of the article) we detect for each article that we do not
ignore:

    pyplot.scatter([i for i, doi in enumerate(citations) if not citations[doi]['ignore']],
                   [len(citations[doi]['counter'].keys()) for doi in citations if not citations[doi]['ignore']])

    <matplotlib.collections.PathCollection at 0xb762c10>

![png]({{site.imgbase}}/2014-01-19-PLoS_Biology_Parse_Citations_files/2014-01-19-PLoS_Biology_Parse_Citations_12_1.png)

[Daniel Lemire](http://lemire.me/blog/archives/2013/11/18/not-all-citations-are-equal-identifying-key-citations-automatically/)
and coauthors observed an average of 30 references cited per article.

To see if our citation counts make any sense, let us see what average we
observe:

    numpy.mean([len(citations[doi]['counter'].keys()) for doi in citations if not citations[doi]['ignore']])

    58.891266628108731

The mean that we observe is about twice as big as that of Lemire et al. so this
may either be a red flag for
our data collection method or due to the difference in research fields.

Let us now take a look at the greatest number of times a reference is mentioned
in each article:

    pyplot.scatter([i for i, doi in enumerate(citations) if not citations[doi]['ignore']],
                   [citations[doi]['counter'].most_common(1)[0][1] for doi in citations if not citations[doi]['ignore']])

    <matplotlib.collections.PathCollection at 0xb624e50>

![png]({{site.imgbase}}/2014-01-19-PLoS_Biology_Parse_Citations_files/2014-01-19-PLoS_Biology_Parse_Citations_17_1.png)

    for doi in citations:
        if not citations[doi]['ignore'] and citations[doi]['counter'].most_common(1)[0][1] > 30:
            print doi
            print citations[doi]['counter'].most_common(1)

[10.1371/journal.pbio.0040362](http://www.plosbiology.org/article/info%3Adoi%2F10.1371/journal.pbio.0040362)

- [(16, 39)]

[10.1371/journal.pbio.1001546](http://www.plosbiology.org/article/info%3Adoi%2F10.1371/journal.pbio.1001546)

- [(29, 47)]
    
[10.1371/journal.pbio.0050092](http://www.plosbiology.org/article/info%3Adoi%2F10.1371/journal.pbio.0050092)

- [(1, 38)]
    
[10.1371/journal.pbio.1001115](http://www.plosbiology.org/article/info%3Adoi%2F10.1371/journal.pbio.1001115)

- [(13, 32)]
    
[10.1371/journal.pbio.1001126](http://www.plosbiology.org/article/info%3Adoi%2F10.1371/journal.pbio.1001126)
    
- [(41, 32)]
    
[10.1371/journal.pbio.1000132](http://www.plosbiology.org/article/info%3Adoi%2F10.1371/journal.pbio.1000132)
    
- [(1, 42)]
    
[10.1371/journal.pbio.1001524](http://www.plosbiology.org/article/info%3Adoi%2F10.1371/journal.pbio.1001524)
    
- [(18, 34)]
    
[10.1371/journal.pbio.0060072](http://www.plosbiology.org/article/info%3Adoi%2F10.1371/journal.pbio.0060072)
    
- [(17, 42)]

(Read the above output as [(reference number article, number of times this
reference is cited)])

Going through the above articles by hand, it is interesting to see that most of
the counted citations
are actually real: reference 13 in *10.1371/journal.pbio.1001115* is actually
cited 32 times in the main text
and once in the figure captions (the latter are not part of my data set).

However, reference 1 in 10.1371/journal.pbio.1000132 occurs only five times in
the main text while the string *[1]* as part of the table in the
format that I downloaded these articles in occurs many more times.

In the same article, 10.1371/journal.pbio.1000132, the editors seem to have made
use of a different typographical scheme for indicating
ranges of references as in this article we find ranges such as *[1] - [3]* as
opposed to the far more common *[1 - 3]*.

The gist of this digression is that my regular expressions above are not
bulletproof and we will count some references too often
and some other references not often enough.

With this caveat in mind, let us now take a look at the distribution of citation
counts across all PLoS Biology articles
that we do not ignore (see above).

    counts = []
    for doi in citations:
        if not citations[doi]['ignore']:
            counts += citations[doi]['counter'].values()


    n, bins, patches = pyplot.hist(counts, normed=True, bins=range(1,max(counts)))

![png]({{site.imgbase}}/2014-01-19-PLoS_Biology_Parse_Citations_files/2014-01-19-PLoS_Biology_Parse_Citations_21_0.png)

    n, bins, patches = pyplot.hist(counts, normed=True, bins=range(1,10))


![png]({{site.imgbase}}/2014-01-19-PLoS_Biology_Parse_Citations_files/2014-01-19-PLoS_Biology_Parse_Citations_22_0.png)

    n, bins, patches = pyplot.hist(counts, normed=False, bins=range(1,10))

![png]({{site.imgbase}}/2014-01-19-PLoS_Biology_Parse_Citations_files/2014-01-19-PLoS_Biology_Parse_Citations_23_0.png)

The bins in the above histograms
are \[1,2\[, \[2,3\[, etc.
([see this question](https://stackoverflow.com/questions/15177203/how-is-the-pyplot-histogram-bins-interpreted))
so that, by far, the greatest number of times references are cited is once.

However, as mentioned multiple times in this post, I need to go back and
validate my method of data collection better to
have greater trust in this result.
