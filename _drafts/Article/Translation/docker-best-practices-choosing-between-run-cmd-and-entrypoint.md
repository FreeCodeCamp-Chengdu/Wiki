---
title: "Docker Best Practices: Choosing Between RUN, CMD, and ENTRYPOINT"
date: 2024-07-17T00:00:00.000Z
author: Jay Schmidt
authorURL: https://www.docker.com/author/jay-schmidt/
originalURL: https://www.docker.com/blog/docker-best-practices-choosing-between-run-cmd-and-entrypoint/
translator: ""
reviewer: ""
---

# Docker Best Practices: Choosing Between RUN, CMD, and ENTRYPOINT

<!-- more -->

![](https://www.docker.com/wp-content/uploads/2024/04/jay-schmidt.jpeg)

[Jay Schmidt][1]  

Docker’s flexibility and robustness as a containerization tool come with a complexity that can be daunting. Multiple methods are available to accomplish similar tasks, and users must understand the pros and cons of the available options to choose the best approach for their projects.

One confusing area concerns the [`RUN`][2], [`CMD`][3], and [`ENTRYPOINT`][4] Dockerfile instructions. In this article, we will discuss the differences between these instructions and describe use cases for each.

![2400x1260 choosing between run cmd and entrypoint](https://www.docker.com/wp-content/uploads/2024/07/2400x1260_choosing-between-run-cmd-and-entrypoint-1110x583.png "- 2400X1260 Choosing Between Run Cmd And Entrypoint")

### RUN

The `RUN` instruction is used in [Dockerfiles][5] to execute commands that build and configure the Docker image. These commands are executed during the image build process, and each `RUN` instruction creates a new layer in the Docker image. For example, if you create an image that requires specific software or libraries installed, you would use `RUN` to execute the necessary installation commands.

The following example shows how to instruct the Docker build process to update the `apt cache` and install Apache during an image build:

RUN apt update && apt -y install apache2

`RUN` instructions should be used judiciously to keep the image layers to a minimum, combining related commands into a single `RUN` instruction where possible to reduce image size.

### CMD

The `CMD` instruction specifies the default command to run when a container is started from the Docker image. If no command is specified during the container startup (i.e., in the `docker run` command), this default is used. `CMD` can be overridden by supplying command-line arguments to `docker run`.

`CMD` is useful for setting default commands and easily overridden parameters. It is often used in images as a way of defining default run parameters and can be overridden from the command line when the container is run. 

For example, by default, you might want a web server to start, but users could override this to run a shell instead:

CMD \["apache2ctl", "-DFOREGROUND"\]

Users can start the container with `docker run -it <image> /bin/bash` to get a Bash shell instead of starting Apache.  

### ENTRYPOINT

The `ENTRYPOINT` instruction sets the default executable for the container. It is similar to `CMD` but is overridden by the command-line arguments passed to `docker run`. Instead, any command-line arguments are appended to the `ENTRYPOINT` command.

**Note:** Use `ENTRYPOINT` when you need your container to always run the same base command, and you want to allow users to append additional commands at the end. 

`ENTRYPOINT` is particularly useful for turning a container into a standalone executable. For example, suppose you are packaging a custom script that requires arguments (e.g., `“my_script extra_args”`). In that case, you can use `ENTRYPOINT` to always run the script process (`“my_script”`) and then allow the image users to specify the `“extra_args”` on the `docker run` command line. You can do the following:

ENTRYPOINT \["my\_script"\]

### Combining CMD and ENTRYPOINT

The `CMD` instruction can be used to provide default arguments to an `ENTRYPOINT` if it is specified in the exec form. This setup allows the entry point to be the main executable and `CMD` to specify additional arguments that can be overridden by the user.

For example, you might have a container that runs a Python application where you always want to use the same application file but allow users to specify different command-line arguments:

ENTRYPOINT \["python", "/app/my\_script.py"\]
CMD \["--default-arg"\]

Running `docker run myimage --user-arg` executes `python /app/my_script.py --user-arg`.

The following table provides an overview of these commands and use cases.

#### **Command description and use cases**

<table><tbody><tr><td><strong>Command</strong></td><td><strong>Description</strong></td><td><strong>Use Case</strong></td></tr><tr><td><a href="https://docs.docker.com/engine/reference/builder/#cmd" rel="nofollow noopener" target="_blank">CMD</a></td><td>Defines the default executable of a Docker image. It can be overridden by <code>docker run</code> arguments.</td><td>Utility images allow users to pass different executables and arguments on the command line.</td></tr><tr><td><a href="https://docs.docker.com/engine/reference/builder/#entrypoint" rel="nofollow noopener" target="_blank">ENTRYPOINT</a></td><td>Defines the default executable. It can be overridden by the <code>“--entrypoint”&nbsp; docker run</code> arguments.</td><td>Images built for a specific purpose where overriding the default executable is not desired.</td></tr><tr><td><a href="https://docs.docker.com/engine/reference/builder/#run" rel="nofollow noopener" target="_blank">RUN</a></td><td>Executes commands to build layers.</td><td>Building an image</td></tr></tbody></table>

### What is PID 1 and why does it matter?

In the context of Unix and Unix-like systems, including Docker containers, PID 1 refers to the first process started during system boot. All other processes are then started by PID 1, which in the process tree model is the parent of every process in the system. 

In Docker containers, the process that runs as PID 1 is crucial, because it is responsible for managing all other processes inside the container. Additionally, PID 1 is the process that reviews and handles signals from the Docker host. For example, a `SIGTERM` into the container will be caught and processed by PID 1, and the container should gracefully shut down.

When commands are executed in Docker using the shell form, a shell process (`/bin/sh -c`) typically becomes PID 1. Still, it does not properly handle these signals, potentially leading to unclean shutdowns of the container. In contrast, when using the exec form, the command runs directly as PID 1 without involving a shell, which allows it to receive and handle signals directly. 

This behavior ensures that the container can gracefully stop, restart, or handle interruptions, making the exec form preferable for applications that require robust and responsive signal handling.

### Shell and exec forms

In the previous examples, we used two ways to pass arguments to the `RUN`, `CMD`, and `ENTRYPOINT` instructions. These are referred to as shell form and exec form. 

**Note:** The key visual difference is that the exec form is passed as a comma-delimited array of commands and arguments with one argument/command per element. Conversely, shell form is expressed as a string combining commands and arguments. 

Each form has implications for executing commands within containers, influencing everything from signal handling to environment variable expansion. The following table provides a quick reference guide for the different forms.

#### **Shell and exec form reference**

<table><tbody><tr><td><strong>Form</strong></td><td><strong>Description</strong></td><td><strong>Example</strong></td></tr><tr><td><strong>Shell Form</strong></td><td>Takes the form of <code>&lt;INSTRUCTION&gt; &lt;COMMAND&gt;</code>.</td><td><code>CMD echo TEST </code>or<code> ENTRYPOINT echo TEST</code></td></tr><tr><td><strong>Exec Form</strong></td><td>Takes the form of <code>&lt;INSTRUCTION&gt; ["EXECUTABLE", "PARAMETER"]</code>.</td><td><code>CMD ["echo", "TEST"] </code>or<code> ENTRYPOINT ["echo", "TEST"]</code></td></tr></tbody></table>

In the shell form, the command is run in a subshell, typically `/bin/sh -c` on Linux systems. This form is useful because it allows shell processing (like variable expansion, wildcards, etc.), making it more flexible for certain types of commands (see this [shell scripting article][9] for examples of shell processing). However, it also means that the process running your command isn’t the container’s PID 1, which can lead to issues with signal handling because signals sent by Docker (like `SIGTERM` for graceful shutdowns) are received by the shell rather than the intended process.

The exec form does not invoke a command shell. This means the command you specify is executed directly as the container’s PID 1, which is important for correctly handling signals sent to the container. Additionally, this form does not perform shell expansions, so it’s more secure and predictable, especially for specifying arguments or commands from external sources.

### Putting it all together

To illustrate the practical application and nuances of Docker’s `RUN`, `CMD`, and `ENTRYPOINT` instructions, along with the choice between shell and exec forms, let’s review some examples. These examples demonstrate how each instruction can be utilized effectively in real-world Dockerfile scenarios, highlighting the differences between shell and exec forms. 

Through these examples, you’ll better understand when and how to use each directive to tailor container behavior precisely to your needs, ensuring proper configuration, security, and performance of your Docker containers. This hands-on approach will help consolidate the theoretical knowledge we’ve discussed into actionable insights that can be directly applied to your Docker projects.

### RUN instruction

For `RUN`, used during the Docker build process to install packages or modify files, choosing between shell and exec form can depend on the need for shell processing. The shell form is necessary for commands that require shell functionality, such as pipelines or file globbing. However, the exec form is preferable for straightforward commands without shell features, as it reduces complexity and potential errors.

\# Shell form, useful for complex scripting
RUN apt-get update && apt-get install -y nginx

# Exec form, for direct command execution
RUN \["apt-get", "update"\]
RUN \["apt-get", "install", "-y", "nginx"\]

### CMD and ENTRYPOINT

These instructions control container runtime behavior. Using exec form with `ENTRYPOINT` ensures that the container’s main application handles signals directly, which is crucial for proper startup and shutdown behavior.  `CMD` can provide default parameters to an `ENTRYPOINT` defined in exec form, offering flexibility and robust signal handling.

\# ENTRYPOINT with exec form for direct process control
ENTRYPOINT \["httpd"\]

# CMD provides default parameters, can be overridden at runtime
CMD \["-D", "FOREGROUND"\]

### Signal handling and flexibility

Using `ENTRYPOINT` in exec form and `CMD` to specify parameters ensures that Docker containers can handle operating system signals gracefully, respond to user inputs dynamically, and maintain secure and predictable operations. 

This setup is particularly beneficial for containers that run critical applications needing reliable shutdown and configuration behaviors. The following table shows key differences between the forms.

#### **Key differences between shell and exec**

<table><tbody><tr><td></td><td><strong>Shell Form</strong></td><td><strong>Exec Form</strong></td></tr><tr><td><strong>Form</strong></td><td>Commands without <code>[]</code> brackets. Run by the container’s shell, e.g., <code>/bin/sh -c</code>.</td><td>Commands with <code>[]</code> brackets. Run directly, not through a shell.</td></tr><tr><td><strong>Variable Substitution</strong></td><td>Inherits environment variables from the shell, such as <code>$HOME</code> and <code>$PATH</code>.</td><td>Does not inherit shell environment variables but behaves the same for <code>ENV</code> instruction variables.</td></tr><tr><td><strong>Shell Features</strong></td><td>Supports sub-commands, piping output, chaining commands, I/O redirection, etc.</td><td>Does not support shell features.</td></tr><tr><td><strong>Signal Trapping &amp; Forwarding</strong></td><td>Most shells do not forward process signals to child processes.</td><td>Directly traps and forwards signals like <code>SIGINT</code>.</td></tr><tr><td><strong>Usage with ENTRYPOINT</strong></td><td>Can cause issues with signal forwarding.</td><td>Recommended due to better signal handling.</td></tr><tr><td><strong>CMD as ENTRYPOINT Parameters</strong></td><td>Not possible with the shell form.</td><td>If the first item in the array is not a command, all items are used as parameters for the <code>ENTRYPOINT</code>.</td></tr></tbody></table>

Figure 1 provides a decision tree for using `RUN`, `CMD`, and `ENTRYPOINT` in building a Dockerfile.

![2400x1260 run cmd entrypoint](data://image/svg+xml,%3Csvg%20xmlns='http://www.w3.org/2000/svg'%20viewBox='0%200%201110%20583'%3E%3C/svg%3E "- 2400X1260 Run Cmd Entrypoint")

![2400x1260 run cmd entrypoint](https://www.docker.com/wp-content/uploads/2024/07/2400x1260_run-cmd-entrypoint-1110x583.png "- 2400X1260 Run Cmd Entrypoint")

Figure 1: Decision tree — RUN, CMD, ENTRYPOINT.

Figure 2 shows a decision tree to help determine when to use exec form or shell form.

![2400x1260 decision tree exec vs shell](data://image/svg+xml,%3Csvg%20xmlns='http://www.w3.org/2000/svg'%20viewBox='0%200%201110%20583'%3E%3C/svg%3E "- 2400X1260 Decision Tree Exec Vs Shell")

![2400x1260 decision tree exec vs shell](https://www.docker.com/wp-content/uploads/2024/07/2400x1260_decision-tree-exec-vs-shell-1110x583.png "- 2400X1260 Decision Tree Exec Vs Shell")

Figure 2: Decision tree — exec vs. shell form.

## Examples

The following section will walk through the high-level differences between `CMD` and `ENTRYPOINT`. In these examples, the `RUN` command is not included, given that the only decision to make there is easily handled by reviewing the two different formats.

### Test Dockerfile

\# Use syntax version 1.3-labs for Dockerfile
# syntax=docker/dockerfile:1.3-labs

# Use the Ubuntu 20.04 image as the base image
FROM ubuntu:20.04

# Run the following commands inside the container:
# 1. Update the package lists for upgrades and new package installations
# 2. Install the apache2-utils package (which includes the 'ab' tool)
# 3. Remove the package lists to reduce the image size
#
# This is all run in a HEREDOC; see
# https://www.docker.com/blog/introduction-to-heredocs-in-dockerfiles/
# for more details.
#
RUN <<EOF
apt-get update;
apt-get install -y apache2-utils;
rm -rf /var/lib/apt/lists/\*;
EOF

# Set the default command
CMD ab

### First build

We will build this image and tag it as `ab`.

$ docker build -t ab .

\[+\] Building 7.0s (6/6) FINISHED                                                               docker:desktop-linux
 => \[internal\] load .dockerignore                                                                              0.0s
 => => transferring context: 2B                                                                                0.0s
 => \[internal\] load build definition from Dockerfile                                                           0.0s
 => => transferring dockerfile: 730B                                                                           0.0s
 => \[internal\] load metadata for docker.io/library/ubuntu:20.04                                                0.4s
 => CACHED \[1/2\] FROM docker.io/library/ubuntu:20.04@sha256:33a5cc25d22c45900796a1aca487ad7a7cb09f09ea00b779e  0.0s
 => \[2/2\] RUN <<EOF (apt-get update;...)                                                                       6.5s
 => exporting to image                                                                                         0.0s
 => => exporting layers                                                                                        0.0s
 => => writing image sha256:99ca34fac6a38b79aefd859540f88e309ca759aad0d7ad066c4931356881e518                   0.0s
 => => naming to docker.io/library/ab 

### **Run with `CMD ab`**

Without any arguments, we get a usage block as expected.

$ docker run ab
ab: wrong number of arguments
Usage: ab \[options\] \[http\[s\]://\]hostname\[:port\]/path
Options are:
    -n requests     Number of requests to perform
    -c concurrency  Number of multiple requests to make at a time
    -t timelimit    Seconds to max. to spend on benchmarking
                    This implies -n 50000
    -s timeout      Seconds to max. wait for each response
                    Default is 30 seconds
<-- SNIP -->

However, if I run `ab` and include a URL to test, I initially get an error:

$ docker run --rm ab https://jayschmidt.us
docker: Error response from daemon: failed to create task for container: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: exec: "https://jayschmidt.us": stat https://jayschmidt.us: no such file or directory: unknown.

The issue here is that the string supplied on the command line — `https://jayschmidt.us` — is overriding the `CMD` instruction, and that is not a valid command, resulting in an error being thrown. So, we need to specify the command to run:

$ docker run --rm  ab ab https://jayschmidt.us/
This is ApacheBench, Version 2.3 <$Revision: 1843412 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking jayschmidt.us (be patient).....done


Server Software:        nginx
Server Hostname:        jayschmidt.us
Server Port:            443
SSL/TLS Protocol:       TLSv1.2,ECDHE-ECDSA-AES256-GCM-SHA384,256,256
Server Temp Key:        X25519 253 bits
TLS Server Name:        jayschmidt.us

Document Path:          /
Document Length:        12992 bytes

Concurrency Level:      1
Time taken for tests:   0.132 seconds
Complete requests:      1
Failed requests:        0
Total transferred:      13236 bytes
HTML transferred:       12992 bytes
Requests per second:    7.56 \[#/sec\] (mean)
Time per request:       132.270 \[ms\] (mean)
Time per request:       132.270 \[ms\] (mean, across all concurrent requests)
Transfer rate:          97.72 \[Kbytes/sec\] received

Connection Times (ms)
              min  mean\[+/-sd\] median   max
Connect:       90   90   0.0     90      90
Processing:    43   43   0.0     43      43
Waiting:       43   43   0.0     43      43
Total:        132  132   0.0    132     132

### Run with ENTRYPOINT

In this run, we remove the `CMD ab` instruction from the Dockerfile, replace it with `ENTRYPOINT ["ab"]`, and then rebuild the image.

This is similar to but different from the `CMD` command — when you use `ENTRYPOINT`, you cannot override the command unless you use the `–entrypoint` flag on the `docker run` command. Instead, any arguments passed to `docker run` are treated as arguments to the `ENTRYPOINT`.

$ docker run --rm  ab "https://jayschmidt.us/"
This is ApacheBench, Version 2.3 <$Revision: 1843412 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking jayschmidt.us (be patient).....done


Server Software:        nginx
Server Hostname:        jayschmidt.us
Server Port:            443
SSL/TLS Protocol:       TLSv1.2,ECDHE-ECDSA-AES256-GCM-SHA384,256,256
Server Temp Key:        X25519 253 bits
TLS Server Name:        jayschmidt.us

Document Path:          /
Document Length:        12992 bytes

Concurrency Level:      1
Time taken for tests:   0.122 seconds
Complete requests:      1
Failed requests:        0
Total transferred:      13236 bytes
HTML transferred:       12992 bytes
Requests per second:    8.22 \[#/sec\] (mean)
Time per request:       121.709 \[ms\] (mean)
Time per request:       121.709 \[ms\] (mean, across all concurrent requests)
Transfer rate:          106.20 \[Kbytes/sec\] received

Connection Times (ms)
              min  mean\[+/-sd\] median   max
Connect:       91   91   0.0     91      91
Processing:    31   31   0.0     31      31
Waiting:       31   31   0.0     31      31
Total:        122  122   0.0    122     122

### What about syntax?

In the example above, we use `ENTRYPOINT ["ab"]` syntax to wrap the command we want to run in square brackets and quotes. However, it is possible to specify `ENTRYPOINT ab` (without quotes or brackets). 

Let’s see what happens when we try that.

$ docker run --rm  ab "https://jayschmidt.us/"
ab: wrong number of arguments
Usage: ab \[options\] \[http\[s\]://\]hostname\[:port\]/path
Options are:
    -n requests     Number of requests to perform
    -c concurrency  Number of multiple requests to make at a time
    -t timelimit    Seconds to max. to spend on benchmarking
                    This implies -n 50000
    -s timeout      Seconds to max. wait for each response
                    Default is 30 seconds
<-- SNIP -->

Your first thought will likely be to re-run the `docker run` command as we did for `CMD ab` above, which is giving both the executable and the argument:

$ docker run --rm ab ab "https://jayschmidt.us/"
ab: wrong number of arguments
Usage: ab \[options\] \[http\[s\]://\]hostname\[:port\]/path
Options are:
    -n requests     Number of requests to perform
    -c concurrency  Number of multiple requests to make at a time
    -t timelimit    Seconds to max. to spend on benchmarking
                    This implies -n 50000
    -s timeout      Seconds to max. wait for each response
                    Default is 30 seconds
<-- SNIP -->

This is because `ENTRYPOINT` can only be overridden if you explicitly add the `–entrypoint` argument to the `docker run` command. The takeaway is to always use `ENTRYPOINT` when you want to force the use of a given executable in the container when it is run.

## Wrapping up: Key takeaways and best practices

The decision-making process involving the use of `RUN`, `CMD`, and `ENTRYPOINT`, along with the choice between shell and exec forms, showcases Docker’s intricate nature. Each command serves a distinct purpose in the Docker ecosystem, impacting how containers are built, operate, and interact with their environments. 

By selecting the right command and form for each specific scenario, developers can construct Docker images that are more reliable, secure, and optimized for efficiency. This level of understanding and application of Docker’s commands and their formats is crucial for fully harnessing Docker’s capabilities. Implementing these best practices ensures that applications deployed in Docker containers achieve maximum performance across various settings, enhancing development workflows and production deployments.

## Learn more

-   [CMD][10]
-   [ENTRYPOINT][11]
-   [RUN][12]
-   Stay updated on the latest Docker news! Subscribe to the [Docker Newsletter][13].
-   Get the latest release of [Docker Desktop][14].
-   New to Docker? [Get started][15].
-   Have questions? The [Docker community is here to help][16].

[developers][17], [Docker][18], [Docker Desktop][19], [dockerfile][20]

[

][21]

#### [Docker Desktop 4.32: Beta Releases of Compose File Viewer, Terminal Shell Integration, and Volume Backups to Cloud Providers][22]

By [Deanna Sparks][23]

[

][24]

#### [How an AI Assistant Can Help Configure Your Project’s Git Hooks][25]

By [Docker Labs][26] July 15, 2024

[

][27]

#### [How to Run Hugging Face Models Programmatically Using Ollama and Testcontainers][28]

By [Ignasi Lopez Luna][29] July 11, 2024

#### Posted

Jul 15, 2024

-   [][30]
-   [][31]
-   [][32]

#### Post Tags

[developers][33][Docker][34][Docker Desktop][35][dockerfile][36]

#### Categories

-   [Community][37]
-   [Company][38]
-   [Engineering][39]
-   [Products][40]

[1]: https://www.docker.com/author/jay-schmidt/ "Posts by Jay Schmidt"
[2]: https://docs.docker.com/engine/reference/builder/#run
[3]: https://docs.docker.com/engine/reference/builder/#cmd
[4]: https://docs.docker.com/engine/reference/builder/#entrypoint
[5]: https://docs.docker.com/reference/dockerfile/
[6]: https://docs.docker.com/engine/reference/builder/#cmd
[7]: https://docs.docker.com/engine/reference/builder/#entrypoint
[8]: https://docs.docker.com/engine/reference/builder/#run
[9]: https://en.wikibooks.org/wiki/Bourne_Shell_Scripting/Variable_Expansion
[10]: https://docs.docker.com/engine/reference/builder/#cmd
[11]: https://docs.docker.com/engine/reference/builder/#entrypoint
[12]: https://docs.docker.com/engine/reference/builder/#run
[13]: https://www.docker.com/newsletter-subscription/
[14]: https://www.docker.com/products/docker-desktop/
[15]: https://docs.docker.com/desktop/
[16]: https://www.docker.com/community/
[17]: https://www.docker.com/blog/tag/developers/
[18]: https://www.docker.com/blog/tag/docker/
[19]: https://www.docker.com/blog/tag/docker-desktop/
[20]: https://www.docker.com/blog/tag/dockerfile/
[21]: https://www.docker.com/blog/docker-desktop-4-32/
[22]: https://www.docker.com/blog/docker-desktop-4-32/
[23]: https://www.docker.com/author/deanna-sparks/ "Posts by Deanna Sparks"
[24]: https://www.docker.com/blog/how-an-ai-assistant-can-help-configure-your-projects-git-hooks/
[25]: https://www.docker.com/blog/how-an-ai-assistant-can-help-configure-your-projects-git-hooks/
[26]: https://www.docker.com/author/docker-labs/ "Posts by Docker Labs"
[27]: https://www.docker.com/blog/how-to-run-hugging-face-models-programmatically-using-ollama-and-testcontainers/
[28]: https://www.docker.com/blog/how-to-run-hugging-face-models-programmatically-using-ollama-and-testcontainers/
[29]: https://www.docker.com/author/ignasi-lopez-luna/ "Posts by Ignasi Lopez Luna"
[30]: https://twitter.com/intent/tweet?url=https%3A%2F%2Fwww.docker.com%2Fblog%2Fdocker-best-practices-choosing-between-run-cmd-and-entrypoint%2F
[31]: https://www.linkedin.com/sharing/share-offsite/?url=https%3A%2F%2Fwww.docker.com%2Fblog%2Fdocker-best-practices-choosing-between-run-cmd-and-entrypoint%2F
[32]: https://www.facebook.com/sharer/sharer.php?u=https%3A%2F%2Fwww.docker.com%2Fblog%2Fdocker-best-practices-choosing-between-run-cmd-and-entrypoint%2F
[33]: https://www.docker.com/blog/tag/developers/
[34]: https://www.docker.com/blog/tag/docker/
[35]: https://www.docker.com/blog/tag/docker-desktop/
[36]: https://www.docker.com/blog/tag/dockerfile/
[37]: https://www.docker.com/blog/category/community-content/
[38]: https://www.docker.com/blog/category/company/
[39]: https://www.docker.com/blog/category/engineering/
[40]: https://www.docker.com/blog/category/products/