FROM alpine:3

###
# Base image setup
###

# Setup the basic necessities
RUN apk update --y -qq
# RUN apk add -y -qq software-properties-common gcc-multilib g++-multilib
RUN apk add  -qq ccache cmake git ninja ninja-build
RUN apk add  -qq make gcc
RUN apk add  -qq make libssl3
RUN apk add  -qq make  wget bash
RUN apk add  -qq make wget zip
RUN apk add -qq make  wget zip  gettext unzip build-base cmake zip unzip curl git
RUN apk add -qq make wget zip gettext unzip  libffi-dev curl zlib openssh autoconf

# Setup VCPKG n a mounted volume TODO: figure out how to cache this
ARG vcpkg_url
ARG vcpkg_commit
RUN mkdir /vcpkg && \
    cd /vcpkg && \
    git init && \
    git remote add origin $vcpkg_url && \
    git fetch origin $vcpkg_commit && \
    git checkout $vcpkg_commit && \
    ./bootstrap-vcpkg.sh
ENV VCPKG_ROOT=/vcpkg
ENV VCPKG_TOOLCHAIN_PATH=/vcpkg/scripts/buildsystems/vcpkg.cmake

# Common environment variables

# Specify where we expect the extension to be mounted and use that as working dir
VOLUME /duckdb_build_dir
WORKDIR /duckdb_build_dir

# Mount for ccache to allow restoring ccache in GH actions
VOLUME /ccache_dir
ENV CCACHE_DIR=/ccache_dir
ENV CCACHE_COMPRESS=TRUE
ENV CCACHE_COMPRESSLEVEL=6
ENV CCACHE_MAXSIZE=400M

###
# Conditionally configure some extra dependencies
###
# a `;` separated list of extra toolchains to install (passed in like this to makes things easier through GitHub Actions)
# Note that it should start and end with a `;` e.g. `;rust;parser_tools;`
ARG extra_toolchains

# NOTE: the weird case conditionals are because of bash limitations in the ubuntu image used (see https://stackoverflow.com/questions/229551/how-to-check-if-a-string-contains-a-substring-in-bash)

# Install Parser tools
RUN case "$extra_toolchains" in \
  *\;parser_tools\;*) \
    apk add -y -qq bison flex \
  ;; \
esac

# Install Fortran
RUN case "$extra_toolchains" in \
  *\;fortran\;*) \
    apk add -y -qq gfortran \
  ;; \
esac

# Configure Rust
RUN case "$extra_toolchains" in \
  *\;rust\;*) \
    curl https://sh.rustup.rs -sSf | bash -s -- -y \
  ;; \
esac
ENV PATH="/root/.cargo/bin:${PATH}"

# Configure go
RUN case "$extra_toolchains" in \
  *\;go\;*) \
    apk add -y -qq go \
  ;; \
esac
ENV PATH="/usr/local/go/bin:${PATH}"

# Install Python3
RUN case "$extra_toolchains" in \
  *\;python3\;*) \
    apk add -y -qq python3 \
  ;; \
esac
