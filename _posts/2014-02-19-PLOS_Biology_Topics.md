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

    topic # 1: 0.019*residues + 0.017*domain + 0.008*peptide + 0.007*domains + 0.006*structures + 0.006*residue + 0.006*crystal + 0.006*mutant + 0.005*structural + 0.005*conformation
    
    topic # 2: 0.018*strains + 0.017*strain + 0.011*bacterial + 0.009*bacteria + 0.009*coli + 0.008*host + 0.005*medium + 0.005*albicans + 0.004*enzymes + 0.004*phage
    
    topic # 3: 0.024*transcription + 0.013*transcriptional + 0.012*promoter + 0.011*methylation + 0.010*chip + 0.009*promoters + 0.009*chromatin + 0.007*microarray + 0.006*enriched + 0.006*genomic
    
    topic # 4: 0.025*membrane + 0.021*infection + 0.013*infected + 0.012*virus + 0.012*viral + 0.010*vesicles + 0.009*syntaxin + 0.009*fusion + 0.008*recruitment + 0.007*vesicle
    
    topic # 5: 0.019*memory + 0.015*sleep + 0.013*learning + 0.012*neurons + 0.012*brain + 0.011*animals + 0.011*hippocampal + 0.010*rats + 0.008*trials + 0.007*hippocampus
    
    topic # 6: 0.011*genome + 0.008*chromosome + 0.007*selection + 0.007*recombination + 0.006*2003 + 0.006*2002 + 0.005*genomic + 0.005*alleles + 0.005*mutations + 0.004*evolution
    
    topic # 7: 0.023*neurons + 0.019*synaptic + 0.010*channels + 0.010*cortical + 0.009*channel + 0.008*current + 0.008*synapses + 0.008*membrane + 0.006*cortex + 0.006*microtubules
    
    topic # 8: 0.012*motif + 0.009*network + 0.008*motifs + 0.008*targets + 0.006*networks + 0.006*conserved + 0.005*mirna + 0.005*modules + 0.005*regulatory + 0.005*hairy
    
    topic # 9: 0.021*population + 0.012*populations + 0.007*rates + 0.006*estimates + 0.006*modern + 0.006*density + 0.006*diversity + 0.005*estimated + 0.005*variation + 0.004*selection
    
    topic # 10: 0.010*actin + 0.007*surface + 0.007*force + 0.007*images + 0.007*image + 0.006*density + 0.006*myosin + 0.005*fibers + 0.005*video + 0.005*distance
    
    topic # 11: 0.028*plants + 0.015*plant + 0.015*circadian + 0.013*auxin + 0.012*clock + 0.011*arabidopsis + 0.010*retina + 0.010*retinal + 0.009*period + 0.009*phase
    
    topic # 12: 0.063*mice + 0.009*mouse + 0.009*animals + 0.007*blood + 0.007*insulin + 0.007*liver + 0.006*glucose + 0.006*muscle + 0.006*tissue + 0.005*controls
    
    topic # 13: 0.011*phosphorylation + 0.009*signaling + 0.009*transfected + 0.007*treated + 0.007*treatment + 0.007*fluorescence + 0.006*receptor + 0.006*membrane + 0.006*kinase + 0.006*antibody
    
    topic # 14: 0.009*mrna + 0.006*buffer + 0.006*mutant + 0.005*vitro + 0.005*complexes + 0.004*assay + 0.004*reaction + 0.004*translation + 0.004*domain + 0.004*nuclear
    
    topic # 15: 0.017*rnai + 0.017*mutants + 0.016*animals + 0.015*elegans + 0.014*mrna + 0.011*splicing + 0.011*exon + 0.010*pathway + 0.009*transcripts + 0.009*worms
    
    topic # 16: 0.016*cohesin + 0.016*chromosome + 0.014*nuclear + 0.012*mitotic + 0.011*chromosomes + 0.009*xist + 0.009*cycle + 0.009*division + 0.009*spindle + 0.008*nuclei
    
    topic # 17: 0.021*embryos + 0.011*mutant + 0.011*neurons + 0.009*mutants + 0.007*stage + 0.007*signaling + 0.007*dorsal + 0.007*embryo + 0.007*axons + 0.006*drosophila
    
    topic # 18: 0.014*differentiation + 0.010*tumor + 0.008*proliferation + 0.007*tumors + 0.006*stem + 0.006*mice + 0.006*hscs + 0.006*mouse + 0.005*apoptosis + 0.005*progenitor
    
    topic # 19: 0.014*females + 0.013*males + 0.009*fitness + 0.008*male + 0.008*female + 0.006*food + 0.005*flight + 0.005*host + 0.004*individuals + 0.004*variation
    
    topic # 20: 0.011*stimulus + 0.008*stimuli + 0.008*responses + 0.008*firing + 0.007*neurons + 0.007*trials + 0.006*visual + 0.006*task + 0.005*cortex + 0.005*frequency
    


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
    
<style>

text {
  font: 10px sans-serif;
}

</style>

<script src="//cdnjs.cloudflare.com/ajax/libs/d3/3.4.1/d3.min.js">

</script>

<script>
var root = {'children': [{'children': [{'name': 'state', 'size': 0.014027120760192772},
    {'name': 'network', 'size': 0.0122754309070018},
    {'name': 'fluorescence', 'size': 0.0069818890929627692},
    {'name': 'networks', 'size': 0.0063824398216250468},
    {'name': 'dynamics', 'size': 0.0063165510379729192},
    {'name': 'parameters', 'size': 0.0062569057579803921},
    {'name': 'force', 'size': 0.0059521217199271512},
    {'name': 'feedback', 'size': 0.0053634819997756596},
    {'name': 'constant', 'size': 0.0051344960645075127},
    {'name': 'rates', 'size': 0.0049777773635393931}],
   'name': 'topic_0'},
  {'children': [{'name': 'strains', 'size': 0.019288758455905716},
    {'name': 'mrna', 'size': 0.019129536696917433},
    {'name': 'strain', 'size': 0.016924902356001123},
    {'name': 'motif', 'size': 0.011603745579452832},
    {'name': 'splicing', 'size': 0.0090241923574943816},
    {'name': 'yeast', 'size': 0.0088613410885865079},
    {'name': 'exon', 'size': 0.0085509191263928759},
    {'name': 'deletion', 'size': 0.0074036730897024672},
    {'name': 'mutant', 'size': 0.0070528255921478954},
    {'name': 'motifs', 'size': 0.0065137716176034526}],
   'name': 'topic_1'},
  {'children': [{'name': 'neurons', 'size': 0.023955360683196321},
    {'name': 'embryos', 'size': 0.01981370990711705},
    {'name': 'axons', 'size': 0.0089215049359675953},
    {'name': 'dorsal', 'size': 0.0081751200040579824},
    {'name': 'synaptic', 'size': 0.0074853705979132804},
    {'name': 'axon', 'size': 0.0065595867576193571},
    {'name': 'stage', 'size': 0.005994851655044842},
    {'name': 'embryo', 'size': 0.0059626695682523257},
    {'name': 'ventral', 'size': 0.0058604029524139348},
    {'name': 'muscle', 'size': 0.0057370171613367327}],
   'name': 'topic_2'},
  {'children': [{'name': 'transcription', 'size': 0.024016158772247796},
    {'name': 'promoter', 'size': 0.012882150743487628},
    {'name': 'transcriptional', 'size': 0.011537078888604623},
    {'name': 'mrna', 'size': 0.01036319399223423},
    {'name': 'targets', 'size': 0.0094187159119675946},
    {'name': 'chip', 'size': 0.0079161367805673771},
    {'name': 'promoters', 'size': 0.0076364615947420869},
    {'name': 'chromatin', 'size': 0.0071839103394654566},
    {'name': 'regulation', 'size': 0.0064638031139786639},
    {'name': 'regulatory', 'size': 0.0055068911067800161}],
   'name': 'topic_3'},
  {'children': [{'name': 'mutants', 'size': 0.026943401007582728},
    {'name': 'animals', 'size': 0.022416005729422414},
    {'name': 'rnai', 'size': 0.017613142119984181},
    {'name': 'elegans', 'size': 0.017327010580101554},
    {'name': 'mutant', 'size': 0.013695650887971204},
    {'name': 'larvae', 'size': 0.012560461966238768},
    {'name': 'worms', 'size': 0.0097169901123682987},
    {'name': 'phenotype', 'size': 0.0082521330322630881},
    {'name': 'pathway', 'size': 0.0069054230126535309},
    {'name': ':gfp', 'size': 0.0064240699787520977}],
   'name': 'topic_4'},
  {'children': [{'name': 'mice', 'size': 0.015986624967884888},
    {'name': 'differentiation', 'size': 0.01578319744647191},
    {'name': 'tumor', 'size': 0.011129841970479714},
    {'name': 'proliferation', 'size': 0.0088507838392918991},
    {'name': 'tumors', 'size': 0.0083384303166775444},
    {'name': 'stem', 'size': 0.0077956278442888414},
    {'name': 'adult', 'size': 0.0076076053430986328},
    {'name': 'mouse', 'size': 0.007176346158845881},
    {'name': 'hscs', 'size': 0.00634557839682026},
    {'name': 'cancer', 'size': 0.0058015537199410175}],
   'name': 'topic_5'},
  {'children': [{'name': 'chromosome', 'size': 0.020856658836037874},
    {'name': 'cohesin', 'size': 0.011789525715462948},
    {'name': 'methylation', 'size': 0.010941995343835951},
    {'name': 'chromosomes', 'size': 0.010795679979173417},
    {'name': 'mirna', 'size': 0.0090080759836123521},
    {'name': 'mitotic', 'size': 0.0077756137706331916},
    {'name': 'male', 'size': 0.0065410327666861751},
    {'name': 'males', 'size': 0.0064483405631109783},
    {'name': 'mirnas', 'size': 0.0064013806058340322},
    {'name': 'melanogaster', 'size': 0.0063692430010434866}],
   'name': 'topic_6'},
  {'children': [{'name': 'genome', 'size': 0.014552428692555635},
    {'name': 'recombination', 'size': 0.0086418475041967827},
    {'name': 'selection', 'size': 0.0075836470604873068},
    {'name': 'genomic', 'size': 0.0062649775757052106},
    {'name': 'chromosome', 'size': 0.0058167958510554904},
    {'name': 'mutations', 'size': 0.0052933745043048956},
    {'name': 'evolution', 'size': 0.0047103941093790342},
    {'name': 'divergence', 'size': 0.0046675862565455943},
    {'name': 'clusters', 'size': 0.0046526649283968741},
    {'name': 'conserved', 'size': 0.0042640621290373493}],
   'name': 'topic_7'},
  {'children': [{'name': 'circadian', 'size': 0.017020000751596116},
    {'name': 'phase', 'size': 0.015677114374813481},
    {'name': 'auxin', 'size': 0.014747769677089521},
    {'name': 'clock', 'size': 0.014262150910979375},
    {'name': 'period', 'size': 0.013961062172626599},
    {'name': 'cycle', 'size': 0.012457610463457619},
    {'name': 'temperature', 'size': 0.0096598736942255098},
    {'name': 'plants', 'size': 0.0092506823262986856},
    {'name': 'dark', 'size': 0.0084621532113019621},
    {'name': 'oscillations', 'size': 0.0082258852059390077}],
   'name': 'topic_8'},
  {'children': [{'name': 'phosphorylation', 'size': 0.0085617528557889021},
    {'name': 'buffer', 'size': 0.0071421891674349524},
    {'name': 'vitro', 'size': 0.0064887813189541997},
    {'name': 'transfected', 'size': 0.0056998659415852942},
    {'name': 'assay', 'size': 0.0056581393931337265},
    {'name': 'kinase', 'size': 0.0054010894849014241},
    {'name': 'antibodies', 'size': 0.0052253572082241496},
    {'name': 'treated', 'size': 0.0049470794736423233},
    {'name': 'lane', 'size': 0.0049279148809745791},
    {'name': 'vivo', 'size': 0.00488704347588705}],
   'name': 'topic_9'}],
 'name': 'plos'};
 
var diameter = 960,
    format = d3.format(",d"),
    color = d3.scale.category20c();

var bubble = d3.layout.pack()
    .sort(null)
    .size([diameter, diameter])
    .padding(1.5);

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


d3.select(self.frameElement).style("height", diameter + "px");
</script>
