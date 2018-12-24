---
layout: post
title: Using the Hygieia Dashboard
---

From here

https://github.com/capitalone/Hygieia

Needs Mongo

docker run -d -p 27017:27017 --name mongodb -v ./mongo:/data/db mongo:latest  mongod --smallfiles

docker run -d -p 3000:80 --name hygui capitaloneio/hygieia-ui

