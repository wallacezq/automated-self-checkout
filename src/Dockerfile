#
# Copyright (C) 2024 Intel Corporation.
#
# SPDX-License-Identifier: Apache-2.0
#

#FROM intel/dlstreamer:2024.1.1-dev-ubuntu22 as base
FROM intel/dlstreamer:2025.0.1.2-dev-ubuntu24 as base
USER root

FROM base as build-default
COPY ./requirements.txt /requirements.txt
RUN pip3 install --upgrade pip --no-cache-dir -r /requirements.txt
RUN pip3 install --pre torch torchvision torchaudio --index-url https://download.pytorch.org/whl/nightly/xpu
WORKDIR /
COPY /extensions /home/pipeline-server/extensions
COPY /pipelines /home/pipeline-server/pipelines
COPY entrypoint.sh /script/entrypoint.sh
RUN chmod +x /script/entrypoint.sh

# Install latest libva
RUN mkdir -p /tmp/newva
WORKDIR /tmp/newva
RUN git clone https://github.com/intel/libva.git
WORKDIR /tmp/newva/libva
RUN ./autogen.sh --prefix=/usr --libdir=/usr/lib/x86_64-linux-gnu && make -j"$(cat < /proc/cpuinfo |grep -c proc)" && make install
WORKDIR /tmp/newva
RUN git clone https://github.com/intel/gmmlib.git
WORKDIR /tmp/newva/gmmlib
RUN mkdir build && cd build && cmake -DCMAKE_INSTALL_PREFIX=/usr/ -DCMAKE_INSTALL_LIBDIR=/usr/lib/x86_64-linux-gnu -DCMAKE_BUILD_TYPE=Release .. && \
    make -j"$(cat < /proc/cpuinfo |grep -c proc)" && make install
WORKDIR /tmp/newva/
RUN git clone https://github.com/intel/media-driver.git
RUN mkdir build_media && cd build_media && \
    cmake -DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_INSTALL_LIBDIR=/usr/lib/x86_64-linux-gnu ../media-driver && \
    make -j"$(cat < /proc/cpuinfo |grep -c proc)" && make install
    


FROM base as build-realsense
RUN if [ -n "$HTTP_PROXY" ] ; then  echo "Acquire::http::Proxy \"$HTTP_PROXY\";" >  /etc/apt/apt.conf; fi
RUN apt-get update -y || true; DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    build-essential \
    autoconf \
    git \
    libssl-dev \
    libusb-1.0-0-dev \
    libudev-dev \
    pkg-config \
    libgtk-3-dev \
    libglfw3-dev \
    libgl1-mesa-dev \
    libglu1-mesa-dev \
    nasm \
    ninja-build \
    cmake  \
    python3  \
    python3-pip  \
    meson && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
    
# Install realsense
RUN mkdir -p /rs
WORKDIR /rs
RUN git clone https://github.com/gwen2018/librealsense.git
WORKDIR /rs/librealsense
RUN git checkout stream_d436_b
COPY ./patch/libusb.h /rs/librealsense/src/libusb/libusb.h
RUN mkdir build
WORKDIR /rs/librealsense/build
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
RUN cmake ../ \
        -DBUILD_SHARED_LIBS=true \
        -DBUILD_WITH_JPEGTURBO=true \
        -DBUILD_PYTHON_BINDINGS:bool=true \
        -DBUILD_WITH_CUDA=false \
        -DFORCE_RSUSB_BACKEND=false \
        -DPYTHON_EXECUTABLE=/usr/bin/python3 \
        -DBUILD_GLSL_EXTENSIONS=false \
        -DBUILD_WITH_CPU_EXTENSIONS=true \
        -DBUILD_UNIT_TESTS=false \
	-DBUILD_GRAPHICAL_EXAMPLES=false \
	-DCMAKE_BUILD_TYPE=Release && \
    make -j"$(cat < /proc/cpuinfo |grep -c proc)" && \
    make install && \
    export PYTHONPATH="$PYTHONPATH":/usr/lib/python3/dist-packages/pyrealsense2 && \
    python3 -c "import pyrealsense2 as rs; print(rs)"
RUN mv /rs/librealsense/build/libjpeg-turbo/lib/libturbojpeg.so* /usr/local/lib
# Build gst realsense element. Use github version once pull request is accepted with bug fixes
WORKDIR /rs
RUN git clone https://github.com/brian-intel/realsense-gstreamer
WORKDIR /rs/realsense-gstreamer
RUN /usr/bin/meson . build && ninja -C build && \
    cp /rs/realsense-gstreamer/build/src/libgstrealsense_meta.so /opt/intel/dlstreamer/gstreamer/lib/ && \
    cp /rs/realsense-gstreamer/build/src/libgstrealsensesrc.so /opt/intel/dlstreamer/gstreamer/lib/gstreamer-1.0 && \
    cp /usr/local/lib/libturbojpeg.so* /opt/intel/dlstreamer/gstreamer/lib/
