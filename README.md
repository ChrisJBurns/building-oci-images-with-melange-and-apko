# Building OCI Images with `melange` & `apko`

Chainguard has two specific open-source products that allow you build OCI images using a declarative language base on YAML. I will be using [`apko`](https://edu.chainguard.dev/open-source/apko/overview/) to actually do the OCI image building and [`melange`](https://edu.chainguard.dev/open-source/melange/overview/) in order to create custom `apk` packages.

The reason this is a pretty nifty thing, is because currently in Dockerfiles, if someone wants to pull something into the image, they just do a `RUN curl url.to.script.that.downloads.stuff.sh`. Now, this is convienient, sure, but can be incredibly insecure if the engineer doesn't know what really lies in the script they are `curl`ing

In comes `apko`. `apko` builds OCI images declaratively that don't give you the option of `curl`ing a script or file from the internet. Here, you have to use already existing packages within the `apk` system, or you can use your own `apk`'s. This is then where `melange` comes in. `melange`, allows you to custom build your own `apk`'s for use by `akpo`  to build your OCI image.

The examples of this can be endless, but let's pick one. Let's say you have a custom binary that you use in your Organisation that does some stuff. Well, in `Dockerfile` land, there are many ways you can include this binary. It may be loading it locally from the folder that the `Dockerfile` lives in (which isn't great because if many images use that same binary, it may end up in many folders across your repositories, making it a nightmare to propagate updated versions of the binary), or you host it somewhere in an artifact repository (Nexus, Artifactory, Github Packages etc etc), you may even include it into a base image, many ways. Some easier to manage than others, but what you probably don't have is the assurance that you know who/what the binary was built by. Incomes `melanage`, when building an `apk`, `melange` will sign it (not mandatory, but strongly encouraged, and easy to do) so you can verify that it has been built by the correct person/process.

In this repository, we will be going over how to do this in a couple different languges, feel free to follow along.

## Prerequisites

Before you walk through some of the examples, make sure you:

- Download the [`melange` image](https://edu.chainguard.dev/open-source/melange/tutorials/getting-started-with-melange/#step-1--downloading-the-melange-image)
- Download the [`apko` image](https://edu.chainguard.dev/open-source/apko/getting-started-with-apko/#step-1--download-the-apko-image)

## Getting Started

In this repository, there is a [images](./images) directory that contains builds for OCI images using `apko` and custom application packages by `melange` in different languages and frameworks.

### Note

I won't be explaning each and every step in detail as I feel the Chainguard documentation does this very well for both `akpo` and `melange`. I will be explaing parts that I find are important only.
