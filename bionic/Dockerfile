FROM ubuntu:bionic
MAINTAINER Robbert van der Helm <mail@robbertvanderhelm.nl>

# This Dockerfile is made solely for the purpose of building yabridge on older
# versions of Ubuntu. This will install everything needed to compile yabridge
# under Ubuntu 18.04. See how this is used in the workflow files under
# https://github.com/robbert-vdh/yabridge/tree/master/.github/workflows

# dpkg might otherwise prompt us to select a timezoone while installing tzdata,
# even though we can't
ENV DEBIAN_FRONTEND=noninteractive

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
RUN apt-get update && \
    apt-get install -y --install-recommends winehq-devel wine-devel-dev && \
    rm -rf /var/lib/apt/lists/*

# Install the rest of the build dependencies
RUN apt-get update && \
    apt-get install -y build-essential cmake pkg-config python3-pip && \
    apt-get install -y libdbus-1-dev libxcb1-dev libxcb1-dev:i386 && \
    pip3 install meson ninja && \
    rm -rf /var/lib/apt/lists/*

# Install and default to GCC 10 now that every major supported distro other than
# Ubuntu 18.04 ships with GCC 10 so we can make use of some newer language
# features. We will be statically linking to libstdc++ since Ubuntu 18.04 ships
# an older version by default, but that won't cause any issues since we're only
# exposing C interfaces.
RUN add-apt-repository -y ppa:ubuntu-toolchain-r/test && \
    apt-get update && \
    apt-get install -y gcc-10 gcc-10-multilib g++-10 g++-10-multilib && \
    update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-10 110 --slave /usr/bin/g++ g++ /usr/bin/g++-10 && \
    rm -rf /var/lib/apt/lists/*
