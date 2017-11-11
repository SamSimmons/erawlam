---
title: "Sqli Part Three"
date: 2017-11-06T20:08:26Z
draft: true
tags: ["SQL injection"]
description: "A look at at one of the most popular tools for sql injection,
sqlmap, and a walkthrough of how it works with some of our previous examples"
---

In this post we're diving back into SQL injection. In the last post we covered
union based injection, worked through an example manually of how it works using
the damn vulnerable web application (DVWA). This post we're going to introduce
the most widely used tool used in SQL injection, and then we're going to work
through the same example as last time. Except this time, we're going to
automate the process.

#### SQLmap

[Sqlmap](https://github.com/sqlmapproject/sqlmap) is really cool. It's a super powerful tool for finding snd exploiting sql
vulnerabilities. It can do everything from identifying vulns, db
fingerprinting, it can even provide a console to run whatever queries you want
against a db. Also it's open source so it's free to play with. You can download
it and use it standalone, or it comes packaged up by default in kali (A popular
linux distro for pentesting).

Why use sqlmap instead of doing it yourself? Because in most cases it's likely better than
you. Unless you're a real pro using sqlmap will save you time, and make the
whole process a hell of a lot less boring, especially if it's something gnarly
like blind sqli. It's not always the best choice, sometimes the only approach
is something crazy or out there that sqlmap will have no idea about. But it's
a really good shotgun approach, quick and dirty.

Cool so let's boot up the damn vulnerable web app again and get our hands
dirty. It will follow similar steps to the last post except we'll make our
lives easier.

By entering an id into the input, we can see the url it corresponds to. I'm running the DVWA from a docker container and for me it translates into
"http://172.28.128./vulnerabilities/sqli/?id=1&Submit=Submit#", I'm going to use sqlmap from a virtual machine running kali. But the steps are still
basically the same no matter how you run it. Just to see what happens I'm just going to run `sqlmap  -u "http://172.28.128.1/vulnerabilities/sqli/?id=1&Submit=Submit#"`
with no other paramaters. The request is going to get bounced because it expects the user to be logged in, and the request coming from sqlmap has no idea about being logged in.

![dvwa screenshot](/sqli-three/1.png)

Even without anything else, sqlmap gives us some useful information here, it picks up that the url is vulnerable to a union attack, and it fingerprints the db as being MySQL.
Now we know that the same request doesn't get redirected when we make it from the browser, so let's pop open the network tab of the dev tools and check out what get's sent with a request.
I'm using firefox, but I can vouch for chrome being pretty similar. To open the dev tools, you can hit `ctrl + shift + k` in firefox or `f12` in chrome. Check out the network tab, it
will record what's going on when you make a request in the browser. So with the network tab open, enter an id in the DVWA input and hit submit, then find the request that gets made
to our vulnerable url.

![dvwa screenshot](/sqli-three/2.png)

Looking at the request headers, the one that jumps out is Cookie, sites will use cookies to manage user sessions, and we can guess that not having a valid cookie with our request is
likely the reason it got redirected earlier. So let's run our last sqlmap query back, but add in the cookie. Copy the cookie value from the network tab, and paste it in with the url
like `sqlmap  -u "http://172.28.128.1/vulnerabilities/sqli/?id=1&Submit=Submit#" PHPSESSID=nbvfmt1t95ept6atilvaq8tuc5; security=low`, sqlmap will identify the db as being MySQL and
prompt for whether to include tests for other DB's, and some other prompts, you can happily mash the enter key, selecting the default options. In the end you should get a message
informing you that the url is indeed vulnerable to injection. Cool, lets find out some more, lets grab some more info, add `--current-user` to your previous query and run it back.
Then same again but with `--current-db`.

![dvwa screenshot](/sqli-three/3.png)
![dvwa screenshot](/sqli-three/4.png)

Try grabbing the tables with `--tables`, then we dig into any intersesting ones further by providing it as a param to sqlmap, add `-T "users"` to a query to only target that table.
You can grab the columns of that table with `--columns`, we can see the user, and password columns listed here, let's grab those columns out with `-C user,password --dump`. Sqlmap will even be
super helpful here and offer to have a go at cracking them for us, or we can dump them to a file and user john the ripper.

![dvwa screenshot](/sqli-three/5.png)
![dvwa screenshot](/sqli-three/6.png)
![dvwa screenshot](/sqli-three/7.png)

Sqlmap is awesome, there's a huge list of things to try out [here](https://github.com/sqlmapproject/sqlmap/wiki/Usage) so have fun with it.
