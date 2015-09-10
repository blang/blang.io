---
title: How to build the smallest docker containers
date: 2015-04-18
comments: true
---

```
FROM ubuntu
...
```

Today most docker containers are based on ubuntu or debian images, which result in quite a big amount of data per container. Most included data is never needed by the containing application.

`docker images` gives a good overview about the sizes of images.

Especially containers containing small amounts of data like tooling scripts profit from a smaller base image.

There are a couple of ways to get a smaller base image, which are shown below.

# From scratch - 0 MB

If your container brings all needed dependencies or is a statically compiled binary your best choice is to begin from scratch. Which means a 0-Byte empty base image.

You might pull the `scratch` image from docker hub, i experienced that's not possible with some versions of docker, but you can simply build it yourself:

```
tar cv --files-from /dev/null | docker import - empty
```
This command creates an empty image called `empty`.

```
FROM empty
...
```

Bare in mind that this container does not even has a shell.

# Busybox - 5.6 MB

If you need at least a small shell to look if everything is in the right place, busybox might be a good choice. `busybox` comes with the busybox shell and the basic small unix utilities.

```
docker pull busybox
```

```
FROM busybox
...
```

For some scripts the basic busybox shell is not sufficient and there's no bash included.

# Busybox with bash - 5.9 MB
 
Fear not, there's also a way to build busybox with bash.

```
FROM progrium/busybox

RUN opkg-install bash
...
```

Or simply pull [blang/busybox-bash](https://github.com/blang/busybox-bash-docker).

# Alpine - 5.0 MB

A very good base for a small image is `gliderlabs/alpine` ([Docs](https://github.com/gliderlabs/docker-alpine)). It comes with a small package manager `apk`.

```
docker pull gliderlabs/alpine
```

```
FROM gliderlabs/alpine
...
```

Alpine comes with a small shell, if you need a full bash, the next option is your choice.

# Alpine with bash - 6.5 MB

Alpine with an installed bash is probably the best choice if you need an extendable image with bash and a package mananger.

```
FROM gliderlabs/alpine
RUN apk-install bash
...
```

You might try this, pull [blang/alpine-bash](https://github.com/blang/alpine-bash-docker).
