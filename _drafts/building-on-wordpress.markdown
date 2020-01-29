---
title: Building on WordPress
date: 2020-01-29 12:12:00 Z
tags:
- Development
---

A project I am working on is related to the REST API for WooCommerce.  This has meant that for development purposes I need to stand up a WooCommerce instance that I can interact with.

We've recently started getting into docker more and more, so I thought I'd try my hand at building a docker image.  I borrowed quite heavily from the example docker-compose.yml file and the documentation from the official WordPress docker image - located here : [https://hub.docker.com/_/wordpress](https://hub.docker.com/_/wordpress)