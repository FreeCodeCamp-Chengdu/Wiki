---
title: Understanding Container Image Layers
date: 2024-06-07T03:12:52.216Z
author: Ken Muse
authorURL: https://github.com/kenmuse
originalURL: https://www.kenmuse.com/blog/understanding-container-image-layers/
translator: ""
reviewer: ""
---

# Understanding Container Image Layers

<!-- more -->

**Category:**

-   [DevOps][1]

**Tags:**

-   [#DevOps][2]

-   [#Containers][3]

-   [#Docker][4]

**Published:** [January 27][5], [2024][6] **Reading Time:** 9 min

---

Containers are pretty amazing. They allow simple processes to act like virtual machines. Underneath this elegance is a set of patterns and practices that ultimately makes everything work. At the root of the design is _layers_. Layers are the fundamental way of storing and distributing the contents of a containerized file system. The design is both surprisingly simple and at the same time very powerful. In today’s post, I’ll explain what layers are and how they work conceptually.

## Building layered images

When you create an image, you typically use a `Dockerfile` to define the contents of the container. It contains a series of commands, such as:

```bash
1FROM scratch
2RUN echo "hello" > /work/message.txt
3COPY content.txt /work/content.txt
4RUN rm -rf /work/message.txt
```

Under the covers, the container engine will execute these commands in order, creating a “layer” for each. But what is really happening? It’s easiest to think of each layer as a directory that holds all of the modified files.

Let’s step through an example of a possible implementation approach.

1.  `FROM scratch` indicates that this container is starting with no contents. This is the first layer, and it could be represented by an empty directory, `/img/layer1`.
2.  Create a second directory, `/img/layer2` and copy everything from `/img/layer1` into it. Then, execute the next command from the Dockerfile (which writes a file to `/work/message.txt`). Those contents are written to `/img/layer2/work/message.txt`. This is the second layer.
3.  Create a third directory, `/img/layer3` copying everything from `img/layer2` into it. The next Dockerfile command requires copying `content.txt` from the host to that directory. That file is written to `/img/layer3/work/content.txt`. This is the third layer.
4.  Finally, create a fourth directory, `/img/layer4` copying everything from `img/layer3` into it. The next command deletes the message file, `img/layer4/work/message.txt`. This is the fourth layer.

To share these layers, the easiest approach is to create a compressed `.tar.gz` for each directory. To reduce the total file size, any files that are unmodified copies of data from a previous layer would be removed. To make it clear when a file was deleted, a “whiteout file” could be used as a placeholder. The file would simply prefix `.wh.` to the original filename. For example, the fourth layer would replace the deleted file with a placeholder named `.wh.message.txt`. When a layer is unpacked, any files that start with `.wh.` can be deleted.

Continuing our example, the compressed files would contain:

| File | Contents |
| --- | --- |
| `layer1.tar.gz` | Empty file |
| `layer2.tar.gz` | Contains `/work/message.txt` |
| `layer3.tar.gz` | Contains `/work/content.txt` (since `message.txt` was not modified) |
| `layer4.tar.gz` | Contains `/work/.wh.message.txt` (since `message.txt` was deleted).  
The file `content.txt` was not modified, so it is not included. |

Building lots of images this way would result in lots of “layer1” directories. To make sure the name is unique, the compressed file is named based on a digest of the contents. This is similar to how Git works. It has the benefit of identifying identical content while identifying any corruption of the files while downloading. If the digest of the contents does not match the file name, the file is corrupt.

To make the results reproducible, one more thing is required — a file that explains how to order the layers (a manifest). The manifest would identify which files to be downloaded the order for unpacking them. This enables recreating the directory structures. It also provides an important benefit: layers can be reused and shared between images. This minimizes the local storage requirements.

In practice, there are more optimizations available. For example, `FROM scratch` really means there is no parent layer, so our example really starts with the contents of `layer2`. The engine can also look at the files used in the build to determine whether or not a layer needs to be recreated. This is the basis for layer caching, which minimizes the need to build or recreate layers. As an additional optimizing, layers that don’t depend on the previous layer can use `COPY --link` to indicate that the layer won’t need to delete or modify any files from the previous layer. This allows the compressed layer file to be created in parallel to the other steps.

## Snapshots

Before a container can run, it needs a file system to mount. In essence, it needs a directory with all of the files that need to be available. The compressed layer files contain the components of the file system, but they can’t be directly mounted and used. Instead, they need to be unpacked and organized into a file system. This unpacked directory is called a _snapshot_ (well, it’s one of a few things with that name 😄).

The process of creating a snapshot is the opposite of image building. It starts by downloading the manifest and building a list of layers to download. For each layer, a directory is created with the contents of the layer’s parent. This directory is called the _active snapshot_. Next, a _diff applier_ is responsible for unpacking the compressed layer file and applying the changes to the active snapshot. The resulting directory is then called a _committed snapshot_. The final committed snapshot is the one that is mounted as the container’s file system.

Using our earlier example:

1.  The initial layer, `FROM scratch`, means that we can start with the next layer and an empty directory. There is no parent.
2.  A directory for `layer2` is created. This empty directory is now an active snapshot. The file `layer2.tar.gz` is downloaded, validated (by comparing the digest to the file name), and unpacked into the directory. The result is a directory containing `/work/message.txt`. This is the first committed snapshot.
3.  A directory for `layer3` is created, and the contents of `layer2` are copied into it. This is a new active snapshot. The file `layer3.tar.gz` is downloaded, validated, and unpacked. The result is a directory containing `/work/message.txt` and `/work/content.txt`. This is now the second committed snapshot.
4.  A directory for `layer4` is created, and the contents of `layer3` are copied into it. The file `layer4.tar.gz` is downloaded, validated, and unpacked. The diff applier recognizes the whiteout file, `/work/.wh.message.txt`, and deletes `/work/message.txt`. This leaves just `/work/content.txt`. This is the third committed snapshot.
5.  Since `layer4` was the last layer, it is the basis for a container. To enable it to support read and write operations, a new snapshot directory is created and the contents of `layer4` are copied into it. This directory is mounted as the container’s file system. Any changes made by the running container will happen in this directory.

If any of those directories already exist, it indicates that another image had the same dependency. As a result, the engine can skip the download and diff applier. It can use the layer as-is. In practice, each of these directories and files is named based on the digest of the contents to make that easier to identify. For example, a set of snapshots might look like this:

```bash
1/var/path/to/snapshots/blobs
2└─ sha256
3   ├─ 635944d2044d0a54d01385271ebe96ec18b26791eb8b85790974da36a452cc5c
4   ├─ 9de59f6b211510bd59d745a5e49d7aa0db263deedc822005ed388f8d55227fc1
5   ├─ fb0624e7b7cb9c912f952dd30833fb2fe1109ffdbcc80d995781f47bd1b4017f
6   └─ fb124ec4f943662ecf7aac45a43b096d316f1a6833548ec802226c7b406154e9
```

or alternatively:

| Image | Parent |
| --- | --- |
| sha256:635944d2044d0a54d01385271ebe96ec18b26791eb8b85790974da36a452cc5c |  |
| sha256:9de59f6b211510bd59d745a5e49d7aa0db263deedc822005ed388f8d55227fc1 | sha256:635944d2044d0a54d01385271ebe96ec18b26791eb8b85790974da36a452cc5c |
| sha256:fb0624e7b7cb9c912f952dd30833fb2fe1109ffdbcc80d995781f47bd1b4017f | sha256:9de59f6b211510bd59d745a5e49d7aa0db263deedc822005ed388f8d55227fc1 |
| sha256:fb124ec4f943662ecf7aac45a43b096d316f1a6833548ec802226c7b406154e9 | sha256:fb0624e7b7cb9c912f952dd30833fb2fe1109ffdbcc80d995781f47bd1b4017f |

The actual snapshot system supports plugins that can improve some of these behaviors. For example, it can allow the snapshots to be pre-composed and unpacked, speeding up the process. This allows the snapshots to be stored remotely. It also allows for special optimizations, such as just-in-time downloading of the needed files and layers.

## Overlays

While it’s easy to mount, the snapshot approach we just described creates a lot of file churn and lots of duplicate files. This slows down the process of starting a container the first time and wastes space. Thankfully, this is one of many aspects of the containerization process that can be handled by the file system. Linux natively supports mounting directories as overlays, implementing most of the process for us.

In Linux (or a Linux container running as `--privileged` or with `--cap-add=SYS_ADMIN`):

1.  Create a `tmpfs` mount (memory-based file system that will be used to explore the overlay process)
    
    ```bash
    1mkdir /tmp/overlay
    2mount -t tmpfs tmpfs /tmp/overlay
    ```
    
2.  Create directories for our process. We’ll use `lower` for the lower (parent) layer, `upper` for the upper (child) layer, `work` as a working directory for the file system, and `merged` to contain the merged file system.
    
    ```bash
    1mkdir /tmp/overlay/{lower,upper,work,merged}
    ```
    
3.  Create some files for the experiment. Optionally, you can add files in `upper` as well.
    
    ```bash
    1cd /tmp/overlay
    2echo hello > lower/hello.txt
    3echo "I'm only here for a moment" > lower/delete-me.txt
    4echo message > upper/upper-message.txt
    ```
    
4.  Mount these directories as an `overlay` type file system. This will create a new file system in the `merged` directory that contains the combined contents of the `lower` and `upper` directory. The `work` directory will be used to track changes to the file system.
    
    ```bash
    1mount -t overlay overlay -o lowerdir=lower,upperdir=upper,workdir=work merged
    ```
    
5.  Explore the file system. You’ll notice that `merged` contains the combined contents of `upper` and `lower`. Then, make some changes:
    
    ```bash
    1rm -rf merged/delete-me.txt
    2echo "I'm new" > merged/new.txt
    3echo world >> merged/hello.txt
    ```
    
6.  As expected, `delete-me.txt` is removed from `merged` and a new file, `new.txt` is created in the same directory. If you `tree` the directories, you’ll see something interesting:
    
    ```txt
       |-- lower
       |   |-- delete-me.txt
       |   `-- hello.txt
       |-- merged
       |   |-- hello.txt
       |   |-- new.txt
       |   `-- upper-message.txt
       |-- upper
       |   |-- delete-me.txt
       |   |-- hello.txt
       |   |-- new.txt
       |   `-- upper-message.txt
    ```
    
    And running `ls -l upper` shows
    
    ```bash
    1total 12
    2c--------- 2 root root 0, 0 Jan 20 00:17 delete-me.txt
    3-rw-r--r-- 1 root root   12 Jan 20 00:20 hello.txt
    4-rw-r--r-- 1 root root    8 Jan 20 00:17 new.txt
    5-rw-r--r-- 1 root root    8 Jan 20 00:17 upper-message.txt
    ```
    

While `merged` shows the effects of our changes, `upper` (as the parent layer) stores the changes similar to the example in our manual process. It contains the new file, `new.txt` and the modified `hello.txt`. It has also created a whiteout file. For the overlay filesystem, this involves replacing the file with a character device (and a 0, 0 device number). In short, it has everything we need to be able to package up the directories!

You can see how this approach could also be used to implement a snapshot system. The `mount` command can natively take a colon (`:`) delimited list of `lowerdir` paths, all of which are unioned together into a single file system. This is part of the nature of modern containers – the containers are composed using native operating system features.

That’s really all there is to creating a basic system. In fact, the `containerd` runtime used by Kubernetes (and the recently release Docker Desktop 4.27.0) uses a similar approach to build and manage its images (with the deeper details covered in [Content Flow][7]). Hope this has helped to demystify the way container images work!

[1]: /categories/devops
[2]: /tags/devops
[3]: /tags/containers
[4]: /tags/docker
[5]: /blog/2024/01
[6]: /blog/2024
[7]: https://github.com/containerd/containerd/blob/main/docs/content-flow.md