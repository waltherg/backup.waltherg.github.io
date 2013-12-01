---
layout: default
title: "PLoS ONE Time to Publication"
date: 2013-08-12
tags: wip
---

**This post is work in progress**

This blog post is based on an IPython Notebook that I sketched out on
[GitHub](https://gist.github.com/waltherg/6211587) some time ago.
You can find accompanying data there.

This post is work in progress since conversion from my original
IPython Notebook to Markdown (for this post) produced some glitches
that I need to amend.

# PLoS ONE Time to Publication

I recemtly picked up working with PLOS's API and downloaded over 63,000 articles
from PLOS ONE which required about 6 hours, pagination of my requests to the
API, and around 2.5 GB of hard disk space.

Here is some analysis I have been doing with this data -- just some fun in the
summer while on vacation.

As you will notice, I am by no means experienced with data analysis and I am
grateful for any advice and criticism you may wish to offer.

This notebook can be found [on GitHub's
Gist](https://gist.github.com/waltherg/6211587) where you can leave comments at
the bottom of the page.
Or shoot me a tweet [on Twitter](https://twitter.com/mbgrw).

If you are interested in the data file I use in this notebook just get in touch.


    import sys
    import cPickle as pickle
    import gzip
    from datetime import date
    import numpy as np
    from matplotlib import pylab as pl
    import matplotlib


    home = !(echo $HOME)
    home = home[0]
    plos_dat = home+'/private/plos_one/editors_plos_one_dates.dat'


    dat_file = gzip.open(plos_dat, 'rb')
    dat = pickle.load(dat_file)
    dat_file.close()


    durations = []
    
    durations_annual = {}
    review_time_annual = {}
    publication_time_annual = {}
    editors_annual = {}
    
    received = []
    published = []
    
    for key in dat.keys():
        dur = []
        for pub in dat[key]:
            dt = date(*pub['publication_date'])-date(*pub['received_date'])
            dur.append(dt.days)
            
            if date(*pub['received_date']).year in durations_annual.keys():
                durations_annual[date(*pub['received_date']).year].append(dt.days)
            else:
                durations_annual[date(*pub['received_date']).year] = [dt.days]
                
            dt = date(*pub['accepted_date'])-date(*pub['received_date'])
            if date(*pub['received_date']).year in review_time_annual.keys():
                review_time_annual[date(*pub['received_date']).year].append(dt.days)
            else:
                review_time_annual[date(*pub['received_date']).year] = [dt.days]
                
            dt = date(*pub['publication_date'])-date(*pub['accepted_date'])
            if date(*pub['accepted_date']).year in publication_time_annual.keys():
                publication_time_annual[date(*pub['accepted_date']).year].append(dt.days)
            else:
                publication_time_annual[date(*pub['accepted_date']).year] = [dt.days]
            
            if date(*pub['received_date']).year in editors_annual.keys():
                editors_annual[date(*pub['received_date']).year].append(key)
            else:
                editors_annual[date(*pub['received_date']).year] = [key]
                
            received.append(date(*pub['received_date']))
            published.append(date(*pub['publication_date']))
        
        durations.append(dur)

Let's get a few details about the data we have.


    earliest_submission = min(received)
    latest_submission = max(received)
    earliest_publication = min(published)
    latest_publication = max(published)
    no_submissions = len(received)


    print 'number of submissios',no_submissions
    print 'earliest submission',earliest_submission
    print 'latest submission',latest_submission
    print 'earliest publication',earliest_publication
    print 'latest publication',latest_publication

    number of submissios 63753
    earliest submission 2006-08-01
    latest submission 2013-08-02
    earliest publication 2006-12-01
    latest publication 2013-07-31


Our data covers 63,753 publications in PLOS ONE -- with the earliest publication
dating back to 2006 and the most recent publication from 2013.

Let us take a closer look at the earliest and most recent submissions and
publications.


    for key in dat.keys():
        for pub in dat[key]:
            
            if date(*pub['publication_date']) == earliest_publication:
                print 'earliest publication',pub['id']
                
            if date(*pub['received_date']) == earliest_submission:
                print 'earliest submission',pub['id']
                
            if date(*pub['publication_date']) == latest_publication:
                print 'latest publication',pub['id']
                
            if date(*pub['received_date']) == latest_submission:
                print 'latest submission',pub['id']

    earliest submission 10.1371/journal.pone.0000003
    latest publication 10.1371/journal.pone.0070002
    latest publication 10.1371/journal.pone.0069696
    latest publication 10.1371/journal.pone.0069531
    earliest publication 10.1371/journal.pone.0000044
    latest publication 10.1371/journal.pone.0070022
    latest publication 10.1371/journal.pone.0069692
    latest publication 10.1371/journal.pone.0066849
    latest publication 10.1371/journal.pone.0069684
    latest publication 10.1371/journal.pone.0069848
    latest publication 10.1371/journal.pone.0069517
    latest publication 10.1371/journal.pone.0069574
    latest publication 10.1371/journal.pone.0070039
    latest submission 10.1371/journal.pone.0052595


The earliest submission in our dataset carries a doi ending in **3** and not
**1** and the most recent submission in our dataset with the doi
**10.1371/journal.pone.0052595** seems to have mislabeled meta data:

The submission date for this article is August 2, 2013 and its publication date
is given as  January 28, 2013.

This seems to be the only oddity in our dataset.


    no_annual_publ = {}
    no_annual_recv = {}
    
    for pub in published:
        if pub.year in no_annual_publ.keys():
            no_annual_publ[pub.year]+=1
        else:
            no_annual_publ[pub.year] = 1
            
    for pub in received:
        if pub.year in no_annual_recv.keys():
            no_annual_recv[pub.year]+=1
        else:
            no_annual_recv[pub.year] = 1


    print 'annual received',no_annual_recv
    print 'annual published',no_annual_publ

    annual received {2006: 265, 2007: 1943, 2008: 3532, 2009: 5019, 2010: 9590, 2011: 16966, 2012: 21140, 2013: 5298}
    annual published {2006: 137, 2007: 1170, 2008: 2757, 2009: 4631, 2010: 6498, 2011: 13864, 2012: 17287, 2013: 17409}


The annual numbers of publications in our dataset follow closely those stated on
[Wikipedia](http://en.wikipedia.org/wiki/PLOS_ONE) -- with the exception of the
year 2012.

Graphically, this data looks as follows:


    fig, ax = pl.subplots()
    ax.plot(no_annual_recv.keys(), no_annual_recv.values(),'o',color='blue')
    ax.plot(no_annual_publ.keys(), no_annual_publ.values(),'o',color='red')
    ax.ticklabel_format(useOffset=False)
    ax.set_xlim(2005,2014)
    ax.set_ylim(-500,22000)
    ax.set_xlabel('year')
    ax.set_ylabel('no. of articles')
    pl.show()


![png](https://dl.dropboxusercontent.com/u/129945779/georgio/plos_one_files/plos_one_15_0.png)


There seems to be a widening gap between the number of articles published (red
disks) and received (blue disks) annually. Since all articles in our dataset
were published eventually this trend may point to an increase in review time
over the years.


    fig, ax = pl.subplots()
    da_median = [np.median(durations_annual[key]) for key in durations_annual.keys()]
    ax.plot(durations_annual.keys(), da_median, 'o', color='blue')
    ax.ticklabel_format(useOffset=False)
    ax.set_xlim(2005,2014)
    ax.set_xlabel('year')
    ax.set_ylabel('median total time to publication / days')
    pl.show()


![png](https://dl.dropboxusercontent.com/u/129945779/georgio/plos_one_files/plos_one_17_0.png)


In the above plot we show the median number of days between submission and
publication sorted by the year of submission -- i.e. an article submitted in
2006 and published in 2007 is included in the data point for 2006.

There seems to be a clear increase in the number of days that authors have to
wait before their articles are published. The median for 2013 is relatively low
since any article submitted in 2013 and published in the same year would lie at
the low end of the distribution of wait times -- those articles that will raise
the median time to publication are likely to get published later this year or
even next year.

The **total time to publication** we plotted above includes the entire time span
between submission and publication of articles. Let us break this down into the
time spans between submission to acceptance and acceptance to publication of the
articles in our dataset:

* By **review time** we mean the amount of time between submission and
acceptance, and
* by **publication time** we mean the amount of time between acceptance and
publication.


    fig, ax = pl.subplots()
    rta_median = [np.median(review_time_annual[key]) for key in review_time_annual.keys()]
    pta_median = [np.median(publication_time_annual[key]) for key in publication_time_annual.keys()]
    ax.plot(publication_time_annual.keys(), pta_median, 'o', color='red', markersize=10)
    ax.plot(review_time_annual.keys(), rta_median, 'o', color='blue')
    ax.ticklabel_format(useOffset=False)
    ax.set_xlim(2005,2014)
    ax.set_ylim(20, 120)
    ax.set_xlabel('year')
    ax.set_ylabel('median time / days')
    pl.show()


![png](https://dl.dropboxusercontent.com/u/129945779/georgio/plos_one_files/plos_one_21_0.png)



    rta_median




    [47.0, 80.0, 83.0, 90.0, 105.0, 110.0, 108.0, 76.0]



While the median publication time (red disks) is relatively stable, the median
review time shows a steady increase -- the **median review time more than
doubles between 2006 and 2012** (ignoring 2013 for the same reason as stated
above).

Even if one ignored 2006 as a special year for PLOS ONE then the **median review
time increased by roughly one month between 2007 and 2012**.

One can speculate what causes this steady increase in review time. The one
possible factor we can look at with our dataset is the ratio of editors to
received (and published) articles.


    ea = [len(set(editors_annual[key])) for key in editors_annual.keys()]


    ea




    [153, 532, 637, 807, 1517, 2295, 3381, 2087]



These figures are the numbers of editors that worked on PLOS ONE submissions
received in the corresponding year: An article received in 2006 and published in
2007 is included in the editor count for 2006.

From these figures we can see that **in 2006 153 editors** worked for PLOS ONE
while this figure was **3,381 in 2012**.

Let us now see how the ratio of received (and published) articles to editors
scaled over time.


    fig, ax = pl.subplots()
    ed_recv_ratio = [float(recv)/float(ed) for recv, ed in zip(no_annual_recv.values(), ea)]
    ax.plot(editors_annual.keys(), ed_recv_ratio, 'o', color='red')
    ax.ticklabel_format(useOffset=False)
    ax.set_xlim(2005,2014)
    ax.set_xlabel('year')
    ax.set_ylabel('no. received articles / editors')
    pl.show()


![png](https://dl.dropboxusercontent.com/u/129945779/georgio/plos_one_files/plos_one_29_0.png)



    ed_recv_ratio




    [1.7320261437908497,
     3.6522556390977443,
     5.544740973312402,
     6.219330855018588,
     6.321687541199736,
     7.392592592592592,
     6.252587991718427,
     2.5385721130809773]



The **ratio of submitted articles to editors** handling those submissions
increased from around **1.73 in 2006** to around **6.25 in 2012** with a peak of
**7.39 in 2011**. These values would mean that on average one editor handled
just over 6 manuscripts per year.

This increase in the average number of submissions handled by editors may
contribute to the observed increase in review time and total time to
publication.

Mean values are sometimes biased by extreme values in the dataset and we should
probably look at the median number of submissions handled by each editor sorted
by year.

Since there appears to be some editorial impact, let us look at the spread of
the **total time to publication** across the editors in our dataset.

Our *durations* variable holds the total time to publication for all editors in
days. If an editor appears on multiple published papers then *durations*
contains a list of total times for that editor.

Here is a sample of what this looks like:


    print durations[0]
    print durations[2]

    [127, 150, 95, 120, 144, 62, 84]
    [140]


**Editor 0** appears on **7 published articles** while **editor 2** appears on
**1 published article** -- this one article took a total of 140 days to be
published (from the date of submission to the date of publication).

Let us analyze the median total time per editor. To this end we plot a histogram
of the median total time (per editor).


    n, bins, patches = pl.hist([np.median(ed) for ed in durations], normed=True, bins=20)
    pl.xlabel('median total time to publication / days')
    pl.ylabel('frequency')
    pl.show()


![png](https://dl.dropboxusercontent.com/u/129945779/georgio/plos_one_files/plos_one_37_0.png)



    print sorted([(count, tt) for count, tt in zip(n,bins)], key=lambda x: x[0], reverse=True)

    [(0.011719646020428425, 125.2), (0.0082885654420793084, 151.75), (0.0073324890562022575, 98.650000000000006), (0.0037626231960322851, 178.30000000000001), (0.0025443968333824854, 72.099999999999994), (0.0014957969262915218, 204.84999999999999), (0.00076331905001474597, 45.549999999999997), (0.00070934699597329821, 231.40000000000001), (0.00038551467172461982, 257.95000000000005), (0.00022359850960027903, 284.5), (0.00013107498838637046, 311.05000000000001), (0.00010794410808289338, 19.0), (7.71029343449238e-05, 337.60000000000002), (6.1682347475939173e-05, 364.15000000000003), (1.5420586868984759e-05, 470.35000000000002), (1.5420586868984759e-05, 496.90000000000003), (7.7102934344923966e-06, 523.45000000000005), (7.7102934344923796e-06, 390.69999999999999), (7.7102934344923796e-06, 417.25), (7.7102934344923796e-06, 443.80000000000001)]


We observe a peak in the frequency of the median total times at around **125
days**. This most likely median is slightly higher than the median [reported for
a subset of our dataset](http://metarabbit.wordpress.com/2013/06/03/how-long-
does-plos-one-take-to-accept-a-paper/).

Let us visualize the spread of the median total times in a scatter plot:


    pl.scatter(range(len(durations)), [np.median(ed) for ed in durations])
    pl.xlabel('editor')
    pl.ylabel('median total time to publication / days')
    pl.show()


![png](https://dl.dropboxusercontent.com/u/129945779/georgio/plos_one_files/plos_one_41_0.png)


It is surprising to see that **for some editors** the **median total time to
publication** is greater than **300 days**.

Are these editors that handled one, yet *tough*, submission?


    print [len(ed) for ed in durations if np.median(ed) >= 300]
    print 'number of editors with median >= 300 and one submission', len([ed for ed in durations if np.median(ed) >= 300 and len(ed)==1])
    print 'number of editors with median >= 300 and more than one submission', len([ed for ed in durations if np.median(ed) >= 300 and len(ed)>1])

    [1, 2, 1, 1, 5, 3, 1, 1, 1, 1, 1, 1, 3, 1, 1, 1, 2, 4, 1, 1, 1, 3, 5, 1, 1, 1, 1, 4, 1, 3, 1, 3, 2, 2, 1, 1, 1, 1, 1, 2, 1, 1, 4, 1, 3, 2, 5, 1, 3, 1, 8, 1, 3]
    number of editors with median >= 300 and one submission 32
    number of editors with median >= 300 and more than one submission 21


As the above shows, **32 editors** whose handled submissions had a **median
total time to publication above 300 days** have only handled one single
submission -- however **21 editors** had handled more than one submission.

One editor has even handled eight submissions, all of which had consistently
long total times to publication:


    print [ed for ed in durations if np.median(ed) >= 300 and len(ed) == 8]

    [[325, 265, 580, 272, 427, 158, 369, 309]]


If we classify editors by how many manuscripts they have handled from submission
to publication, then the above figures seem to indicate that outliers (long
median total times) occur across multiple classes.

Let us see if general patterns in total time to publication repeat themselves
across difference classes.

To this end we put all observed total times to publication into classes of
experience -- we classify our dataset by how many submissions have been handled
by the corresponding editors.


    classes = {}
    for ed in durations:
        a_class = len(ed)
        if a_class in classes.keys():
            classes[a_class].append(ed)
        else:
            classes[a_class] = [ed]


    print 'least number of submissions handled to publication', min(classes.keys())
    print 'greatest number of submissions handled to publication', max(classes.keys())

    least number of submissions handled to publication 1
    greatest number of submissions handled to publication 441



    print len(classes[441])

    1


Our dataset seems to include one editor that has handled 441 manuscripts from
submission to publication.

Is it possible that these are in fact multiple editors with the same name? **A
quick check of our data confirms that this particular editor and other editors
that have handled a large number of manuscripts appear to be unique names.**

Let us now visualize the spread of total times to publication for each class in
a box plot.


    bp_data = []
    for cl in classes.keys():
        dummy = []
        for ed in classes[cl]:
            for val in ed:
                dummy.append(val)
        bp_data.append(dummy)
        
    fig, ax = pl.subplots()
    ax.boxplot(bp_data, positions=classes.keys())
    ax.set_xlabel('number of submissions handled')
    ax.set_ylabel('total time to publication / days')
    ax.set_xticks(range(1,max(classes.keys())+50, 50))
    pl.show()


![png](https://dl.dropboxusercontent.com/u/129945779/georgio/plos_one_files/plos_one_52_0.png)


Apart from the aforementioned negative total time (DOI:
10.1371/journal.pone.0052595) that is due to mislabeled meta data, we observe
one extreme outlier near 2000 days.

Apart from these outliers most total times appear to fall into a range between 0
and 1,000 days. Let us zoom into this time frame:


    bp_data = []
    for cl in classes.keys():
        dummy = []
        for ed in classes[cl]:
            for val in ed:
                dummy.append(val)
        bp_data.append(dummy)
        
    fig, ax = pl.subplots()
    ax.boxplot(bp_data, positions=classes.keys())
    ax.set_xlabel('number of submissions handled')
    ax.set_ylabel('total time to publication / days')
    ax.set_xticks(range(1,max(classes.keys())+50, 50))
    ax.set_ylim(0, 1200)
    pl.show()


![png](https://dl.dropboxusercontent.com/u/129945779/georgio/plos_one_files/plos_one_54_0.png)


This boxplot suggests that there is no clear correlation between the number of
articles handled and the total time to publication for individual articles.

Put another way, the **experience of editors does not appear to alter the total
time to publication of their handled submissions**: less experienced and more
experienced editors seem to be largerly indistinguishable from one another.

Let us look at the spread of total times between editors in the same class.

As a first step, let us look at the one editor that has handled 441 submissions
to date.


    ed_id = None
    ed_durations = None
    for ed_i, ed in enumerate(durations):
        if len(ed) == 441:
            ed_id = ed_i
            ed_durations = ed
    
    print ed_id

    977


The one editor that has handled 441 submissions is **editor 977** in our
dataset.


    pl.scatter(range(len(durations[ed_id])), durations[ed_id])
    pl.xlabel('submission number handled by editor 977')
    pl.ylabel('total time to publication / days')
    print 'median total time for editor 977:',np.median(durations[977])

    median total time for editor 977: 102.0



![png](https://dl.dropboxusercontent.com/u/129945779/georgio/plos_one_files/plos_one_59_1.png)


The median total time to publication for editor 977 is 102 days over the 441
submission they have handled.

What is the overall pattern in the median total time to publication across all
editors irrespective of the number of submissions handled?


    pl.scatter(range(len(durations)), sorted([np.median(ed) for ed in durations]))
    pl.xlabel('editor')
    pl.ylabel('median total time to publication / days')
    pl.xlim(-100,len(durations)+100)
    pl.ylim(0, 600)
    pl.show()


![png](https://dl.dropboxusercontent.com/u/129945779/georgio/plos_one_files/plos_one_62_0.png)


Here we plot the median total time to publication for each editor in our
dataset, sorted by that median value.

There is still a multitude of potentially important factors in this data, such
as the editor's experience and the scientific field their assigned submissions
fall into.

Let us take a look at the potential effects due to the editor's experience.
Since at least one class of experience (441 handled submissions) contains just
one editor, we will restrict the following plots to just those classes that
contain at least 50 editors and fewer than 200 editors -- otherwise we would
just have too many curves to look at.


    for cl in classes.keys():
        dummy = []
        for ed in durations:
            if len(ed) == cl:
                dummy.append(ed)
        if len(dummy) > 50 and len(dummy) < 200:
            pl.scatter(range(len(dummy)), sorted([np.median(ed) for ed in dummy]))
            print 'class',cl
            
    pl.xlabel('editor')
    pl.ylabel('median total time to publication / days')
    pl.show()

    class 7
    class 8
    class 9
    class 10
    class 11
    class 12
    class 13
    class 14
    class 15
    class 16
    class 17
    class 18
    class 19
    class 20
    class 21
    class 23



![png](https://dl.dropboxusercontent.com/u/129945779/georgio/plos_one_files/plos_one_64_1.png)


Visualizing the median total time to publication for this narrow band of editor
classes (between 7 and 23 handled submissions) shows at least two things:

* The maximal median values of around 600 days are not captured by this subset
of classes, but, and more importantly,

* there is a pattern in median total times that repeats itself across the
different classes considered here: all classes appear to vary between the same
extrema in median total times.

This scatter plot seems to corroborate the conclusion we have drawn from the
above boxplots: Editorial experience does not appear to impact the total time to
publication of the submissions they handle.
