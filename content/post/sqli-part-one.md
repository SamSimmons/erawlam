+++
date = "2017-07-15T11:50:22+12:00"
draft = false
title = "Sql injection - part one"

+++

Web apps have all kinds of good stuff that an attacker could want. Personal information, usernames, passwords, credit card information. Because there is lots and lots of this good stuff, web apps need some way to keep it all organized and accessable. The most common kind of data store is some form of sql database.

SQL is a language used to to talk to the database. If the web application want's to change something it will be translated into SQL and used to update the database. Let's say you have a web app `badNetflix` which allows users to search for tv shows. A user comes on and wants to find a hilarious show about a horse in hollywood. They type 'Bojack Horseman' in the searchbar and hit enter. A request goes up to the server with the string 'Bojack Horseman' and the server goes to the database and forms a query like:
{{< highlight sql >}}
SELECT * FROM shows WHERE WHERE title = 'Bojack Horseman';
{{< / highlight >}}
Everything works fine, they binge watch all seasons of Bojack back to back since it's a great show. They lose their job and become an alcoholic and decide to spend their days watching other tv shows. They decide to watch It's Always Sunny in Philadelphia and they're passionate about using apostrophes, so they search for that next, the web app sends it up to the server and oh shit everything breaks. The server creates the query:
{{< highlight sql >}}
SELECT * FROM shows WHERE WHERE title = 'It's Always Sunny in Philadelphia';
{{< / highlight >}}
See the problem? The single quote in "It's" closes the string, which is invalid syntax. Now our user is an alcoholic tv junkie with nothing but time on their hands and a need to make money somehow to pay for their badNetflix bill. This is terrible. Because the developer made some mistakes not securing the site, our attacker can have a field day. They could steal data or credentials. Maybe just drop the table for fun, in some cases SQL injection can lead to an attacker being able to control the entire server.

A common way to test for sql injection is just to start chucking in single quotes willy nilly into input fields and url bars and see what breaks.  If you get back something funny then it's worth investigatinf further. With the badNetflix example, if our degenerate TV aficianado was to input {{<highlight sql>}}' or 1 = 1 --{{</ highlight>}} instead of a title, our code would naively send back all the information in the shows database. The `--` starts a comment, so will just blank out anything coming after, so the server will create the query:
SELECT * FROM shows WHERE title = '' or 1=1
Which will check the db for every tv show where the title is '' (which would return nothing) or 1 = 1 (which is always true). So would send back every single show. Which isn't really great.

Anyway that's a quick introduction to SQL injection, in part two we're going to cover how to grab data from other tables with the union operator and run through some practical examples (legally, of course).
