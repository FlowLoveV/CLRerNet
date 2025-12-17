FROM nvidia/cuda:12.1.1-devel-ubuntu22.04

# 1. 更换APT国内源
RUN sed -i 's|http://archive.ubuntu.com|http://mirrors.aliyun.com|g' /etc/apt/sources.list && \
    sed -i 's|http://security.ubuntu.com|http://mirrors.aliyun.com|g' /etc/apt/sources.list

# 2. 设置时区避免交互
ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=Asia/Shanghai
RUN ln -fs /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# packages
RUN apt-get update 
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    sudo git zip curl \
    libopencv-dev libffi-dev liblapack-dev libsqlite3-dev \
    build-essential libssl-dev libbz2-dev libreadline-dev \
    ssh-client wget vim python3-opencv &&\
    rm -rf /var/lib/apt/lists/* /var/cache/apt/archives/*

# user setting
ARG UID
RUN useradd docker -l -u $UID -G sudo -s /bin/bash -m
RUN echo 'Defaults visiblepw' >> /etc/sudoers
RUN echo 'docker ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
USER docker

# python
ENV PYENV_ROOT=/home/docker/.pyenv
ENV PATH=$PYENV_ROOT/shims:$PYENV_ROOT/bin:$PATH
# 3. 使用原始GitHub源（清华大学镜像可能有问题）
RUN curl -L https://raw.githubusercontent.com/pyenv/pyenv-installer/master/bin/pyenv-installer | bash
ARG PYTHON_VERSION
RUN pyenv install ${PYTHON_VERSION} && pyenv global ${PYTHON_VERSION}

# 4. 配置pip国内源
RUN pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple && \
    pip config set global.trusted-host pypi.tuna.tsinghua.edu.cn

# torch
ARG TORCH_VERSION
ARG TORCH_CUDA_ARCH_LIST
ARG TORCHVISION_VERSION
RUN pip install "numpy<2.0"
# 5. 更换PyTorch国内源
RUN pip install torch==${TORCH_VERSION} torchvision==${TORCHVISION_VERSION} \
    -f https://mirrors.tuna.tsinghua.edu.cn/pytorch-wheels/torch_stable.html

# mmcv
ARG MIM_VERSION
ARG MMCV_VERSION
ARG MMENGINE_VERSION
ARG MMDET_VERSION
RUN pip install -U openmim==${MIM_VERSION}
# 6. 配置mim国内源
# RUN mim config set set base-url https://openmmlab.oss-accelerate.aliyuncs.com
RUN mim install mmcv==${MMCV_VERSION}
RUN mim install mmengine==${MMENGINE_VERSION}
RUN mim install mmdet==${MMDET_VERSION}

# requirements
RUN pip install -U pip setuptools
COPY requirements.txt /tmp/requirements.txt
RUN pip install -r /tmp/requirements.txt
RUN pip install "numpy<2.0"

# lane NMS
COPY --chown=docker libs/models/layers/nms /tmp/nms
WORKDIR /tmp/nms
RUN python /tmp/nms/setup.py install

# enable deterministic training
ENV CUBLAS_WORKSPACE_CONFIG=:4096:8

# path
ENV PYTHONPATH=/work:/home/docker/

WORKDIR /work

ENTRYPOINT ["/bin/bash"]