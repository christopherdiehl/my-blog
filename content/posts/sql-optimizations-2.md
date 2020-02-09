+++
date = "2020-02-08T07:24:44-05:00"
title = "Optimizing SQL queries on a Raleigh's crime dataset"
tags = ["sql","optimization", "Raleigh"]
keywords = ["sql optimization","sql explain analyze", "Raleigh crime data"]
draft = true
+++

#### This post builds upon the previous post. If you haven't checked it out already, please do so [here](https://christopherdiehl.github.io/posts/sql-optimizations/)


In my opinion, SQL is easy to get working initially, but optimizing SQL is a whole different beast. Thankfully, I recently had the chance to optimize a whole bunch of SQL for one of our critical platforms resulting in an over 300% reduction in execution time. While I can't promise the same result for anyone following this article, I do outline some of my learnings below using the most recent Raleigh Crime dataset.

## First query.


If you want to read more about Postgres query plans or the different scanning methods available please checkout the following resources:

- https://www.postgresql.org/docs/11/planner-optimizer.html
- https://severalnines.com/database-blog/overview-various-scan-methods-postgresql
- https://www.postgresql.org/docs/8.1/performance-tips.html
- https://dba.stackexchange.com/questions/119386/understanding-bitmap-heap-scan-and-bitmap-index-scan