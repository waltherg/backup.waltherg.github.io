---
layout: default
title: "PLOS Biology Outlier Detection"
date: 2014-03-06
tags: plos nltk svd isomap dimensionality
---

# PLOS Biology Outlier Detection

In this blog post I play with dimensionality reduction techniques SVD and Isomap
to map a corpus of 1,754 PLOS Biology articles from 27,210-dimensional feature
space to 2-dimensional space.

This sort of approach is used oftentimes to estimate which data points are near
each other.
Here, I realized that the data points far away from the bulk discuss (almost)
consistently neurobiological topics.

As far as I am aware PLOS Biology publish articles on all biological topics so
there is probably no editorial factor our observation here.


    %matplotlib inline
    %autosave 10

    import gensim
    import cPickle as pickle
    from sklearn import *
    from sklearn.manifold import Isomap
    import numpy
    from matplotlib import pyplot
    from mpl_toolkits.mplot3d import Axes3D

    articles = pickle.load(open('data/plos_biology_articles_unfurled.list','r'))

    dois = pickle.load(open('data/plos_biology_dois.list','r'))

The first article in this data set looks as follows

    articles[0][:10]

    ['introduction',
     'during',
     '1980s',
     '1990s',
     'methods',
     'molecular',
     'genetics',
     'used',
     'determine',
     'contributions']

And the corresponding DOI of this article is

    dois[0]

    '10.1371/journal.pbio.1000584'



Checking the main text of the above DOI we make certain that the article stored
in `articles[0]` corresponds to the DOI stored in `dois[0]`.

Let us now load the same corpus as in `articles` but already formatted as a
numerical matrix that represents each article (row of the matrix) as a bag of
words.
We generated this corpus and the corresponding dictionary
[earlier](http://georg.io/2014/02/PLOS_Biology_Topics).


    corpus = gensim.corpora.MmCorpus('data/plos_biology_corpus.mm')
    dictionary = dictionary = gensim.corpora.dictionary.Dictionary().load('data/plos_biology.dict')


    corpus_mat = gensim.matutils.corpus2csc(corpus)
    corpus_mat = corpus_mat.T
    print corpus_mat.shape

    (1754, 27210)


## SVD


    svd = decomposition.TruncatedSVD(n_components=2)


    corpus_mat_transform = svd.fit_transform(corpus_mat)


    pyplot.scatter(corpus_mat_transform[:,0], corpus_mat_transform[:,1])
    pyplot.scatter(numpy.median(corpus_mat_transform[:,0]), numpy.median(corpus_mat_transform[:,1]), color='red')

    <matplotlib.collections.PathCollection at 0x39dd4690>

![png]({{imgbase}}/PLOS_Biology_Outlier_Detecton_files/PLOS_Biology_Outlier_Detecton_15_1.png)


### Outliers far Away from Median

As we can see there are a few articles that lie relatively far away from the
bulk of the corpus (denoted by the red disk which marks the median).
Let's focus on some of these:


    corpus_mat_transform[corpus_mat_transform[:,0]>150]

    array([[ 188.00455202,  207.0323185 ],
           [ 173.92204694,  149.59252031],
           [ 153.2889464 ,  215.74459155],
           [ 162.25069234,  102.40518113],
           [ 150.996759  ,  145.03623767]])




    numpy.where(corpus_mat_transform[:,0]>150)

    (array([  35, 1074, 1109, 1371, 1544]),)




    for index in numpy.where(corpus_mat_transform[:,0]>150)[0]:
        print 'http://www.plosbiology.org/article/info:doi/%s' % dois[index]

- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.1001060]
- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.1001657]
- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.1001283]
- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.1000135]
- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.1000373]


The first of these "outliers" in the above reduced space is a *Synopsis*
articles so it may be understandable why that one sticks out.
However, the remaining articles are research articles that all deal with
neurobiological topics - so off-hand it is not obvious to me why these would lie
a bit further away from the bulk of the articles.

### Articles Near Median

For comparison with these outliers, let us take a look at articles near the
median (red disk in the above scatter plot).


    median = (numpy.median(corpus_mat_transform[:,0]), numpy.median(corpus_mat_transform[:,1]))
    print median

    (41.996541953291668, -11.862843898802915)



    distances = numpy.asarray([numpy.linalg.norm(vec) for vec in corpus_mat_transform-median])
    near_median = numpy.where(distances<1.5)
    print near_median

    (array([ 233,  810,  872, 1096, 1353, 1408, 1705]),)



    for index in near_median[0]:
        print 'http://www.plosbiology.org/article/info:doi/%s' % dois[index]

- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.0030233]
- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.0060002]
- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.0050237]
- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.1001513]
- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.0060160]
- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.1000091]
- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.0020367]


Articles near the median discuss topics such as gene expression in *C. elegans*,
development and patterning of the neural plat, and stomata in *Arabidopsis*.

## Isomap

[Isomap](http://www.ncbi.nlm.nih.gov/pubmed/11125149) is another dimensionality
reduction tool that promises to preserve the higher-dimensional shape of your
data cloud better than SVD.


    isomap = Isomap(n_components=2)


    isomap_transformed = isomap.fit_transform(corpus_mat.toarray())


    pyplot.scatter(isomap_transformed[:,0], isomap_transformed[:,1])
    pyplot.scatter(numpy.median(isomap_transformed[:,0]), numpy.median(isomap_transformed[:,1]), color='red')

    <matplotlib.collections.PathCollection at 0x37bc4910>

![png]({{imgbase}}/PLOS_Biology_Outlier_Detecton_files/PLOS_Biology_Outlier_Detecton_32_1.png)


### Outlying Group of Articles with 1st Component > 400

Let us focus on the group of articles on the right in the above plot.


    indeces = numpy.where(isomap_transformed[:,0]>450)
    print indeces
    for index in indeces[0]:
        print 'http://www.plosbiology.org/article/info:doi/%s' % dois[index]

    (array([ 388,  397,  486,  521,  676,  949, 1087, 1109, 1221, 1325, 1427]),)
    
- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.0060246]
- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.0030271]
- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.1001194]
- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.1000479]
- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.0040369]
- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.1001212]
- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.1001506]
- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.1001283]
- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.0060152]
- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.0060037]
- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.1000041]


Clicking through these, we realize that most of these articles deal with
neurobiological topics again - except for articles on insulin resistance and B
cell lymphomas.

### Two Articles in the Top Left Corner


    indeces = numpy.where((isomap_transformed[:,0]<-100) & (isomap_transformed[:,1]>600))
    print indeces
    for index in indeces[0]:
        print 'http://www.plosbiology.org/article/info:doi/%s' % dois[index]

    (array([  35, 1074]),)
    
- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.1001060]
- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.1001657]
    

Also these two outliers deal with neurobiological topics.

### One Article Center Top


    indeces = numpy.where((isomap_transformed[:,0]>100) & (isomap_transformed[:,1]>600))
    print indeces
    for index in indeces[0]:
        print 'http://www.plosbiology.org/article/info:doi/%s' % dois[index]

    (array([421]),)
    
- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.1000230]


And again an article about a neurobiological topic.

### Articles Near the Median


    median = (numpy.median(isomap_transformed[:,0]), numpy.median(isomap_transformed[:,1]))
    print median

    (-22.134749426543411, -4.3751913714736927)



    distances = numpy.asarray([numpy.linalg.norm(vec) for vec in isomap_transformed-median])
    near_median = numpy.where(distances<2)
    print near_median

    (array([  49,  323,  350,  772,  773,  889,  908,  960, 1038, 1158, 1285,
           1309, 1321, 1530]),)

    for index in near_median[0]:
        print 'http://www.plosbiology.org/article/info:doi/%s' % dois[index]

- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.1000607]
- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.0060301]
- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.0060263]
- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.0020379]
- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.0020418]
- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.0050299]
- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.1001082]
- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.1001138]
- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.1001616]
- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.0060114]
- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.0040138]
- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.0040188]
- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.0040156]
- [http://www.plosbiology.org/article/info:doi/10.1371/journal.pbio.1000320]


Articles near the median discuss topics such as stochastic gene expression, DNA
transcription and repair, metabolic symbiosis, T cell differentiation,
chromatin, an RNAi screen for cytokinesis inhibitors, and a study on p53.
