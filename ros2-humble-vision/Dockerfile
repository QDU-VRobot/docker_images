# syntax=docker/dockerfile:1.6

FROM ros:humble-ros-base-jammy

ENV DEBIAN_FRONTEND=noninteractive
ENV ROS_DISTRO=humble
SHELL ["/bin/bash", "-lc"]

ARG VISION_REPO_SSH="git@github.com:QDU-VRobot/vision_dev.git"
ARG VISION_BRANCH=""

ARG CUDA_PKG_VER="12-5"
ARG TENSORRT_PKG_VER="10.3.0.26-1+cuda12.5"
ARG NVIDIA_REPO_ARCH="x86_64"

RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates \
    curl \
    gnupg \
    gpg \
    wget \
    && rm -rf /var/lib/apt/lists/*

# Intel OpenVINO repo
RUN curl -fsSL https://apt.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB \
    | gpg --dearmor -o /usr/share/keyrings/intel-openvino.gpg && \
    echo "deb [signed-by=/usr/share/keyrings/intel-openvino.gpg] https://apt.repos.intel.com/openvino ubuntu22 main" \
    > /etc/apt/sources.list.d/intel-openvino.list

# NVIDIA CUDA/TensorRT repo
RUN wget -q https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/${NVIDIA_REPO_ARCH}/cuda-keyring_1.1-1_all.deb && \
    dpkg -i cuda-keyring_1.1-1_all.deb && \
    rm -f cuda-keyring_1.1-1_all.deb

# Kitware CMake repo repo
RUN wget -O /tmp/kitware-archive-latest.asc https://apt.kitware.com/keys/kitware-archive-latest.asc && \
    gpg --dearmor -o /usr/share/keyrings/kitware-archive-keyring.gpg /tmp/kitware-archive-latest.asc && \
    echo "deb [signed-by=/usr/share/keyrings/kitware-archive-keyring.gpg] https://apt.kitware.com/ubuntu/ jammy main" \
      > /etc/apt/sources.list.d/kitware.list && \
    rm -f /tmp/kitware-archive-latest.asc


RUN apt-get update && apt-get install -y --no-install-recommends \
    git \
    openssh-client \
    unzip \
    build-essential \
    cmake \
    ninja-build \
    python3-pip \
    python3-rosdep \
    python3-colcon-common-extensions \
    libwpa-client-dev \
    libnm-dev \
    libudev-dev \
    libasio-dev \
    openvino-2026.1.0 \
    cuda-compiler-${CUDA_PKG_VER} \
    cuda-cudart-dev-${CUDA_PKG_VER} \
    libnvinfer10=${TENSORRT_PKG_VER} \
    libnvinfer-dev=${TENSORRT_PKG_VER} \
    libnvinfer-headers-dev=${TENSORRT_PKG_VER} \
    libnvinfer-plugin10=${TENSORRT_PKG_VER} \
    libnvinfer-plugin-dev=${TENSORRT_PKG_VER} \
    libnvinfer-headers-plugin-dev=${TENSORRT_PKG_VER} \
    libnvonnxparsers10=${TENSORRT_PKG_VER} \
    libnvonnxparsers-dev=${TENSORRT_PKG_VER} \
    && (find /usr/lib/x86_64-linux-gnu \
      \( -name "libnvinfer*.a" \
         -o -name "libnvonnxparser*.a" \
         -o -name "libnvparsers*.a" \) \
      -delete 2>/dev/null || true) \
    && rm -rf \
         /var/lib/apt/lists/* \
         /tmp/* \
         /var/tmp/* \
         /usr/local/cuda*/samples \
         /usr/local/cuda*/doc \
         /usr/local/cuda*/extras \
    && apt-get clean

ENV CUDA_HOME=/usr/local/cuda-12.5
ENV CUDAToolkit_ROOT=/usr/local/cuda-12.5
ENV PATH=${CUDA_HOME}/bin:${PATH}
ENV LD_LIBRARY_PATH=${CUDA_HOME}/lib64:${LD_LIBRARY_PATH}
ENV CMAKE_CUDA_COMPILER=${CUDA_HOME}/bin/nvcc
ENV CMAKE_CUDA_HOST_COMPILER=/usr/bin/g++

RUN rosdep init || true

RUN mkdir -p /root/.ssh && chmod 700 /root/.ssh && \
    ssh-keyscan github.com >> /root/.ssh/known_hosts && \
    chmod 644 /root/.ssh/known_hosts

WORKDIR /opt/ws
RUN --mount=type=ssh \
    set -euxo pipefail; \
    if [[ -n "${VISION_BRANCH}" ]]; then \
      git clone --branch "${VISION_BRANCH}" "${VISION_REPO_SSH}" vision_dev; \
    else \
      git clone "${VISION_REPO_SSH}" vision_dev; \
    fi; \
    cd vision_dev; \
    git config --global url."git@github.com:".insteadOf "https://github.com/"; \
    git submodule sync --recursive; \
    git submodule update --init --recursive

WORKDIR /opt/ws/vision_dev
RUN set -euxo pipefail; \
    apt-get update; \
    rosdep update; \
    rosdep install --rosdistro ${ROS_DISTRO} --from-paths src --ignore-src -r -y; \
    rm -rf /var/lib/apt/lists/*

ENV DEBIAN_FRONTEND=dialog
