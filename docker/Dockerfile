# ----------------------------------------------- Base Docker Image -----------------------------------------------
FROM nvidia/cuda:10.0-devel-ubuntu18.04

ENV PYTORCH_VERSION=1.4.0
ENV TORCHVISION_VERSION=0.5.0
ENV CUDNN_VERSION=7.6.0.64-1+cuda10.0
ENV NCCL_VERSION=2.4.7-1+cuda10.0

# Workaround for deadlock issue. To be removed with next major Pytorch release
ENV NCCL_LL_THRESHOLD=0

# Python 2.7 or 3.6 is supported by Ubuntu Bionic out of the box
ARG python=3.6
ENV PYTHON_VERSION=${python}
ENV DEBIAN_FRONTEND=noninteractive

# Set default shell to /bin/bash
SHELL ["/bin/bash", "-cu"]

RUN apt-get update && apt-get install -y --allow-downgrades --allow-change-held-packages --no-install-recommends \
    build-essential \
    cmake \
    g++-4.8 \
    git \
    curl \
    docker.io \
    vim \
    wget \
    ca-certificates \
    libcudnn7=${CUDNN_VERSION} \
    libnccl2=${NCCL_VERSION} \
    libnccl-dev=${NCCL_VERSION} \
    libjpeg-dev \
    libpng-dev \
    python${PYTHON_VERSION} \
    python${PYTHON_VERSION}-dev \
    python3-tk \
    librdmacm1 \
    libibverbs1 \
    ibverbs-providers \
    libgtk2.0-dev \
    unzip \
    bzip2 \
    htop  \
    libsm6 \
    libxext6 \
    libxrender-dev

# Instal Python and pip
RUN if [[ "${PYTHON_VERSION}" == "3.6" ]]; then \
    apt-get install -y python${PYTHON_VERSION}-distutils; \
    fi

RUN ln -sf /usr/bin/python${PYTHON_VERSION} /usr/bin/python

RUN curl -O https://bootstrap.pypa.io/get-pip.py && \
    python get-pip.py && \
    rm get-pip.py

# Install PyTorch
RUN pip install future typing numpy awscli
RUN pip install https://download.pytorch.org/whl/cu100/torch-${PYTORCH_VERSION}-cp36-cp36m-linux_x86_64.whl
RUN pip install https://download.pytorch.org/whl/cu100/torchvision-${TORCHVISION_VERSION}-cp36-cp36m-linux_x86_64.whl


# Configure environment variables - default working directory is "/workspace"
WORKDIR /workspace
ENV PYTHONPATH="/workspace"

# Install dependencies
RUN pip install ninja yacs cython matplotlib opencv-python tqdm onnx onnxruntime coloredlogs scipy pycuda
RUN pip uninstall -y pillow
RUN pip install pillow-simd==6.2.2.post1 pycocotools 

# Install apex
WORKDIR /workspace
RUN git clone https://github.com/NVIDIA/apex.git
WORKDIR /workspace/apex
RUN git checkout 82dac9c9419035110d1ccc49b2608681337903ed
RUN pip install -v --no-cache-dir --global-option="--cpp_ext" --global-option="--cuda_ext" .

RUN wget -o 'nv-tensorrt-repo-ubuntu1804-cuda10.0-trt7.0.0.11-ga-20191216_1-1_amd64.deb' https://www.dropbox.com/s/afuvlpj4qiw3bn1/nv-tensorrt-repo-ubuntu1804-cuda10.0-trt7.0.0.11-ga-20191216_1-1_amd64.deb?dl=0
RUN dpkg -i 'nv-tensorrt-repo-ubuntu1804-cuda10.0-trt7.0.0.11-ga-20191216_1-1_amd64.deb'
RUN apt-key add /var/nv-tensorrt-repo-cudax.x-trt7.x.x.x-gayyyymmdd/7fa2af80.pub
RUN apt-get update
RUN apt-get install tensorrt
RUN apt-get install python3-libnvinfer


ENV version=7.0.0-1+cuda10.0
RUN apt-get install libnvinfer7=${version} \
    libnvonnxparsers7=${version} \
    libnvparsers7=${version} \
    libnvinfer-plugin7=${version} \
    libnvinfer-dev=${version} \
    libnvonnxparsers-dev=${version} \
    libnvparsers-dev=${version} \
    libnvinfer-plugin-dev=${version} \
    python-libnvinfer=${version} \
    python3-libnvinfer=${version}

# Copy repo
ENV PYTHONPATH="/workspace/panoptic:$PYTHONPATH"
COPY . /workspace/panoptic
WORKDIR /workspace/panoptic


