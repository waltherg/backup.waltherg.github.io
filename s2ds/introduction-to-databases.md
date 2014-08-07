---
layout: default
title: "S2DS Introduction to Databases"
date: 2014-08-07
tags: s2ds
---

# Introduction to Databases

These notes are based on the intro to databases that
[Matthew Eric Bassett](https://twitter.com/mebassett) delivered at S2DS 2014.

## Database Management System (DBMS)

- simplest example is a file system
- needs to provide an interface between stored files and what we call ...?
- manages file storage and user permissions (security)
- records access patterns (to time requests)
- ACID compliance:
    * Atomicity: all or nothing communication, no partial data updates
    * Consistency: all defined rules are always upheld
    * Isolation: no crosstalk between users and their actions
    * Durability: data are backed up etc.

## Database Models

- relational, based on relations (i.e. tables)
- [object-oriented databases](https://en.wikipedia.org/wiki/Object_database)
- graph databases
- key/value: e.g. memcache, Redis
- SQLite: file-based

## [Relational Model](https://en.wikipedia.org/wiki/Relational_model)

- "schema" and "database" are used interchangeably here
- a schema is a set of tables
- "data types" are understood in the sense of C, Java, etc. data types
- attributes are columns in a table
- tuples are rows in a table
- relations are tables / are encoded in tables
- primary key (unique) or keys (?)
    * primary keys are unique identifiers of tuples in a table
    * can consist of one or multiple attributes
    * are usually an artificial integer attribute "ID" that is not
      related to the real world modeled by your schema
- side note: NoSQL databases oftentimes are about not predefining a
  schema but rather hoping that one emerges over time and with experience

## Subqueries and Joins

- subqueries are queries within another query
- [Joins](https://en.wikipedia.org/wiki/Join_(SQL))
- inner join v. outer join
    * [StackOverflow: INNER and OUTER joins](https://stackoverflow.com/questions/38549/difference-between-inner-and-outer-joins)

## Random Notes

- `tinyint(1)` == `bool`
- [InnoDB](https://en.wikipedia.org/wiki/InnoDB) is a MySQL storage engine
  that provides ACID-compliant transactions
- use dagger quotes around variable names to use names that correspond
  to reserved keywords
- MySQL is case-sensitive if your operating system is case sensitive
  (e.g. Windows is not case sensitive)
- `having` v. `where`
    * `having` for grouped attributes
    * `where` for ungrouped attributed
    * [MySQL: HAVING vs. WHERE](http://lists.mysql.com/mysql/134036)
    * [StackOverflow HAVING vs. WHERE](https://stackoverflow.com/questions/2905292/where-vs-having)
- indexing
    * [index types available in Microsoft SQL Server](http://msdn.microsoft.com/en-us/library/ms175049.aspx)
    * [an explanation of indexing with binary trees](https://www.simple-talk.com/sql/learn-sql-server/sql-server-index-basics/)
- trigger
    * procedures that fire when a predefined database event occurs
    * [CREATE TRIGGER in Microsoft SQL Server](http://msdn.microsoft.com/en-us/library/ms189799.aspx)
- transactions
