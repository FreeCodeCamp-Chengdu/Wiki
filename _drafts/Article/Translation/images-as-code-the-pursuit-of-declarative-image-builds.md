---
title: "Images as Code: The pursuit of declarative image builds"
date: 2024-01-22T00:00:00.000Z
authorURL: ""
originalURL: https://www.chainguard.dev/unchained/images-as-code-the-pursuit-of-declarative-image-builds
translator: ""
reviewer: ""
---

Product

<!-- more -->

# Images as Code: The pursuit of declarative image builds

Matt Moore, CTO

copied

### In the beginning...

When I introduced the [original distroless][1] images to the world, I said "[I am in pursuit of a better way of building containers.][2]" Much of the need for this ties back to one thing: `RUN`. This powerful and sticky Dockerfile directive in many ways enabled the ascension of Docker to the ubiquity it has today. However, this directive is the root of so many problems as well. The imperative nature of `RUN` requires activating the container and running things inside of it, which makes things like multi-tenancy (e.g. hosted build services) and multi-architecture (e.g. amd64 and arm64) builds hard. The fact that it lets users execute arbitrary commands also makes any claims of reproducibility pretty trivially refutable.

‍

My journey in this space started in 2014 with [Blaze at Google][3] (now Open Source as Bazel), where I wrote the original [`rules_docker`][4] (which has now been deprecated and replaced by `rules_oci`) with the aim to provide every directive of Dockerfile except `RUN`. These were without a doubt the first generally reproducible docker build rules (likely by several years), and by the time I ultimately left Google there were hundreds of teams building with these internally.

‍

Blaze `BUILD` files were a natural early vehicle for this work because of their declarative representation of what developers intend to build. In declarative models, users express their desired state vs. imperative models where they express the procedure to get there (a la `RUN`); it is a declaration of intention. While Bazel (open source Blaze) `BUILD` files did not exactly become a runaway sensation, it is not alone in its use of declarative expression, and there are two great examples which have gained massive mindshare: Kubernetes and Terraform (both have cameos below, but no spoilers). Terraform has come to define the category known as “[Infrastructure as Code][5]” and uses a declarative expression of the infrastructure a user wants deployed. It also inspired the title of my pursuit of declarative image builds: “Images as Code”.

‍

### Solving the last mile

My journey with Bazel ultimately led me to exploring rules like [`go_image`][6] that acted as a drop-in-replacement for [`go_binary`][7]. The aim of this was to make it trivial for users that had already taken on the non-trivial overhead of adopting Bazel to turn their applications into containers. Part of the motivation for the original distroless images was the need for suitable runtime environments for these rules without the kitchen sink of your typical base images, similar to how Googlers used the “Google Runtime Environment” (aka GRTE) with [`rules_docker`][8]. These `foo_image` rules made me realize that when users are expressing their intent declaratively (e.g. put my app in an image), it enables the tooling to achieve that intent in ways that can be much more sophisticated than if a user were defining the procedure imperatively (I touched on this during [my talk at the first BazelCon][9]).

‍

For example, something like this (this is in Python, but many languages have a _fairly_ similar pattern):

‍

\-- CODE language-bash -- FROM cgr.dev/chainguard/python:latest-dev ADD requirements.txt . RUN pip install -r requirements.txt ADD . .

‍

I call this class of images “**last-mile images**” (and their build tools “last-mile builders”), and it inspired and informed a significant amount of downstream innovation, including [“Faster than Light Builds” aka FTL][10] (with Dan Lorenc and Aaron Prindle), the [ko][11] project (with Jason Hall and Jon Johnson), the [jib][12] project (with Appu Goundan, Q Chen, and Patrick Flynn), and [CNCF buildpacks][13] (with Jacques Chester and Stephen Levine). In particular, it has been humbling to see folks wishing for and attempting to replicate [ko][14] for many other language ecosystems over the years. In my (incredibly biased) opinion, what we managed to achieve with [ko][15] is so close to optimal that I consider the “last mile” for Go images a largely solved problem and I’m skeptical we will do _much_ better. **Huge** kudos to Jason Hall and Jon Johnson for all of their innovative work on this project over the years, continually taking it to new heights.

‍

### Creating distroless

The other major class of images is “base images,” but few tools aim to specialize as “base image builders.” Instead they want to own the whole problem! Where “last-mile images” handle the application and its language-level dependencies, “base images” generally consist of the system-level dependencies comprising the runtime environment. Most typical base images start from a distro image (e.g. alpine, debian, ubuntu, or my fave: Wolfi) and use `RUN` (which needs a shell) to install things via its package manager (e.g. `apk`, `apt`, `yum`). Some unfortunate base images need things unavailable via the distro package manager (or at a different version) and fallback to `curl` piped to `tar` (or worse `bash`), which mean that standard Software Composition Analysis (aka SCA) tooling may not find them (e.g. for producing SBOMs or performing vulnerability scans).

‍

In the distroless philosophy, the only packages that should exist in the final image are those needed by the application itself at runtime, not the logic to bootstrap that environment. My “go to” analogy here is that it is like the Java Runtime Environment (JRE) vs the Java Development Kit (JDK). The vast majority of applications simply do not need a shell (needed for `RUN`), or a package manager (common reason for `RUN`) at runtime. This makes the traditional model of producing base images largely untenable for producing distroless images because it fundamentally requires things in the image that should not be there.

‍

![Dan Lorenc and Matt Moore photoshopped into a painting with text reading "distroless is a philosophy".](https://assets-global.website-files.com/6228fdbc6c971401d02a9c42/65ae80e31ea2318f0a49e8de_distroless%20is%20a%20philosophy.png)

Dan Lorenc and I spreading the good word of distroless. Image credit: Dan "Pop" Papandrea

‍

When we built the original distroless images we leveraged a fluke of history: Bazel’s [`rules_docker`][16] supports the equivalent of “`ADD foo.deb`” by expanding the filesystem portion of the `deb` and ignoring install hooks completely. In other words it could “install” a `deb` without the package manager or shell inside of the final container (assuming of course that those hooks weren’t important!). Around this we fashioned additional Bazel rules to download and feed in a list of `deb` files and we were off to the races! By not going through the package manager we hit all kinds of fun issues over the years; as an example, we didn’t have a package database for SCA tooling. We had to invent one that worked with how we were building up the images, and every scanner on the planet eventually reverse engineered this format as the images gained traction. To call this approach a solution would be generous, but it proved the concept and slowly gained traction. Huge kudos to Appu Goundan and others for keeping distroless going over the years.

‍

### Distroless done right

In the several years it took for the distroless philosophy to become more mainstream, Ariadne Conill had started to experiment with making `apk-tools` itself do what we had abused Bazel to do through her [Witchery][17] project. Revisiting distroless was also one of the projects we discussed extensively in the early days of Chainguard, and we saw huge potential in Ariadne’s work for what I describe as “distroless done right.” We recruited her to our merry band, and [`apko`][18] was born.

‍

Like [ko][19], [`apko`][20] was a tool born with a singular focus: Solve the problem of declaratively bootstrapping an image from scratch given a list of APKs you want in it. Like using ko, using `apko` for the first few times felt magical. We bootstrapped `apko` using Alpine to prove the concept, but we knew we would ultimately have to build what would become [Wolfi][21] to achieve the GNU compatibility we would need to ultimately replace distroless, and to ensure a complete enough library of packages to keep people from reaching for `curl` and `tar`. Having now built hundreds of images with `apko`, in my (incredibly biased) opinion we have solved another large segment of the declarative image creation space. Huge kudos to Ariadne Conill for her vision and leadership of this next generation of distroless tooling (and Wolfi!).

‍

### Houston, we have a problem

However, with the growing number of specialized tools, we created a new orchestration problem: these specialized tools each solve one part of the problem, but how do we compose them? We first encountered this with ko thinking about how we compose `ko build` with other container runtimes besides Kubernetes (e.g. [ECS][22], [Cloud Run][23], [Lambda][24]). However, this same desire for composability applies to when we think about how we verify supply chain properties of base images, sign resulting images, and attest to produced SBOMs.

‍

![A chart illustrating a hypothetical composition of ko with other tools.](https://assets-global.website-files.com/6228fdbc6c971401d02a9c42/65ae8c0b89f277307ea4592b_BLOG_%20Images%20as%20Code.jpeg)

‍

One morning I had the “crazy idea” to create a [terraform provider for ko][25] so that we could leverage the powerful composability of Terraform and tap into its vast ecosystem of runtime providers. I don’t think I have ever nerd sniped Jason Hall as effectively prior or since, because he, with help from Nathan Smith, had a proof-of-concept working by lunch! Over the following months we used this provider (at times in anger) and learned and refined things a lot. While we immediately had the idea to do the same for `apko`, it wasn’t ready because it still relied on the host’s `apk-tools`, which meant it had to run in an Alpine or Wolfi-based environment. However, in parallel to our learning with [the ko provider][26], Avi Deitcher was creating a [pure go `apk-tools` library][27], which enabled us to run `apko` on any host system (I now run it natively on my Apple M2). As soon as this implementation landed in `apko`, I immediately put together an initial [terraform provider for `apko`][28] (there was no way I was going to let Jason steal all the fun this time!), so that it too could compose with tools the way we were with the ko provider. At this point, the provider for `apko` was still very much experimental (only supporting `apko_build`), and we were still learning a lot.

‍

### Terraforming Chainguard Images

As I was taking stock of the complexity that the Chainguard Images build system had organically become (as we rapidly scaled our image count), I realized there was an enormous amount of imperative orchestration happening around our nice declarative `apko`, and so I decided to write out my ideal terraform to express the composition of tools currently spread across Github Actions, sooo much bash, and 3-4 dialects of yaml. This sketch included our experimental [provider for `apko`][29], but also unwritten [cosign][30] and [OCI][31] providers (the core of this sketch would become our [publisher module][32]).

‍

![A chart illustrating an early sketch of Chainguard declarative image builds.I](https://assets-global.website-files.com/6228fdbc6c971401d02a9c42/65ae86912774f640c812f859_BLOG_%20Images%20as%20Code%20(1).jpg)

This sketch was compelling enough a solution that I decided to immediately put together a preliminary provider for [cosign][33] to sign images (`cosign_sign`), attest image claims (`cosign_attest`, e.g. SBOMs, SLSA), and verify policies over images (`cosign_verify`). Trying to compose these providers was also very educational and things matured quickly after that. In parallel, Jason Hall and Jon Johnson had started to build an [OCI provider][34] to support tagging / testing images, and suddenly we had all of the pieces we needed to make it real. Josh Dolitsky then injected some of his trademark YOLO, and our early canary images showed the viability of the approach. We pulled the trigger, and now 100% of Chainguard’s Images are built completely declaratively!

‍

### Happily ever after

While we are now using these pieces to orchestrate our Chainguard Image builds, what we are doing barely scratches the surface of what is already possible. I grew up with legos, and the best part is being able to take them apart and build new things (and part of the appeal of Terraform was an enormous ecosystem for us to compose with). With these pieces, you can deploy a “last mile image” built with ko, overlaid on a “base image” assembled from a custom set of APKs tailored to your application’s needs. You can sign / attest them all with cosign, and then deploy them to a runtime environment of your choosing:

![A chart illustrating the assembly of an entire container from APKs and Go, then deploying it to ECS.](https://assets-global.website-files.com/6228fdbc6c971401d02a9c42/65ae8c83fc153371b8f7ff31_BLOG_%20Images%20as%20Code%20(2).jpeg)

We have come a long way since the early days of “_Images as Code_”, and I want to thank everyone who has helped (far too many to name) to advance my endless pursuit of “[_better way\[s\] of building containers_][35].” If you are interested in learning more, or collaborating on taking things to the next level, then please reach out! If you are interested in learning more about Chainguard Images as part of your container security strategy, [contact our team][36] today.

[Share on Twitter][37] [Share on LinkedIn][38] [Share on Email][39]

##### Related articles

![](https://assets-global.website-files.com/6228fdbc6c971401d02a9c42/664bb8f8a1a7027189abcdac_Card%20Images.png)

Product

How to transition to secure container images with new migration guides

May 22, 2024

[][40]

![](https://assets-global.website-files.com/6228fdbc6c971401d02a9c42/66436bb3fd98682163f2733b_Card%20Images.png)

Product

Achieve PCI DSS v4.0 compliance with Chainguard Images

May 20, 2024

[][41]

![](https://assets-global.website-files.com/6228fdbc6c971401d02a9c42/663504e768d5ea1bef3e329a_Chainguard_Images_Card.png)

Product

Chainguard Images April 2024: Secure, reliable, feature-rich

May 9, 2024

[][42]

[1]: https://github.com/GoogleContainerTools/distroless
[2]: https://www.youtube.com/watch?v=lviLZFciDv4
[3]: https://www.youtube.com/watch?v=RS1aiQqgUTA
[4]: https://github.com/bazelbuild/rules_docker
[5]: https://en.wikipedia.org/wiki/Infrastructure_as_code
[6]: https://github.com/bazelbuild/rules_docker/tree/master#go_image
[7]: https://github.com/bazelbuild/rules_go/blob/master/docs/go/core/rules.md#go_binary
[8]: https://github.com/bazelbuild/rules_docker
[9]: https://www.youtube.com/watch?v=RS1aiQqgUTA
[10]: https://docs.google.com/presentation/d/e/2PACX-1vQg-zft_wBGRa0vsiFFDUXyT_PxiZ3L-flyH6g2BG0UtsTWQOeAJ-pkNGOZD5Q2koO9cFIeqkNFvmcu/pub?start=false&loop=false&delayms=3000
[11]: https://github.com/ko-build/ko
[12]: https://github.com/GoogleContainerTools/jib
[13]: https://buildpacks.io/
[14]: https://github.com/ko-build/ko
[15]: https://github.com/ko-build/ko
[16]: https://github.com/bazelbuild/rules_docker
[17]: https://ariadne.space/2021/09/09/introducing-witchery-tools-for-building-distroless-images-with-alpine/
[18]: https://github.com/chainguard-dev/apko
[19]: https://github.com/ko-build/ko
[20]: https://github.com/chainguard-dev/apko
[21]: https://github.com/wolfi-dev
[22]: https://github.com/ko-build/terraform-provider-ko/tree/main/provider-examples/ecs
[23]: https://github.com/ko-build/terraform-provider-ko/tree/main/provider-examples/cloudrun
[24]: https://github.com/ko-build/terraform-provider-ko/tree/main/provider-examples/lambda
[25]: https://github.com/ko-build/terraform-provider-ko
[26]: https://github.com/ko-build/terraform-provider-ko
[27]: https://github.com/chainguard-dev/go-apk
[28]: https://github.com/chainguard-dev/terraform-provider-apko
[29]: https://github.com/chainguard-dev/terraform-provider-apko
[30]: https://github.com/chainguard-dev/terraform-provider-cosign
[31]: https://github.com/chainguard-dev/terraform-provider-oci
[32]: https://github.com/chainguard-dev/terraform-publisher-apko/blob/main/main.tf
[33]: https://github.com/sigstore/cosign
[34]: https://github.com/chainguard-dev/terraform-provider-oci
[35]: https://www.youtube.com/watch?v=lviLZFciDv4
[36]: https://www.chainguard.dev/contact
[37]: https://twitter.com/intent/tweet?text=Images as Code: The pursuit of declarative image builds&url=https://www.chainguard.dev/unchained/images-as-code-the-pursuit-of-declarative-image-builds "Share on Twitter"
[38]: https://www.linkedin.com/shareArticle?mini=true&url=chainguard.dev/images-as-code-the-pursuit-of-declarative-image-builds "Share on LinkedIn"
[39]: mailto:?&subject=&cc=&bcc=&body=[your URL path/{slug variable}] "Share via Email"
[40]: /unchained/how-to-transition-to-secure-container-images-with-new-migration-guides
[41]: /unchained/achieve-pci-dss-v4-0-compliance-with-chainguard-images
[42]: /unchained/chainguard-images-april-2024-secure-reliable-feature-rich