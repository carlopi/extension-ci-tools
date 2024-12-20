FROM alpine:3

###
# Base image setup
###

# Setup the basic necessities
RUN apk update -y -qq
# RUN apk add -y -qq software-properties-common gcc-multilib g++-multilib
RUN apk add -y -qq ccache cmake git
RUN apk add -y -qq ninja-build make
RUN apk add -y -qq ninja-build make libssl3
RUN apk add -y -qq ninja-build make  wget
RUN apk add -y -qq ninja-build make wget zip
RUN apk add -y -qq ninja-build make  wget zip  gettext unzip
RUN apk add -y -qq ninja-build make wget zip gettext unzip  libffi-dev curl zlib openssh autoconf

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
ENV GEN=ninja

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
    apt-get install -y -qq bison flex \
  ;; \
esac

# Install Fortran
RUN case "$extra_toolchains" in \
  *\;fortran\;*) \
    apt-get install -y -qq gfortran gfortran-aarch64-linux-gnu \
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
    apk add -y -qq golang-go \
  ;; \
esac
ENV PATH="/usr/local/go/bin:${PATH}"

# Install Python3
RUN case "$extra_toolchains" in \
  *\;python3\;*) \
    apk add install -y -qq python3 \
  ;; \
esac
