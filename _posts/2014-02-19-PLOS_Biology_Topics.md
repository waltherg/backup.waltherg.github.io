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
on a set of 1,754 PLOS Biology articles to work out what a possible collection
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


    articles_unfurled = pickle.load(open('plos_biology_articles_unfurled.list', 'r'))

## Dictionary and Corpus Creation

Create a dictionary of all words (tokens) that appear in our collection of PLOS
Biology articles
and create a bag of words object for each article (`doc2bow`).


    dictionary = gensim.corpora.Dictionary(articles_unfurled)


    dictionary.save('plos_biology.dict')


    dictionary = gensim.corpora.dictionary.Dictionary().load('plos_biology.dict')

I noticed that the word *figure* occurs rather frequently in these articles, so
let us exclude this and any other words
that appear in more than half of the articles in this data set ([thanks to
Radim](https://twitter.com/RadimRehurek/status/436136774906044416)
for pointing this out to me).


    dictionary.filter_extremes()


    corpus = [dictionary.doc2bow(article) for article in articles_unfurled]


    gensim.corpora.MmCorpus.serialize('plos_biology_corpus.mm', corpus)


    model = gensim.models.ldamodel.LdaModel(corpus, id2word=dictionary, update_every=1, chunksize=100, passes=2, num_topics=20)

And these are the twenty topics we find in 1,754 PLOS Biology articles:


    for topic_i, topic in enumerate(model.print_topics(20)):
        print('topic # %d: %s\n' % (topic_i+1, topic))

    topic # 1: 0.014*host + 0.010*bacterial + 0.009*bacteria + 0.007*infection + 0.007*strain + 0.006*strains + 0.006*plant + 0.005*tree + 0.005*phylogenetic + 0.005*genome
    
    topic # 2: 0.015*memory + 0.013*sleep + 0.013*trials + 0.012*learning + 0.012*task + 0.008*participants + 0.008*performance + 0.008*brain + 0.008*trial + 0.007*behavioral
    
    topic # 3: 0.021*signaling + 0.018*mutant + 0.012*flies + 0.011*pathway + 0.008*drosophila + 0.007*overexpression + 0.006*receptor + 0.006*expressing + 0.005*mutants + 0.005*staining
    
    topic # 4: 0.012*stimulus + 0.012*neurons + 0.009*responses + 0.009*stimuli + 0.009*firing + 0.006*visual + 0.006*frequency + 0.005*cortex + 0.005*trials + 0.005*location
    
    topic # 5: 0.027*membrane + 0.026*syntaxin + 0.022*tiles + 0.021*vesicles + 0.020*vesicle + 0.017*recruitment + 0.017*fusion + 0.017*endocytosis + 0.012*snare + 0.011*endosomes
    
    topic # 6: 0.014*domain + 0.013*residues + 0.007*structures + 0.007*domains + 0.007*structural + 0.004*surface + 0.004*peptide + 0.004*residue + 0.004*crystal + 0.004*conformation
    
    topic # 7: 0.010*population + 0.007*populations + 0.005*variation + 0.005*selection + 0.004*density + 0.003*rates + 0.003*fitness + 0.003*estimates + 0.003*estimated + 0.003*individuals
    
    topic # 8: 0.015*membrane + 0.012*images + 0.011*actin + 0.011*fluorescence + 0.008*microscopy + 0.008*video + 0.008*localization + 0.007*surface + 0.007*migration + 0.007*image
    
    topic # 9: 0.053*mice + 0.010*animals + 0.008*glucose + 0.008*insulin + 0.008*treatment + 0.007*mouse + 0.007*mitochondrial + 0.005*muscle + 0.005*release + 0.005*cholesterol
    
    topic # 10: 0.016*infection + 0.015*mice + 0.011*virus + 0.010*infected + 0.010*viral + 0.008*immune + 0.008*t-cells + 0.006*c57bl/6 + 0.006*treg + 0.005*thymocytes
    
    topic # 11: 0.009*phosphorylation + 0.007*buffer + 0.006*vitro + 0.006*transfected + 0.006*assay + 0.005*kinase + 0.005*antibodies + 0.005*treated + 0.005*lane + 0.005*vivo
    
    topic # 12: 0.017*circadian + 0.016*phase + 0.015*auxin + 0.014*clock + 0.014*period + 0.012*cycle + 0.010*temperature + 0.009*plants + 0.008*dark + 0.008*oscillations
    
    topic # 13: 0.015*genome + 0.009*recombination + 0.008*selection + 0.006*genomic + 0.006*chromosome + 0.005*mutations + 0.005*evolution + 0.005*divergence + 0.005*clusters + 0.004*conserved
    
    topic # 14: 0.021*chromosome + 0.012*cohesin + 0.011*methylation + 0.011*chromosomes + 0.009*mirna + 0.008*mitotic + 0.007*male + 0.006*males + 0.006*mirnas + 0.006*melanogaster
    
    topic # 15: 0.016*mice + 0.016*differentiation + 0.011*tumor + 0.009*proliferation + 0.008*tumors + 0.008*stem + 0.008*adult + 0.007*mouse + 0.006*hscs + 0.006*cancer
    
    topic # 16: 0.027*mutants + 0.022*animals + 0.018*rnai + 0.017*elegans + 0.014*mutant + 0.013*larvae + 0.010*worms + 0.008*phenotype + 0.007*pathway + 0.006*:gfp
    
    topic # 17: 0.024*transcription + 0.013*promoter + 0.012*transcriptional + 0.010*mrna + 0.009*targets + 0.008*chip + 0.008*promoters + 0.007*chromatin + 0.006*regulation + 0.006*regulatory
    
    topic # 18: 0.024*neurons + 0.020*embryos + 0.009*axons + 0.008*dorsal + 0.007*synaptic + 0.007*axon + 0.006*stage + 0.006*embryo + 0.006*ventral + 0.006*muscle
    
    topic # 19: 0.019*strains + 0.019*mrna + 0.017*strain + 0.012*motif + 0.009*splicing + 0.009*yeast + 0.009*exon + 0.007*deletion + 0.007*mutant + 0.007*motifs
    
    topic # 20: 0.014*state + 0.012*network + 0.007*fluorescence + 0.006*networks + 0.006*dynamics + 0.006*parameters + 0.006*force + 0.005*feedback + 0.005*constant + 0.005*rates

Let us visualize these topics as color-coded bubbles ... see at the bottom of this page.

<style>

text {
  font: 12px sans-serif;
}

</style>

<script src="//cdnjs.cloudflare.com/ajax/libs/d3/3.4.1/d3.min.js">

</script>

<div id="#bubbles">
<script type="text/javascript">

var root = {
    "name": "plos",
    "children": [
        {
            "name": "topic_1",
            "children": [
                {
                    "name": "state",
                    "size": 0.014027120760192772
                },
                {
                    "name": "network",
                    "size": 0.0122754309070018
                },
                {
                    "name": "fluorescence",
                    "size": 0.006981889092962769
                },
                {
                    "name": "networks",
                    "size": 0.006382439821625047
                },
                {
                    "name": "dynamics",
                    "size": 0.006316551037972919
                },
                {
                    "name": "parameters",
                    "size": 0.006256905757980392
                },
                {
                    "name": "force",
                    "size": 0.005952121719927151
                },
                {
                    "name": "feedback",
                    "size": 0.00536348199977566
                },
                {
                    "name": "constant",
                    "size": 0.005134496064507513
                },
                {
                    "name": "rates",
                    "size": 0.004977777363539393
                }
            ]
        },
        {
            "name": "topic_2",
            "children": [
                {
                    "name": "strains",
                    "size": 0.019288758455905716
                },
                {
                    "name": "mrna",
                    "size": 0.019129536696917433
                },
                {
                    "name": "strain",
                    "size": 0.016924902356001123
                },
                {
                    "name": "motif",
                    "size": 0.011603745579452832
                },
                {
                    "name": "splicing",
                    "size": 0.009024192357494382
                },
                {
                    "name": "yeast",
                    "size": 0.008861341088586508
                },
                {
                    "name": "exon",
                    "size": 0.008550919126392876
                },
                {
                    "name": "deletion",
                    "size": 0.007403673089702467
                },
                {
                    "name": "mutant",
                    "size": 0.007052825592147895
                },
                {
                    "name": "motifs",
                    "size": 0.0065137716176034526
                }
            ]
        },
        {
            "name": "topic_3",
            "children": [
                {
                    "name": "neurons",
                    "size": 0.02395536068319632
                },
                {
                    "name": "embryos",
                    "size": 0.01981370990711705
                },
                {
                    "name": "axons",
                    "size": 0.008921504935967595
                },
                {
                    "name": "dorsal",
                    "size": 0.008175120004057982
                },
                {
                    "name": "synaptic",
                    "size": 0.00748537059791328
                },
                {
                    "name": "axon",
                    "size": 0.006559586757619357
                },
                {
                    "name": "stage",
                    "size": 0.005994851655044842
                },
                {
                    "name": "embryo",
                    "size": 0.005962669568252326
                },
                {
                    "name": "ventral",
                    "size": 0.005860402952413935
                },
                {
                    "name": "muscle",
                    "size": 0.005737017161336733
                }
            ]
        },
        {
            "name": "topic_4",
            "children": [
                {
                    "name": "transcription",
                    "size": 0.024016158772247796
                },
                {
                    "name": "promoter",
                    "size": 0.012882150743487628
                },
                {
                    "name": "transcriptional",
                    "size": 0.011537078888604623
                },
                {
                    "name": "mrna",
                    "size": 0.01036319399223423
                },
                {
                    "name": "targets",
                    "size": 0.009418715911967595
                },
                {
                    "name": "chip",
                    "size": 0.007916136780567377
                },
                {
                    "name": "promoters",
                    "size": 0.007636461594742087
                },
                {
                    "name": "chromatin",
                    "size": 0.0071839103394654566
                },
                {
                    "name": "regulation",
                    "size": 0.006463803113978664
                },
                {
                    "name": "regulatory",
                    "size": 0.005506891106780016
                }
            ]
        },
        {
            "name": "topic_5",
            "children": [
                {
                    "name": "mutants",
                    "size": 0.02694340100758273
                },
                {
                    "name": "animals",
                    "size": 0.022416005729422414
                },
                {
                    "name": "rnai",
                    "size": 0.01761314211998418
                },
                {
                    "name": "elegans",
                    "size": 0.017327010580101554
                },
                {
                    "name": "mutant",
                    "size": 0.013695650887971204
                },
                {
                    "name": "larvae",
                    "size": 0.012560461966238768
                },
                {
                    "name": "worms",
                    "size": 0.009716990112368299
                },
                {
                    "name": "phenotype",
                    "size": 0.008252133032263088
                },
                {
                    "name": "pathway",
                    "size": 0.006905423012653531
                },
                {
                    "name": ":gfp",
                    "size": 0.006424069978752098
                }
            ]
        },
        {
            "name": "topic_6",
            "children": [
                {
                    "name": "mice",
                    "size": 0.015986624967884888
                },
                {
                    "name": "differentiation",
                    "size": 0.01578319744647191
                },
                {
                    "name": "tumor",
                    "size": 0.011129841970479714
                },
                {
                    "name": "proliferation",
                    "size": 0.008850783839291899
                },
                {
                    "name": "tumors",
                    "size": 0.008338430316677544
                },
                {
                    "name": "stem",
                    "size": 0.007795627844288841
                },
                {
                    "name": "adult",
                    "size": 0.007607605343098633
                },
                {
                    "name": "mouse",
                    "size": 0.007176346158845881
                },
                {
                    "name": "hscs",
                    "size": 0.00634557839682026
                },
                {
                    "name": "cancer",
                    "size": 0.0058015537199410175
                }
            ]
        },
        {
            "name": "topic_7",
            "children": [
                {
                    "name": "chromosome",
                    "size": 0.020856658836037874
                },
                {
                    "name": "cohesin",
                    "size": 0.011789525715462948
                },
                {
                    "name": "methylation",
                    "size": 0.01094199534383595
                },
                {
                    "name": "chromosomes",
                    "size": 0.010795679979173417
                },
                {
                    "name": "mirna",
                    "size": 0.009008075983612352
                },
                {
                    "name": "mitotic",
                    "size": 0.007775613770633192
                },
                {
                    "name": "male",
                    "size": 0.006541032766686175
                },
                {
                    "name": "males",
                    "size": 0.006448340563110978
                },
                {
                    "name": "mirnas",
                    "size": 0.006401380605834032
                },
                {
                    "name": "melanogaster",
                    "size": 0.006369243001043487
                }
            ]
        },
        {
            "name": "topic_8",
            "children": [
                {
                    "name": "genome",
                    "size": 0.014552428692555635
                },
                {
                    "name": "recombination",
                    "size": 0.008641847504196783
                },
                {
                    "name": "selection",
                    "size": 0.007583647060487307
                },
                {
                    "name": "genomic",
                    "size": 0.006264977575705211
                },
                {
                    "name": "chromosome",
                    "size": 0.0058167958510554904
                },
                {
                    "name": "mutations",
                    "size": 0.005293374504304896
                },
                {
                    "name": "evolution",
                    "size": 0.004710394109379034
                },
                {
                    "name": "divergence",
                    "size": 0.004667586256545594
                },
                {
                    "name": "clusters",
                    "size": 0.004652664928396874
                },
                {
                    "name": "conserved",
                    "size": 0.004264062129037349
                }
            ]
        },
        {
            "name": "topic_9",
            "children": [
                {
                    "name": "circadian",
                    "size": 0.017020000751596116
                },
                {
                    "name": "phase",
                    "size": 0.01567711437481348
                },
                {
                    "name": "auxin",
                    "size": 0.01474776967708952
                },
                {
                    "name": "clock",
                    "size": 0.014262150910979375
                },
                {
                    "name": "period",
                    "size": 0.013961062172626599
                },
                {
                    "name": "cycle",
                    "size": 0.012457610463457619
                },
                {
                    "name": "temperature",
                    "size": 0.00965987369422551
                },
                {
                    "name": "plants",
                    "size": 0.009250682326298686
                },
                {
                    "name": "dark",
                    "size": 0.008462153211301962
                },
                {
                    "name": "oscillations",
                    "size": 0.008225885205939008
                }
            ]
        },
        {
            "name": "topic_10",
            "children": [
                {
                    "name": "phosphorylation",
                    "size": 0.008561752855788902
                },
                {
                    "name": "buffer",
                    "size": 0.007142189167434952
                },
                {
                    "name": "vitro",
                    "size": 0.0064887813189542
                },
                {
                    "name": "transfected",
                    "size": 0.005699865941585294
                },
                {
                    "name": "assay",
                    "size": 0.0056581393931337265
                },
                {
                    "name": "kinase",
                    "size": 0.005401089484901424
                },
                {
                    "name": "antibodies",
                    "size": 0.00522535720822415
                },
                {
                    "name": "treated",
                    "size": 0.004947079473642323
                },
                {
                    "name": "lane",
                    "size": 0.004927914880974579
                },
                {
                    "name": "vivo",
                    "size": 0.00488704347588705
                }
            ]
        },
        {
            "name": "topic_11",
            "children": [
                {
                    "name": "infection",
                    "size": 0.01577266666974835
                },
                {
                    "name": "mice",
                    "size": 0.015361720289496545
                },
                {
                    "name": "virus",
                    "size": 0.011007241525944013
                },
                {
                    "name": "infected",
                    "size": 0.010457767374836181
                },
                {
                    "name": "viral",
                    "size": 0.009623267229053416
                },
                {
                    "name": "immune",
                    "size": 0.008151419939990936
                },
                {
                    "name": "t-cells",
                    "size": 0.007903491474715888
                },
                {
                    "name": "c57bl/6",
                    "size": 0.006157590821503989
                },
                {
                    "name": "treg",
                    "size": 0.0057852493316896585
                },
                {
                    "name": "thymocytes",
                    "size": 0.005382845291039275
                }
            ]
        },
        {
            "name": "topic_12",
            "children": [
                {
                    "name": "mice",
                    "size": 0.0526794404550929
                },
                {
                    "name": "animals",
                    "size": 0.00958093288121088
                },
                {
                    "name": "glucose",
                    "size": 0.008250963000271942
                },
                {
                    "name": "insulin",
                    "size": 0.008145911804443715
                },
                {
                    "name": "treatment",
                    "size": 0.007516761593140702
                },
                {
                    "name": "mouse",
                    "size": 0.007410167814008957
                },
                {
                    "name": "mitochondrial",
                    "size": 0.0073920587440700165
                },
                {
                    "name": "muscle",
                    "size": 0.005277263226406868
                },
                {
                    "name": "release",
                    "size": 0.005246088035204043
                },
                {
                    "name": "cholesterol",
                    "size": 0.005011236509993229
                }
            ]
        },
        {
            "name": "topic_13",
            "children": [
                {
                    "name": "membrane",
                    "size": 0.015184017447007824
                },
                {
                    "name": "images",
                    "size": 0.012172087924975981
                },
                {
                    "name": "actin",
                    "size": 0.011256938516447932
                },
                {
                    "name": "fluorescence",
                    "size": 0.010942907809257687
                },
                {
                    "name": "microscopy",
                    "size": 0.007739957194264501
                },
                {
                    "name": "video",
                    "size": 0.007716775541748253
                },
                {
                    "name": "localization",
                    "size": 0.007597669806112658
                },
                {
                    "name": "surface",
                    "size": 0.007346784325209209
                },
                {
                    "name": "migration",
                    "size": 0.006561423206679801
                },
                {
                    "name": "image",
                    "size": 0.006533641183371362
                }
            ]
        },
        {
            "name": "topic_14",
            "children": [
                {
                    "name": "population",
                    "size": 0.010150343223103476
                },
                {
                    "name": "populations",
                    "size": 0.007072277913863973
                },
                {
                    "name": "variation",
                    "size": 0.00465454790046441
                },
                {
                    "name": "selection",
                    "size": 0.004500700344937965
                },
                {
                    "name": "density",
                    "size": 0.003915153882702561
                },
                {
                    "name": "rates",
                    "size": 0.00343203503481455
                },
                {
                    "name": "fitness",
                    "size": 0.003286580667362158
                },
                {
                    "name": "estimates",
                    "size": 0.0032753623328261455
                },
                {
                    "name": "estimated",
                    "size": 0.0030765654591539543
                },
                {
                    "name": "individuals",
                    "size": 0.003069874941554179
                }
            ]
        },
        {
            "name": "topic_15",
            "children": [
                {
                    "name": "domain",
                    "size": 0.014231839613620274
                },
                {
                    "name": "residues",
                    "size": 0.01298791943557803
                },
                {
                    "name": "structures",
                    "size": 0.007222981647717879
                },
                {
                    "name": "domains",
                    "size": 0.007044342037705193
                },
                {
                    "name": "structural",
                    "size": 0.006989240970416582
                },
                {
                    "name": "surface",
                    "size": 0.0043223670217884735
                },
                {
                    "name": "peptide",
                    "size": 0.004257577705663562
                },
                {
                    "name": "residue",
                    "size": 0.004109514546214206
                },
                {
                    "name": "crystal",
                    "size": 0.0038185694899268165
                },
                {
                    "name": "conformation",
                    "size": 0.003673887557220687
                }
            ]
        },
        {
            "name": "topic_16",
            "children": [
                {
                    "name": "membrane",
                    "size": 0.02659207486796983
                },
                {
                    "name": "syntaxin",
                    "size": 0.025690650844904815
                },
                {
                    "name": "tiles",
                    "size": 0.021837862752907212
                },
                {
                    "name": "vesicles",
                    "size": 0.02111232701380241
                },
                {
                    "name": "vesicle",
                    "size": 0.019586908406923953
                },
                {
                    "name": "recruitment",
                    "size": 0.01681972829888044
                },
                {
                    "name": "fusion",
                    "size": 0.01680497769353098
                },
                {
                    "name": "endocytosis",
                    "size": 0.016631567035261433
                },
                {
                    "name": "snare",
                    "size": 0.011956395148843538
                },
                {
                    "name": "endosomes",
                    "size": 0.01125477185037516
                }
            ]
        },
        {
            "name": "topic_17",
            "children": [
                {
                    "name": "stimulus",
                    "size": 0.0123412060680319
                },
                {
                    "name": "neurons",
                    "size": 0.012190142614242679
                },
                {
                    "name": "responses",
                    "size": 0.009199600088360203
                },
                {
                    "name": "stimuli",
                    "size": 0.008929838283185675
                },
                {
                    "name": "firing",
                    "size": 0.008678468384180318
                },
                {
                    "name": "visual",
                    "size": 0.006169848515046801
                },
                {
                    "name": "frequency",
                    "size": 0.005553068324853105
                },
                {
                    "name": "cortex",
                    "size": 0.005448263267141143
                },
                {
                    "name": "trials",
                    "size": 0.004933235038498286
                },
                {
                    "name": "location",
                    "size": 0.00475140632230544
                }
            ]
        },
        {
            "name": "topic_18",
            "children": [
                {
                    "name": "signaling",
                    "size": 0.02148768265612107
                },
                {
                    "name": "mutant",
                    "size": 0.01774419480293616
                },
                {
                    "name": "flies",
                    "size": 0.012001528736965764
                },
                {
                    "name": "pathway",
                    "size": 0.010911270191325876
                },
                {
                    "name": "drosophila",
                    "size": 0.007933576809114029
                },
                {
                    "name": "overexpression",
                    "size": 0.007156869156681172
                },
                {
                    "name": "receptor",
                    "size": 0.005824582968084336
                },
                {
                    "name": "expressing",
                    "size": 0.005724910689993441
                },
                {
                    "name": "mutants",
                    "size": 0.005466215145468003
                },
                {
                    "name": "staining",
                    "size": 0.005412321403521954
                }
            ]
        },
        {
            "name": "topic_19",
            "children": [
                {
                    "name": "memory",
                    "size": 0.014690218497751888
                },
                {
                    "name": "sleep",
                    "size": 0.012833179523530445
                },
                {
                    "name": "trials",
                    "size": 0.012715129877910479
                },
                {
                    "name": "learning",
                    "size": 0.01192443513073574
                },
                {
                    "name": "task",
                    "size": 0.011804447187987122
                },
                {
                    "name": "participants",
                    "size": 0.008448286041762116
                },
                {
                    "name": "performance",
                    "size": 0.008310515420201016
                },
                {
                    "name": "brain",
                    "size": 0.008120687855453777
                },
                {
                    "name": "trial",
                    "size": 0.007573599340399715
                },
                {
                    "name": "behavioral",
                    "size": 0.007438183861358561
                }
            ]
        },
        {
            "name": "topic_20",
            "children": [
                {
                    "name": "host",
                    "size": 0.014154065580705285
                },
                {
                    "name": "bacterial",
                    "size": 0.009657752702001539
                },
                {
                    "name": "bacteria",
                    "size": 0.008972041435179276
                },
                {
                    "name": "infection",
                    "size": 0.007183251766963715
                },
                {
                    "name": "strain",
                    "size": 0.006587834273635868
                },
                {
                    "name": "strains",
                    "size": 0.00624765312732708
                },
                {
                    "name": "plant",
                    "size": 0.005570793457917716
                },
                {
                    "name": "tree",
                    "size": 0.005342732391290373
                },
                {
                    "name": "phylogenetic",
                    "size": 0.005336733351555545
                },
                {
                    "name": "genome",
                    "size": 0.005281806812584402
                }
            ]
        }
    ]
};

var diameter = 2000,
    format = d3.format(",d"),
    color = d3.scale.category20c();

var bubble = d3.layout.pack()
    .sort(null)
    .size([diameter, diameter])
    .padding(6.0);

var svg = d3.select("body").append("svg")
    .attr("width", diameter)
    .attr("height", diameter)
    .attr("class", "bubble");

var node = svg.selectAll(".node")
      .data(bubble.nodes(classes(root))
      .filter(function(d) { return !d.children; }))
    .enter().append("g")
      .attr("class", "node")
      .attr("transform", function(d) { return "translate(" + d.x + "," + d.y + ")"; });

  node.append("title")
      .text(function(d) { return d.className + ": " + format(d.value); });

  node.append("circle")
      .attr("r", function(d) { return d.r; })
      .style("fill", function(d) { return color(d.packageName); });

  node.append("text")
      .attr("dy", ".3em")
      .style("text-anchor", "middle")
      .text(function(d) { return d.className.substring(0, d.r / 3); });


// Returns a flattened hierarchy containing all leaf nodes under the root.
function classes(root) {
  var classes = [];

  function recurse(name, node) {
    if (node.children) node.children.forEach(function(child) { recurse(node.name, child); });
    else classes.push({packageName: name, className: node.name, value: node.size});
  }

  recurse(null, root);
  return {children: classes};
}

d3.select(self.frameElement).style("height", diameter + "px");

</script>
</div>

## Topics with Lemmatized Tokens

As we can notice, some of the tokens in the above topics are just singular and
plural forms of the same word.

Let us see what topics we find after lemmatizing all of our tokens.


    from nltk.stem import WordNetLemmatizer
    wnl = WordNetLemmatizer()
    articles_lemmatized = []
    for article in articles_unfurled:
        articles_lemmatized.append([wnl.lemmatize(token) for token in article])

    pickle.dump(articles_lemmatized, open('plos_biology_articles_lemmatized.list', 'w'))

    dictionary_lemmatized = gensim.corpora.Dictionary(articles_lemmatized)

    dictionary_lemmatized.save('plos_biology_lemmatized.dict')

    dictionary_lemmatized.filter_extremes()

    corpus_lemmatized = [dictionary_lemmatized.doc2bow(article) for article in articles_lemmatized]

    gensim.corpora.MmCorpus.serialize('plos_biology_corpus_lemmatized.mm', corpus_lemmatized)

    model_lemmatized = gensim.models.ldamodel.LdaModel(corpus_lemmatized, id2word=dictionary_lemmatized, update_every=1, chunksize=100, passes=2, num_topics=20)

And here are the twenty topics we find with lemmatized tokens:

    for topic_i, topic in enumerate(model_lemmatized.print_topics(20)):
        print('topic # %d: %s\n' % (topic_i+1, topic))

    topic # 1: 0.052*embryo + 0.011*dorsal + 0.011*zebrafish + 0.009*neural + 0.008*anterior + 0.008*ventral + 0.007*cartilage + 0.007*signaling + 0.007*posterior + 0.007*muscle
    
    topic # 2: 0.015*infection + 0.015*elegans + 0.014*host + 0.014*rnai + 0.012*worm + 0.009*parasite + 0.009*larva + 0.009*mosquito + 0.008*adult + 0.008*bacteria
    
    topic # 3: 0.025*promoter + 0.023*transcription + 0.012*transcriptional + 0.011*chromatin + 0.011*chip + 0.010*cohesin + 0.010*methylation + 0.009*nuclear + 0.007*histone + 0.006*ino1
    
    topic # 4: 0.017*microtubule + 0.013*nucleus + 0.011*mitotic + 0.011*insulin + 0.011*glucose + 0.011*spindle + 0.010*nuclear + 0.009*retina + 0.008*mitochondrial + 0.007*retinal
    
    topic # 5: 0.024*stimulus + 0.015*neuron + 0.014*trial + 0.010*firing + 0.007*spike + 0.007*task + 0.007*current + 0.007*frequency + 0.006*recording + 0.006*memory
    
    topic # 6: 0.020*phosphorylation + 0.016*kinase + 0.013*signaling + 0.012*antibody + 0.007*inhibitor + 0.007*receptor + 0.006*phosphorylated + 0.006*transfected + 0.006*infection + 0.006*peptide
    
    topic # 7: 0.061*neuron + 0.031*axon + 0.022*synaptic + 0.013*dendrite + 0.012*brain + 0.011*neuronal + 0.010*synapsis + 0.010*cortical + 0.010*dendritic + 0.008*axonal
    
    topic # 8: 0.023*fly + 0.017*drosophila + 0.012*larva + 0.012*signaling + 0.012*phenotype + 0.010*clone + 0.010*defect + 0.009*rescue + 0.008*disc + 0.008*genotype
    
    topic # 9: 0.010*network + 0.010*dynamic + 0.008*force + 0.007*parameter + 0.007*distance + 0.007*movement + 0.006*simulation + 0.006*video + 0.005*motor + 0.005*direction
    
    topic # 10: 0.019*residue + 0.011*peptide + 0.007*substrate + 0.007*reaction + 0.007*structural + 0.007*enzyme + 0.006*crystal + 0.006*subunit + 0.006*molecule + 0.005*conformation
    
    topic # 11: 0.038*chromosome + 0.025*allele + 0.018*recombination + 0.015*locus + 0.008*hybrid + 0.008*female + 0.008*male + 0.008*marker + 0.007*genomic + 0.007*primer
    
    topic # 12: 0.016*tumor + 0.012*differentiation + 0.010*tissue + 0.008*blood + 0.007*proliferation + 0.007*culture + 0.007*wound + 0.007*liver + 0.006*treatment + 0.006*stem
    
    topic # 13: 0.038*strain + 0.010*yeast + 0.008*plasmid + 0.007*coli + 0.006*grown + 0.006*deletion + 0.006*codon + 0.006*ribosome + 0.006*phage + 0.005*culture
    
    topic # 14: 0.033*fiber + 0.026*receptor + 0.016*aggregation + 0.015*aggregate + 0.008*trkb + 0.007*granule + 0.007*liposome + 0.007*bdnf + 0.006*body + 0.006*signaling
    
    topic # 15: 0.015*genome + 0.009*cluster + 0.006*motif + 0.005*selection + 0.005*dataset + 0.005*2003 + 0.005*family + 0.004*2002 + 0.004*conserved + 0.004*read
    
    topic # 16: 0.007*estimate + 0.007*female + 0.007*male + 0.006*variation + 0.006*selection + 0.006*fitness + 0.005*variable + 0.005*density + 0.005*trait + 0.005*bird
    
    topic # 17: 0.022*membrane + 0.010*antibody + 0.008*fluorescence + 0.008*vesicle + 0.006*surface + 0.006*transfected + 0.006*expressing + 0.006*fusion + 0.005*localization + 0.005*particle
    
    topic # 18: 0.025*plant + 0.019*cycle + 0.014*circadian + 0.013*clock + 0.013*phase + 0.012*auxin + 0.012*period + 0.010*feedback + 0.010*rhythm + 0.010*oscillation
    
    topic # 19: 0.052*mrna + 0.025*transcript + 0.020*exon + 0.011*splicing + 0.011*motif + 0.010*transcription + 0.009*regulation + 0.008*translation + 0.008*association + 0.007*intron
    
    topic # 20: 0.014*subject + 0.014*brain + 0.014*cortex + 0.012*participant + 0.008*task + 0.007*visual + 0.007*word + 0.007*object + 0.005*neural + 0.005*trial
    
