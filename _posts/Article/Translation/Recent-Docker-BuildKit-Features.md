---
title: Recent Docker BuildKit Features You're Missing Out On
date: 2024-05-13T00:00:00.000Z
authorURL: ""
originalURL: https://martinheinz.dev/blog/111
translator: ""
reviewer: ""
---
# Recent Docker BuildKit Features You're Missing Out On | Martin Heinz | Personal Website & Blog
With introduction of BuildKit - the improved builder backend for Docker - many new features has been added to Docker, many of which are little known. So, here's a rundown of the ones you definitely need to know about and should start using to make better use of Docker.

Debugging
---------

Starting with the most common of tasks - debugging. Debugging `docker build` has always been a pain - if some `RUN` or `COPY` command fails you can hardly view context and debug what went wrong, usually resorting to adding `RUN ls -la` and similar to get more info. That however now changes with introduction of `docker buildx debug`:

```

export BUILDX_EXPERIMENTAL=1
docker buildx debug --invoke /bin/sh --on=error build .

[+] Building 1.2s (14/18)                docker:default
...
------
 > [builder 5/6] RUN exit 1:
------
Dockerfile:10
--------------------
   8 |     RUN pip3 install -r requirements.txt
   9 |
  10 | >>> RUN exit 1
  11 |
  12 |     COPY . /app
--------------------
ERROR: process "/bin/sh -c exit 1" did not complete successfully: exit code: 1
[+] Building 0.0s (0/0)                  docker:default
Launching interactive container. Press Ctrl-a-c to switch to monitor console
Interactive container was restarted with process "u6agxp1ywqapemxrt8iexfv4h". Press Ctrl-a-c to switch to the new container
/ # ls -la
total 72
drwxr-xr-x    1 root     root          4096 May  5 12:59 .
drwxr-xr-x    1 root     root          4096 May  5 12:59 ..
drwxr-xr-x    1 root     root          4096 May  4 10:11 app
...

```


As we can see in the snippet above, we start by enabling experimental BuildKit features with `BUILDX_EXPERIMENTAL` environment variable. We then start build through `docker buildx debug` - if the build fails at any point we will be dropped into the container and can explore the context and debug.

Notice that we included the `--on=error` option that will only start the debug session if the build fails.

For more details see [debugging docs](https://github.com/docker/buildx/blob/master/docs/debugging.md).

Environment Variables
---------------------

If you've run a build with BuildKit before, then you've noticed the fancy new log output. It does look nice but it's not very practical, especially when debugging. There's however an environment variable to switch to plain log output:

```

export BUILDKIT_PROGRESS=plain

```


You can also set it to `rawjson` which is definitely not human-readable, but could be useful if you want to process the logs in some way.

Alternatively, if you like the TTY-based dynamic output, but dislike the colors, then you can simply change them with:

```

BUILDKIT_COLORS="run=green:warning=yellow:error=red:cancel=cyan" docker buildx debug --invoke /bin/sh --on=error build .

```


Making the output look like:

![Buildx Colors](https://i.imgur.com/gTasZHC.png)

See [docs](https://docs.docker.com/build/building/variables/#build-tool-configuration-variables) for other environment variables.

Exporters
---------

BuildKit also introduces concept of _exporters_, which define how the output of a build will be saved. The 2 most useful options are `image` and `registry`. `image` -as you could expect - saves output as a container image, while `registry` exporter automatically pushes to specified registry:

```

docker buildx build --output type=registry,name=martinheinz/testimage:latest .

```


All we need to do is specify `--output` option with type `registry` and the destination. This option additionally supports specifying multiple registries at once:

```

docker buildx build --output type=registry,\"name=docker.io/martinheinz/testimage,docker.io/martinheinz/testimage2\" .

```


Finally, we can also provide `--cache-to` and `--cache-from` options to e.g. use existing image from registry as a cache source:

```

docker buildx build --output type=registry,name=martinheinz/testimage:latest \
 --cache-to type=inline \
 --cache-from type=registry,ref=docker.io/martinheinz/testimage .

...
 => CACHED docker-image://docker.io/docker/dockerfile:1.4@sha256:9ba7531bd80fb0a8...1e24ef1a0dbc
...
 => CACHED [builder 2/5] WORKDIR /app                                                                            0.0s
 => CACHED [builder 3/5] COPY requirements.txt /app                                                              0.0s
 => CACHED [builder 4/5] RUN --mount=type=cache,target=/root/.cache/pip     pip3 install -r requirements.txt     0.0s
 => CACHED [builder 5/5] COPY . /app                                                                             0.0s
 => CACHED [dev-envs 1/3] RUN <<EOF (apk update...)                                                              0.0s
 => CACHED [dev-envs 2/3] RUN <<EOF (addgroup -S docker...)                                                      0.0s
 => CACHED [dev-envs 3/3] COPY --from=gloursdocker/docker / /                                                    0.0s
 => preparing layers for inline cache                                                                            0.0s
...

```


Imagetools
----------

Simple, but handy subcommand of `docker buildx` called `imagetools`, allows us to inspect of an image in registry without having to pull it. The [documentation](https://docs.docker.com/reference/cli/docker/buildx/imagetools/inspect/) includes a lot of examples, but the most useful one for me is getting a digest of remote image:

```

docker buildx imagetools inspect alpine --format "{{json .Manifest}}" | jq .digest
"sha256:c5b1261d6d3e43071626931fc004f70149baeba2c8ec672bd4f27761f8e1ad6b"

```


Latest Dockerfile Syntax
------------------------

With BuildKit comes also new Dockerfile syntax through what's called _[Dockerfile frontend](https://docs.docker.com/build/dockerfile/frontend/)_. To enable current latest syntax we need to add a directive to the top of the Dockerfile, e.g.:

```

# syntax=docker/dockerfile:1.3
FROM ...

```


To find version you can check `dockerfile-upstream` [Docker Hub repository](https://hub.docker.com/r/docker/dockerfile-upstream).

Here-docs
---------

First of these Dockerfile syntax improvements I want to mention is _here-docs_, which allows us to pass multiline scripts into `RUN` and `COPY` commands:

```

# syntax = docker/dockerfile:1.3-labs
FROM debian
RUN <<eot bash
  apt-get update
  apt-get install -y vim
eot

# Same as:
RUN apt-get update && apt-get install -y vim

```


In the past we would have to use `&&` if we wanted to put multiple commands into single `RUN`, now with here-docs, we can write a normal script.

Additionally, first line can specify interpreter, so we can - for example - write a Python script too:

```

# syntax = docker/dockerfile:1.3-labs
FROM python:3.6
RUN <<eot
#!/usr/bin/env python
print("hello world")
eot

```


`COPY` and `ADD` Features
-------------------------

In this new Dockerfile syntax, there are also more subtle changes and improvements to `COPY` and `ADD` in form of new options.

`COPY` now supports `--parents` option:

```

# syntax=docker/dockerfile:1.7.0-labs
FROM ubuntu

COPY ./one/two/some.txt /normal/

RUN find /normal
#10 [3/5] RUN find /normal
#10 0.223 /normal
#10 0.223 /normal/some.txt

COPY --parents ./one/two/some.txt /parents/

RUN find /parents
#12 [5/5] RUN find /parents
#12 0.509 /parents
#12 0.509 /parents/one
#12 0.509 /parents/one/two
#12 0.509 /parents/one/two/some.txt

```


If you copy a nested file with normal `COPY`, the image will contain only the file itself without the parent directories, with `--parents` the whole file tree is copied, similar to how `cp --parents` does it.

Similarly to `--parents` option, you can also use `--exclude`:

```

COPY --exclude=*.txt ./some-dir/* ./some-dest

```


Which will omit excluded files (and patterns) when copying.

Finally, `ADD` command has received an improvement too - it is now possible to directly add Git repository:

```

# syntax=docker/dockerfile:1.7.0-labs
FROM ubuntu

ADD git@github.com:kelseyhightower/helloworld.git /repo
RUN ls -la /repo

```


And when running build for this Dockerfile, we will get:

```

docker buildx build --ssh default --progress=plain .
#8 [2/3] ADD git@github.com:kelseyhightower/helloworld.git /repo
#8 0.478 Warning: Permanently added 'github.com' (ED25519) to the list of known hosts.
#8 1.738 ref: refs/heads/master HEAD
#8 1.738 96a652519d1aaca11085ca3a7806bead4d2c273f HEAD
#8 3.478 96a652519d1aaca11085ca3a7806bead4d2c273f refs/heads/master
#8 1.829 ref: refs/heads/master HEAD
#8 1.829 96a652519d1aaca11085ca3a7806bead4d2c273f HEAD
#8 3.838 From github.com:kelseyhightower/helloworld
#8 3.838  * [new branch]      master     -> master
#8 3.838  * [new branch]      master     -> origin/master
#8 DONE 7.4s

#9 [2/3] ADD git@github.com:kelseyhightower/helloworld.git /repo
#9 DONE 0.0s

```


This will also work for [private repositories](https://docs.docker.com/reference/dockerfile/#adding-private-git-repositories).

See more interesting options in [docs](https://docs.docker.com/reference/dockerfile/#add), such `ADD --keep-git-dir` or `ADD --checksum` for validating artifact checksums

Bonus: Indentation
------------------

And while not a BuildKit feature, one thing I discovered recently is that you can indent lines in Dockerfile and it will work just fine, which allows for more readability, especially with multistage builds:

```

# syntax=docker/dockerfile:1
FROM golang:1.21
  WORKDIR /src

  COPY main.go .
  RUN go build -o /bin/hello ./main.go

FROM scratch
  COPY --from=0 /bin/hello /bin/hello
  CMD ["/bin/hello"]

```


It looks weird at first glance, but it's more readable in my opinion, making it more clear where each stage starts and which commands belong to it.

Conclusion
----------

The examples in this article only show the features that I find the most useful, but there's plenty more, so be sure to check out official [Docker docs](https://docs.docker.com/reference/cli/docker/buildx/), but also [BuildKit docs](https://github.com/moby/buildkit/tree/master/docs), which have the latest changes. Docker blog is also a great resources, posts tagged _[buildkit](https://www.docker.com/blog/tag/buildkit/)_ or _[buildx](https://www.docker.com/blog/tag/buildx/)_ specifically.