---
title: Docker tooling - Latex
date: 2015-04-19
comments: true
---

[Latex](http://www.latex-project.org/) is a known typesetting system. 

Docker containers are ideal if you need an environment to compile sources without installing the actual compiler on your machine. A full latex installation uses quite a lot of space and containerizing the whole installation has many pros. 

# Latex image

To install your latex image, there's nothing else to do than to pull `blang/latex`.

```
docker pull blang/latex
```

Of course you can build it by yourself, have a look at the [Dockerfile](https://github.com/blang/latex-docker/blob/master/Dockerfile).

This image contains a full latex installation but you might add some packages, so feel free to build your own image based on the Dockerfile.

# Create an alias

To work with this container it's much more comfortable if you create a small shell script.

`dockercmd.sh`
```
#!/bin/sh
exec docker run --rm -i --user="$(id -u):$(id -g)" -v $PWD:/data blang/latex "$@"
```

First let me explain what it does.

Since latex compiles sources, you need to map the sources via volume to docker:
`-v "$PWD:/data"` mounts your current working directory to the containers `/data`, the working directory of the container.

You also want latex to work with your users UID and GID, otherwise you end up with directories owned by root.
`--user="$(id -u):$(id -g)"`

# Using the container
```
./dockercmd.sh pdflatex example.tex
```

This command mounts your working directory as a volume and executes `pdflatex example.tex`. Simple as that.

You can execute every latex command using the prefix `./dockercmd.sh`

Check out my [example.tex](https://github.com/blang/latex-docker/blob/master/example/example.tex) to get started.

# Small limitations

Since every execution of `dockercmd.sh` spins up a docker container, execution might be slower than a local installation, especially if you're running `pdflatex` multiple time in one compile cycle.

Multiple executions:
```
./dockercmd.sh pdflatex example.tex
./dockercmd.sh pdflatex example.tex
```

Here's a cool trick to work inside one container:

```
latexdockercmd.sh /bin/sh -c "pdflatex example.tex && pdflatex example.tex"
```