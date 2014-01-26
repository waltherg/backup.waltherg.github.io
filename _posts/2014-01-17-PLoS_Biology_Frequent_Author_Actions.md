---
layout: default
title: "Most Frequent Author Actions in PLoS Biology Articles"
date: 2014-01-17
tags: plos nltk
---

# Update: Web Application in Development

I started developing a web application based on the simple steps lined
out in this blog post.
I am entirely new to developing and deploying web applications so any
feedback and criticism is greatly appreciated!

Check out the app: [TLDRMed](http://tldrmed.com).

# Most Frequent Author Actions in PLoS Biology Articles

In a [previous blog post](http://georg.io/2014/01/PLoS_Biology_Author_Action) I
attempted to summarize
PLoS Biology articles by extracting sentences that contained the claimed author
action "we have shown".

As asked by [Noam Ross](https://twitter.com/noamross/status/422062035627569152),
it might be interesting to see
what the most frequent author actions besides "show" are.

Most of this code is just the same as in my previous blog post - skip to the
bottom for a list of the twenty most
frequent claimed author actions.


    import plospy
    import os
    
    from nltk.tokenize import sent_tokenize
    
    from matplotlib import pyplot
    %matplotlib inline
    
    from nltk.tag import RegexpTagger, BigramTagger, UnigramTagger
    from nltk.corpus import brown
    import nltk


    all_names = [name for name in os.listdir('../plos/plos_biology/plos_biology_data') if '.dat' in name]


    article_bodies = {}
    article_titles = {}
    
    for name_i, name in enumerate(all_names):
        docs = plospy.PlosXml('../plos/plos_biology/plos_biology_data/'+name)
        for article in docs.docs:
            article_bodies[article['id']] = article['body']
            article_titles[article['id']] = article['title']

Let us now use one of the standard sentence tokenizers shipped with NLTK:


    sentences = {}
    for doi in article_bodies.keys():
        tokens = sent_tokenize(article_bodies[doi])
        sentences[doi] = []
        for sentence in tokens:
            sentences[doi].append(sentence)

Let us now train both a [UnigramTagger](http://nltk.org/api/nltk.tag.html) and a
[BigramTagger](http://nltk.org/api/nltk.tag.html)
and [chain them together](http://nltk.org/api/nltk.tag.html#module-
nltk.tag.sequential) for better results.


    brown_train = brown.tagged_sents()


    len(brown_train)
    
    57340




    unigram_tagger = UnigramTagger(brown_train)
    bigram_tagger = BigramTagger(brown_train, backoff=unigram_tagger)

As opposed to my [previous blog
post](http://georg.io/2014/01/PLoS_Biology_Author_Action), here we do not
save raw sentences in `author_actions` but rather tagged sentences.


    author_actions = {}
    
    for doi in sentences.keys():
        author_actions[doi] = []
        for sentence in sentences[doi]:
            if any([token == 'we' or token == 'We' for token in sentence.split()]):
                author_actions[doi].append(bigram_tagger.tag(sentence.split()))


    author_actions[author_actions.keys()[2]][0]

    [(u'We', 'PPSS'),
     (u'reasoned', 'VBD'),
     (u'that', 'CS'),
     (u'imaging', 'NN'),
     (u'the', 'AT'),
     (u'macaque', None),
     (u'auditory', None),
     (u'cortex', 'NN'),
     (u'would', 'MD'),
     (u'be', 'BE'),
     (u'invaluable', 'JJ'),
     (u'for', 'IN'),
     (u'functionally', 'RB'),
     (u'localizing', None),
     (u'ACFs.', None)]



Let us now extract all verbs in these sentences but grabbing those words tagged
with any variation of **VB** except
for **VBG** (gerunds / -ing forms).


    verbs = []
    
    for key in author_actions.keys():
        for sentence in author_actions[key]:
            for (word, tag) in sentence:
                if tag and tag != 'VBG' and tag.startswith('VB'):
                    verbs.append(word.lower())


    len(verbs)

    172765



As we can see though, this list of verbs also contains past tense (and likely
other) forms:


    verbs[:10]

    [u'demonstrate',
     u'demonstrated',
     u'expected',
     u'find',
     u'given',
     u'limited',
     u'add',
     u'infer',
     u'add',
     u'shown']



See the following [StackOverflow](http://stackoverflow.com/a/9764418/3078529)
answer for how to get the
infinitive form of these verbs:


    from nltk.stem.wordnet import WordNetLemmatizer


    lemmatizer = WordNetLemmatizer()


    lemmatizer.lemmatize(verbs[1], 'v')

    u'demonstrate'




    verbs_inf = [unicode(lemmatizer.lemmatize(verb, 'v')) for verb in verbs]


    verbs_inf[:20]

    [u'demonstrate',
     u'demonstrate',
     u'expect',
     u'find',
     u'give',
     u'limit',
     u'add',
     u'infer',
     u'add',
     u'show',
     u'use',
     u'identify',
     u'employ',
     u'develop',
     u'base',
     u'examine',
     u'minimize',
     u'confuse',
     u'arise',
     u'employ']



The occurrence of the verb *confuse* here is odd so let us take a look at an
occurrence of this:


    for doi in author_actions.keys():
        for sentence in author_actions[doi]:
            if 'confuse' in nltk.tag.untag(sentence):
                print sentence
                break

    [(u'By', 'IN'), (u'studying', 'VBG'), (u'a', 'AT'), (u'single', 'AP'), (u'population', 'NN'), (u'in', 'IN'), (u'which', 'WDT'), (u'the', 'AT'), (u'major', 'JJ'), (u'axis', 'NN'), (u'of', 'IN'), (u'environmental', 'JJ'), (u'variation', 'NN'), (u'is', 'BEZ'), (u'temporal,', None), (u'we', 'PPSS'), (u'can', 'MD'), (u'minimize', 'VB'), (u'the', 'AT'), (u'potential', 'JJ'), (u'to', 'TO'), (u'confuse', 'VB'), (u'environmental', 'JJ'), (u'effects', 'NNS'), (u'on', 'IN'), (u'genetic', 'JJ'), (u'variance', 'NN'), (u'with', 'IN'), (u'demographic', 'JJ'), (u'effects', 'NNS'), (u'that', 'WPS'), (u'may', 'MD'), (u'arise', 'VB'), (u'with', 'IN'), (u'comparisons', 'NNS'), (u'between', 'IN'), (u'distinct', 'JJ'), (u'populations', 'NNS'), (u'over', 'IN'), (u'a', 'AT'), (u'spatially', 'RB'), (u'heterogeneous', 'JJ'), (u'environment.', None)]


As we can see, the phrase that *confuse* occurs in is *we can minimize the
potential to confuse* - where
*confuse* is certainly not a claimed author action.

We should keep this in mind when interpreting our results.

To count the verbs in our list `verbs_inf` we use the standard Python container
`Counter`:


    from collections import Counter


    verb_counter = Counter(verbs_inf)

And these are the twenty most frequent verbs, by total count, in our data set:


    verb_counter.most_common(20)

    [(u'find', 7419),
     (u'use', 6248),
     (u'observe', 5000),
     (u'show', 4035),
     (u'test', 3632),
     (u'identify', 3140),
     (u'examine', 2975),
     (u'determine', 2949),
     (u'compare', 2492),
     (u'perform', 2029),
     (u'investigate', 1915),
     (u'express', 1769),
     (u'analyze', 1688),
     (u'require', 1518),
     (u'detect', 1490),
     (u'generate', 1490),
     (u'measure', 1468),
     (u'base', 1466),
     (u'describe', 1363),
     (u'obtain', 1341)]



To see how frequent each of these verbs is, we normalize their counts by the
length of `verbs_inf` which holds
all verbs we have extracted.

Bear in mind though that these frequencies are smaller than they should be since
a number of verbs in
`verbs_inf` are certainly not claimed author actions (such as *confuse* - see
comment above).


    most_frequent = verb_counter.most_common(20)
    most_frequent = [(word, float(count)/float(len(verbs_inf))) for (word, count) in most_frequent]
    for (word, count) in most_frequent:
        print('%s: %.4f' % (word, count))

    find: 0.0429
    use: 0.0362
    observe: 0.0289
    show: 0.0234
    test: 0.0210
    identify: 0.0182
    examine: 0.0172
    determine: 0.0171
    compare: 0.0144
    perform: 0.0117
    investigate: 0.0111
    express: 0.0102
    analyze: 0.0098
    require: 0.0088
    detect: 0.0086
    generate: 0.0086
    measure: 0.0085
    base: 0.0085
    describe: 0.0079
    obtain: 0.0078
