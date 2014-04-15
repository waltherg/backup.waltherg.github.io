---
layout: default
title: "PLoS Time to Publication"
date: 2013-10-06
tags: plos publishing
---

This blog post is based on an IPython Notebook that I sketched out on
[GitHub](https://gist.github.com/waltherg/6860035) some time ago.

# PLoS Time to Publication

In [an earlier IPython Notebook](http://nbviewer.ipython.org/6211587) I took a
look at the time to publication in PLoS ONE.

Here I'll take a comparative look at the time to publication in PLoS ONE,
Biology, Computational Biology, and Genetics.

[Drop me a line](https://twitter.com/mbgrw) for any feedback.


    import gzip
    import cPickle as pickle
    from datetime import date
    import numpy as np


    data_one = pickle.load(gzip.open('plos_one.gzip', 'rb'))
    data_biology = pickle.load(gzip.open('plos_biology.gzip', 'rb'))
    data_comp_biology = pickle.load(gzip.open('plos_comp_biology.gzip', 'rb'))
    data_genetics = pickle.load(gzip.open('plos_genetics.gzip', 'rb'))

## Overview of Data

The data we just loaded are dictionaries whose keys are randomly assigned,
unique integer identifiers for each editor
and whose values are the list of articles that these editors have handled.


    print 'number of PLoS ONE editors in data set', len(data_one.keys())
    print 'number of PLoS Biology editors', len(data_biology.keys())
    print 'number of PLoS Computational Biology editors', len(data_comp_biology.keys())
    print 'number of PLoS Genetics editors', len(data_genetics.keys())

    number of PLoS ONE editors in data set 4885
    number of PLoS Biology editors 675
    number of PLoS Computational Biology editors 570
    number of PLoS Genetics editors 626



    print 'number of PLoS ONE articles in data set', len([1 for ed_i in data_one.keys() for article in data_one[ed_i]])
    print 'number of PLoS Biology articles', len([1 for ed_i in data_biology.keys() for article in data_biology[ed_i]])
    print 'number of PLoS Comp. Biology articles', len([1 for ed_i in data_comp_biology.keys() for article in data_comp_biology[ed_i]])
    print 'number of PLoS Genetics articles', len([1 for ed_i in data_genetics.keys() for article in data_genetics[ed_i]])

    number of PLoS ONE articles in data set 63753
    number of PLoS Biology articles 1754
    number of PLoS Comp. Biology articles 2490
    number of PLoS Genetics articles 3371



    print 'earliest PLoS ONE publication', min([date(*article['publication_date']) for ed_i in data_one.keys() for article in data_one[ed_i]])
    print 'most recent PLoS ONE publication', max([date(*article['publication_date']) for ed_i in data_one.keys() for article in data_one[ed_i]])
    print ''
    print 'earliest PLoS Biology publication', min([date(*article['publication_date']) for ed_i in data_biology.keys() for article in data_biology[ed_i]])
    print 'most recent PLoS Biology publication', max([date(*article['publication_date']) for ed_i in data_biology.keys() for article in data_biology[ed_i]])
    print ''
    print 'earliest PLoS Comp Biology publication', min([date(*article['publication_date']) for ed_i in data_comp_biology.keys() for article in data_comp_biology[ed_i]])
    print 'most recent PLoS Comp Biology publication', max([date(*article['publication_date']) for ed_i in data_comp_biology.keys() for article in data_comp_biology[ed_i]])
    print ''
    print 'earliest PLoS Genetics publication', min([date(*article['publication_date']) for ed_i in data_genetics.keys() for article in data_genetics[ed_i]])
    print 'most recent PLoS Genetics publication', max([date(*article['publication_date']) for ed_i in data_genetics.keys() for article in data_genetics[ed_i]])

    earliest PLoS ONE publication 2006-12-01
    most recent PLoS ONE publication 2013-07-31
    
    earliest PLoS Biology publication 2003-08-18
    most recent PLoS Biology publication 2013-10-01
    
    earliest PLoS Comp Biology publication 2005-06-24
    most recent PLoS Comp Biology publication 2013-09-26
    
    earliest PLoS Genetics publication 2005-06-17
    most recent PLoS Genetics publication 2013-09-26


For each journal, let's throw all publications into one vat and take a look at
the empirical distribution of
the amount of time elapsed between submission and publication.


    # one mislabeled article in PLoS ONE, http://nbviewer.ipython.org/6211587
    # exclude that one mislabeled article whose 'time to publication' is less than zero
    duration_one = [(date(*article['publication_date'])-date(*article['received_date'])).days for ed_i in data_one.keys() for article in data_one[ed_i] if (date(*article['publication_date'])-date(*article['received_date'])).days >= 0]
    duration_biology = [(date(*article['publication_date'])-date(*article['received_date'])).days for ed_i in data_biology.keys() for article in data_biology[ed_i]]
    duration_comp_biology = [(date(*article['publication_date'])-date(*article['received_date'])).days for ed_i in data_comp_biology.keys() for article in data_comp_biology[ed_i]]
    duration_genetics = [(date(*article['publication_date'])-date(*article['received_date'])).days for ed_i in data_genetics.keys() for article in data_genetics[ed_i]]

## Distribution of Time to Publication


    n, bins, patches = hist([duration for duration in duration_one], normed=True, bins=range(600))
    title_one = title('PLoS ONE Time to Publication')
    _ = xlabel('time to publication / days')
    _ = ylabel('frequency')
    show()
    
    n, bins, patches = hist([duration for duration in duration_biology], normed=True, bins=range(600))
    title_biology = title('PLoS Biology Time to Publication')
    _ = xlabel('time to publication / days')
    _ = ylabel('frequency')
    show()
    
    n, bins, patches = hist([duration for duration in duration_comp_biology], normed=True, bins=range(600))
    title_biology = title('PLoS Comp. Biology Time to Publication')
    _ = xlabel('time to publication / days')
    _ = ylabel('frequency')
    show()
    
    n, bins, patches = hist([duration for duration in duration_genetics], normed=True, bins=range(600))
    title_biology = title('PLoS Genetics Time to Publication')
    _ = xlabel('time to publication / days')
    _ = ylabel('frequency')
    show()


![png](https://dl.dropboxusercontent.com/u/129945779/georgio/plos_time_to_publication_files/plos_time_to_publication_12_0.png)



![png](https://dl.dropboxusercontent.com/u/129945779/georgio/plos_time_to_publication_files/plos_time_to_publication_12_1.png)



![png](https://dl.dropboxusercontent.com/u/129945779/georgio/plos_time_to_publication_files/plos_time_to_publication_12_2.png)



![png](https://dl.dropboxusercontent.com/u/129945779/georgio/plos_time_to_publication_files/plos_time_to_publication_12_3.png)



    _ = boxplot([duration_one,
                 duration_biology,
                 duration_comp_biology,
                 duration_genetics])
    xticks([1, 2, 3, 4], ['PLoS ONE',
                          'Biology',
                          'Comp. Biology',
                          'Genetics'])
    _ = ylabel('time to publication / days')
    show()
    
    _ = boxplot([duration_one,
                 duration_biology,
                 duration_comp_biology,
                 duration_genetics])
    xticks([1, 2, 3, 4], ['PLoS ONE',
                          'Biology',
                          'Comp. Biology',
                          'Genetics'])
    ylim([0, 600])
    _ = ylabel('time to publication / days')
    show()


![png](https://dl.dropboxusercontent.com/u/129945779/georgio/plos_time_to_publication_files/plos_time_to_publication_13_0.png)



![png](https://dl.dropboxusercontent.com/u/129945779/georgio/plos_time_to_publication_files/plos_time_to_publication_13_1.png)


## Editorial Influence in Time to Publication

One interesting observation we made previously is that the time to publication
appears to depend somewhat on the handling editor.

Let's see if we observe similar trends in the other PLoS journals.


    lists = []
    for a_dict_i, a_dict in enumerate([data_one, data_biology, data_comp_biology, data_genetics]):
        a_list = []
        for key in a_dict.keys():
            _ = []
            for article in a_dict[key]:
                _.append((date(*article['publication_date'])-date(*article['received_date'])).days)
            a_list.append(_)
        lists.append(a_list)
        
    data_one_lists = sorted(lists[0], key=np.median)
    data_biology_lists = sorted(lists[1], key=np.median)
    data_comp_biology_lists = sorted(lists[2], key=np.median)
    data_genetics_lists = sorted(lists[3], key=np.median)


    print 'PLoS ONE data okay', len(data_one_lists) == len(data_one) 
    print 'PLoS Biology data okay', len(data_biology_lists) == len(data_biology)
    print 'PLoS Comp. Biology data okay', len(data_comp_biology_lists) == len(data_comp_biology)
    print 'PLoS Genetics okay', len(data_genetics_lists) == len(data_genetics)

    PLoS ONE data okay True
    PLoS Biology data okay True
    PLoS Comp. Biology data okay True
    PLoS Genetics okay True



    _ = scatter(range(len(data_one_lists)), [np.median(a_list) for a_list in data_one_lists])
    xlabel('editor')
    ylabel('median time to publication / days')
    xlim(-100, len(data_one_lists)+100)
    ylim(0, 600)
    _ = title('PLoS ONE Median Time to Publication')
    show()
    
    _ = scatter(range(len(data_biology_lists)), [np.median(a_list) for a_list in data_biology_lists])
    xlabel('editor')
    ylabel('median time to publication / days')
    xlim(-10, len(data_biology_lists)+10)
    ylim(0, 600)
    _ = title('PLoS Biology Median Time to Publication')
    show()
    
    _ = scatter(range(len(data_comp_biology_lists)), [np.median(a_list) for a_list in data_comp_biology_lists])
    xlabel('editor')
    ylabel('median time to publication / days')
    xlim(-10, len(data_comp_biology_lists)+10)
    ylim(0, 600)
    _ = title('PLoS Comp. Biology Median Time to Publication')
    show()
    
    _ = scatter(range(len(data_genetics_lists)), [np.median(a_list) for a_list in data_genetics_lists])
    xlabel('editor')
    ylabel('median time to publication / days')
    xlim(-10, len(data_genetics_lists)+10)
    ylim(0, 600)
    _ = title('PLoS Genetics Median Time to Publication')
    show()


![png](https://dl.dropboxusercontent.com/u/129945779/georgio/plos_time_to_publication_files/plos_time_to_publication_18_0.png)



![png](https://dl.dropboxusercontent.com/u/129945779/georgio/plos_time_to_publication_files/plos_time_to_publication_18_1.png)



![png](https://dl.dropboxusercontent.com/u/129945779/georgio/plos_time_to_publication_files/plos_time_to_publication_18_2.png)



![png](https://dl.dropboxusercontent.com/u/129945779/georgio/plos_time_to_publication_files/plos_time_to_publication_18_3.png)

