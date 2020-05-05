FROM ubuntu:focal
MAINTAINER Robbert van der Helm <mail@robbertvanderhelm.nl>

# This Dockerfile is similar to the other image made for Ubuntu 18.04. It also
# does not include Boost, since installing `libboost-all-dev:i386` from the
# repositories is for some reason an incredible challenge in and of itself. See
# how this is used in the workflow files under
# https://github.com/robbert-vdh/yabridge/tree/master/.github/workflows

# dpkg might otherwise prompt us to select a timezoone while installing tzdata,
# even though we can't
ENV DEBIAN_FRONTEND=noninteractive

# Set up repositories needed for Wine and its development toolchain
RUN apt-get update && \
    apt-get install -y software-properties-common git wget && \
    dpkg --add-architecture i386 && \
    wget -O - https://dl.winehq.org/wine-builds/winehq.key | apt-key add - && \
    add-apt-repository 'deb https://dl.winehq.org/wine-builds/ubuntu/ bionic main' && \
    rm -rf /var/lib/apt/lists/*

# Install Wine and its development headers
# NOTE: As of writing this, the latest Wine release (5.7) has a regression with
#       winelib applications like yabridge, so we need to explicitely install
#       Wine Staging 5.6. Because of this we need to mmanually specify all of
#       these extra `wine-staging*` packages as apt will otherwise try to
#       install 5.7 anyway.
# NOTE: Also, we're installing it from Wine's repo meant for Ubuntu 18.04. This
#       sounds like a horrible idea but official repo intended for Ubuntu 20.04
#       only has versions 5.7 and up.
RUN apt-get update && \
    apt-get install -y --install-recommends winehq-staging=5.6~bionic wine-staging-dev=5.6~bionic wine-staging=5.6~bionic wine-staging-amd64=5.6~bionic wine-staging-i386=5.6~bionic && \
    rm -rf /var/lib/apt/lists/*

# Install the rest of the build dependencies
# Ubuntu 20.04 comes with GCC 9 so there's no need to install anything special there anymore
RUN apt-get update && \
    apt-get install -y build-essential gcc-multilib g++-multilib pkg-config python3-pip nodejs && \
    apt-get install -y libxcb1-dev libxcb1-dev:i386 && \
    pip3 install meson ninja && \
    rm -rf /var/lib/apt/lists/*
