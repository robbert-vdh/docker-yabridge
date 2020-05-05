FROM ubuntu:bionic
MAINTAINER Robbert van der Helm <mail@robbertvanderhelm.nl>

# This Dockerfile is made solely for the purpose of building yabridge on older
# versions of Ubuntu. This will install everything needed to compile yabridge
# except for Boost, since compiling the few Boost modules we need is very quick
# anyway and our needs might chance over time. See how this is used in the
# workflow files under
# https://github.com/robbert-vdh/yabridge/tree/master/.github/workflows

# Set up repositories needed for Wine and its development toolchain. Wine
# requires faudio which is missing from Ubuntu 18.04's repositories, so we have
# to obtain that library through a backports PPA.
RUN apt-get update && \
    apt-get install -y software-properties-common git wget && \
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
