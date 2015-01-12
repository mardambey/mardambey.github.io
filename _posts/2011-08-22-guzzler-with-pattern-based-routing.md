---
layout: post
title: Guzzler with pattern based routing
date: '2011-08-22T03:42:00-04:00'
tags:
- Code
- mysql
- guzzler
- rabbitmq
tumblr_url: http://www.hisham.cc/post/30203544867/guzzler-with-pattern-based-routing
---
Guzzler now publishes binlogs and allows pattern based subscriptions using “dbName.tableName.opName” with wild-card support.

dbName is the database, tableName is the table being acted upon and opName
is one of update, insert, or delete. Wild cards are also supported.
