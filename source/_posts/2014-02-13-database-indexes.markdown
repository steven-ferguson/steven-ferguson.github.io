---
layout: post
title: "Database Indexes"
date: 2014-02-13 15:43:48 -0800
comments: true
categories: [SQL, PostgreSQL, Rails]
---

Prior to today, I knew database indexes were something you used to help speed up a query, but I didn't really understand their importance. I read this great [Thoughtbot blog](http://robots.thoughtbot.com/a-grand-piano-for-your-violin) to learn more about them.

####Here is what I learned today:

1. When you perform a database query on a column without an index the database will look at each row and compare its value with the query value. If you don't have many records this isn't a big deal, but when you are talking about thousands or millions of rows that is a lot of work! Indexes provide a quick reference for the database, so it can retreive the matching records without having to look at every row. 

2. If you are using PostgreSQL (and apparently most other SQL databases), the primary key is index automatically. This means that in Rails the "id" column is indexed automatically because Rails tells the database the "id" column is the primary key.

3. Basically any column you have to query for a set of records should be indexed.

4. There are two downsides to adding an index. First, anytime you INSERT or UPDATE, the database must write both the record and the index reference which means more work for the database. Second, if you are creating an index for an existing set of records it could take a while. The Thoughtbot blog talks about a database migration that took 7 hours!  