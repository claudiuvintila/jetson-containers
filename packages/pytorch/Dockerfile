#---
# name: pytorch
# alias: torch
# group: pytorch
# config: config.py
# depends: [cuda, cudnn, tensorrt, numpy, onnx]
# test: test.py
# docs: |
#  Containers for PyTorch with CUDA support.
#  Note that the [`l4t-pytorch`](/packages/l4t/l4t-pytorch) containers also include PyTorch, torchvision, and torchaudio.
#---
ARG BASE_IMAGE
FROM ${BASE_IMAGE}

ENV DEBIAN_FRONTEND=noninteractive

ARG PYTORCH_URL
ARG PYTORCH_WHL
ARG TORCH_VERSION=1.10.0

COPY --from=tonistiigi/xx / /

#RUN if [ "$(xx-info march)" = 'aarch64' ]; then \
#      apt-get update \
#        && apt-get install -y wget \
#        && wget http://ports.ubuntu.com/pool/main/o/openssl/libssl1.1_1.1.1f-1ubuntu2_$(xx-info pkg-arch).deb \
#        && apt-get install ./libssl1.1_1.1.1f-1ubuntu2_$(xx-info pkg-arch).deb \
#        && rm ./libssl1.1_1.1.1f-1ubuntu2_$(xx-info pkg-arch).deb; \
#    else \
#      apt-get update \
#        && apt-get install -y wget \
#        && wget http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.1f-1ubuntu2_amd64.deb \
#        && apt-get install ./libssl1.1_1.1.1f-1ubuntu2_amd64.deb \
#        && rm ./libssl1.1_1.1.1f-1ubuntu2_amd64.deb; \
#    fi

RUN apt-get update \
    && apt-get install -y wget curl git gpg \
    && wget -O - https://apt.kitware.com/keys/kitware-archive-latest.asc 2>/dev/null | gpg --dearmor - | tee /usr/share/keyrings/kitware-archive-keyring.gpg >/dev/null \
    && echo 'deb [signed-by=/usr/share/keyrings/kitware-archive-keyring.gpg] https://apt.kitware.com/ubuntu/ bionic main' | tee /etc/apt/sources.list.d/kitware.list >/dev/null \
    && apt-get update \
    && rm /usr/share/keyrings/kitware-archive-keyring.gpg \
    && apt-get install kitware-archive-keyring -y \
    && apt-get install cmake -y

RUN pip3 install --upgrade setuptools pip wheel \
  && pip3 install MarkupSafe==2.0.0 Cython==0.29.37

RUN git clone --depth 1 --branch v${TORCH_VERSION} https://github.com/pytorch/pytorch \
    && cd pytorch \
    && git submodule sync \
    && git submodule update --init --recursive \
    && pip3 install -r requirements.txt \
    && export PYTORCH_BUILD_VERSION=$(git describe --tags --exact | sed 's/^v//') \
    && export PYTORCH_BUILD_NUMBER=0 \
    && python3 setup.py bdist_wheel

# install prerequisites
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
		  libopenblas-dev \
		  libopenmpi-dev \
            openmpi-bin \
            openmpi-common \
		  gfortran \
		  libomp-dev \
    && rm -rf /var/lib/apt/lists/* \
    && apt-get clean && \
    \
    # download and install the PyTorch wheel
    cd /opt && \
    wget --quiet --show-progress --progress=bar:force:noscroll --no-check-certificate ${PYTORCH_URL} -O ${PYTORCH_WHL} && \
    pip3 install --verbose ${PYTORCH_WHL} && \
    python3 -c 'import torch; print(f"PyTorch version: {torch.__version__}"); print(f"CUDA available:  {torch.cuda.is_available()}"); print(f"cuDNN version:   {torch.backends.cudnn.version()}"); print(torch.__config__.show());' && \
    \
    # patch for https://github.com/pytorch/pytorch/issues/45323
    PYTHON_ROOT=`pip3 show torch | grep Location: | cut -d' ' -f2` && \
    TORCH_CMAKE_CONFIG=$PYTHON_ROOT/torch/share/cmake/Torch/TorchConfig.cmake && \
    echo "patching _GLIBCXX_USE_CXX11_ABI in ${TORCH_CMAKE_CONFIG}" && \
    sed -i 's/  set(TORCH_CXX_FLAGS "-D_GLIBCXX_USE_CXX11_ABI=")/  set(TORCH_CXX_FLAGS "-D_GLIBCXX_USE_CXX11_ABI=0")/g' ${TORCH_CMAKE_CONFIG} && \
    \
    # PyTorch C++ extensions frequently use ninja parallel builds
    pip3 install --no-cache-dir scikit-build && \
    pip3 install --no-cache-dir ninja

# set the CUDA architectures that PyTorch extensions get built for
ARG TORCH_CUDA_ARCH_ARGS
ENV TORCH_CUDA_ARCH_LIST=${TORCH_CUDA_ARCH_ARGS} \
    TORCH_HOME=/data/models/torch
