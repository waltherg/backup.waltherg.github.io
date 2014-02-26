---
layout: default
title: "PLOS Biology Topics Part II"
date: 2014-02-26
tags: plos nltk gensim topic
---

# PLOS Biology Topics Part II

I have received some great suggestions from both
[Radim](https://twitter.com/RadimRehurek/status/436527298951798784)
and [Asim](http://georg.io/2014/02/PLOS_Biology_Topics/#comment-1259524342) that
I wanted to follow up on in this brief blog post.

## The Effect of Changing `passes`

Following on from the unlemmatized dictionary we built in the
[last blog post](http://georg.io/2014/02/PLOS_Biology_Topics#dictionary-and-
corpus-creation)
it is interesting to see how the quality of topics changes when altering the
`passes` parameter of the `LdaModel` object.


    %matplotlib inline
    from matplotlib import pyplot
    import numpy


    import gensim
    import cPickle as pickle


    dictionary = gensim.corpora.dictionary.Dictionary().load('plos_biology.dict')


    dictionary.filter_extremes()


    corpus = gensim.corpora.MmCorpus('plos_biology_corpus.mm')

### Lowering `passes`


    model = gensim.models.ldamodel.LdaModel(corpus, id2word=dictionary, update_every=1, chunksize=100, passes=1, num_topics=20)


    for topic_i, topic in enumerate(model.print_topics(20)):
        print('topic # %d: %s\n' % (topic_i+1, topic))

    topic # 1: 0.008*genome + 0.006*2003 + 0.006*2002 + 0.006*selection + 0.005*population + 0.005*chromosome + 0.004*recombination + 0.004*genomic + 0.004*variation + 0.004*2001
    
    topic # 2: 0.019*tumor + 0.014*tumors + 0.012*damage + 0.010*mitotic + 0.009*survival + 0.009*cancer + 0.009*death + 0.009*spindle + 0.009*wound + 0.008*cyclin
    
    topic # 3: 0.029*embryos + 0.011*differentiation + 0.010*stage + 0.009*embryo + 0.009*zebrafish + 0.007*cartilage + 0.007*signaling + 0.007*neural + 0.006*anterior + 0.006*dorsal
    
    topic # 4: 0.014*trials + 0.013*cortex + 0.010*memory + 0.010*task + 0.010*location + 0.010*brain + 0.009*sleep + 0.008*responses + 0.007*neurons + 0.007*learning
    
    topic # 5: 0.018*transcription + 0.016*mrna + 0.012*promoter + 0.008*nuclear + 0.008*transcriptional + 0.006*translation + 0.006*promoters + 0.006*transcripts + 0.005*upstream + 0.005*methylation
    
    topic # 6: 0.019*plants + 0.013*plant + 0.011*males + 0.011*auxin + 0.011*females + 0.010*fitness + 0.009*pouch + 0.006*male + 0.006*temperature + 0.006*host
    
    topic # 7: 0.018*state + 0.011*states + 0.009*viral + 0.008*virus + 0.006*reaction + 0.006*remembered + 0.006*fluorescence + 0.006*molecules + 0.005*particles + 0.005*temperature
    
    topic # 8: 0.058*mice + 0.010*disease + 0.010*liver + 0.010*glucose + 0.009*mouse + 0.008*hscs + 0.008*blood + 0.008*insulin + 0.007*c57bl/6 + 0.006*hepatocytes
    
    topic # 9: 0.026*neurons + 0.020*synaptic + 0.010*stimulation + 0.008*current + 0.008*channels + 0.008*responses + 0.008*synapses + 0.008*calcium + 0.007*spike + 0.007*amplitude
    
    topic # 10: 0.011*membrane + 0.008*transfected + 0.007*antibody + 0.007*antibodies + 0.006*expressing + 0.004*western + 0.004*treated + 0.004*phosphorylation + 0.004*images + 0.004*localization
    
    topic # 11: 0.032*neurons + 0.016*axons + 0.012*axon + 0.010*retina + 0.010*mutants + 0.009*muscle + 0.008*retinal + 0.008*mutant + 0.008*axonal + 0.007*sections
    
    topic # 12: 0.017*rnai + 0.016*targets + 0.015*motif + 0.014*mirna + 0.014*hairy + 0.010*modules + 0.010*syntaxin + 0.010*elegans + 0.009*mirnas + 0.009*motifs
    
    topic # 13: 0.019*flies + 0.014*mutant + 0.010*drosophila + 0.010*larvae + 0.008*circadian + 0.006*cycle + 0.006*clock + 0.006*adult + 0.006*signaling + 0.005*mutants
    
    topic # 14: 0.026*animals + 0.020*mice + 0.016*mutants + 0.013*mutant + 0.011*signaling + 0.010*muscle + 0.010*pathway + 0.007*mitochondrial + 0.006*cholesterol + 0.006*elegans
    
    topic # 15: 0.008*tiles + 0.006*myosin + 0.006*images + 0.005*fiber + 0.005*video + 0.005*tile + 0.005*actin + 0.005*epidermal + 0.004*5.75 + 0.004*distal
    
    topic # 16: 0.013*phosphorylation + 0.009*treg + 0.008*albicans + 0.007*apcs + 0.007*naïve + 0.007*kinase + 0.006*memory + 0.006*induction + 0.006*signaling + 0.006*state
    
    topic # 17: 0.020*domain + 0.018*residues + 0.010*domains + 0.009*structural + 0.008*structures + 0.008*peptide + 0.006*residue + 0.006*bound + 0.005*peptides + 0.005*crystal
    
    topic # 18: 0.021*cohesin + 0.021*infection + 0.017*chromosome + 0.012*infected + 0.010*replication + 0.010*host + 0.009*chromosomes + 0.008*chromatin + 0.007*pericentric + 0.007*centromere
    
    topic # 19: 0.010*strain + 0.010*coli + 0.009*strains + 0.007*bacterial + 0.006*mutant + 0.006*bacteria + 0.005*enzymes + 0.004*grown + 0.004*concentrations + 0.004*membrane
    
    topic # 20: 0.007*stimulus + 0.005*firing + 0.004*stimuli + 0.004*frequency + 0.004*noise + 0.004*rates + 0.004*network + 0.004*power + 0.003*population + 0.003*correlation
    


Looking at topic 10, we can see that lowering `passes` had a pretty detrimental
effect on the topic that deals with plants and
[auxin](https://en.wikipedia.org/wiki/Auxin) as this topic is now mixed up with
flies.

### Increasing `passes`


    model = gensim.models.ldamodel.LdaModel(corpus, id2word=dictionary, update_every=1, chunksize=100, passes=3, num_topics=20)


    for topic_i, topic in enumerate(model.print_topics(20)):
        print('topic # %d: %s\n' % (topic_i+1, topic))

    topic # 1: 0.014*infection + 0.011*treatment + 0.010*infected + 0.008*circadian + 0.008*phase + 0.007*treated + 0.007*cycle + 0.007*clock + 0.006*medium + 0.006*period
    
    topic # 2: 0.024*2003 + 0.023*2002 + 0.017*2001 + 0.016*2000 + 0.012*1999 + 0.010*1998 + 0.009*conserved + 0.009*motif + 0.009*drosophila + 0.008*1997
    
    topic # 3: 0.019*strains + 0.017*strain + 0.009*yeast + 0.006*grown + 0.005*coli + 0.005*genome + 0.005*bacterial + 0.005*cerevisiae + 0.005*medium + 0.005*bacteria
    
    topic # 4: 0.024*mrna + 0.009*translation + 0.009*nuclear + 0.008*mrnas + 0.007*splicing + 0.007*lanes + 0.006*exon + 0.005*buffer + 0.005*vitro + 0.005*lane
    
    topic # 5: 0.031*transcription + 0.018*promoter + 0.015*transcriptional + 0.010*chromatin + 0.010*chip + 0.010*promoters + 0.009*methylation + 0.007*locus + 0.006*histone + 0.006*replication
    
    topic # 6: 0.014*peptide + 0.010*peptides + 0.006*aggregation + 0.006*buffer + 0.005*structural + 0.005*mass + 0.005*amino + 0.004*united + 0.004*states + 0.004*molecules
    
    topic # 7: 0.018*membrane + 0.014*fluorescence + 0.011*synaptic + 0.007*images + 0.006*imaging + 0.006*release + 0.005*channels + 0.005*microscopy + 0.005*image + 0.005*vesicles
    
    topic # 8: 0.012*stimulus + 0.011*neurons + 0.009*trials + 0.009*responses + 0.009*stimuli + 0.008*firing + 0.007*task + 0.006*visual + 0.006*cortex + 0.005*location
    
    topic # 9: 0.052*mice + 0.011*animals + 0.011*mouse + 0.009*neurons + 0.009*adult + 0.008*insulin + 0.008*brain + 0.007*tissue + 0.006*muscle + 0.006*glucose
    
    topic # 10: 0.025*neurons + 0.022*embryos + 0.013*axons + 0.010*dorsal + 0.009*axon + 0.008*zebrafish + 0.008*migration + 0.008*muscle + 0.007*mutants + 0.007*ventral
    
    topic # 11: 0.016*embryos + 0.013*signaling + 0.011*stage + 0.010*neural + 0.008*differentiation + 0.008*embryo + 0.008*auxin + 0.007*signalling + 0.007*fate + 0.007*division
    
    topic # 12: 0.020*domain + 0.017*residues + 0.009*domains + 0.007*structures + 0.007*state + 0.006*bound + 0.005*crystal + 0.005*conformation + 0.005*complexes + 0.005*mutant
    
    topic # 13: 0.014*state + 0.012*memory + 0.010*feedback + 0.009*olfactory + 0.008*odor + 0.006*network + 0.005*responses + 0.005*switching + 0.005*switch + 0.005*stability
    
    topic # 14: 0.010*selection + 0.010*population + 0.006*variation + 0.006*populations + 0.005*evolution + 0.005*rates + 0.005*diversity + 0.004*humans + 0.004*estimated + 0.004*divergence
    
    topic # 15: 0.032*mutants + 0.025*mutant + 0.025*animals + 0.017*rnai + 0.015*elegans + 0.013*mutation + 0.011*mutations + 0.009*phenotype + 0.009*worms + 0.008*larvae
    
    topic # 16: 0.028*mice + 0.011*tumor + 0.008*tumors + 0.007*mouse + 0.007*blood + 0.006*proliferation + 0.006*hscs + 0.006*cancer + 0.005*t-cells + 0.005*differentiation
    
    topic # 17: 0.009*network + 0.006*density + 0.005*parameters + 0.005*distance + 0.005*actin + 0.005*dynamics + 0.004*force + 0.004*shape + 0.004*networks + 0.004*rates
    
    topic # 18: 0.026*chromosome + 0.020*genome + 0.013*chromosomes + 0.010*recombination + 0.009*cohesin + 0.008*assembly + 0.008*genomic + 0.007*reads + 0.005*mitotic + 0.005*spindle
    
    topic # 19: 0.018*flies + 0.014*females + 0.013*males + 0.009*fitness + 0.009*plants + 0.008*male + 0.008*female + 0.007*plant + 0.007*food + 0.006*temperature
    
    topic # 20: 0.011*signaling + 0.010*mutant + 0.009*phosphorylation + 0.009*antibody + 0.008*antibodies + 0.007*expressing + 0.007*transfected + 0.007*kinase + 0.007*pathway + 0.006*western
    


In this run, topic 16 deals with plants and auxin and now contains tokens that
have to do with methylation,
presumably from work on
[epigenetics](https://en.wikipedia.org/wiki/Epigenetics), and
some growth factor called [EPS8](https://en.wikipedia.org/wiki/EPS8).

It is hard to say if increasing the number of `passes` improved the topic
quality.

## Increasing the Number of Topics

Another parameter that can be altered is `num_topics` and intuitively I would
expect that increasing this parameter allows for finer-grained topics.

Regarding this topic that covers plants and auxin, I would expect a sufficiently
large `num_topics` value would mulitple different topics that
each deal with different aspects of plants.


    model = gensim.models.ldamodel.LdaModel(corpus, id2word=dictionary, update_every=1, chunksize=100, passes=2, num_topics=60)


    for topic_i, topic in enumerate(model.print_topics(60)):
        print('topic # %d: %s\n' % (topic_i+1, topic))

    topic # 1: 0.012*abundance + 0.011*flight + 0.010*habitat + 0.010*water + 0.009*diversity + 0.009*land + 0.008*density + 0.008*birds + 0.007*area + 0.007*webs
    
    topic # 2: 0.038*state + 0.012*feedback + 0.010*parameters + 0.009*states + 0.008*transition + 0.008*dynamics + 0.007*concentrations + 0.007*degradation + 0.007*noise + 0.007*constant
    
    topic # 3: 0.032*residues + 0.026*domain + 0.015*crystal + 0.014*structures + 0.013*conformation + 0.013*structural + 0.011*helix + 0.011*surface + 0.010*interface + 0.010*loop
    
    topic # 4: 0.008*distance + 0.007*direction + 0.007*force + 0.006*movement + 0.005*motion + 0.005*field + 0.004*tuning + 0.004*curves + 0.004*rates + 0.004*spatial
    
    topic # 5: 0.042*hybrid + 0.035*maternal + 0.026*germ + 0.024*hybrids + 0.022*sperm + 0.020*mothers + 0.018*zygotic + 0.016*eggs + 0.014*oocytes + 0.012*crosses
    
    topic # 6: 0.030*population + 0.014*modern + 0.014*populations + 0.011*humans + 0.011*estimates + 0.007*probability + 0.007*estimated + 0.005*prey + 0.005*density + 0.005*estimate
    
    topic # 7: 0.022*membrane + 0.012*fusion + 0.011*viral + 0.010*virus + 0.010*syntaxin + 0.006*complexes + 0.006*membranes + 0.005*united + 0.005*hiv-1 + 0.005*cellular
    
    topic # 8: 0.031*synaptic + 0.026*neurons + 0.018*hippocampal + 0.017*memory + 0.015*rats + 0.014*synapses + 0.011*hippocampus + 0.010*psd-95 + 0.010*brain + 0.010*neuronal
    
    topic # 9: 0.057*mutants + 0.031*mutant + 0.029*animals + 0.028*elegans + 0.018*worms + 0.014*rnai + 0.012*:gfp + 0.011*pathway + 0.010*mutation + 0.010*dauer
    
    topic # 10: 0.050*muscle + 0.027*animals + 0.022*pharyngeal + 0.021*body + 0.015*mutants + 0.013*fiber + 0.013*bodies + 0.012*degeneration + 0.012*fibers + 0.011*muscles
    
    topic # 11: 0.062*uptake + 0.052*coupling + 0.050*flux + 0.045*neck + 0.039*metabolites + 0.031*glutamate + 0.027*fluid + 0.026*transients + 0.024*transporters + 0.021*release
    
    topic # 12: 0.051*neurons + 0.028*axons + 0.021*axon + 0.015*dendritic + 0.012*axonal + 0.012*dendrites + 0.012*motor + 0.010*synaptic + 0.009*synapse + 0.008*dendrite
    
    topic # 13: 0.086*strains + 0.079*strain + 0.032*yeast + 0.025*medium + 0.021*grown + 0.019*induction + 0.016*glucose + 0.015*cerevisiae + 0.014*hac1 + 0.014*cultures
    
    topic # 14: 0.018*cortex + 0.017*task + 0.016*trials + 0.015*brain + 0.013*subjects + 0.013*attention + 0.012*participants + 0.011*location + 0.011*areas + 0.010*neural
    
    topic # 15: 0.030*network + 0.028*motif + 0.024*clusters + 0.019*networks + 0.018*motifs + 0.018*cluster + 0.014*modules + 0.011*regulatory + 0.009*profiles + 0.008*score
    
    topic # 16: 0.046*cycle + 0.035*division + 0.019*proliferation + 0.016*progression + 0.013*arrest + 0.012*stem + 0.012*-gfp + 0.011*differentiation + 0.011*colonies + 0.011*cyclin
    
    topic # 17: 0.015*insulin + 0.012*glucose + 0.010*disease + 0.010*mitochondrial + 0.010*liver + 0.010*cholesterol + 0.009*animals + 0.008*tissue + 0.008*aging + 0.007*food
    
    topic # 18: 0.026*neurons + 0.022*neural + 0.015*progenitor + 0.012*cortical + 0.011*differentiation + 0.011*neuronal + 0.010*adult + 0.009*explants + 0.009*cortex + 0.009*developing
    
    topic # 19: 0.052*sleep + 0.023*remembered + 0.019*period + 0.011*heads + 0.011*night + 0.010*mper2 + 0.009*circadian + 0.008*responsiveness + 0.008*diff + 0.008*deprivation
    
    topic # 20: 0.059*migration + 0.039*wound + 0.021*apical + 0.015*video + 0.015*healing + 0.014*epithelial + 0.014*migratory + 0.013*adhesion + 0.013*migrating + 0.013*cdk5
    
    topic # 21: 0.064*fluorescence + 0.035*images + 0.027*imaging + 0.027*image + 0.026*recruitment + 0.020*intensity + 0.016*events + 0.013*fluorescent + 0.013*laser + 0.011*bcl-x
    
    topic # 22: 0.070*mrna + 0.024*mrnas + 0.024*exon + 0.022*splicing + 0.017*translation + 0.016*transcripts + 0.013*exons + 0.012*elements + 0.011*intron + 0.011*splice
    
    topic # 23: 0.014*subunit + 0.014*domain + 0.013*substrate + 0.013*domains + 0.011*enzyme + 0.010*residues + 0.010*enzymes + 0.010*reaction + 0.008*subunits + 0.007*catalytic
    
    topic # 24: 0.019*chromosome + 0.012*methylation + 0.011*transcription + 0.009*genomic + 0.009*genome + 0.008*chromosomes + 0.008*locus + 0.008*microarray + 0.008*transcripts + 0.007*xist
    
    topic # 25: 0.021*genome + 0.020*genomes + 0.019*evolution + 0.018*conserved + 0.015*phylogenetic + 0.014*evolutionary + 0.014*tree + 0.010*alignment + 0.009*branch + 0.008*lineage
    
    topic # 26: 0.048*infection + 0.045*host + 0.029*infected + 0.021*bacterial + 0.019*bacteria + 0.016*parasites + 0.013*immune + 0.012*mosquitoes + 0.012*virulence + 0.012*hosts
    
    topic # 27: 0.015*aggregation + 0.011*aggregates + 0.010*buffer + 0.009*solution + 0.009*spectra + 0.008*water + 0.007*charge + 0.007*domain + 0.006*concentrations + 0.006*structural
    
    topic # 28: 0.015*coli + 0.008*codon + 0.008*plasmid + 0.008*synthesis + 0.008*products + 0.007*initiation + 0.007*translation + 0.007*ribosomal + 0.006*poly + 0.005*bacterial
    
    topic # 29: 0.040*peptide + 0.028*peptides + 0.024*domain + 0.021*lane + 0.017*vitro + 0.017*lanes + 0.014*affinity + 0.014*purified + 0.014*vivo + 0.012*buffer
    
    topic # 30: 0.074*plants + 0.051*plant + 0.027*arabidopsis + 0.025*pollen + 0.016*thaliana + 0.014*rice + 0.012*sorghum + 0.012*maize + 0.012*ovules + 0.010*flowers
    
    topic # 31: 0.049*2003 + 0.047*2002 + 0.034*2001 + 0.034*2000 + 0.025*1999 + 0.020*1998 + 0.018*1997 + 0.016*1996 + 0.013*genome + 0.013*2004
    
    topic # 32: 0.021*trials + 0.021*learning + 0.019*auditory + 0.019*performance + 0.017*task + 0.015*training + 0.015*visual + 0.012*discrimination + 0.012*responses + 0.011*olfactory
    
    topic # 33: 0.021*selection + 0.016*genome + 0.014*alleles + 0.013*chromosome + 0.012*allele + 0.009*divergence + 0.008*genomic + 0.008*population + 0.007*locus + 0.007*loci
    
    topic # 34: 0.019*fitness + 0.017*selection + 0.016*populations + 0.013*population + 0.009*adaptation + 0.008*cost + 0.007*individuals + 0.007*treatment + 0.007*environment + 0.007*bees
    
    topic # 35: 0.036*targets + 0.026*rnai + 0.021*mirna + 0.019*hairy + 0.015*mirnas + 0.011*drosophila + 0.011*conserved + 0.009*regulation + 0.008*screen + 0.006*phenotypes
    
    topic # 36: 0.036*channel + 0.033*channels + 0.027*retina + 0.026*retinal + 0.024*current + 0.022*membrane + 0.019*calcium + 0.018*currents + 0.016*voltage + 0.013*photoreceptors
    
    topic # 37: 0.075*recombination + 0.067*mutations + 0.056*mutation + 0.017*frequency + 0.016*repair + 0.016*conversion + 0.013*substitution + 0.013*substitutions + 0.013*rates + 0.011*integration
    
    topic # 38: 0.038*nuclear + 0.025*assembly + 0.022*particles + 0.020*replication + 0.018*localization + 0.013*foci + 0.012*nucleus + 0.012*ssdna + 0.009*particle + 0.009*budding
    
    topic # 39: 0.029*stimulus + 0.027*neurons + 0.023*firing + 0.022*responses + 0.018*stimuli + 0.013*frequency + 0.013*neuron + 0.012*stimulation + 0.012*spike + 0.008*amplitude
    
    topic # 40: 0.066*actin + 0.031*myosin + 0.019*video + 0.018*filaments + 0.018*leading + 0.016*eps8 + 0.016*motility + 0.015*polymerization + 0.014*edge + 0.014*f-actin
    
    topic # 41: 0.024*molecules + 0.011*ligand + 0.011*inhibition + 0.011*closure + 0.010*lifetime + 0.010*ligands + 0.010*fret + 0.010*concentrations + 0.008*agonist + 0.007*molecule
    
    topic # 42: 0.057*females + 0.056*males + 0.044*melanogaster + 0.044*male + 0.034*female + 0.019*mating + 0.017*simulans + 0.016*dosage + 0.016*drosophila + 0.014*flies
    
    topic # 43: 0.030*fmrp + 0.029*cftr + 0.025*hairpin + 0.022*fractions + 0.020*neurites + 0.019*cdc42 + 0.019*chemotaxis + 0.017*septum + 0.015*vegetal + 0.015*tfiih
    
    topic # 44: 0.038*auxin + 0.036*plants + 0.033*tiles + 0.019*tile + 0.019*mutant + 0.017*plant + 0.016*arabidopsis + 0.014*mutants + 0.014*petal + 0.013*pin1
    
    topic # 45: 0.043*mutant + 0.025*flies + 0.024*larvae + 0.018*drosophila + 0.017*mutants + 0.014*signaling + 0.014*clones + 0.012*pathway + 0.011*larval + 0.010*rescue
    
    topic # 46: 0.020*differentiation + 0.011*mouse + 0.010*β-catenin + 0.010*t-cells + 0.009*adult + 0.009*signaling + 0.008*proliferation + 0.008*skin + 0.008*induction + 0.007*staining
    
    topic # 47: 0.142*pole + 0.091*tailbud + 0.069*ced-1 + 0.069*psba + 0.057*optic + 0.049*dkk1 + 0.037*phagosomal + 0.029*erps + 0.021*prechordal + 0.021*lefty
    
    topic # 48: 0.167*association + 0.075*lipid + 0.047*snps + 0.041*associations + 0.023*variants + 0.020*thetaiotaomicron + 0.018*lipolysis + 0.017*tunnel + 0.014*inkt + 0.014*synthetic
    
    topic # 49: 0.084*albicans + 0.032*opaque + 0.031*ubiquitination + 0.031*biofilm + 0.027*ubiquitin + 0.025*golgi + 0.025*white + 0.019*arf1 + 0.018*ligase + 0.017*switching
    
    topic # 50: 0.028*phase + 0.024*circadian + 0.021*clock + 0.018*cycle + 0.017*period + 0.016*oscillations + 0.014*hexamers + 0.013*flies + 0.012*rhythms + 0.011*dark
    
    topic # 51: 0.006*variation + 0.006*rates + 0.004*correlation + 0.004*variable + 0.003*groups + 0.003*dataset + 0.003*categories + 0.003*bias + 0.003*proportion + 0.003*best
    
    topic # 52: 0.062*embryos + 0.020*embryo + 0.019*stage + 0.018*zebrafish + 0.014*anterior + 0.014*dorsal + 0.012*posterior + 0.010*cartilage + 0.009*ventral + 0.009*domain
    
    topic # 53: 0.150*mice + 0.025*mouse + 0.013*sections + 0.012*animals + 0.006*injection + 0.005*littermates + 0.005*controls + 0.005*knockout + 0.005*medulla + 0.005*blood
    
    topic # 54: 0.037*membrane + 0.030*receptor + 0.029*vesicles + 0.023*vesicle + 0.021*endocytosis + 0.017*plasma + 0.016*receptors + 0.013*vessel + 0.012*transport + 0.012*endosomes
    
    topic # 55: 0.015*phage + 0.012*compounds + 0.012*united + 0.012*states + 0.011*viruses + 0.010*mutant + 0.009*compound + 0.009*cysteine + 0.009*tail + 0.008*redox
    
    topic # 56: 0.039*mice + 0.022*hscs + 0.014*thymocytes + 0.013*bone + 0.013*c57bl/6 + 0.011*sca-1 + 0.011*foxp3 + 0.010*flow + 0.010*naïve + 0.009*marrow
    
    topic # 57: 0.009*damage + 0.009*survival + 0.007*blood + 0.007*trkb + 0.007*vivo + 0.006*regeneration + 0.006*patients + 0.006*ifn-γ + 0.006*cytokine + 0.006*t-cell
    
    topic # 58: 0.045*transcription + 0.039*promoter + 0.026*cohesin + 0.024*transcriptional + 0.021*promoters + 0.018*chip + 0.016*chromatin + 0.016*ino1 + 0.012*yeast + 0.010*histone
    
    topic # 59: 0.024*mitotic + 0.022*microtubules + 0.020*microtubule + 0.020*spindle + 0.016*mitosis + 0.015*plk1 + 0.013*centromere + 0.013*mad2 + 0.013*meiosis + 0.011*meiotic
    
    topic # 60: 0.017*phosphorylation + 0.013*transfected + 0.011*treated + 0.011*treatment + 0.010*kinase + 0.010*western + 0.008*signaling + 0.008*antibodies + 0.008*antibody + 0.007*inhibitor
    


Even cranking up the `num_topics` parameter to 60 does not split the topic that
contains plants into more refined subtopics.

Either I misunderstand the API of `LdaModel` or increasing the number of modeled
topics is the wrong way of going about it.
