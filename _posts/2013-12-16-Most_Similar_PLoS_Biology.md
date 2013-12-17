---
layout: default
date: 2013-12-16
title: "The Ten Most Similar PLoS Biology Articles"
tags: Python XML OpenData PLoS Sklearn
---

# The Ten Most Similar PLoS Biology Articles

... at least by some measure.

I recently downloaded 1754 PLoS Biology articles as XML files through the PLoS
API
and have looked at the distribution of the [time to
publication](http://georg.io/2013/10/PLoS_Time_to_Publication/)
of PLoS Biology and other PLoS journals.

Here I will play a little with [scikit-learn](http://scikit-learn.org/stable/)
to see if I can discover those
PLoS Biology articles (in my data set) that are most similar to one another.

## Import Packages

I started writing a [Python package
(PLoSPy)](https://github.com/waltherg/PLoSPy) for more convient parsing
of the XML files I have download from PLoS.

    import plospy
    import os
    from sklearn.feature_extraction.text import TfidfVectorizer
    from sklearn.metrics.pairwise import linear_kernel
    import itertools

## Discover Data Files on Hard Disk

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

    print len(all_names)

    18

## Vectorize all Articles

To reduce memory use, I wrote the following method that returns an iterator over
all article bodies.
In passing this iterator to the vectorizer, we avoid loading all articles into
memory at once - despite
the use of an iterator here, I have not been able to repeat this experiment with
all 65,000-odd PLoS ONE
articles without running out of memory.

    ids = []
    titles = []
    
    def get_corpus(all_names):
        for name_i, name in enumerate(all_names):
            docs = plospy.PlosXml('../plos/plos_biology/plos_biology_data/'+name)
            for article in docs.docs:
                ids.append(article['id'])
                titles.append(article['title'])
                yield article['body']

    corpus = get_corpus(all_names)
    tfidf = TfidfVectorizer().fit_transform(corpus)

Just as a sanity check, the number of DOIs in our data set should now equal 1754
as this is the number
of articles [I downloaded in the first
place](http://georg.io/2013/10/PLoS_Time_to_Publication).

    len(ids)

    1754

The vectorizer generated a matrix with 139,748 columns (these are the tokens,
i.e. probably unique words used in
all 1754 PLoS Biology articles) and 1754 rows (corresponding to individual
articles).

    tfidf.shape

    (1754, 139748)

Let us now compute all pairwise cosine distances betweeen all 1754 vectors
(articles) in matrix `tfidf`.
I copied and pasted most of this from a StackOverflow answer that I cannot find
now - I will
add a link to the answer when I come across it again.

To get the ten most similar articles, we track the top five pairwise matches.

    top_five = [[-1,-1,-1] for i in range(5)]
    threshold = -1.
    
    for index in range(len(ids)):
        cosine_similarities = linear_kernel(tfidf[index:index+1], tfidf).flatten()
        related_docs_indices = cosine_similarities.argsort()[:-5:-1]
        
        first = related_docs_indices[0]
        second = related_docs_indices[1]
        
        if first != index:
            print 'Error'
            break
    
        if cosine_similarities[second] > threshold:
            if first not in [top[0] for top in top_five] and first not in [top[1] for top in top_five]:
                scores = [top[2] for top in top_five]
                replace = scores.index(min(scores))
                # print 'replace',replace
                top_five[replace] = [first, second, cosine_similarities[second]]
                # print 'old threshold',threshold
                threshold = min(scores)
                # print 'new threshold',threshold

## The Most Similar Articles

Let us now take a look at the results!

    for tf in top_five:
        print ''
        print('Cosine Similarity: %.2f' % tf[2])
        print('Title 1: %s' %titles[tf[0]])
        print('http://www.plosbiology.org/article/info%3Adoi%2F'+str(ids[tf[0]]))
        print ''
        print('Title 2: %s' %titles[tf[1]])
        print('http://www.plosbiology.org/article/info%3Adoi%2F'+str(ids[tf[1]]))
        print ''

    
**Cosine Similarity: 0.89**
[Title 1: MYRF Is a Membrane-Associated Transcription Factor That Autoproteolytically Cleaves to Directly Activate Myelin Genes](http://www.plosbiology.org/article/info%3Adoi%2F10.1371/journal.pbio.1001625)
    
[Title 2: A Bacteriophage Tailspike Domain Promotes Self-Cleavage of a Human Membrane-Bound Transcription Factor, the Myelin Regulatory Factor MYRF](http://www.plosbiology.org/article/info%3Adoi%2F10.1371/journal.pbio.1001624)
    
    
**Cosine Similarity: 0.92**
[Title 1: Gamma-Secretase Represents a Therapeutic Target for the Treatment of Invasive Glioma Mediated by the p75 Neurotrophin Receptor](http://www.plosbiology.org/article/info%3Adoi%2F10.1371/journal.pbio.0060289)
    
[Title 2: The p75 Neurotrophin Receptor Is a Central Regulator of Glioma Invasion](http://www.plosbiology.org/article/info%3Adoi%2F10.1371/journal.pbio.0050212)
    
    
**Cosine Similarity: 0.86**
[Title 1: Coupling of a Core Post-Translational Pacemaker to a Slave Transcription/Translation Feedback Loop in a Circadian System](http://www.plosbiology.org/article/info%3Adoi%2F10.1371/journal.pbio.1000394)
    
[Title 2: Elucidating the Ticking of an In Vitro Circadian Clockwork](http://www.plosbiology.org/article/info%3Adoi%2F10.1371/journal.pbio.0050093)
    
    
**Cosine Similarity: 0.86**
[Title 1: PICK1 and ICA69 Control Insulin Granule Trafficking and Their Deficiencies Lead to Impaired Glucose Tolerance](http://www.plosbiology.org/article/info%3Adoi%2F10.1371/journal.pbio.1001541)
    
[Title 2: PICK1 Deficiency Impairs Secretory Vesicle Biogenesis and Leads to Growth Retardation and Decreased Glucose Tolerance](http://www.plosbiology.org/article/info%3Adoi%2F10.1371/journal.pbio.1001542)
    
    
**Cosine Similarity: 0.91**
[Title 1: Insights into Mad2 Regulation in the Spindle Checkpoint Revealed by the Crystal Structure of the Symmetric Mad2 Dimer](http://www.plosbiology.org/article/info%3Adoi%2F10.1371/journal.pbio.0060050)
    
[Title 2: The Influence of Catalysis on Mad2 Activation Dynamics](http://www.plosbiology.org/article/info%3Adoi%2F10.1371/journal.pbio.1000010)
    
