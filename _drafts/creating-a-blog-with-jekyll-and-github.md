---
title: "tools: Creating a blog with Jekyll and github"
categories: "tools"
---

# tools: Creating a blog with Jekyll and github

A friend of mine was recently having an issue with WordPress, I have a site there myself, but other people who heard about this asked why he wasnt using github to service a static site, this got me intregued.

I started off by performing a basic search for what this was, and found about the default provision of a site via github was really straight forward and would allow you have a basic site setup in next to no time, it would even be hosted as a subdomain under github.io, so you get to feel like you are speciall with an io domain for free.

The first things I did were setup the blogging template, this requires a directory called `_posts` to be created, which you then put all your pages in.

The basic setup of a page consists of naming the page with a date and a short name and then a header section with a title and some categories.

this page is called `2017-12-15-creating-a-blog-using-github-and-jekyll

```
---
title: "tools: Creating a blog with Jekyll and github"
categories: "tools"
---
```

This create a folder structure of `/tools/2017/12/15/create-a-blog-with-jekyll-and-github`