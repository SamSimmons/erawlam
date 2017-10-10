+++
date = "2017-07-15T11:50:22+12:00"
draft = false
title = "hello world"

+++

what is sql injection

Web apps have all kinds of good stuff that an attacker could want. Personal information, usernames, passwords, credit card information. Because there is lots of this good stuff, web apps need to keep it all organized, most common kind of data store is some form of sql database.

sql is the query language used to talk to the data store to tell it what you want. ** expand on sql have a code example of a simple sql query (use a example similar to calhoun go blog)

if the developer is not careful when using client provided data they can leave themselves open to sqli. By modifying the data sent up to the server/db they can make the app provide data it really shouldn't or provide access to an area the attacker shouldn't be able to get too (eg bypassing login) or even take control of the server the db is running on

what types of sqli are there and what do they look like

basic vulnerability

say you have a web app badNetflix which allows users to search for tv shows. A user comes on and wants to find a hilarious show about a horse in hollywood. They type 'Bojack Horseman' in the searchbar and hit enter. A request goes up to the server with the string 'Bojack Horseman' and the server goes to the database and forms a query like SELECT * FROM shows WHERE title = 'Bojack Horseman' which will search all the rows in the database for any shows with a matching title, and return them to the web app to display them as search results.

Now say the user decides he wants to watch It's Always Sunny in Philadelphia and they're passionate about using apostrophes, so they search for that next, the web app sends it up to the server and oh shit everything breaks. The server creates the query SELECT * FROM shows WHERE title = 'It's Always Sunny in Philadelphia' and in doing so creates invalid syntax because the quote in "It's" closes the string.

Knowing that the search query is likely getting passed straight into the sql query an attacker can start to make some more mischievous queries, a common test string is `' or 1 = 1 --`, the `--` starts a comment, so will just blank out anything coming after, so the server will create the query SELECT * FROM shows WHERE title = '' or 1=1 which will check the db for every tv show where the title is '' (which would return nothing) or 1 = 1 (which is always true). So would return everything. Which isn't really great, though in this case not hugely damaging, (although if this was an login box with the same vulnerability, and an attacker knew someone else's username and could effectively skip the password verification by forcing a query like SELECT * FROM permissions WHERE username = 'admin' AND password = '' or 1=1)

sqlmap and other tools?

walk through of sqli exploits in nodegoat/webgoat, & metasploitable.
