---
title: "Building containers from scratch: Layers"
date: 2024-06-07T03:14:58.423Z
authorURL: ""
originalURL: https://depot.dev/blog/building-container-layers-from-scratch
translator: ""
reviewer: ""
---

At Depot, we're focused on providing the fastest build service for container images. We accomplish this primarily by:

<!-- more -->

1.  Providing instant access to powerful compute and storage
2.  Optimizing the build process itself to be as fast as possible

We run Depot on top of AWS, using large 16-core machines for each Depot project. These machines use native Intel and Arm CPUs, avoiding emulation for multi-platform images. And we provide them with distributed cache storage using a Ceph cluster with NVMe SSDs. This all makes it quick to execute `RUN` statements and makes cache lookups and writes fast.

For the build process itself, in addition to many [high-level optimizations][1] of the build process, we're currently working on a number of low-level optimizations to the build process itself.

To better understand some of these optimizations, it's helpful to understand the OCI container image layer format itself.

## [][2]Layer format

They're just tarballs!

The [OCI image spec][3] for container images ("Docker images") defines a container image as a collection of "layers" and metadata. Each layer is a collection of files stored as a [tar archive][4].

When an image is unpacked, layers stack on each other to form the filesystem of the container. Conceptually, this can be seen as extracting each tarball on top of each other until you have the entire filesystem, or "unioning" all the layers together.

Take this Dockerfile for example:

```
FROM ubuntu:22.04
COPY hello.txt .
COPY world.txt .
```

When built, this will produce a container image with three layers:

1.  A "layer" (tarball) with the Ubuntu 22.04 base image files
2.  A "layer" (tarball) that contains the `hello.txt` file
3.  A "layer" (tarball) which contains the `world.txt` file

After unpacking the resulting image, the container filesystem would look like this:

```
/
├── bin/
├── boot/
├── dev/
├── etc/
├── home/
├── lib/
├── media/
├── mnt/
├── opt/
├── proc/
├── root/
├── run/
├── sbin/
├── srv/
├── sys/
├── tmp/
├── usr/
├── var/
├── hello.txt
└── world.txt
```

This is all the files from the Ubuntu 22.04 base layer, plus the `hello.txt` file from the second layer, plus the `world.txt` file from the third layer.

Today, tools like Docker and BuildKit produce one tar layer for each `RUN`, `COPY`, `ADD`, etc. statement in the Dockerfile (or in the target stage of a multi-stage Dockerfile). Note that this is not a requirement of the OCI image spec, which doesn't specify how layers are created.

An OCI container image is just a collection of tarballs.

## [][5]Assembling an OCI image

If container image layers are just tarballs, and they are unioned together to form the container filesystem, how is it possible to have files that are removed or modified in later layers?

### [][6]Handling modified files

Modified files are straightforward: if a file is included in multiple layers, the last layer to include that file "wins". For example:

```
FROM scratch
RUN echo "hello" > example.txt
RUN echo "world" > example.txt
```

This will produce an image with two layers:

1.  A tarball with the `example.txt` file containing the text `hello`
2.  A tarball with the `example.txt` file containing the text `world`

When unpacked, the first layer's `example.txt` file will be overwritten by the second layer's `example.txt` file, resulting in a container filesystem with a single `example.txt` file containing the text `world`.

### [][7]Handling removed files

But what about files that are removed? For example:

```
FROM scratch
RUN echo "hello" > example.txt
RUN rm example.txt
```

For this, the OCI image spec defines a special kind of file called a "whiteout" file.

Whiteout files are empty files with a special name that tells the container runtime that a path should be removed from the container filesystem. Whiteout files have a special prefix of `.wh.` plus the name of the file to be removed.

For example, the second layer produced by the above example would contain a zero-byte file named `.wh.example.txt`. That instructs the container runtime to remove the `example.txt` file from the container filesystem when unpacking the layer.

**Note:** This is a common cause of security vulnerabilities with container builds. If a file is included in an earlier layer and then removed in a later layer, the file's contents still exist in the container image contents. This can result in leaking sensitive information, such as credentials or private keys.

There is one additional special whiteout file called the "opaque whiteout", named `.wh..wh..opq`. This file instructs the container runtime to remove all files and directories in the same directory as the opaque whiteout file. For example, if a layer contains a file named `/example/.wh..wh..opq`, that instructs the container runtime to remove all files and directories that are children of the `/example` directory.

Finally, layers are often distributed as gzipped tarballs, with the `.tar.gz` extension, to save storage space and reduce network data transfer. Note that the spec also supports uncompressed tarballs (`.tar`) and zstd-compressed tarballs (`.tar.zstd`).

## [][8]Overlay filesystems

Container runtimes like containerd or podman are responsible for taking an image's layers (tarballs) and unpacking them into a directory before running the container. This is called the container's "rootfs", or root filesystem.

Actually un-tarring each layer into the rootfs directory sequentially, taking care to apply file modifications or whiteout deletes, for each container launched, would be very slow. Instead, container runtimes often use special filesystems to efficiently combine the layers into a single filesystem.

One such filesystem is [overlayfs][9], which is a Linux kernel feature that allows multiple directories to be combined into a single directory. This is the default filesystem used by Docker and Podman.

With overlayfs, each layer is unpacked into a separate directory, and then the container runtime tells the Linux kernel to mount these directories on top of each other. The kernel then presents the combined directories as a single directory to the container runtime.

Overlayfs supports the concept of whiteout files natively, so the container runtime translates the OCI image whiteout files into overlayfs whiteout files when mounting the layers.

This allows each image layer to only be unpacked once, so running multiple containers from the same image is very fast, and sharing common base layers between images is very efficient. And the Linux kernel handles all the complexity of combining the layers into a single filesystem.

## [][10]Lazy image layers with eStargz

While image layers are "just" tar archives containing files, it is possible to extend the format with additional capabilities for more efficient storage and transfer. One example of this is the [eStargz][11] image format.

eStargz images are still valid tar archives, but they are constructed in a special way that allows a metadata file to describe the location of each individual file inside the compressed tarball. This allows access to those individual files without needing to download the entire layer from the registry and decompress.

For example, to locate a specific file in an eStargz layer, the container runtime would:

1.  Fetch the end of the compressed tarball, which contains the index of all files in the layer (the "TOC")
2.  Read the byte offset and length of the file from the TOC
3.  Fetch the specified byte range from the registry

For large layers, this can result in significant savings in download size and time.

This kind of optimization can be useful when starting a container because the container is able to begin running before the entire image has been downloaded. Then, as the container runs and files are accessed, those files are lazy-loaded from the registry.

This can be beneficial for building images as well. We built [depot.ai][12] as one example of this. With the depot.ai images, popular machine-learning models from Hugging Face are packaged as eStargz-compatible images. Then when copying those ML models into a new container image, Depot is able to only download the files that the `COPY` requests. This can be much faster than downloading the entire model repository, especially if the model repository contains multiple formats of the model where only one is needed.

Other examples of this kind of optimization include the [SOCI snapshotter][13] from AWS, the [Nydus][14] image format, and [`zstd:chunked`][15] from Red Hat.

## [][16]Optimizing layer construction

Considering that container image layers are tarballs, we're exploring a variety of optimizations to the layer construction process to make it faster and more efficient:

1.  Depot supports both creating and consuming eStargz images today.
2.  Depot parallelizes the compression of tarballs using multiple CPU cores. Today we divide layer tarballs into chunks and gzip each chunk in parallel, then concatenate the chunks together. This results in a slightly larger compressed tarball, but with vastly improved compression speed.
3.  We're investigating parallelizing single-layer construction — "building" a layer means creating a tarball from a directory of files. Today, this is done sequentially, but the construction of the tar archive headers can be parallelized.
4.  We're exploring alternate ways of constructing layers that don't require a `Dockerfile`. Today, Dockerfiles are the most common way of describing a container build, but given that layers are tarballs, it's possible to "just" make a tarball.

---

There are many more facets to optimizing the container build process, and we hope to share more of our work in the future. If any of this is interesting to you, feel free to reach out on [Twitter][17] or [Discord][18].

[1]: https://twitter.com/kylegalbraith/status/1746161367290167705
[2]: #layer-format
[3]: https://github.com/opencontainers/image-spec/blob/main/spec.md
[4]: https://en.wikipedia.org/wiki/Tar_(computing)
[5]: #assembling-an-oci-image
[6]: #handling-modified-files
[7]: #handling-removed-files
[8]: #overlay-filesystems
[9]: https://www.kernel.org/doc/html/latest/filesystems/overlayfs.html
[10]: #lazy-image-layers-with-estargz
[11]: https://github.com/containerd/stargz-snapshotter/blob/main/docs/estargz.md
[12]: https://depot.ai/
[13]: https://github.com/awslabs/soci-snapshotter
[14]: https://nydus.dev/
[15]: https://www.redhat.com/sysadmin/faster-container-image-pulls
[16]: #optimizing-layer-construction
[17]: https://twitter.com/jacobwgillespie
[18]: https://discord.gg/MMPqYSgDCg