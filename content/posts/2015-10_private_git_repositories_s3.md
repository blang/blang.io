---
title: Private git repositories with s3 backup
date: 2015-10-06
comments: true
---

Github is a bit too expensive to offload my private repositories which i'm not currently working on. It's quite easy to create you own git server and setup a backup to s3. Here is how:

<!--more-->

Setup git server
----------------
    apt-get install git
    useradd git

Install your ssh keys

    cp authorized_keys /home/git/.ssh/authorized_keys 

Install aws cli

    apt-get install python-pip
    pip install awscli


Configure aws credentials:

    su -s /bin/bash -c "aws configure" git

Use `eu-central-1` as default region if you're storing in Frankfurt

Script to create a new repo on your server (/root/createrepo.sh)
-------------
```bash
#!/bin/sh

proj="$1"

if [ -z "$proj" ]; then
        echo "No valid project given"
        exit 1
fi

if [ -d "/home/git/$proj.git" ]; then
        echo "Project already exists"
        exit 1
fi
echo "Creating git repository: $proj"
su -s /bin/bash -c "mkdir /home/git/${proj}.git && cd /home/git/${proj}.git && git init --bare && ln -s /home/git/githook.sh /home/git/${proj}.git/hooks/post-receive" git
```

Post-receive hook (/home/git/githook.sh)
------------
```bash
#!/bin/sh

S3_BUCKET="[yourbucketname]"
repo=${PWD##*/}
aws s3 sync --delete ./ s3://$S3_BUCKET/$repo
```

Create a new repository
----------

    ./createrepo.sh myproject

First push
----------
Change the `origin` url of your git repository and push
```bash
$ git remote set-url origin git@git.[yourhost]:[yourrepo].git
$ git push origin master
Counting objects: 232, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (108/108), done.
Writing objects: 100% (232/232), 52.32 KiB | 0 bytes/s, done.
Total 232 (delta 119), reused 232 (delta 119)
remote: upload: refs/heads/master to s3://[yourbucket]/[repo].git/refs/heads/master
...
To git@git.[yourhost]:[yourrepo].git
 * [new branch]      master -> master
```

If everything worked, you will see the upload process as output of your git push.

Transfer repository from github to your server
-----------

```bash
git clone --mirror git@github.com:[user]/[repo].git
git remote set-url --push origin git@git.[server]:[repo].git
git push --mirror
```
