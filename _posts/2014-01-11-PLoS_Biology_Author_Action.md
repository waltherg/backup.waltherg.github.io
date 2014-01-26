---
layout: default
title: "PLoS Biology Author Actions"
date: 2014-01-11
tags: plos nltk
---

# Update: Web Application in Development

I started developing a web application based on the simple steps lined
out in this blog post.
I am entirely new to developing and deploying web applications so any
feedback and criticism is greatly appreciated!

Check out the app: [TLDRMed](http://tldrmed.com).

# Author Actions in PLoS Biology Articles

I have downloaded a set of just over 1,700 PLoS Biology articles and have plaid
a little
with some Python packages to work out
[the ten most similar PLoS Biology
articles](http://georg.io/2013/12/Most_Similar_PLoS_Biology/) and
[the most frequent bigrams in PLoS
Biology](http://georg.io/2014/01/PLoS_Biology_Bigrams).

Here I will play a tiny bit with sentence tokenization and token matching to
work out author actions:
Oftentimes, scientific articles are written in a descriptive way and authors
commonly refer to themselves with *we*.

To parse my set of PLoS Biology XML files I will make use of a small utility
library,
[PLoSPy](https://github.com/waltherg/PLoSPy) that I have started to put
together.


    import plospy
    import os
    
    from nltk.tokenize import sent_tokenize
    
    from matplotlib import pyplot
    %matplotlib inline


    all_names = [name for name in os.listdir('../plos/plos_biology/plos_biology_data') if '.dat' in name]


    article_bodies = {}
    article_titles = {}
    
    for name_i, name in enumerate(all_names):
        docs = plospy.PlosXml('../plos/plos_biology/plos_biology_data/'+name)
        for article in docs.docs:
            article_bodies[article['id']] = article['body']
            article_titles[article['id']] = article['title']

There are 1,754 articles in my data set:


    len(article_bodies)

    1754



Here are the first five DOI's in my data set:


    article_bodies.keys()[:5]

    ['10.1371/journal.pbio.0040216',
     '10.1371/journal.pbio.0040215',
     '10.1371/journal.pbio.0040210',
     '10.1371/journal.pbio.0040229',
     '10.1371/journal.pbio.0040368']



Let us now use one of the standard sentence tokenizers shipped with NLTK:


    sentences = {}
    for doi in article_bodies.keys():
        tokens = sent_tokenize(article_bodies[doi])
        sentences[doi] = []
        for sentence in tokens:
            sentences[doi].append(sentence)

The data structure `sentences` is a dictionary that stores the list of sentences
detected for a given DOI -
where the key to access this list is just the DOI itself.


    len(sentences)

    1754



For the first DOI in our data set, the first four detected sentences are the
following:


    sentences.keys()[0]

    '10.1371/journal.pbio.0040216'




    sentences[sentences.keys()[0]][:4]

    [u'  Introduction  Evolution is expected to occur when selection acts on a trait that has a heritable basis of phenotypic variation.',
     u'Quantitative genetic models allow an evolutionary trajectory to be predicted from the strength of selection and the amount of genetic variance, usually expressed as the heritability, h 2 [ 1 ].',
     u'However, while simple theoretical models assume a constant environment, environmental heterogeneity has long been recognised as an important factor influencing the evolutionary dynamics of fitness-related traits in the wild [ 2 ].',
     u'Specifically, selection can vary considerably from year to year within a population [ 3 , 4 ], and it is increasingly recognised that environmental conditions also influence the heritability on which any response to selection depends [ 5 , 6 ].']



Let us now apply an oversimplified rule to detect author actions of the form "We
....":
All sentences that contain the pronoun *we* (or *We*) are assumed to describe
some author action.


    author_actions = {}
    
    for doi in sentences.keys():
        author_actions[doi] = []
        for sentence in sentences[doi]:
            if any([token == 'we' or token == 'We' for token in sentence.split()]):
                author_actions[doi].append(sentence.split())

The data structure `author_actions` is similar to `sentences` but stores only
those sentences that are assumed to describe
author action.


    len(author_actions)

    1754



Let us now count the number of author action sentences detected per article and
plot a histogram.


    no_author_actions = [len(author_actions[doi]) for doi in author_actions.keys()]


    n, bins, patches = pyplot.hist(no_author_actions, max(no_author_actions)+1, normed=1, facecolor='g', alpha=0.75)
    pyplot.xlabel('Number of Author Actions per Article')
    pyplot.ylabel('Frequency')
    pyplot.show()


![png]({{site.imgbase}}/2014-01-11-PLoS_Biology_Author_Actions_files/2014-01-11-PLoS_Biology_Author_Actions_22_0.png)


This histogram shows that, most frequently, articles will describe somewhere
around 30 to 50 author actions.

Anyone familiar with scientific literature in the life sciences will know that
authors often describe
what their article is intended to present by pointing out what they *have
shown*.

So let us go through a few of the articles in our data set and print out all
those author action sentences
that contain the phrase *have shown*.

As you can see in the sample output below, the phrase *we have shown* is used to
describe both what was shown
in the current article and what has been shown in earlier work by the same
authors.

I wonder to what extent this sort of approach may be used to summarize and
condense scientific articles.


    for doi in author_actions.keys()[:50]:
        if any(['have shown' in ' '.join(sentence) for sentence in author_actions[doi]]):
            print('Title: '+article_titles[doi])
            print('http://www.plosbiology.org/article/info%3Adoi%2F'+doi)
            for sentence in author_actions[doi]:
                if 'have shown' in ' '.join(sentence):
                    print ' '.join(sentence)
            print '\n'

[Use of a Dense Single Nucleotide Polymorphism Map for In Silico Mapping in the Mouse](http://www.plosbiology.org/article/info%3Adoi%2F10.1371/journal.pbio.0020393):

- We have shown that they can be used to identify Mendelian traits and replicate classical QTL associations.
    
    
[Translation Repression in Human Cells by MicroRNA-Induced Gene Silencing Requires RCK/p54](http://www.plosbiology.org/article/info%3Adoi%2F10.1371/journal.pbio.0040210):

- We have shown that depletion of RCK/p54 disrupts P-bodies and disperses the cytoplasmic localization of Ago2.
    
    
[SIRT1 Regulates HIV Transcription via Tat Deacetylation](http://www.plosbiology.org/article/info%3Adoi%2F10.1371/journal.pbio.0030041):

- We and others have shown that Tat is acetylated at lysine 50 by the transcriptional coactivators p300 and human GCN5 (general control of amino acid synthesis 5) [ 7 , 8 , 9 , 10 ].
    
    
[HIV-1 Tat Stimulates Transcription Complex Assembly through Recruitment of TBP in the Absence of TAFs](http://www.plosbiology.org/article/info%3Adoi%2F10.1371/journal.pbio.0030044):

- Tat and P-TEFb Direct Recruitment of TBP in the Absence of TAFs We have shown that transcription from the HIV-1 LTR involves TBP but not TFIID.
- However, we have shown that when tethered to DNA or nascent RNA, P-TEFb can also stimulate TC assembly and dictate the composition of the TC.
    

[Exdpf Is a Key Regulator of Exocrine Pancreas Development Controlled by Retinoic Acid and ptf1a in Zebrafish](http://www.plosbiology.org/article/info%3Adoi%2F10.1371/journal.pbio.0060293):

- We have shown in our previous result that overexpression of exdpf increased exocrine cell number due to overproliferation of these cells ( Figure 5 B, [center]).
- The exdpf Gene Acts Genetically Downstream of RA in Regulating Exocrine Pancreas Development We have shown that exdpf is essential for the exocrine pancreas differentiation and expansion; overexpression of exdpf gene leads to increased exocrine size.
    
    
[USP8 Promotes Smoothened Signaling by Preventing Its Ubiquitination and Changing Its Subcellular Localization](http://www.plosbiology.org/article/info%3Adoi%2F10.1371/journal.pbio.1001238):

- Here, we have shown that Smo accumulates on the cell surface when Shi is inactivated, suggesting that Smo functions through Shi-mediated endocytosis.
    
    
[Hedgehog-Regulated Ubiquitination Controls Smoothened Trafficking and Cell Surface Expression in Drosophila](http://www.plosbiology.org/article/info%3Adoi%2F10.1371/journal.pbio.1001239):

- Using antibody uptake assay in S2 cells, we have shown that Smo reaches the cell surface but quickly internalizes in the absence of Hh and that Hh stimulation diminishes internalized Smo with a concomitant increase in cell surface Smo [13] .
    
    
[Raf Activation Is Regulated by Tyrosine 510 Phosphorylation in Drosophila](http://www.plosbiology.org/article/info%3Adoi%2F10.1371/journal.pbio.0060128):

- We have shown that Src64B Δ17 mutant flies possess reduced Draf kinase activity (see Figure 1 B), which can be attributed to lack of tyrosine phosphorylation of Draf by Src64B (see Figure 1 C).
    
    
[Notch-Deficient Skin Induces a Lethal Systemic B-Lymphoproliferative Disorder by Secreting TSLP, a Sentinel for Epidermal Integrity](http://www.plosbiology.org/article/info%3Adoi%2F10.1371/journal.pbio.0060123):

- With this system, we have shown previously that Notch loss also involves non-cell autonomous alteration in transforming growth factor ß and insulin-like growth factor signaling [ 15 ].
    
    
[Nongenetic Individuality in the Host–Phage Interaction](http://www.plosbiology.org/article/info%3Adoi%2F10.1371/journal.pbio.0060120):

- We have shown that although persister λcI857KnR lysogens can survive the transient heat induction by being in a dormant state, they cannot survive infection, since the effect of the phage comes into action once persisters switch to normal growth.
- Conclusion We have shown that the nongenetic individuality in the exit from stationary phase found in populations that persist intensive antibiotic treatments can dramatically affect the interaction between bacteria and λ-phages.
- However, we have shown that lytic phages are able to infect persistent cells, wait for their switching back to the normally growing state, and then eliminate them by lysis.
    
    
[A Feedback Loop between Dynamin and Actin Recruitment during Clathrin-Mediated Endocytosis](http://www.plosbiology.org/article/info%3Adoi%2F10.1371/journal.pbio.1001302):

- As we have shown here productive CME scission events do occur in the presence of actin-disrupting drugs, similar to previous results [7] , [33] , [46] .

