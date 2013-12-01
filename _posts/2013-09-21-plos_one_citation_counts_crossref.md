---
layout: default
title: "PLoS ONE Citation Counts Recorded in CrossRef"
date: 2013-09-21
tags: wip
---

**This post is work in progress**

This blog post is based on an IPython Notebook that I sketched out on
[GitHub](https://gist.github.com/waltherg/6645140) some time ago.
You can find accompanying data there.

This post is work in progress since conversion from my original
IPython Notebook to Markdown (for this post) produced some glitches
that I need to amend.

# PLoS ONE Citation Counts

The data set used in this notebook has been described here:

http://nbviewer.ipython.org/6211587

Here, we'll use that same data and call the CrossRef OpenURL service
to retrieve citation counts for all PLoS ONE publications of the year 2010.

We will assume that a publication came out in 2010 when it was accepted
for publication in that year.

Oddly, we'll find that we are counting **7263 publications in 2010** while other
data suggest there were 6730 publications in 2010:

http://www.plos.org/wp-content/uploads/2013/09/progress_update_Measureing-
Impact.jpg

Feel free to give me a shout at https://twitter.com/mbgrw or play with this
notebook and data here
https://gist.github.com/waltherg/6645140.


    import cPickle as pickle
    import gzip
    import requests
    from BeautifulSoup import BeautifulSoup
    import time


    dat_file = gzip.open('editors_plos_one_dates_numbered.dat', 'rb')
    dat = pickle.load(dat_file)
    dat_file.close()


    print dat[0][0]['id']
    print dat[0][0]['accepted_date'][0]

    10.1371/journal.pone.0057865
    2013



    publications = {}
    for editor in dat.keys():
        for pub in dat[editor]:
            year = pub['accepted_date'][0]
            if year not in publications.keys():
                publications[year] = []
            publications[year].append(pub['id'])


    publications.keys()




    [2006, 2007, 2008, 2009, 2010, 2011, 2012, 2013]




    len(publications[2010])




    7263




    publications[2010][0]




    '10.1371/journal.pone.0011195'


    
    # We'll use the CrossRef OpenURL service
    # register your email here: http://www.crossref.org/requestaccount/
    # explanation here: https://github.com/articlemetrics/alm/wiki/Crossref
    
    crossref_email = 'youremail'
    
    no_citations_2010 = []
    for doi_i, doi in enumerate(publications[2010]):
        res = requests.get('http://www.crossref.org/openurl/?pid='+crossref_email+'&id=doi:'+doi+'&noredirect=true')
        xml = BeautifulSoup(res.text)
        attributes = dict(xml.findAll('query')[0].attrs)
        no_citations_2010.append(int(attributes['fl_count']))
        
        print doi_i,'of',len(publications[2010]),'doi',doi,'no. citations',int(attributes['fl_count'])
        
        # not sure of API limitations but sleep for five seconds between calls
        time.sleep(5)
        
    # redo into handy dictionary doi -> citation count
    crossref_citations = {}
    for doi_i, doi in enumerate(publications[2010]):
        crossref_citations[doi] = no_citations_2010[doi_i]
        
    # save dictionary to hard disk
    pickle.dump(crossref_citations, open('plos_one_accepted_2010_crossref_citations.dict', 'wb'))


    cit_dict = pickle.load(open('plos_one_accepted_2010_crossref_citations.dict', 'rb'))


    cit = cit_dict.values()


    print 'number of unique citation counts',len(set(cit))
    print 'smallest citation count:',min(cit),'biggest citation count:',max(cit)

    number of unique citation counts 84
    smallest citation count: 0 biggest citation count: 201



    bins = range(202)


    hist(cit, bins)




    (array([303, 437, 553, 568, 575, 537, 489, 436, 383, 283, 252, 217, 197,
           163, 158, 107, 124,  87,  80,  68,  47,  52,  55,  41,  35,  32,
            32,  25,  14,  20,  15,  17,  12,  14,   8,   7,  10,  10,  12,
            12,  12,   9,   9,   4,   3,   6,   3,   3,   4,   3,   9,   3,
             3,   6,   0,   4,   2,   1,   1,   3,   1,   4,   0,   2,   1,
             2,   1,   0,   0,   0,   0,   1,   1,   0,   3,   1,   0,   2,
             1,   0,   0,   1,   0,   0,   1,   0,   1,   2,   0,   0,   0,
             0,   0,   0,   0,   1,   1,   0,   0,   0,   0,   0,   0,   0,
             0,   0,   0,   0,   0,   1,   0,   0,   0,   0,   0,   1,   0,
             0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,
             0,   0,   0,   0,   0,   0,   0,   1,   0,   0,   0,   0,   0,
             0,   0,   0,   0,   1,   1,   0,   0,   0,   0,   0,   0,   0,
             0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,
             0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,
             1,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,
             0,   0,   0,   0,   0,   1]),
     array([  0,   1,   2,   3,   4,   5,   6,   7,   8,   9,  10,  11,  12,
            13,  14,  15,  16,  17,  18,  19,  20,  21,  22,  23,  24,  25,
            26,  27,  28,  29,  30,  31,  32,  33,  34,  35,  36,  37,  38,
            39,  40,  41,  42,  43,  44,  45,  46,  47,  48,  49,  50,  51,
            52,  53,  54,  55,  56,  57,  58,  59,  60,  61,  62,  63,  64,
            65,  66,  67,  68,  69,  70,  71,  72,  73,  74,  75,  76,  77,
            78,  79,  80,  81,  82,  83,  84,  85,  86,  87,  88,  89,  90,
            91,  92,  93,  94,  95,  96,  97,  98,  99, 100, 101, 102, 103,
           104, 105, 106, 107, 108, 109, 110, 111, 112, 113, 114, 115, 116,
           117, 118, 119, 120, 121, 122, 123, 124, 125, 126, 127, 128, 129,
           130, 131, 132, 133, 134, 135, 136, 137, 138, 139, 140, 141, 142,
           143, 144, 145, 146, 147, 148, 149, 150, 151, 152, 153, 154, 155,
           156, 157, 158, 159, 160, 161, 162, 163, 164, 165, 166, 167, 168,
           169, 170, 171, 172, 173, 174, 175, 176, 177, 178, 179, 180, 181,
           182, 183, 184, 185, 186, 187, 188, 189, 190, 191, 192, 193, 194,
           195, 196, 197, 198, 199, 200, 201]),
     <a list of 201 Patch objects>)




![png](https://dl.dropboxusercontent.com/u/129945779/georgio/plos_one_citation_counts_crossref_openurl_files/plos_one_citation_counts_crossref_openurl_15_1.png)



    hist(cit, bins, normed=1)




    (array([ 0.04584657,  0.06612195,  0.08367378,  0.08594341,  0.08700257,
            0.08125284,  0.07399001,  0.06597065,  0.05795128,  0.0428204 ,
            0.03812982,  0.03283401,  0.02980784,  0.02466334,  0.02390679,
            0.01619004,  0.01876229,  0.01316387,  0.01210471,  0.010289  ,
            0.00711151,  0.00786806,  0.00832199,  0.00620366,  0.00529581,
            0.00484188,  0.00484188,  0.00378272,  0.00211832,  0.00302618,
            0.00226963,  0.00257225,  0.00181571,  0.00211832,  0.00121047,
            0.00105916,  0.00151309,  0.00151309,  0.00181571,  0.00181571,
            0.00181571,  0.00136178,  0.00136178,  0.00060524,  0.00045393,
            0.00090785,  0.00045393,  0.00045393,  0.00060524,  0.00045393,
            0.00136178,  0.00045393,  0.00045393,  0.00090785,  0.        ,
            0.00060524,  0.00030262,  0.00015131,  0.00015131,  0.00045393,
            0.00015131,  0.00060524,  0.        ,  0.00030262,  0.00015131,
            0.00030262,  0.00015131,  0.        ,  0.        ,  0.        ,
            0.        ,  0.00015131,  0.00015131,  0.        ,  0.00045393,
            0.00015131,  0.        ,  0.00030262,  0.00015131,  0.        ,
            0.        ,  0.00015131,  0.        ,  0.        ,  0.00015131,
            0.        ,  0.00015131,  0.00030262,  0.        ,  0.        ,
            0.        ,  0.        ,  0.        ,  0.        ,  0.        ,
            0.00015131,  0.00015131,  0.        ,  0.        ,  0.        ,
            0.        ,  0.        ,  0.        ,  0.        ,  0.        ,
            0.        ,  0.        ,  0.        ,  0.        ,  0.00015131,
            0.        ,  0.        ,  0.        ,  0.        ,  0.        ,
            0.00015131,  0.        ,  0.        ,  0.        ,  0.        ,
            0.        ,  0.        ,  0.        ,  0.        ,  0.        ,
            0.        ,  0.        ,  0.        ,  0.        ,  0.        ,
            0.        ,  0.        ,  0.        ,  0.        ,  0.        ,
            0.        ,  0.        ,  0.00015131,  0.        ,  0.        ,
            0.        ,  0.        ,  0.        ,  0.        ,  0.        ,
            0.        ,  0.        ,  0.00015131,  0.00015131,  0.        ,
            0.        ,  0.        ,  0.        ,  0.        ,  0.        ,
            0.        ,  0.        ,  0.        ,  0.        ,  0.        ,
            0.        ,  0.        ,  0.        ,  0.        ,  0.        ,
            0.        ,  0.        ,  0.        ,  0.        ,  0.        ,
            0.        ,  0.        ,  0.        ,  0.        ,  0.        ,
            0.        ,  0.        ,  0.        ,  0.        ,  0.        ,
            0.        ,  0.        ,  0.00015131,  0.        ,  0.        ,
            0.        ,  0.        ,  0.        ,  0.        ,  0.        ,
            0.        ,  0.        ,  0.        ,  0.        ,  0.        ,
            0.        ,  0.        ,  0.        ,  0.        ,  0.        ,
            0.00015131]),
     array([  0,   1,   2,   3,   4,   5,   6,   7,   8,   9,  10,  11,  12,
            13,  14,  15,  16,  17,  18,  19,  20,  21,  22,  23,  24,  25,
            26,  27,  28,  29,  30,  31,  32,  33,  34,  35,  36,  37,  38,
            39,  40,  41,  42,  43,  44,  45,  46,  47,  48,  49,  50,  51,
            52,  53,  54,  55,  56,  57,  58,  59,  60,  61,  62,  63,  64,
            65,  66,  67,  68,  69,  70,  71,  72,  73,  74,  75,  76,  77,
            78,  79,  80,  81,  82,  83,  84,  85,  86,  87,  88,  89,  90,
            91,  92,  93,  94,  95,  96,  97,  98,  99, 100, 101, 102, 103,
           104, 105, 106, 107, 108, 109, 110, 111, 112, 113, 114, 115, 116,
           117, 118, 119, 120, 121, 122, 123, 124, 125, 126, 127, 128, 129,
           130, 131, 132, 133, 134, 135, 136, 137, 138, 139, 140, 141, 142,
           143, 144, 145, 146, 147, 148, 149, 150, 151, 152, 153, 154, 155,
           156, 157, 158, 159, 160, 161, 162, 163, 164, 165, 166, 167, 168,
           169, 170, 171, 172, 173, 174, 175, 176, 177, 178, 179, 180, 181,
           182, 183, 184, 185, 186, 187, 188, 189, 190, 191, 192, 193, 194,
           195, 196, 197, 198, 199, 200, 201]),
     <a list of 201 Patch objects>)




![png](https://dl.dropboxusercontent.com/u/129945779/georgio/plos_one_citation_counts_crossref_openurl_files/plos_one_citation_counts_crossref_openurl_16_1.png)



    median(cit)




    6.0




    mean(cit)




    8.8766833106370093


