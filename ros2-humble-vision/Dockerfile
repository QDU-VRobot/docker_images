# syntax=docker/dockerfile:1.6

FROM ros:humble-ros-base-jammy

ENV DEBIAN_FRONTEND=noninteractive
SHELL ["/bin/bash", "-lc"]

ARG VISION_REPO_SSH="git@github.com:QDU-VRobot/vision_dev.git"
ARG VISION_BRANCH=""

# Pin CUDA / TensorRT versions
ARG CUDA_PKG_VER="12-5"
ARG TENSORRT_PKG_VER="10.3.0.26-1+cuda12.5"
ARG NVIDIA_REPO_ARCH="x86_64"

# Intel OpenVINO repo
RUN wget https://apt.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB && \
    sudo apt-key add GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB && \
    echo "deb https://apt.repos.intel.com/openvino ubuntu22 main" | sudo tee /etc/apt/sources.list.d/intel-openvino.list

RUN apt-get update && apt-get install -y --no-install-recommends \
    git \
    openssh-client \
    ca-certificates \
    curl \
    unzip \
    gnupg \
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
    cuda-toolkit-${CUDA_PKG_VER} \
    tensorrt=${TENSORRT_PKG_VER} \
    tensorrt-dev=${TENSORRT_PKG_VER} \
    python3-libnvinfer=${TENSORRT_PKG_VER} \
    python3-libnvinfer-dev=${TENSORRT_PKG_VER}

# Optional but usually helpful for build systems
ENV CUDA_HOME=/usr/local/cuda-12.5
ENV PATH=${CUDA_HOME}/bin:${PATH}
ENV LD_LIBRARY_PATH=${CUDA_HOME}/lib64:${LD_LIBRARY_PATH}

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
RUN set -exo pipefail; \
    source /opt/ros/humble/setup.bash; \
    rosdep update; \
    rosdep install --from-paths src --ignore-src -r -y && \
    rm -rf /var/lib/apt/lists/*

ENV DEBIAN_FRONTEND=dialog
