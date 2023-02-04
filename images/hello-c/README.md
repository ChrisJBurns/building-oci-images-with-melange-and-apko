# C Program in an OCI Image

Here we will go through the building of an OCI image with a simple Hello World program written in C.

Because this program is going to be customly made for this example, we will be using `melange` in order to package it up into an `apk` that we can then use by `apko` to bake it into our OCI image.

## Prerequisites

You should have the `melange` and `apko` images downloaded already as stated in the [main README.md](../../README.md#prerequisites)

## Getting Started

Our program is simple, there is a `hello.c` file that contains our C code that writes `Hello world!` out to the console. We then have a `Makefile` that builds and installs the program into `/usr/bin`.

### Building the `apk`

The `melange.yaml` contains the information that is required by `melange` to build the `apk`. We specify that the `apk` should be for all target architectures. We install some packages from the main `apk` repository. Then as part of the pipeline step we run the `Makefile` in order to build the program and install it into the `apk`.

First generate a keypair that will be used to sign our custom `apk`:

```shell
docker run --rm -v "${PWD}":/work cgr.dev/chainguard/melange keygen
```

There should now be `melange.rsa` and `melange.rsa.pub` files in the current directory.

To build the `apk`, run

```shell
docker run --privileged --rm -v "${PWD}":/work \
  cgr.dev/chainguard/melange build melange.yaml \
  --signing-key melange.rsa
```

You should see a bunch of logs whizz past where it is building and signing our custom C program `apk` for each of the architectures.

An example:

```logs
...
2023/02/04 15:23:36 melange (hello/ppc64le): generating apk index from packages in /work/packages/ppc64le
2023/02/04 15:23:36 melange: processing package /work/packages/ppc64le/hello-0-r0.apk
2023/02/04 15:23:36 melange: generating index at /work/packages/ppc64le/APKINDEX.tar.gz
2023/02/04 15:23:36 melange: signing apk index at /work/packages/ppc64le/APKINDEX.tar.gz
2023/02/04 15:23:36 melange: signing index /work/packages/ppc64le/APKINDEX.tar.gz with key melange.rsa
2023/02/04 15:23:36 melange: appending signature to index /work/packages/ppc64le/APKINDEX.tar.gz
2023/02/04 15:23:36 melange: writing signed index to /work/packages/ppc64le/APKINDEX.tar.gz
2023/02/04 15:23:36 melange: signed index /work/packages/ppc64le/APKINDEX.tar.gz with key melange.rsa
...
```

You will endup with a `packages/` folder with a bunch of subfolders that contains our program `apk` for each of the architectures. Cool!

### Building the OCI Image

Now the custom `apk` has been built, we use `akpo` to build the OCI image that will contain this `apk` (which has our C program). There is a `hello.yaml` file that will be used by `akpo` in order to build our image. We tell it that we want to use a local repository that is the `packages/` folder that contains our `apk`'s, we then tell it to install the `hello` package that was built by `melange`.
Lastly, we tell the Image to have an entrypoint (like `Dockerfile`'s) that runs our  `hello` C program.

To build the image, run:

```shell
docker run --rm -v ${PWD}:/work cgr.dev/chainguard/apko  build --debug hello.yaml hello:test hello.tar -k melange.rsa.pub
```

Here we say, build the `hello.yaml` file, create an image called `hello:test` and then output it as a `hello.tar`, but to use the `melange.rsa.pub` key so that it can verify the `hello` `apk` package using the same key it was built by.

You should see logs whizz past again where it describes what it's doing, but at the end you will have lines that look something like this:

```logs
...
Feb  4 16:35:47.041 [WARNING] [arch:x86_64] multiple SBOM formats requested, uploading SBOM with media type: spdx+json
Feb  4 16:35:47.153 [INFO] [arch:x86_64] output OCI image file to hello.tar
```

Now if you check in your current directory and you will have a couple of new files.

One of which is `hello.tar` which is a tarball of our OCI image. To load this into your local registry run:
`docker load < hello.tar`

You should now see it with `docker images`.

Lastly, to run the image, run:

```shell
docker run --rm hello:test
Hello world!
```

There we go! Our very own C program OCI image built by `akpo`, with the `apk` package being built by `melange.`

## Further Note

If you have a detailed eye, you will have noticed that there where two other files created after tha `apko` build step:

- `sbom-*.cdx` - this is the [CycloneDX](https://cyclonedx.org/) SBOM of our OCI Image
- `sbom-*.spdx.json` this is the [SPDX](https://spdx.dev/) SBOM of our OCI Image

These files contain information about all of the dependencies and packages that make up the OCI image (including our `hello` `apk` - go have a look for it). This is an incredible powerful feature because this allows you to further look deeper into your images to see what nasties exist if another log4shell was to ever happen.

## Conclusion

Hope you enjoyed!
