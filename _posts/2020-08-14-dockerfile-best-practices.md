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

If you use the normal nginx variant, it will look as follows.

```
FROM nginx:1.19.2
COPY my-static-html-directory /usr/share/nginx/html
```

build this file using :

```
docker build -t nginx-full .
```

You can accomplish the same using an alpine variant of nginx.

```
FROM nginx:1.19.2-alpine
COPY my-static-html-directory /usr/share/nginx/html
```

build this file as well using :

```
docker build -t nginx-small .
```

now if you list docker images:

```
$ docker images
nginx          1.19.2-alpine       6f715d38cfe0        11 hours ago        22.1MB
nginx          1.19.2              4bb46517cac3        11 hours ago        133MB
nginx-small    latest              3376e1ebe4ac        2 days ago          22.1MB
nginx-full     latest              1d3130aefb32        2 days ago          133MB

```

By using alpine images you save more than a 110 MB!

All official images of different products offer many variants of their images like alpine, slim etc. You can check product's docker hub page to find their different variants.

You can choose the lighter versions and install additional tools if needed.

## #2 Order steps correctly in your Dockerfile

It may seem that order of some steps is not really important.
But if you want to use caching, which results in faster subsequent builds, then it is.

**_Order your steps from least to most frequently changing steps to optimize caching._**

Say you want to install ssh to our nginx-alipine image.

```

FROM nginx:1.19.2-alpine
COPY my-static-html-directory /usr/share/nginx/html
RUN apk update
RUN apk add --no-cache openssh

```

above file is fine but every time you change files in my-static-html-directory ssh both `apk` commands are run again.

```
$ docker build .
Sending build context to Docker daemon  3.584kB
Step 1/5 : FROM nginx:1.19.2-alpine
 ---> 6f715d38cfe0
Step 2/5 : COPY my-static-html-directory /usr/share/nginx/html
 ---> 2180c2aa8027
Step 3/5 : RUN apk update
 ---> Running in db6b25979ef2
.
.
Step 4/5 : RUN apk add --no-cache openssh
 ---> Running in ea8332dce143
.
.
 ---> a38e35183f0e
Step 5/5 : COPY app /usr/share/nginx/html
 ---> ab97b1107664
Successfully built ab97b1107664
```

We could leverage caching by moving `COPY` statement down.

```

FROM nginx:1.19.2-alpine
RUN apk update
RUN apk add --no-cache openssh
COPY my-static-html-directory /usr/share/nginx/html

```

after doing this change ssh install is cached and not re-installed everytime there is change in html files.

```
$ docker build .
Sending build context to Docker daemon  3.584kB
Step 1/4 : FROM nginx:1.19.2-alpine
 ---> 6f715d38cfe0
Step 2/4 : RUN apk update
 ---> Using cache
 ---> 28026827b263
Step 3/4 : RUN apk add --no-cache openssh
 ---> Using cache
 ---> 8d49976f95e4
Step 4/4 : COPY my-static-html-directory /usr/share/nginx/html
 ---> 4f82ab71dc89
Successfully built 4f82ab71dc89
```

So whenever you write a dockerfile think about what steps could change frequently and move them towards the bottom if possible.
