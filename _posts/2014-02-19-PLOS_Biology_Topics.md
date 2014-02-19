---
layout: default
title: "PLOS Biology Topics"
date: 2014-02-19
tags: plos nltk gensim topic
---

# PLOS Biology Topics

Ever wonder what topics are discussed in PLOS Biology articles?
Here I will apply an implementation of [Latent Dirichlet
Allocation](https://en.wikipedia.org/wiki/Latent_Dirichlet_allocation) (LDA)
to a set of 1,754 PLOS Biology articles to work out what a possible collection
of underlying topics could be.

I first read about LDA in [Building Machine Learning Systems with
Python](http://www.packtpub.com/building-machine-learning-systems-with-
python/book)
co-authored by [Luis Coelho](https://twitter.com/luispedrocoelho).

LDA seems to have been first described by [Blei *et
al.*](http://jmlr.org/papers/volume3/blei03a/blei03a.pdf) and
I will use the implementation provided by
[gensim](http://radimrehurek.com/gensim/wiki.html#latent-dirichlet-allocation)
which was written by
[Radim Řehůřek](https://twitter.com/RadimRehurek).


    import gensim

    import plospy
    import os

    import nltk

    import cPickle as pickle

With the following lines of code we open, parse, and tokenize all 1,754 PLOS
Biology articles in our collection.

As this takes a bit of time and memory, I carried out all of these steps once
and stored the resulting data structures to my hard disk for later reuse - see
further below.

    all_names = [name for name in os.listdir('../plos/plos_biology/plos_biology_data') if '.dat' in name]

    article_bodies = []
    
    for name_i, name in enumerate(all_names):
        docs = plospy.PlosXml('../plos/plos_biology/plos_biology_data/'+name)
        for article in docs.docs:
            article_bodies.append(article['body'])

We have 1,754 PLOS Biology articles in our collection:

    len(article_bodies)

    1754




    punkt_param = nltk.tokenize.punkt.PunktParameters()
    punkt_param.abbrev_types = set(['et al', 'i.e', 'e.g', 'ref', 'c.f',
                                    'fig', 'Fig', 'Eq', 'eq', 'eqn', 'Eqn',
                                    'dr', 'Dr'])
    sentence_splitter = nltk.tokenize.punkt.PunktSentenceTokenizer(punkt_param)


    sentences = []
    for body in article_bodies:
        sentences.append(sentence_splitter.tokenize(body))


    articles = []
    for body in sentences:
        this_article = []
        for sentence in body:
            this_article.append(nltk.tokenize.word_tokenize(sentence))
        articles.append(this_article)


    pickle.dump(articles, open('plos_biology_articles_tokenized.list', 'w'))


    articles = pickle.load(open('plos_biology_articles_tokenized.list', 'r'))


    is_stopword = lambda w: len(w) < 4 or w in nltk.corpus.stopwords.words('english')

Save each article as one list of tokens and filter out stopwords:

    articles_unfurled = []
    for article in articles:
        this_article = []
        for sentence in article:
            this_article += [token.lower().encode('utf-8') for token in sentence if not is_stopword(token)]
        articles_unfurled.append(this_article)


    pickle.dump(articles_unfurled, open('plos_biology_articles_unfurled.list', 'w'))


    dictionary = gensim.corpora.Dictionary(articles_unfurled)


    dictionary.save('plos_biology.dict')


    corpus = [dictionary.doc2bow(article) for article in articles_unfurled]


    gensim.corpora.MmCorpus.serialize('plos_biology_corpus.mm', corpus)


    model = gensim.models.ldamodel.LdaModel(corpus, id2word=dictionary, update_every=1, chunksize=100, passes=2, num_topics=20)

And these are the twenty topics we find in 1,754 PLOS Biology articles:

    for topic_i, topic in enumerate(model.print_topics(20)):
        print('topic # %d: %s\n' % (topic_i+1, topic))

    topic # 1: 0.026*genes + 0.015*gene + 0.008*sequence + 0.008*2003 + 0.007*sequences + 0.007*human + 0.007*2002 + 0.006*using + 0.006*table + 0.006*identified
    
    topic # 2: 0.011*species + 0.006*data + 0.005*selection + 0.005*figure + 0.005*table + 0.005*population + 0.005*number + 0.004*using + 0.004*used + 0.004*model
    
    topic # 3: 0.023*expression + 0.021*genes + 0.014*gene + 0.013*figure + 0.012*transcription + 0.007*promoter + 0.007*transcriptional + 0.007*levels + 0.005*regions + 0.005*chromatin
    
    topic # 4: 0.066*cells + 0.024*cell + 0.013*figure + 0.006*cohesin + 0.003*shown + 0.003*using + 0.003*data + 0.003*mitotic + 0.003*also + 0.003*experiments
    
    topic # 5: 0.012*figure + 0.009*channels + 0.009*channel + 0.008*current + 0.007*calcium + 0.007*membrane + 0.007*retinal + 0.006*potential + 0.006*currents + 0.006*synaptic
    
    topic # 6: 0.016*figure + 0.015*mrna + 0.011*protein + 0.009*proteins + 0.008*cells + 0.006*translation + 0.006*nuclear + 0.005*mrnas + 0.005*splicing + 0.004*complex
    
    topic # 7: 0.038*mice + 0.012*figure + 0.008*levels + 0.007*animals + 0.006*expression + 0.005*control + 0.005*mouse + 0.005*increased + 0.005*dj-1 + 0.004*insulin
    
    topic # 8: 0.012*figure + 0.007*model + 0.007*force + 0.007*rate + 0.006*state + 0.004*time + 0.004*data + 0.004*distance + 0.004*this + 0.004*movement
    
    topic # 9: 0.023*cells + 0.016*figure + 0.010*protein + 0.009*activity + 0.007*phosphorylation + 0.006*activation + 0.006*cell + 0.006*proteins + 0.005*using + 0.005*transfected
    
    topic # 10: 0.014*memory + 0.011*sleep + 0.009*learning + 0.008*hippocampal + 0.008*group + 0.007*rats + 0.006*olfactory + 0.006*hippocampus + 0.006*odor + 0.006*training
    
    topic # 11: 0.014*figure + 0.011*model + 0.006*time + 0.005*cell + 0.005*network + 0.005*this + 0.005*data + 0.005*different + 0.004*rate + 0.004*number
    
    topic # 12: 0.012*figure + 0.010*protein + 0.010*binding + 0.009*structure + 0.009*residues + 0.008*domain + 0.008*proteins + 0.005*using + 0.005*complex + 0.005*shown
    
    topic # 13: 0.022*neurons + 0.015*figure + 0.007*axons + 0.006*mutants + 0.006*expression + 0.006*synaptic + 0.005*axon + 0.005*spastin + 0.005*mutant + 0.005*flies
    
    topic # 14: 0.013*strains + 0.012*strain + 0.009*figure + 0.007*growth + 0.005*mutants + 0.005*cells + 0.005*mutant + 0.005*yeast + 0.004*grown + 0.004*infection
    
    topic # 15: 0.014*binding + 0.013*sites + 0.010*target + 0.008*sequence + 0.008*figure + 0.007*sequences + 0.007*site + 0.007*motif + 0.006*targets + 0.006*gene
    
    topic # 16: 0.016*figure + 0.013*membrane + 0.009*cells + 0.009*fluorescence + 0.008*cell + 0.006*actin + 0.006*images + 0.006*proteins + 0.005*formation + 0.004*vesicles
    
    topic # 17: 0.040*cells + 0.021*expression + 0.018*cell + 0.017*figure + 0.008*differentiation + 0.005*signaling + 0.005*tumor + 0.004*mutant + 0.004*proliferation + 0.004*also
    
    topic # 18: 0.016*plants + 0.010*plant + 0.008*figure + 0.008*growth + 0.007*auxin + 0.004*arabidopsis + 0.004*species + 0.004*light + 0.004*effects + 0.004*temperature
    
    topic # 19: 0.019*embryos + 0.014*expression + 0.013*figure + 0.007*development + 0.007*stage + 0.007*embryo + 0.006*zebrafish + 0.006*mutants + 0.006*animals + 0.005*wild-type
    
    topic # 20: 0.011*figure + 0.008*activity + 0.007*stimulus + 0.006*time + 0.006*neurons + 0.006*response + 0.005*stimuli + 0.005*responses + 0.005*firing + 0.004*trials

One lesson to be learned here is that the word *Figure* does not carry much
topical information in PLOS Biology articles - and likely in most scientific
articles in general.
