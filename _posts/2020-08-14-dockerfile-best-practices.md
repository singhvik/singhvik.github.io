---
layout: post
title: "Dockerfile Best Practices"
date: 2020-08-14 22:56:26 +1200
categories: [technical, docker]
tags: [docker]
---

When building docker Images you can do some easy optimization to make your build faster and images smaller

## #1 Use the smallest image you need

Let's say you are using nginx as base image and then adding some of your files to it.

```
FROM nginx:1.19.2
COPY my-static-html-directory /usr/share/nginx/html
```

You can accomplish the same using an alpine variant of nginx.

```
FROM nginx:1.19.2-alpine
COPY my-static-html-directory /usr/share/nginx/html
```

By using alpine images you save more than a 100Mb.

```
$ docker images
nginx     1.19.2-alpine       6f715d38cfe0        11 hours ago        22.1MB
nginx     1.19.2              4bb46517cac3        11 hours ago        133MB
```

All official images of different products offer many variants of their images like alpine, slim etc. You can check product's docker hub page to find their different variants.

You can choose the lighter versions and install additional tools if needed.

## #2 Order steps correctly in your Dockerfile

It may seem that order of some statements is not really relevant.
But if you want to use caching, which results in faster builds, then it is.

_Order your steps from least to most frequently changing steps to optimize caching._

Say you want to install ssh to our nginx-alipine image.

```
FROM nginx:1.19.2-alpine
COPY my-static-html-directory /usr/share/nginx/html
RUN apt-get update
RUN apt-get -y install ssh
```

above file is fine but every time you change files in my-static-html-directory ssh both app-get commands are run again.

We could leverage caching by moving `COPY` statement down.

```
FROM nginx:1.19.2-alpine
RUN apt-get update
RUN apt-get -y install ssh
COPY my-static-html-directory /usr/share/nginx/html
```

after doing this change ssh install is cached and not re-installed everytime there is change in html files.
