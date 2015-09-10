---
title: Docker tooling - Jekyll
date: 2015-04-19
comments: true
---

As part of an upcoming docker tooling series we start with [Jekyll](http://jekyllrb.com/), a static website generator used for blogs and websites. [Jekyll](http://jekyllrb.com/) is also supported by [Github pages](https://pages.github.com/), so check it out if you're not familiar with it.

Docker containers are ideal if you need an environment to compile sources without installing the actual compiler on your machine. Especially interesting in case of ruby since setting up a proper ruby environment is hard (and annoying).

Jekyll is a ruby based website generator which takes some sources (Markdown for example) and compiles them to html.

# Jekyll container

To install your jekyll container, there's nothing else to do than to pull `grahamc/jekyll`.

```
docker pull grahamc/jekyll
```

# Create an alias

To work with this container it's much more comfortable if you create a proper alias.

`alias.source`
```
alias jekyll='docker run -it --rm --name jekyll -v "$PWD:/src" --user="$(id -u):$(id -g)" -p 127.0.0.1:4000:4000 grahamc/jekyll'
```

And you need to source this file:
```
source ./alias.source
OR
. ./alias.source
```

After that, you have jekyll ready as if it's installed on you machine using the command `jekyll`. Ok, i guess there are some limitations which shouldn't bother you too much. 

First let me explain what it does.

It binds the docker command to the alias `jekyll` which means you can execute `jekyll serve` and it really executes `docker run ... grahamc/jekyll serve`.

Since jekyll compiles sources, you need to map the sources via volume to docker:
`-v "$PWD:/src"` mounts your current working directory to the containers `/src`, the working directory of jekyll.

You also want jekyll to work with your users UID and GID, otherwise you end up with directories owned by root.
`--user="$(id -u):$(id -g)"`

It also exposes jekylls default port 4000 to your machine.

# Create a new site
```
jekyll new dockersite
```

Jekyll will generate a new directory called `dockersite` inside your current working directory.

# Build and serve your new site
```
cd dockersite
jekyll serve --host 0.0.0.0
```

Now have a look at [http://127.0.0.1:4000](http://127.0.0.1:4000) your site was compiled an is served.
Remember to use the `host` flag because jekyll by default binds to localhost, which is the containers localhost and not yours.


# Summary
You now have a containerized jekyll installation ready without cumbering ruby artifacts on your system.

If you run into trouble, docker might not be able to remove the container and therefor return an error on the next start.
Simply remove it first
```
docker kill jekyll
docker rm jekyll
```

