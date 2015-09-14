---
title: Deploy golang binaries using wercker, glide and gox
date: 2015-09-14
comments: true
---

A short example on how to deploy golang binaries using wercker. It also uses dependency managment via glide and cross compiles your code for many platforms.

At the end you can push a git tag, click deploy and a github release is created containing all your cross compiled binaries.

Have a look at the [release of my project](https://github.com/blang/pushovercli/releases/tag/v0.0.1)

<!--more-->

The deploy steps:

- Install utilities `file` and `jq`
- Get the git tag of your current commit. That's a tricky part described later
- Setup go workspace
- Build source using glide
- Install gox
- Build all binaries using gox
- Create a Github release for your tag
- Upload every binary as an asset to your release

Git Tag
-------

To get the current git tag turns out to be quite tricky. My idea is to only build a release if the current commit is tagged, otherwise you would overwrite an existing release.

Because wercker does not always build before you tag is pushed to origin, my first variant will only work if wercker builds on a tag, not a commit:
```bash
export GIT_TAG=`git describe --tags --exact-match "${WERCKER_GIT_COMMIT}" 2>/dev/null` && test -n "$GIT_TAG"
```

Also a `git fetch` of all tags fails, i haven't looked into it but i guess the `git remote` is missing.

I'm not very familiar with wercker and was looking for a solution that works right out of the box and luckily if found this [pull request](https://github.com/walter-cd/walter/pull/101/commits) by 
[Naoki Ainoya](https://github.com/ainoya). Therefor all credits for getting the current git tag to him:
```bash
export GIT_TAG=$(curl https://api.github.com/repos/blang/pushovercli/tags | jq -r ".[] | select(.commit.sha == \"${WERCKER_GIT_COMMIT}\") | .name")
```
This reads the tag of your current commit from the public github api. It also enables us to push a tag afterwards and deploy after the commit is already done.

But there is still a problem which i could not fix yet: I guess the rate limiter of github rejects an answer and the deploy will fail in lack of the tag information.

The git tag step will fail with the following error:
```
jq: error: Cannot index array with string
```

For a small project that's not a problem since i deploy manually and don't mind to push that deploy button more than once to get this task finished. You might consider to request the github api using your own api token to overcome the rate-limiter on werckers boxes.


wercker.yml

```yaml
box: golang

build:
  steps:
    - setup-go-workspace
    - termie/glide-build@2014.345.145

deploy:
  steps:

    # Install file utility needed by gox
    - script:
      name: apt-update/install file, jq
      code: |
        apt-get update -qy && apt-get install -qy file jq
    # Get the git tag of this commit, otherwise fail on this step
    # Only tagged commits will be deployed
    # # export GIT_TAG=`git describe --tags --exact-match "${WERCKER_GIT_COMMIT}" 2>/dev/null` && test -n "$GIT_TAG"
    - script:
      name: git tag
      code: |
        echo "Wercker commit: $WERCKER_GIT_COMMIT"
        export GIT_TAG=$(curl https://api.github.com/repos/blang/pushovercli/tags | jq -r ".[] | select(.commit.sha == \"${WERCKER_GIT_COMMIT}\") | .name")
        echo "Git tag is: $GIT_TAG"
        test -n "$GIT_TAG"

    # Setup go workspace
    - setup-go-workspace
    # Build source using glide
    - termie/glide-build@2014.345.145

    # Install gox
    - script:
        name: gox install
        code: |
          go get github.com/mitchellh/gox

    # Build all binaries
    - script: 
        name: gox build
        code: |
          $GOPATH/bin/gox

    # Create a release based on the current tag
    - github-create-release:
        token: $GITHUB_TOKEN
        tag: $GIT_TAG

    # Upload all assets
    - github-upload-asset:
        token: $GITHUB_TOKEN
        file: pushovercli_darwin_386
    - github-upload-asset:
        token: $GITHUB_TOKEN
        file: pushovercli_darwin_amd64
    - github-upload-asset:
        token: $GITHUB_TOKEN
        file: pushovercli_freebsd_386
    - github-upload-asset:
        token: $GITHUB_TOKEN
        file: pushovercli_freebsd_amd64
    - github-upload-asset:
        token: $GITHUB_TOKEN
        file: pushovercli_freebsd_arm
    - github-upload-asset:
        token: $GITHUB_TOKEN
        file: pushovercli_linux_386
    - github-upload-asset:
        token: $GITHUB_TOKEN
        file: pushovercli_linux_amd64
    - github-upload-asset:
        token: $GITHUB_TOKEN
        file: pushovercli_linux_arm
    - github-upload-asset:
        token: $GITHUB_TOKEN
        file: pushovercli_netbsd_386
    - github-upload-asset:
        token: $GITHUB_TOKEN
        file: pushovercli_netbsd_amd64
    - github-upload-asset:
        token: $GITHUB_TOKEN
        file: pushovercli_netbsd_arm
    - github-upload-asset:
        token: $GITHUB_TOKEN
        file: pushovercli_openbsd_386
    - github-upload-asset:
        token: $GITHUB_TOKEN
        file: pushovercli_openbsd_amd64
```