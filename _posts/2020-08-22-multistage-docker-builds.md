---
layout: post
title: "Multi stage docker builds"
date: 2020-08-22 11:52:26 +1200
categories: [technical, docker]
tags: [docker]
---

## What is a Multi-stage docker builds

Multi-stage build is a way of dividing dockerfile into stages or sections. Only the artifacts and steps from the final stage end up in the resulting docker image, thus optimizing the image.

## Why use a multi-stage dockerfile ?

Every step of a dockerfile becomes a layer of resulting image. Some of these steps are just for building the target artifact and are not really needed when image is running.

These extra artifacts unnecessarily increase the size of the resulting image. Before multi-stage builds developers had to manually delete these undesired artifacts.

Let's see with examples, how developers achieved optimized images without multi-stage builds

### Method 1 - delete what's not needed.

```
FROM node:14.8
WORKDIR /usr/src/app
RUN yarn global add serve
COPY package.json ./
COPY src/ src/
COPY public/ public/
RUN yarn
RUN yarn build # This steps creates a build folder containing output files
CMD serve -s build
```

> Note: Node app used in docker file generates built artifacts in `build` folder on running `yarn build`

Above dockerfile is creating static production build of a node application and then serving the build output files using `serve` command.

This file has no optimization.
For starters, we could delete the original source code and node_modules which are no longer needed.

```
FROM node:14.8
.
.
RUN yarn build # This steps creates a build folder containing output files
RUN rm -rf node_modules src
CMD serve -s build
```

This is slightly better but we still have node which we don't really need.

Lets improve this file using `builder pattern`.

## Method 2 - Builder pattern

Builder pattern works as follows

1. Create a dockerfile which generates the desired artifacts.
2. Create a container from image created in `step one`
3. Copy required artifacts from the running container to your local machine.
4. Finally, create the desired image using artifacts from `step 3`.

Lets see this in more details.

Modify our dockerfile and rename it to `dockerfile.builder`

```
FROM node:14.8
WORKDIR /usr/src/app
# RUN yarn global add serve
COPY package.json ./
COPY src/ src/
COPY public/ public/
RUN yarn
RUN yarn build # This steps creates a build folder containing output files
# CMD serve -s build
```

In the above dockerfile, I have commented steps related to serving of app, because this is not the final image now.

Lets build this image.

```
docker build -t app-builder . -f dockerfile.builder
```

Once the build is complete, create a container from this intermediate image.

```
docker container create --name build-container app-builder
```

Above commands creates a container and names it `build-container`. The artifacts we require are present in `/usr/src/app/build` directory.

We copy the build folder to our local machine from the docker container.

```
docker container cp build-container:/usr/src/app/build ./app-prod-build
docker rm -f build-container
```

Above command copies artifacts to `app-prod-build` in our current directory . After copying we also clean up by removing the container.

Now we are ready to create our final image using the optimized build created previously.

Make another dockerfile as follows and name it `dockerfile.builder-pattern`

```
FROM nginx:1.19.2-alpine
COPY app-prod-build /usr/share/nginx/html
```

Lets build it.

```
docker build -t final-app-builder-pattern . -f dockerfile.builder-pattern
```

This new dockerfile will create and image only containing our optimized build files and nginx to serve them.

This is all good, but its a tough build process and hard to maintain with multiple build files and intermediate containers.

This is where multi-stage builds comes in.

## Multi-stage build to the rescue

Before we begin remember that every new stage begins with `FROM` keyword.
Above same image can be created using multi-stage build as follows.

```
FROM node:14.8 AS builder
WORKDIR /usr/src/app
COPY package.json ./
COPY src/ src/
COPY public/ public/
RUN yarn
RUN yarn build # This steps creates a build folder containing output files

# stage 2 begins from here
FROM nginx:1.19.2-alpine
COPY --from=builder /usr/src/app/build usr/share/nginx/html
```

So much simpler isn't it ?

In the above dockerfile we have two stages.

First stage named `builder` using the `AS` keyword.
I haven't named the second stage but it can be named as well.

You could also choose not to name stages and use a `0` based index instead, ie. stages can be referred using 0, 1, 2 ... and so on.
for eg. in our dockerfile we could use below instruction instead of using name of first stage.

```
COPY --from=0 /usr/src/app/build usr/share/nginx/html
```

Lets build it. I am assuming this dockerfile is named `dockerfile` so `-f` flag is not required

```
docker build -t final-app-multi-stage .
```

Using multi-stage build we only have one dockerfile, making it easy to modify and maintain.
It has all the advantages of `builder-pattern` with out its pain points.

```
$ docker images
final-app-multi-stage          22.5MB
final-app-builder-pattern      22.5MB
app-builder                    1.27GB
```

> app-builder image was used as step one in builder pattern is quite similar to an unoptimized image created in `Method one`

So we see, multi-stage build is great and easy way to optimize your final images.

For debugging or testing you can also stop build at any stage. For eg. to build just the first stage you could do

```
docker build --target builder -t test-app-image .
```

This command tells docker to stop at `builder` stage.
