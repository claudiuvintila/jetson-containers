#---
# name: python
# alias: python3
# group: build
# depends: [build-essential]
# notes: installs core python3 packages and pip
#---
ARG BASE_IMAGE
FROM ${BASE_IMAGE}

ARG PYTHON_VERSION=3.8.2
ARG PY_VERSION=3.8

ENV PYTHONIOENCODING=utf-8

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
		  python${PY_VERSION}-dev \
		  python3-pip \
    && rm -rf /var/lib/apt/lists/* \
    && apt-get clean && \
    update-alternatives --install /usr/bin/python3 python3 /usr/bin/python${PY_VERSION} 1 && \
    python3 -m pip install --upgrade pip && \
    \
    which python || python --version || pip --version || \
    which python3 || python3 --version || pip3 --version && \
    which pip3 && pip3 --version && \
    \
    pip3 install --upgrade --no-cache-dir \
	 setuptools \
	 packaging \
	 'Cython' \
	 wheel && \
    \
    pip3 install --no-cache-dir --verbose wget psutil

SHELL ["/bin/bash", "-c"]
# cmake
RUN apt-get update && \
    apt-get install -y protobuf-compiler libprotobuf-dev openssl libssl-dev libcurl4-openssl-dev && \
    mkdir /workspace && \
    cd /workspace && \
    wget https://github.com/Kitware/CMake/releases/download/v3.13.5/cmake-3.13.5.tar.gz && \
    tar xvf cmake-3.13.5.tar.gz > /dev/null && \
    cd cmake-3.13.5 && \
    ./bootstrap --system-curl && \
    make -j$(nproc) && \
    make install && \
    cd ..

# prepare headers
RUN cd /workspace && \
    mkdir python${PY_VERSION} && \
    mkdir python${PY_VERSION}/include && \
    apt-get download libpython${PY_VERSION}-dev && \
    ar x libpython${PY_VERSION}-dev_${PYTHON_VERSION}-*_arm64.deb && \
    tar -xvf data.tar.xz && \
    cp ./usr/include/aarch64-linux-gnu/python${PY_VERSION}/pyconfig.h python${PY_VERSION}/include/ && \
    wget https://www.python.org/ftp/python/${PYTHON_VERSION}/Python-${PYTHON_VERSION}.tgz && \
    tar xzf Python-${PYTHON_VERSION}.tgz > /dev/null && \
    cp -r Python-${PYTHON_VERSION}/Include/* python${PY_VERSION}/include/ && \
    ls -las

# cleanup headers
#RUN rm -rf libpython${PY_VERSION}-dev_${PYTHON_VERSION}-*_arm64.deb data.tar.xz control.tar.xz debian-binary


# tensorrt
#COPY ./docker/tensorrt/scripts/build.sh /opt/build.sh
RUN cd /workspace && \
    git clone https://github.com/pybind/pybind11.git && \
    git clone -b release/8.0 https://github.com/NVIDIA/TensorRT.git && \
    cd TensorRT && \
    git submodule update --init --recursive && \
    cd python && \
    PYTHON_MAJOR_VERSION=3 && \
    PYTHON_MINOR_VERSION=8 && \
    TARGET=aarch64 && \
    TARGET=aarch64 && \
    TRT_OSSPATH=${PWD}/.. && \
    EXT_PATH=${PWD}/../.. && \
    ./build.sh && \
    python${PY_VERSION} -m pip install build/dist/tensorrt-8.0.1.6-cp${PY_VERSION/./}-none-linux_aarch64.whl && \
    cd ../..

# this was causing issues downstream (e.g. Python2.7 still around in Ubuntu 18.04,
# and in cmake python enumeration where some packages expect that 'python' is 2.7)
#RUN update-alternatives --install /usr/bin/python python /usr/bin/python3 1 && \
#    update-alternatives --install /usr/bin/pip pip /usr/bin/pip3 1
