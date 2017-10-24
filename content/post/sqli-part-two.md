+++
Categories = []
Description = "Continuing the series on SQL injection, in this post we're going to get an understanding of how a sql vulnerability can used to extract data. As well as get a better handle on the union operator and work through some examples"
Tags = ["SQL injection"]
date = "2017-10-20T11:19:55+13:00"
title = "SQL injection - part two"

+++

This is continuation of the series on SQL injection. In part one we covered what SQL injection is, some examples of what can happen when someone leaves their site open to SQLI, and a quick and dirty test to pick up some low hanging fruit.
In this post we're going to go a bit further into how one would actually exploit a SQL vulnerability, how to figure out what type of database you're working with, and how to us the union operator to extract data, and walkthrough a practical example by testing out new found knowledge against an intentionally vulnerable app.

#### Fingerprinting the Database

There are two main distinctions of databases, there are SQL db's and there are NoSQL dbs's. The difference being surprisingly enough their use of the SQL language. Since we're learning about SQL injection we're only going to concentrate on the ones using SQL. The fundamentals of injection are often the same across different types of SQL based db's, but there are little differences just to make things harder. We're only going to concentrate on three common databases, MySql, Microsoft SQL Server, and Oracle.

You might simply be able to grab the banner by replacing the query to be evaluated with one of the following:

- MS SQL: `SELECT @@version`

- MySQL: `SELECT version()`

- Oracle: `SELECT version FROM v$instance`

You can also try using string concatenation to rule out db's, they handle the following ways of sticking strings together:

- MS SQL: `'a' + 'a'`

- MySQL: `CONCAT('a','a')`

- Oracle: `'a' || 'a' or CONCAT('a', 'a')`

So if you have a vulnerabilty expecting a user id, you can feed it something like `1' AND 'aa' = 'a'+ 'a' -- `, if you get an error you'll know `1' AND 'aa'=CONCAT('a','a') -- `

#### The union operator

walkthrough of a union attack against dvwa
