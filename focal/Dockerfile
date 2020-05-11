FROM ubuntu:focal
MAINTAINER Robbert van der Helm <mail@robbertvanderhelm.nl>

# This Dockerfile is similar to the other image made for Ubuntu 18.04. See how
# this is used in the workflow files under
# https://github.com/robbert-vdh/yabridge/tree/master/.github/workflows

# dpkg might otherwise prompt us to select a timezoone while installing tzdata,
# even though we can't
ENV DEBIAN_FRONTEND=noninteractive

ENV BOOST_URL="https://dl.bintray.com/boostorg/release/1.72.0/source/boost_1_72_0.tar.gz"
ENV BOOST_CHECKSUM="c66e88d5786f2ca4dbebb14e06b566fb642a1a6947ad8cc9091f9f445134143f  boost_1_72_0.tar.gz"
ENV BOOST_VERSION="1_72_0"

# Set up repositories needed for Wine and its development toolchain
RUN apt-get update && \
    apt-get install -y software-properties-common git wget && \
    dpkg --add-architecture i386 && \
    wget -O - https://dl.winehq.org/wine-builds/winehq.key | apt-key add - && \
    add-apt-repository 'deb https://dl.winehq.org/wine-builds/ubuntu/ focal main' && \
    rm -rf /var/lib/apt/lists/*

# Install Wine and its development headers
RUN apt-get update && \
    apt-get install -y --install-recommends winehq-staging wine-staging-dev && \
    rm -rf /var/lib/apt/lists/*

# Install the rest of the build dependencies
# Ubuntu 20.04 comes with GCC 9 so there's no need to install anything special there anymore
RUN apt-get update && \
    apt-get install -y build-essential gcc-multilib g++-multilib pkg-config python3-pip nodejs && \
    apt-get install -y libxcb1-dev libxcb1-dev:i386 && \
    pip3 install meson ninja && \
    rm -rf /var/lib/apt/lists/*

# Compile and install the Boost libraries we need. This should not take longer
# than a few seconds, but the disk IO required slows the jobs on GitHub Actions
# down immensely.
RUN mkdir /tmp/boost && \
    cd /tmp/boost && \
    wget --quiet --max-redirect 3 "$BOOST_URL" && \
    echo "$BOOST_CHECKSUM" | sha256sum -c && \
    tar -xf "boost_${BOOST_VERSION}.tar.gz" && \
    rm "boost_${BOOST_VERSION}.tar.gz" && \
    \
    cd boost_${BOOST_VERSION} && \
    ./bootstrap.sh --with-toolset=gcc --with-icu --with-python= && \
    \
    # 32-bit build
    ./b2 \
    variant=release \
    debug-symbols=off \
    threading=multi \
    runtime-link=shared \
    link=shared,static \
    toolset=gcc \
    address-model=32 \
    cflags="${CPPFLAGS} ${CFLAGS} -m32 -fPIC -O3" \
    cxxflags="${CPPFLAGS} ${CXXFLAGS} -m32 -std=c++14 -fPIC -O3" \
    linkflags="${LDFLAGS} -m32" \
    --with-filesystem \
    --libdir=/usr/local/lib/i386-linux-gnu \
    -j $(nproc) \
    \
    install && \
    \
    # 64-bit build
    ./b2 \
    variant=release \
    debug-symbols=off \
    threading=multi \
    runtime-link=shared \
    link=shared,static \
    toolset=gcc \
    address-model=64 \
    cflags="${CPPFLAGS} ${CFLAGS} -fPIC -O3" \
    cxxflags="${CPPFLAGS} ${CXXFLAGS} -std=c++14 -fPIC -O3" \
    --with-filesystem \
    -j $(nproc) \
    \
    install && \
    \
    cd / && \
    rm -rf /tmp/boost
