---
layout: post
title: Using jekyll
date: '2018-12-24 10:43:17'
categories: Blogging
author: Satish Ramjee
tags:
- jekyll
- github 
- blogging
---

Jekyll is mainly a static site generator.

A good overview here of static site generators:

<https://davidwalsh.name/introduction-static-site-generators>


Jekyll home page:

<https://jekyllrb.com>



## Set up

```
  gem install bundler jekyll

  jekyll new my-awesome-site

  cd my-awesome-site

  bundle exec jekyll serve

```
=> Now browse to <http://localhost:4000>

## Blogging with github pages.

<http://jmcglone.com/guides/github-pages/>

Port your site to jekyll using one the various migration tools

<http://import.jekyllrb.com>

This has a defined structure and to change themes all you need to do is follow the structure.

## Github pages

If you create a github project called {username}.github.io, then the site will be served on **http://{username}.github.io**

<https://help.github.com/articles/about-github-pages-and-jekyll/ >


### Themes

You can handcraft the themes from various offering, or get them set this directly in githug settings page. 

<https://help.github.com/articles/adding-a-jekyll-theme-to-your-github-pages-site-with-the-jekyll-theme-chooser/>


### Custom Domain

Change the setting in github:

<https://help.github.com/articles/adding-or-removing-a-custom-domain-for-your-github-pages-site/>

And then point your domain to {username}.github.io (CNAME, ANAME)

For blog.blah you need to set the apex domain configuration:

<https://help.github.com/articles/setting-up-an-apex-domain/>


## Pages

<https://jekyllrb.com/docs/pages/>


## Drafts

Use of branching for drafts.

<http://qrohlf.com/posts/jekyll-drafts-workflow>

or 

jekyll serve or jekyll build with the --drafts switch


## Comments

<https://eduardoboucas.com/blog/2016/08/10/staticman.html>

<https://eduardoboucas.com/blog/2015/05/11/rethinking-the-commenting-system-for-my-jekyll-site.html>

<https://mademistakes.com/articles/jekyll-static-comments/>



## Using Gitlab instead

<https://www.youtube.com/watch?v=TWqh9MtT4Bg>


