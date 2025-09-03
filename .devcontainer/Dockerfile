FROM pytorch/pytorch:2.8.0-cuda12.9-cudnn9-devel AS common_base
ARG BASE_IMAGE_NAME
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && apt-get install -y --no-install-recommends \
    htop dstat aria2 wget zip libibverbs-dev curl vim git git-lfs zstd pv jq ccache

ARG MAX_JOBS=8
# follow the instructions at https://github.com/NVIDIA/gdrcopy?tab=readme-ov-file#deb-package
RUN mkdir -p /src/ && cd /src/ &&\
    wget https://github.com/NVIDIA/gdrcopy/archive/refs/tags/v2.4.4.tar.gz &&\
    tar -xzf v2.4.4.tar.gz && ls -lha . &&\
    cd gdrcopy-2.4.4/ &&\
    apt-get update && \
    apt-get install -y build-essential devscripts debhelper fakeroot pkg-config dkms &&\
    cd packages &&\
    CUDA=/usr/local/cuda ./build-deb-packages.sh

RUN cd /src/gdrcopy-2.4.4/packages &&\
    dpkg -i *.deb
ADD https://github.com/openucx/ucx/releases/download/v1.19.0/ucx-1.19.0.tar.gz /src/ucx-1.19.0.tar.gz
RUN cd /src/ &&\
    tar vxzf ucx-1.19.0.tar.gz &&\
    cd ucx-1.19.0 &&\
    ./configure                        \
    --enable-shared                    \
    --disable-static                   \
    --disable-doxygen-doc              \
    --enable-optimizations             \
    --enable-cma                       \
    --enable-devel-headers             \
    --with-verbs                       \
    --with-dm                          \
    --enable-mt			       \
    --with-gdrcopy=/usr                \
    --prefix=/opt/ucx     &&           \
    make -j "${MAX_JOBS}" &&           \
    make install-strip &&              \
    ldconfig
RUN pip3 install uv meson ninja pybind11

