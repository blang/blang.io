---
title: golang-alpine - Build golang binaries for alpine linux
date: 2015-04-19
comments: true
---

Since alpine linux and therefor `gliderlabs/alpine` docker containers use `musl` instead of `gnu libc` your golang binaries will not work inside alpine.

There are two ways you can fix this:

## Static linking
```
CGO_ENABLED=0 go build -a -installsuffix cgo
```
## Use this docker image

Use this docker image to build your binary, check the usage below. 

Using `gliderlabs/alpine` it is possible to create an image similiar to `golang` with only 204MB.

You can pull it from docker hub: `docker pull blang/golang-alpine`

# Usage
Use this container to build your project binary and copy it to your production alpine container.

```
docker run --rm -v "$PWD":/go/src/github.com/yourname/yourrepo -w /go/src/github.com/yourname/yourrepo blang/golang-alpine go build -v

docker run --rm -v "$PWD":/go/bin blang/golang-alpine go get github.com/yourname/yourrepo
```

# Dockerfile
The dockerfile is quite simple:

```
FROM gliderlabs/alpine
MAINTAINER Benedikt Lang <mail@blang.io>
RUN apk-install bash go bzr git mercurial subversion openssh-client ca-certificates 

RUN mkdir -p /go/src /go/bin && chmod -R 777 /go
ENV GOPATH /go
ENV PATH /go/bin:$PATH
WORKDIR /go
```

# Limitations
This process depends on the golang port for `alpine linux`. The base `golang` image is build from source and comes in different versions properly tagged.

A better solution to build this image would be a compilation of the golang sources, but there are problems i was not able to solve:

- Alpine uses musl libc, Base image gnu libc
- You can't copy go binaries build on ubuntu, because of musl
- I could not build go using musl because of build errors
