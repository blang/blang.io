---
title: Travis-ci Golang Binary Github Release
date: 2014-07-28
comments: true
---

If you're working on a golang project which results in an usable binary, you might want to let [travis-ci](https://travis-ci.org/) do the work for you. Travis will test and build your project and create a github release with the binary attached.

-----

# .travis.yml

```
language: go
go:
  - 1.3
install:
  - "go get -d -v ./..."
  - "go build -v ./..."
deploy:
  provider: releases
  api_key:
    secure: [YOUR ENCRYPTED OAUTH TOKEN]
  file: "[NAME OF YOUR PRODUCED BINARY]"
  skip_cleanup: true
  on:
    repo: [YOUR REPO e.g. blang/expenv]
    tags: true
    all_branches: true
```

The best way to create this file is by using the [travis command-line tool](http://blog.travis-ci.com/2013-01-14-new-client/):

```
$ travis init
```

This will create nearly everything for you including the encrypted oauth token. Afterwards you want to add the missing lines from above.

To create a release, simply tag the commit with e.g. `v1.0.0` and push the tag: `git push origin v1.0.0`

Note: Travis does not build your project by default, which will not produce a binary, because of that we changed the `install` instructions.