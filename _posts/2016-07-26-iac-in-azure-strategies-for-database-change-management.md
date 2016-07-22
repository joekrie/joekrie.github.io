---
layout: post
title:  "Welcome to Jekyll!"
date:   2016-07-21 23:59:30 -0500
categories: jekyll update
---

Strategies for Database Change Management
=========================================

*This is the fourth post in a series on Microsoft tooling and services to enable teams following DevOps principles.*

When adopting continuous delivery principles for a system, whether it be greenfield or legacy, one of the biggest challenges in managing database changes. Databases provide a challenge for the following reasons. First, they store operational data. Second, particularly for relational databases, schema changes can cause operational data to either be lost if a column is removed, or become invalid if a required column is added, or a data type changes.

Entity Framework Migrations
---------------------------

Entity Framework is a data access framework that provides an object-relational mapper and database migration tools. 

SQL Server Data Tools
---------------------

While Entity Framework provides a fairly comprehensive set of database change management tools, many developers prefer to avoid the high level of abstraction that it provides. Working directly with SQL and SQL Server allows you to take advantage of the intricacies of SQL Server.

RavenDB Rolling Migrations
--------------------------

While using a document database mitigates most of the problems we encounter when migrating a relational database, there are a few cases.

References
----------

* Jez Humble, Continuous Delivery
* https://ayende.com/blog/66563/ravendb-migrations-rolling-updates
* http://www.sqlhammer.com/database-deployment-work-flow-with-sql-server-data-tools-ssdt/
