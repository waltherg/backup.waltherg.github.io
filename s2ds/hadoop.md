---
layout: default
title: "S2DS Hadoop"
date: 2014-08-05
tags: s2ds
---

# Hadoop

[Chris Harris](https://twitter.com/cj_harris5) of Hortonworks gave us an
introduction to Hadoop.
The following notes are based on his talk.

## Motivation

- Sensors appear everywhere and produce large amounts of data
- More modern and interesting business intelligence (BI) entails questions
  about what a customer did before purchasing a product
  (more classical BI would look at the purchase event itself)
- Traditional systems are limited to tracking well-defined events
  such as purchase events (transactional data)
- New data types that become available (e.g. geolocation, tweets, etc.)
  appear in large volumes (cost barrier to storing all that) and most
  of this new data does not fit in rows and columns

## Historical Background

- Yahoo engineers were building a search engine
- Lots of small machines were used to store data distributed across
  these machines
- Data processing was required to happen on the machine where they were stored

## [Enterprise Hadoop](http://hortonworks.com/hadoop-modern-data-architecture/)

- A key idea is to store data in whatever format they appear in
- Data analysis is an iterative process where questions lead to other
  questions hence it is best to store data in its native format to remain
  flexible
- Components:
    * Hadoop Distributed File System (HDFS)
    * YARN: data operating system
    * Map Reduce: batch processing language
    * Falcon: data governance
    * Hive: open-source SQL
- General properties:
    * all data is replicated three times
    * [materialized views](https://en.wikipedia.org/wiki/Materialized_view) dependent on the user's permissions (data safety)

## Architecture of a Hadoop Setup

- ?

## R and Hadoop

- RHadoop
- RHDFS
- R map reduce

## Other Random Notes I Need to Make Sense Of

- siloed data v. data lake:
    * siloed: data split into business units
    * data lake: horizontal combination of data (across business units)
- there are three common types of analytics that are performed with
  [Hortonworks Data Platform (HDP)](http://hortonworks.com/hdp/)
- Mahout: scalable machine learning package in Java
- Apache Spark: streaming processing
- Tableau
- ORCFile: structured file format
