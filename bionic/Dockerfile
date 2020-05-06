FROM ubuntu:bionic
MAINTAINER Robbert van der Helm <mail@robbertvanderhelm.nl>

# This Dockerfile is made solely for the purpose of building yabridge on older
# versions of Ubuntu. This will install everything needed to compile yabridge
# under Ubuntu 18.04. See how this is used in the workflow files under
# https://github.com/robbert-vdh/yabridge/tree/master/.github/workflows

# dpkg might otherwise prompt us to select a timezoone while installing tzdata,
# even though we can't
ENV DEBIAN_FRONTEND=noninteractive

ENV BOOST_URL="https://dl.bintray.com/boostorg/release/1.72.0/source/boost_1_72_0.tar.gz"
ENV BOOST_CHECKSUM="c66e88d5786f2ca4dbebb14e06b566fb642a1a6947ad8cc9091f9f445134143f  boost_1_72_0.tar.gz"
ENV BOOST_VERSION="1_72_0"

# Add a PPA for newer versions of git since GitHub Actions has a hard dependency
# on git 2.18 while Ubuntu 18.04 ships with git 2.17
RUN apt-get update && \
    apt-get install -y software-properties-common && \
    add-apt-repository ppa:git-core/ppa && \
    rm -rf /var/lib/apt/lists/*

# Set up repositories needed for Wine and its development toolchain. Wine
# requires faudio which is missing from Ubuntu 18.04's repositories, so we have
# to obtain that library through a backports PPA.
RUN apt-get update && \
    apt-get install -y git wget && \
    dpkg --add-architecture i386 && \
    wget -O - https://dl.winehq.org/wine-builds/winehq.key | apt-key add - && \
    add-apt-repository 'deb https://dl.winehq.org/wine-builds/ubuntu/ bionic main' && \
    add-apt-repository -y ppa:cybermax-dexter/sdl2-backport && \
    rm -rf /var/lib/apt/lists/*

# Install Wine and its development headers
# NOTE: As of writing this, the latest Wine release (5.7) has a regression with
#       winelib applications like yabridge, so we need to explicitely install
#       Wine Staging 5.6. Because of this we need to mmanually specify all of
#       these extra `wine-staging*` packages as apt will otherwise try to
#       install 5.7 anyway.
RUN apt-get update && \
    apt-get install -y --install-recommends winehq-staging=5.6~bionic wine-staging-dev=5.6~bionic wine-staging=5.6~bionic wine-staging-amd64=5.6~bionic wine-staging-i386=5.6~bionic && \
    rm -rf /var/lib/apt/lists/*

# Install the rest of the build dependencies
RUN apt-get update && \
    apt-get install -y build-essential gcc-8 gcc-8-multilib g++-8 g++-8-multilib pkg-config python3-pip nodejs && \
    apt-get install -y libxcb1-dev libxcb1-dev:i386 && \
    pip3 install meson ninja && \
    rm -rf /var/lib/apt/lists/*

# Finally default to GCC 8, since the default GCC 7's C++17 implementation is
# incomplete
RUN update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-8 800 --slave /usr/bin/g++ g++ /usr/bin/g++-8

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
