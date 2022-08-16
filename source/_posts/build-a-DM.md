---
title: build a DM
date: 2022-08-16 12:57:55
tags:
---

##### These are the dependencies needed for the development
```shell
$ npm init
$ npm install express
$ npm install ws
$ npm install socket.io
$ npm install moment
```

#### This is for constantly watching the events, so we don't have to restart the server every time we make a change
```shell
$ npm install -D nodemon 
```
It will only be used during development, so add -D, which means Dev dependencies.