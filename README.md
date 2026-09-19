<!--
SPDX-FileCopyrightText: NONE
SPDX-License-Identifier: CC0-1.0
-->

# xcursor-viewer

![Release](../../actions/workflows/release.yml/badge.svg)

## Usage

### Download

The easiest method. Simply download and execute the compiled binary from [Releases](../../releases).

### Compile (without Docker)

1. Install dependencies (using `apt` for example)

    ```bash
    apt install -y ninja-build cmake g++ qt5-default
    # if qt5-default is not available, use the following command instead:
    apt install -y ninja-build cmake g++ qtbase5-dev
    ```

1. Clone repository and compile

    ```bash
    git clone https://github.com/drizt/xcursor-viewer.git
    cd xcursor-viewer
    cmake -G Ninja . -B build
    cmake --build build
    ```

1. Execute the compiled binary

    ```bash
    ./build/xcursor-viewer
    ```

### Compile (with Docker)

Base images: `busybox:latest`, `debian:bookworm`

1. Create the Dockerfile

    ```Dockerfile
    # syntax=docker/dockerfile:1.19
    FROM debian:bookworm AS build
    ARG tag=heads/master
    RUN <<EOF
      apt update
      apt install -y ninja-build cmake g++ qtbase5-dev
      apt clean
    EOF
    ADD --link --unpack=true https://github.com/drizt/xcursor-viewer/archive/refs/${tag}.tar.gz /xcursor-viewer
    RUN <<EOF
      cmake -G Ninja /xcursor-viewer/xcursor-viewer-${tag#*/} -B /out && cmake --build /out
    EOF

    FROM scratch
    COPY --from=build /out/xcursor-viewer /

    ENTRYPOINT ["/xcursor-viewer"]
    ```

1. Compile and export binary

    ```bash
    docker buildx build --output=. .
    ```

    By default, the exported binary will be compiled from **master** branch. To specify another ref, provide a `SOURCE_REF` build argument, where `SOURCE_REF` may be `tags/<tag-name>` or `heads/<branch-name>`. For example:

    ```bash
    # compile from v0.0.1 tag and export binary
    docker buildx build --build-arg SOURCE_REF=tags/v0.0.1 --output=. .
    # compile from stable branch and export binary
    docker buildx build --build-arg SOURCE_REF=heads/stable --output=. .
    ```

1. Execute the compiled binary

    ```bash
    ./xcursor-viewer
    ```
