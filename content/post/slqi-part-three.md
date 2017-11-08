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
lives easier. To nab our sensitive information this time we're going to:

- Find the url to attack.
- Find our cookie to let a response through
- Hit our vulnerable url with a get request + cookie to grab the user, and the
  tables
  - explain how the get/post requests work with the command line
  - show the different levels and risk
- Hit the db to grab the users again.
