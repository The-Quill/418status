---
layout: post
title:  "Introducing StackNews"
comments: true
date:   2016-07-24 19:50:00 +1000
categories: code project stack-exchange
---

StackExchange is an amazing network of sites, and trying to keep up with all the news for each site is an arduous task.

That's where [StackNews](http://stacknews.org) comes in, instead of manually visiting >150 sites, you get all Meta posts laid out in one convenient feed.

[StackNews](https://github.com/The-Quill/StackNews) is also available on GitHub!

# How it's built:

StackNews is a three-server approach:

 - Web (runs on Node.js, Express and React/Redux) and cron
 - Redis
 - HAProxy

On the web server, Express is constantly listening on a certain port for requests.

All routes are overlaid to drop all traffic from anyone not matching the reverse proxy's IP.

Reverse proxies are designed to load balance your servers and remove some malformed and potentially malicious traffic.

While a reverse proxy doesn't _exactly_ serve much purpose for this particular project, the codebase is pretty flexible and is designed for scale.

When you reach an the web page, A data-less React template is served, which the browser then proceeds to load the data in.

By doing this, crawlers and non-web clients won't put additional strain on Redis. Assuming the Redis server had more processing power or more slaves, it could probably preload some data.

When the data route is called by the web browser, it requests Redis for the top 30 scored posts in the sorted set of posts.

Via an hourly cron job, that sorted set of posts is being updated with the latest from Stack Exchange via their API.

A daily cron job finds the new sites.

This is a pretty flexible net of sites that allows for scaling on Web, by adding more webservers to HAProxy and on Redis by adding slaves.
