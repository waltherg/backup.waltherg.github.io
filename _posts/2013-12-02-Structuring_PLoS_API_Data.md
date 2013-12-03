---
layout: default
date: 2013-12-02
title: "Structuring PLoS API Data"
tags: Python XML OpenData PLoS
---

# Structuring PLoS API Data

A while ago [I started
using](http://georg.io/2013/08/12/PLoS_ONE_time_to_publication.html) data
provided by the PLoS API.

The PLoS API allows you to download entire articles as [XML
documents](http://en.wikipedia.org/wiki/XML):
you can download batches of articles complete with title, abstract, body (the
actual article) and metadata such as date of publication.

The only problem with XML is that you need to
[parse](http://en.wikipedia.org/wiki/Parsing) it to rediscover the structure of
your document
(e.g. a document contains one title, one abstract, etc.) - this task essentially
turns your XML document into human-readable format.

I encounter continually this parsing problem when wishing to work with the PLoS
data I downloaded and so decided to put together a small
Python module that encapsulates all of these steps and that I can reuse.

Python modules that do exactly this probably exist already - but reinventing the
wheel is part of the learning process ... right?

Nevertheless, some of the Python modules I should probably look into are:

* https://code.google.com/p/plos-search/

## Trial and Error

I will first attempt to find a solution through trial and error and only later
combine everything I find into a Python module.

First off, we know we need to load a number of XML files and parse these: `os`
is the standard Python module for path operations and
[`BeautifulSoup`](http://www.crummy.com/software/BeautifulSoup/) is the Swiss
Army knife for XML and HTML parsing.

We will need to use the class `BeautifulSoup` found in the module
`BeautifulSoup` - hence the somewhat confusing import below.

    import os
    from BeautifulSoup import BeautifulSoup

We know where the PLoS XML files are located relative to this IPython Notebook
(`relative_path`).
We also know that all XML files in that particular directory have the file
extension `.dat`.

From this we work out the path to each XML file with [`os.path.realpath`](http:/
/docs.python.org/2/library/os.path.html#os.path.realpath) so that we can load
each XML file easily.

    relative_path = '../plos/plos_one/plos_one_dump/'
    filenames = os.listdir(relative_path)
    files = [os.path.realpath(relative_path+name) for name in filenames if '.dat' in name]

Let us now focus on just one of these XML files, `files[0]`, open this file and
read its content.

To exemplify what these XML files look like, we print the first 1,000 characters
of this file.

    f_name = files[0]
    f = open(f_name, 'rb')
    f_content = f.read()
    f.close()

Let us print the first 1,000 characters of the content of this file.

    print f_content[:1000]

    <?xml version="1.0" encoding="UTF-8"?>
    <response>
    <lst name="responseHeader"><int name="status">0</int><int name="QTime">1402</int></lst><result name="response" numFound="63683" start="15600"><doc><str name="id">10.1371/journal.pone.0067899</str><arr name="subject_facet"><str>/Biology and life sciences/Genetics/Genomics/Animal genomics/Mammalian genomics</str><str>/Biology and life sciences/Genetics/Alleles</str><str>/Computer and information sciences/Artificial intelligence/Machine learning</str><str>/Biology and life sciences/Neuroscience/Cognitive science/Artificial intelligence/Machine learning</str><str>/Physical sciences/Mathematics/Statistics (mathematics)/Statistical methods/Regression analysis/Linear regression analysis</str><str>/Biology and life sciences/Genetics/Gene expression</str><str>/Biology and life sciences/Genetics/Genetic loci</str><str>/Physical sciences/Physics/Thermodynamics/Entropy</str><str>/Research and analysis methods/Mathematical and statistical techniques

We can already make out one of the articles stored in this particular XML file:
An opening `<doc>` tag is followed by a named tag, `name=id`, that contains the
[DOI](http://en.wikipedia.org/wiki/Digital_object_identifier)
of this article. We can also see one of the metadata tags, `subject_facet`,
provided by the PLoS API.

Let us now parse this XML document with the class `BeautifulSoup` that we
imported earlier.

To parse this XML document, we pass the object that holds the content of the
document, `f_content`, to the constructor of `BeautifulSoup` and instantiate an
object of `BeautifulSoup`.

    f_xml = BeautifulSoup(f_content)

Above we realized that the tag `doc` encloses individual articles. Let us fetch
all articles stored in this XML document through this tag.

    docs_list = f_xml.findAll('doc')

The variable `docs_list` holds a list of `BeautifulSoup.Tag` variables that each
encompass one `doc` tag of the XML document.

    print type(docs_list[0])

    <class 'BeautifulSoup.Tag'>

Let us see how many articles we have in this one XML document.

    print len(docs_list)

    100

As we can see, this one XML document contains 100 articles.

Let us focus on just one article, `docs_list[0]`.

    doc = docs_list[0]

BeautifulSoup generates a [parse tree](http://en.wikipedia.org/wiki/Parse_tree)
from the XML document we feed it with.

Considering a single `doc` tag we essentially look at a
[subtree](http://en.wikipedia.org/wiki/Tree_(data_structure)
of the parse tree with the corresponding `doc` tag at the root of that subtree.

Therefore, to look at all tags contained within this `doc` tag we want to get
the children of the `doc` tag in the parse tree.

    children = doc.findChildren()

Let us print out a number of the children we find.

    print children[:30:2]

    [<str name="id">10.1371/journal.pone.0067899</str>, <str>/Biology and life sciences/Genetics/Genomics/Animal genomics/Mammalian genomics</str>, <str>/Computer and information sciences/Artificial intelligence/Machine learning</str>, <str>/Physical sciences/Mathematics/Statistics (mathematics)/Statistical methods/Regression analysis/Linear regression analysis</str>, <str>/Biology and life sciences/Genetics/Genetic loci</str>, <str>/Research and analysis methods/Mathematical and statistical techniques/Statistical methods/Regression analysis/Linear regression analysis</str>, <str name="journal">PLoS ONE</str>, <date name="received_date">2012-12-13T00:00:00Z</date>, <int name="pagecount">9</int>, <str>Tao Huang</str>, <arr name="editor_facet"><str>Xinping Cui</str></arr>, <arr name="editor_display"><str>Xinping Cui</str></arr>, <arr name="author_affiliate"><str>Institute of Systems Biology, Shanghai University, Shanghai, P. R. China</str><str>Department of Genetics and Genomic Sciences, Mount Sinai School of Medicine, New York, New York, United States of America</str></arr>, <str>Department of Genetics and Genomic Sciences, Mount Sinai School of Medicine, New York, New York, United States of America</str>, <str>University of California, Riverside, United States of America</str>]


Looking at a small selection of children of `doc` we realize that all tags that
have contextual meaning to us are named, such as `name=id` or `name=journal`.
XML tags that are not named are in fact children of named tags.

Let us therefore extract all tag names as the identifiers of the data and
metadata that describe each article.


    identifiers = []
    for child in children:
        try:
            identifiers = identifiers + [child['name']]
        except KeyError:
            continue

These are the `identifiers` we find:

    identifiers

    [u'id',
     u'subject_facet',
     u'journal',
     u'publication_date',
     u'received_date',
     u'accepted_date',
     u'pagecount',
     u'author_facet',
     u'editor_facet',
     u'editor_display',
     u'author_affiliate',
     u'editor_affiliate',
     u'abstract',
     u'body']

It now becomes apparent that a handy data structure to save articles is a
dictionary whose keys are the identifiers we found and whose values are the
content of the corresponding XML tags.

For instance, to extract the content of the identifier `id` we call `find` on
our `doc` object. The content of this identifier is the DOI of the corresponding
article.

    doi = doc.find(True, {'name': 'id'}).contents[0]

And this is the DOI of this particular article:

    print doi

    10.1371/journal.pone.0067899

Let us prepare our dictionary and save the contents of our identifier XML tags
apropriately.


    docs = {}
    for ident in identifiers:
        docs[ident] = doc.find(True, {'name': ident}).contents

The resultant dictionary is easy and intutive to query but some of the saved
values, for instance in `docs['abstract']` still contain XML tags `<str>` that
we should remove.

These are the keys of the dictionary `docs`:

    docs.keys()

    [u'body',
     u'subject_facet',
     u'editor_display',
     u'pagecount',
     u'journal',
     u'abstract',
     u'author_facet',
     u'editor_facet',
     u'received_date',
     u'author_affiliate',
     u'publication_date',
     u'editor_affiliate',
     u'id',
     u'accepted_date']

We can query for the DOI of the article easily:

    docs['id']

    [u'10.1371/journal.pone.0067899']

Other data such as the abstract is also avaiable. Note, however, the XML tag
`<str>` that we carried through and is now stored in our dictionary. We find the
same superfluous tag in other article properties,
such as `docs['subject_facet']`.

    docs['abstract']

    [<str>
    Expression Quantitative Trait Locus (eQTL) analysis is a powerful tool to study the biological mechanisms linking the genotype with gene expression. Such analyses can identify genomic locations where genotypic variants influence the expression of genes, both in close proximity to the variant (cis-eQTL), and on other chromosomes (trans-eQTL). Many traditional eQTL methods are based on a linear regression model. In this study, we propose a novel method by which to identify eQTL associations with information theory and machine learning approaches. Mutual Information (MI) is used to describe the association between genetic marker and gene expression. MI can detect both linear and non-linear associations. What’s more, it can capture the heterogeneity of the population. Advanced feature selection methods, Maximum Relevance Minimum Redundancy (mRMR) and Incremental Feature Selection (IFS), were applied to optimize the selection of the affected genes by the genetic marker. When we applied our method to a study of apoE-deficient mice, it was found that the cis-acting eQTLs are stronger than trans-acting eQTLs but there are more trans-acting eQTLs than cis-acting eQTLs. We compared our results (mRMR.eQTL) with R/qtl, and MatrixEQTL (modelLINEAR and modelANOVA). In female mice, 67.9% of mRMR.eQTL results can be confirmed by at least two other methods while only 14.4% of R/qtl result can be confirmed by at least two other methods. In male mice, 74.1% of mRMR.eQTL results can be confirmed by at least two other methods while only 18.2% of R/qtl result can be confirmed by at least two other methods. Our methods provide a new way to identify the association between genetic markers and gene expression. Our software is available from supporting information.
    </str>]

Based on our previous assumptions about named and unnamed tags, a robust method
of removing these superfluous
`<str>` tags probably involves checking for and removing unnamed `<str>` tags
from our original XML data in `f_xml`.

    unnamed_str = f_xml.findAll('str', {'name': None})

Here is one of those unnamed `<str>` tags:

    unnamed_str[0]

    <str>/Biology and life sciences/Genetics/Genomics/Animal genomics/Mammalian genomics</str>

However, to go down that road we would need to modify the parse tree stored in
`f_xml`.

Let us make life easier for ourselves and just remove the offending `<str>` and
`</str>` tags from the resultant
strings stored in our dictionary's values.

    docs['subject_facet']

    [<str>/Biology and life sciences/Genetics/Genomics/Animal genomics/Mammalian genomics</str>,
     <str>/Biology and life sciences/Genetics/Alleles</str>,
     <str>/Computer and information sciences/Artificial intelligence/Machine learning</str>,
     <str>/Biology and life sciences/Neuroscience/Cognitive science/Artificial intelligence/Machine learning</str>,
     <str>/Physical sciences/Mathematics/Statistics (mathematics)/Statistical methods/Regression analysis/Linear regression analysis</str>,
     <str>/Biology and life sciences/Genetics/Gene expression</str>,
     <str>/Biology and life sciences/Genetics/Genetic loci</str>,
     <str>/Physical sciences/Physics/Thermodynamics/Entropy</str>,
     <str>/Research and analysis methods/Mathematical and statistical techniques/Statistical methods/Regression analysis/Linear regression analysis</str>,
     <str>/Biology and life sciences/Genetics/Heredity/Genetic mapping/Variant genotypes</str>]

To remove the `<str>` and `</str>` we replace these substrings with empty
strings.

The replace method is defined in the standard Python module
[`string`](http://docs.python.org/2/library/string.html#string.replace).

    str(docs['subject_facet'][0]).replace('<str>', '').replace('</str>', '')

    '/Biology and life sciences/Genetics/Genomics/Animal genomics/Mammalian genomics'

## Wrap Up

Let us now put everything that we worked out into a comprehensive class.

We will want to be able to pass the path to a PLoS XML file into this class and
receive a list
of dictionaries, one per article contained within the XML file.

    class PlosXml:
        from BeautifulSoup import BeautifulSoup
        
        def __init__(self, path):
            self.raw = None
            with open(path, 'rb') as f:
                self.raw = f.read()
                
            self.xml = BeautifulSoup(self.raw)
            docs_list = self.xml.findAll('doc')
            
            self.docs = []
            
            for doc in docs_list:
                doc_dict = {}
                children = doc.findChildren()
                
                identifiers = []
                for child in children:
                    try:
                        identifiers = identifiers + [child['name']]
                    except KeyError:
                        continue
                
                for ident in identifiers:
                    contents = doc.find(True, {'name': ident}).contents
                    contents = [str(cont).replace('<str>', '').replace('</str>', '') for cont in contents]
                    doc_dict[ident] = contents
                    
                self.docs = self.docs + [doc_dict]

Let us test this class with one of our XML files.

    docs = PlosXml(files[1])

This XML file also contains 100 articles:

    len(docs.docs)

    100

And here are the DOIs of some of the articles stored in this particular XML
file:

    [doc['id'] for doc in docs.docs[:5]]

    [['10.1371/journal.pone.0010777'],
     ['10.1371/journal.pone.0057451'],
     ['10.1371/journal.pone.0016395'],
     ['10.1371/journal.pone.0060403'],
     ['10.1371/journal.pone.0057450']]
