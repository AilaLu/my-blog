---
title: pub/sub pattern
date: 2022-08-18 19:55:36
tags:
---

pub/sub pattern stands for publish/subscribe pattern, it provides better scalability comparing with other message exchange pattern. you can think of it as the actual publish and subscribe, where `publisher`(sender) can send message to multiple `subscribers`(receiver).
In this pattern, the pub sends message to sub through `broker`, the broker's job is to organizing messages into according categories, so pub only has to focus on publishing messages, without knowing which specific sub, so the amount of sub doesn't affect them. This is why pub/sub pattern has better scalability. 



### pub/sub middleware
- Redis
- PostgreSQL

####references:
[Pub/Sub Examples: 5 Use Cases to Understand the Pattern and its Benefits]https://ably.com/blog/pub-sub-pattern-examples