---
layout: post
title: Ghost on Centos
date: '2018-04-07 22:36:33'
categories: 'Tech'
author: Satish Ramjee
tags:
- ghost-tag
- centos
- blogging
---

Using ghost running on a centos vm backed by sqlite.


[Ghost on centos](https://blog.boardiesitsolutions.com/installing-ghost-1-0-on-centos-7-with-apache/)

Except I used sqlite and ngix. I used forever to run it like so:

```
NODE_ENV=production forever start index.js
```

 [Config of Sqlite](https://docs.ghost.org/docs/config)
 
Ghost editor is similar to markdowns so quite neat.

