---
layout: post
title: GAMOS
---

With FN and Go

fn init --runtime go --trigger http gofn

fn --verbose deploy --app goapp --local

brew install httpie

```
http localhost:8080/t/goapp/gofn-trigger
HTTP/1.1 200 OK
Content-Length: 26
Content-Type: text/plain; charset=utf-8
Date: Sat, 22 Dec 2018 14:27:04 GMT
Fn-Call-Id: 01CZB4ZY0BNG8G00GZJ0000004

{
    "message": "Hello World"
}
```
