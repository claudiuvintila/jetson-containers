#---
# name: tensorrt
# group: cuda
# depends: [cuda, cudnn, python]
# config: config.py
# test: test.sh
#---
ARG BASE_IMAGE
FROM ${BASE_IMAGE}

ARG TENSORRT_URL
ARG TENSORRT_DEB
ARG TENSORRT_PACKAGES

ARG PYTHON_VERSION=3.6.9
ARG PY_VERSION=3.6

# prepare headers
RUN cd /opt && \
    mkdir python${PY_VERSION} && \
    mkdir python${PY_VERSION}/include && \
    wget http://ftp.us.debian.org/debian/pool/main/p/python${PY_VERSION}/libpython${PY_VERSION}-dev_${PYTHON_VERSION}-1_arm64.deb && \
    ar x libpython${PY_VERSION}-dev_${PYTHON_VERSION}-1_arm64.deb && \
    tar -xvf data.tar.xz && \
    cp ./usr/include/aarch64-linux-gnu/python${PY_VERSION}/pyconfig.h python${PY_VERSION}/include/ && \
    cp -r Python-${PYTHON_VERSION}/Include/* python${PY_VERSION}/include/ && \
    ls -las

# cleanup headers
#RUN rm -rf libpython${PY_VERSION}-dev_${PYTHON_VERSION}-1_arm64.deb data.tar.xz control.tar.xz debian-binary


# tensorrt
COPY ./docker/tensorrt/scripts/build.sh /opt/build.sh
RUN cd /opt && \
    git clone https://github.com/pybind/pybind11.git && \
    git clone -b release/8.0 https://github.com/NVIDIA/TensorRT.git && \
    cd TensorRT && \
    git submodule update --init --recursive && \
    cd python && \
    cp ./build.sh ./build.sh.old && \
    mv /opt/build.sh ./build.sh && \
    TRT_OSSPATH=${PWD}/.. EXT_PATH=${PWD}/../.. ./build.sh && \
    python${PY_VERSION} -m pip install build/dist/tensorrt-8.0.1.6-cp${PY_VERSION/./}-none-linux_aarch64.whl && \
    cd ../..

#RUN echo "Downloading ${TENSORRT_DEB}" && \
#    mkdir /tmp/cudnn && cd /tmp/cudnn && \
#    wget --quiet --show-progress --progress=bar:force:noscroll ${TENSORRT_URL} && \
#    dpkg -i *.deb && \
#    cp /var/nv-tensorrt-local-repo-*/nv-tensorrt-local-*-keyring.gpg /usr/share/keyrings/ && \
#    apt-get update && \
#    apt-cache search tensorrt && \
#    apt-get install -y --no-install-recommends ${TENSORRT_PACKAGES} && \
#    rm -rf /var/lib/apt/lists/* && \
#    apt-get clean && \
#    dpkg --list | grep tensorrt && \
#    dpkg -P ${TENSORRT_DEB} && \
#    rm -rf /tmp/tensorrt