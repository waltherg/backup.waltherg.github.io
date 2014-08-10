---
layout: default
title: "S2DS NoSQL"
date: 2014-08-09
tags: s2ds
---

# NoSQL

[Adam Fowler](https://twitter.com/adamfowleruk) of MarkLogic spoke to us
about NoSQL databases.
Check out his blog [here](http://adamfowlerml.wordpress.com/).

The following notes are based on his talk.

## Introduction

- Data are generally growing in volume, velocity of appearance,
  variety, and complexity
- Publishers popularized NoSQL (even though they did not call it NoSQL then)
  because the majority of their data come in the form of XML files
  (i.e. not as rows and columns)
- While relational databases assume that you can define a schema upfront
  we now encounter situations where the format is hard to predict
- NoSQL is generally not thought of as a replacement for relational databases
  but rather as a different technology to deal with different data

## What is NoSQL

- The term NoSQL was coined by [Carlo Strozzi](http://www.strozzi.it/) in 1998:
    * [Wikipedia article on Strozzi NoSQL](https://en.wikipedia.org/wiki/Strozzi_NoSQL_(RDBMS))
    * [Direct link to Strozzi NoSQL](http://www.strozzi.it/cgi-bin/CSA/tw7/I/en_US/nosql/Home%20Page)
- The term NoSQL was repopularized more recently by
  [Eric Evans](http://blog.sym-link.com/) and
  [Johan Oskarsson](https://twitter.com/skr)
- Historically we have been through a number of "data" (?) eras:
    * Hierarchical era in which mainframes would be used to perform
      relatively simple tasks on data
    * Relational era in which normalized structure became common ... ??
    * Any structure era in which data appear in a variety of formats
      (i.e. other than rows and columns)
- Data are stored in their raw format allowing us to browse and explore them
- Many NoSQL platforms are not ACID compliant
  (know the limitations of the NoSQL platform you are using!)
- The lack of ACID compliance in some NoSQL solutions is a sign of the
  evolving market
- In platforms that make use of a predefined schema, existing data need to
  be updated whenever the schema changes
- Often used to store meta data of the actual data ??

## Misconceptions About NoSQL

- Consistency (as in ACID compliance): Neo4j and MarkLogic are in fact
  consistent!
- SQL exposure is possible in the NoSQL world: SQL is just a language that
  can be mapped to queries in the NoSQL world

## NoSQL Vendor Differences

- Graph databases: Individual facts that are combined to a whole.
  (instances / units of
  [Resource Description Framework](https://en.wikipedia.org/wiki/Resource_Description_Framework) (RDF) triples are stored?)
- This probably relates to [triplestores](https://en.wikipedia.org/wiki/Triplestore)

## Bitemporality Problem

- [Bitemporal data warehouse](http://bifuture.blogspot.co.uk/2013/04/bi-temporal-data-warehouse.html)

## Random Notes

- [Wikibon: Big Data Vendor Revenue and Market Forecast](http://wikibon.org/wiki/v/Big_Data_Market_Size_and_Vendor_Revenues)
- [451 Research](https://451research.com/)
- [451 Research Data Platforms Landscape Map](http://blogs.the451group.com/information_management/2014/03/18/updated-data-platforms-landscape-map-february-2014/)
- [CODiE Awards](https://en.wikipedia.org/wiki/Codie_award)
- [Gartner Hype Cycle](http://www.gartner.com/technology/research/methodologies/hype-cycle.jsp)
- [Gartner Magic Quadrant for Enterprise Search](http://www.enterprisesearchblog.com/2014/07/gartner-mq-2014-for-search-surprise.html)
- [MarkLogic user group London](http://www.meetup.com/muglondon/)
- [Extract, transform, load (ETL)](https://en.wikipedia.org/wiki/Extract,_transform,_load)
- Search engine indexes:
    * faster than binary tree search above a certain volume of data
    * index needs to be updated periodically
- [Multiversion concurrency control](https://en.wikipedia.org/wiki/Multiversion_concurrency_control)
- Repeatable reads
    * when concurrent sessions query and update the same database, it may
      happen that a read query returns rows that are being altered by
      a concurrent session before the read operation is complete
    * to avoid this scenario, those rows that are part of the read
      query can be locked
    * [blog post about repeatable reads](http://blogs.msdn.com/b/craigfr/archive/2007/05/09/repeatable-read-isolation-level.aspx)
