---
layout: post
title: "Dockerfile Best Practices"
date: 2020-08-14 22:56:26 +1200
categories: [technical, docker]
tags: [docker]
---

When building docker Images you can do some easy optimization to make your build faster and images smaller.

## #1 Use the smallest image you need

Whenever writing a dockerfile try to use the slim down version of the base image. Generally this is the alpine variant and are much smaller than the normal versions.
Alpine linux docker image is just 5 MBs!

In alpine variant of images only those packages are present which are really needed.
So no common packages like vim, ssh etc would be available, but if you need them you can just install them as needed.

Let's see an example to understand this better.
Say you are using nginx as base image and then adding some of your files to it.

If you use the normal nginx image, it will look as follows.

```
FROM nginx:1.19.2
COPY my-static-html-directory /usr/share/nginx/html
```

build this file using command :

```
docker build -t nginx-full .
```

You can accomplish exactly the same using an alpine variant of nginx.

```
FROM nginx:1.19.2-alpine
COPY my-static-html-directory /usr/share/nginx/html
```

build this file as well using :

```
docker build -t nginx-small .
```

now if you list docker images as follows:

```
$ docker images
nginx          1.19.2-alpine       6f715d38cfe0        11 hours ago        22.1MB
nginx          1.19.2              4bb46517cac3        11 hours ago        133MB
nginx-small    latest              3376e1ebe4ac        2 days ago          22.1MB
nginx-full     latest              1d3130aefb32        2 days ago          133MB

```

Now you can see by using alpine images you saved more than 110 MB!

All official images of different products offer many variants of their docker images like alpine, slim etc. You can check product's docker hub page to find their different variants.
Here is the link to [nginx's dockerhub page](https://hub.docker.com/_/nginx).

You can choose the lighter versions of docker images for your dockerfiles and install additional packages if needed.

## #2 Order steps correctly in your Dockerfile

It may seem that order of some steps is not really important.
But if you want to use caching, which results in faster subsequent builds, then it is.

### How caching in docker build works

A Docker image consists of read-only layers each of which represents a dockerfile instruction/step.
When a docker build is run if the step or instruction is unchanged then a cache from a previos build is used and the step is not actually exectued.
As soon as a step is changed, cache of all the subsequent steps are also invalidated.

This is why you should always:

**_Order your steps from least to most frequently changing steps to optimize caching._**

Lets see an example. Say you want to install ssh to our nginx-alipine image.

```

FROM nginx:1.19.2-alpine
COPY my-static-html-directory /usr/share/nginx/html
RUN apk update
RUN apk add --no-cache openssh

```

Above dockerfile works okay, but every time you change files in my-static-html-directory `step 3 and 4` are run again because cache is invalidated at `step 2`.

```
$ docker build .
Sending build context to Docker daemon  3.584kB
Step 1/4 : FROM nginx:1.19.2-alpine
 ---> 6f715d38cfe0
Step 2/4 : COPY my-static-html-directory /usr/share/nginx/html
 ---> 1301190f87aa
Step 3/4 : RUN apk update
 ---> Running in c531f30073c4
fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/main/x86_64/APKINDEX.tar.gz
.
.
 ---> 9a438091be3c
Step 4/4 : RUN apk add --no-cache openssh
 ---> Running in 68ce2368353d
fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/main/x86_64/APKINDEX.tar.gz
.
.
 ---> 230c9786f74c

```

In this dockerfile `step 2` to `step 4` are executed again when there is a change in my-static-html-directory

We could leverage caching by moving `COPY` statement down.

```
FROM nginx:1.19.2-alpine
RUN apk update
RUN apk add --no-cache openssh
COPY my-static-html-directory /usr/share/nginx/html
```

after doing this change `step 2 and 3` are cached i.e openssh install instruction is not run on every build. Notice the text `Using cache` in the output.

```
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
 ---> 911482c87de4
Successfully built 911482c87de4
```

In this dockerfile only `step 4` is executed again when there is a change in my-static-html-directory

So whenever you write a dockerfile think about what steps could change frequently and move them towards the bottom if possible. This can have a lot of impact on build time of your image.
