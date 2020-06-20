---
layout: post
title: Multi-Stage Docker Builds
date: 2020-05-07 15:32:20 +0000
description: Optimize docker build size # Add post description (optional)
img: multistage-docker.jpg # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Docker, Go]
---

Multi-stage builds are a new feature requiring Docker 17.05 or higher on the daemon and client. Multistage builds are useful to anyone who has struggled to optimize Dockerfiles while keeping them easy to read and maintain.

Let’s start with a simple Go program, in a file named main.go:

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello world!")
}
```

And build it using the golang:1.14-alpine image in a single-stage build. Here’s the Dockerfile:

```docker
FROM golang:1.14-alpine
WORKDIR /app
ADD . /app
RUN cd /app && go build -o main
ENTRYPOINT ./main
```

Now let’s build it and run it:
```
docker build -t idawud/hello .
docker run --rm idawud/hello
```

Ok, works fine, but let’s look at the size with `docker images | grep idawud/hello` . 372 MB, just for our single little Go binary. This is because the image still contains the alpine base image, all the golang compiling and running tools, the source code and the generated go binary, these are lot of things we don't actually need anymore to run the binary.

Now let’s try a multi-stage build using this new Dockerfile:

```docker
# build stage
FROM golang:1.14-alpine AS build-env
ADD . /src
RUN cd /src && go build -o main

# final stage
FROM alpine
WORKDIR /app
COPY --from=build-env /src/main /app/
ENTRYPOINT ./main
```

Let’s build and run it again:

```
docker build -t idawud/hello .
docker run --rm idawud/hello
```

And check the size: 7.64 MB. Now the size reduces drastically because although in the first stage i.e. the build stage the size is equal to first docker image we've seen. In the second stage only the generated binary is imported from the first stage into the second image which will contain only the alpine base image and the generated binary without including all the unnecessarry go tools, thus recuding the size drastically.

### Conclusion 
In this post we looked at multi-stage docker build, though the example used is with go we can build with any other project or language.
