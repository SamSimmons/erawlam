+++
categories = []
description = "Continuing the series on SQL injection, in this post we're going to get an understanding of how a sql vulnerability can used to extract data. As well as get a better handle on the union operator and work through some examples"
tags = ["SQL injection"]
date = "2017-10-20T11:19:55+13:00"
title = "SQL injection - part two"

+++

This is continuation of the series on SQL injection. In part one we covered what SQL injection is, some examples of what can happen when someone leaves their site open to SQLI, and a quick and dirty test to pick up some low hanging fruit.
In this post we're going to go a bit further into how one would actually exploit a SQL vulnerability, how to figure out what type of database you're working with, and how to us the union operator to extract data, and walkthrough a practical example by testing out new found knowledge against an intentionally vulnerable app.

#### Fingerprinting the Database

There are two main distinctions of databases, there are SQL db's and there are NoSQL dbs's. The difference being surprisingly enough their use of the SQL language. Since we're learning about SQL injection we're only going to concentrate on the ones using SQL. The fundamentals of injection are often the same across different types of SQL based db's, but there are little differences just to make things harder. We're only going to concentrate on three common databases, MySql, Microsoft SQL Server, and Oracle.

There are a few different ways of figuring out the type of database you're up against, you might be able to tell from the error message printed when providing bad input.

- MS SQL: `Microsoft SQL Native Client error ‘80040e14’ Unclosed quotation mark after the character string`

- MySQL: `You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''''' at line 1`

- Oracle: `ORA-00933: SQL command not properly ended`

You might simply be able to grab the banner by replacing the query to be evaluated with one of the following:

- MS SQL: `SELECT @@version`

- MySQL: `SELECT version()`

- Oracle: `SELECT version FROM v$instance`

You can also use math functions to check, as the functions listed will only work on those databases:

- MS SQL: `SQUARE(1)`

- MySQL: `POW(1,1)`

- Oracle: `BITAND(1,1)`

All those functions evaluate to `1` so you can use them in a query like `http://fakesite.com/profile?id=2-POW(1,1)` which should error on an oracle or sql server db but gets you the correct page for a MySQL backed site. Once you know what database you're up against you know what syntax to use to investigate further and you'll have a better chance of exploiting the vulnerability further.


#### The union operator

The `Union` operator is used to mix together two SELECT statements into one set of results, it is one of the most common ways to get arbitary data when you've got a SQLI vulnerabilty that returns results directly. Using the union operator you can basically tack on a totally seperate nasty query with the one that's already running. In the previous post we used the example of a web app returning tv show information using something like `SELECT * FROM shows WHERE title = 'Bojack Horseman';`, we saw how a hacker could twist the query into returning everything in the shows table.

Getting that extra data probably isn't that interesting to a hacker, they want usernames and passwords. But by using the union operator they might be able to get just that. Entering the input `Bojack Horseman' UNION SELECT * FROM users --` would get the hacker all the data from the users table (if there is a table called users). There is one major gotcha to using the union operator, the union query must have the same number of columns as the first query and the column types must match.

If the shows table has three columns, `name`, `creator`, `star_rating`, and the users table might have `name`, `password`, `date_joined`, `email` then our earlier query would fail with an error about matching the number of columns. But you could do something like `Bojack Horseman' UNION SELECT name, password, email FROM users --` assuming the types matched.

There are a few fancy tricks around converting types to make sure they match and figuring out how many columns are in the first table but they're probably out of scope of this post. Let's get into the example.

#### Damn Vulnerable Web app

[Damn Vulnerable Web App](https://github.com/ethicalhack3r/DVWA) (DVWA) is an intentionally vulnerable web application created for people to learn about security in a safe and legal environment. I'm not going to go through the process of setting it up, there are pretty good instructions on the github page. But once you've got it up and running, you want to login with the username `admin` and the password `password`, you'll likely need to setup/reset the DB (by clicking the button) and in the DVWA Security tab, set the difficulty level to low. Then if you navigate to the SQL injection tab you should see a screen like this:

![dvwa screenshot](/sqli-two/1.png)

The way it's supposed to work is by entering a user id in the box and you'll get back some basic information about that user. If we test this input out by entering a `'` into the box, we get a error page saying to check our SQL syntax. Cool we know this input is vulnerable to sql injection. Also we know that we're working with a MySQL db since it mentions it in the error message. Helpful.

Let's try and get it to send back something it shouldn't. How about information for all the users in one query. We can probably guess that the server is doing something like `SELECT id, first_name, last_name FROM users WHERE id = 'something'`. So if we can feed the server an OR condition that's always true we can probably get everything in the table, try entering `' or 1=1 #` which is going to check each row in the table and see if the the id matches `''` (which it won't) or if the number 1 is equal to 1(which it always is) so return all the rows in the table.

![dvwa screenshot](/sqli-two/2.png)

Cool, we've verified that we can trick the server into returning extra information. But we would really love passwords rather than everyones names/ids. We want to use our new union trick, but to do that we need to know a few more things. We want the name of a table likely to hold users logins/passwords and we need to know the columns in this table so we can combine them with our union attack.

We know we're dealing with a MySQL table, to get a list of all the tables in a MySQL db you can run a query like `SELECT table_name FROM information_schema.tables`. If you're up against an oracle or SQL server db then the syntax will be different, something like `SELECT table_name FROM all_tables` for oracle and depending on the version of sql server `SELECT table_name FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_TYPE='BASE TABLE'`.

We can use our query to find the table names with the union query, the only trick is because the first query is expecting an integer as the first argument we'll need to use null in our union so the types won't clash. But using something like `' and 1=0 union select null, table_name from information_schema.tables #` we can get the server to send us back all the tables in the database. Using `1=0` makes sure we don't get any valid results in the first query, instead it only sends back table names. This is cool, we can see some interesting tables in there amongst lots of fluff. To filter out some of the MySQL specific fluff we could add in a where clause as well like `WHERE table_schema != 'mysql' AND table_schema != 'information_schema'`

Anyway you want to look through the table names and find something that might hold interesting data, the `users` table immedietly jumps out at me, it's probably got passwords and other sensitive information. Now that we've got a sql vulnerability open to union attacks and name for the table we want, we can dig a bit further. The next step will be to find column names. We use the same tactic as before, `' and 1=0 union SELECT null, COLUMN_NAME FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME = 'users'; #`

![dvwa screenshot](/sqli-two/3.png)

Cool, now we can see some columns we're interested in, `user` and `password` look particularly promising. Lets' grab some login credentials. Again we'll use the same process here, the only problem is we want to grab both the username and password hash and we've only got one slot to squeeze them in. No problem, we can use the concat function to stick both columns together. `' UNION SELECT null, concat(user, ':' ,password) FROM users #`

![dvwa screenshot](/sqli-two/3.png)

Cool, we now have usernames and passwords for all the users of the web app. The passwords are hashed though so we cant read them. These passwords happen to hashed using the MD5 algorithm, which can be cracked reasonably easily. If you're interested you can crack them with a program called `john the ripper` but I won't cover that here.
