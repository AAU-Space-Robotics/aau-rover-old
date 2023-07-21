# Use the latest version of Jetpack as base image
FROM nvcr.io/nvidia/l4t-jetpack:r35.3.1

# Avoid prompts with apt-get
ENV DEBIAN_FRONTEND=noninteractive

# Update apt-get and install necessary packages
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
            python3-pip \
            python3-dev \
            libopenblas-dev \
            libopenmpi-dev \
            openmpi-bin \
            openmpi-common \
            gfortran \
            libomp-dev && \
    rm -rf /var/lib/apt/lists/* && \
    apt-get clean

# Install pip packages
RUN pip3 install --no-cache-dir setuptools Cython wheel numpy

###################################
###       Install PyTorch       ###
###################################

# Use a wheel from Nvidia instead of building from source
# Check https://developer.download.nvidia.com/compute/redist/jp/v512/pytorch/ for more details
ARG PYTORCH_URL=https://developer.download.nvidia.com/compute/redist/jp/v512/pytorch/torch-2.1.0a0+41361538.nv23.06-cp38-cp38-linux_aarch64.whl
ARG PYTORCH_WHL=torch-2.1.0a0+41361538.nv23.06-cp38-cp38-linux_aarch64.whl

RUN wget --quiet --show-progress --progress=bar:force:noscroll --no-check-certificate ${PYTORCH_URL} -O ${PYTORCH_WHL} && \
    pip3 install --no-cache-dir --verbose ${PYTORCH_WHL} && \
    rm ${PYTORCH_WHL}

###################################
###         Install ROS         ###
###################################

# Update system locale
RUN apt-get update && apt-get install -y locales && \
    locale-gen en_US en_US.UTF-8 && \
    update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8

ENV LANG=en_US.UTF-8
ENV PYTHONIOENCODING=utf-8

# Set Python3 as default
RUN update-alternatives --install /usr/bin/python python /usr/bin/python3 1

# Install necessary packages for ROS
RUN apt-get install -y software-properties-common && \
    add-apt-repository universe && \
    apt-get update && \
    apt-get install -y curl

# Add ROS keys
RUN curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | tee /etc/apt/sources.list.d/ros2.list > /dev/null

# Install ROS and necessary python packages
RUN apt-get update && \
    apt-get install -y \
      python3-flake8-docstrings \
      python3-pip \
      python3-pytest-cov \
      ros-dev-tools && \
    python3 -m pip install -U \
       flake8-blind-except \
       flake8-builtins \
       flake8-class-newline \
       flake8-comprehensions \
       flake8-deprecated \
       flake8-import-order \
       flake8-quotes \
       "pytest>=5.3" \
       pytest-repeat \
       pytest-rerunfailures

ARG ROS_PKG=ros_base
ENV ROS_DISTRO=humble
ENV ROS_ROOT=/opt/ros/${ROS_DISTRO}

# Clone ROS repos
RUN mkdir -p ~/${ROS_ROOT}/src && \
    cd ~/${ROS_ROOT} && \
    vcs import --input https://raw.githubusercontent.com/ros2/ros2/${ROS_DISTRO}/ros2.repos src 

# Upgrade system and initialize rosdep
RUN apt-get upgrade -y && \
    cd ~/${ROS_ROOT} && \
    rosdep init && \
    rosdep update && \
    rosdep install --from-paths src --ignore-src -y --skip-keys "fastcdr rti-connext-dds-6.0.1 urdfdom_headers"

# Cleanup python3.9 if exists
RUN apt-get purge -y python3.9 libpython3.9* || echo "python3.9 not found, skipping removal"

# Build ROS2
RUN cd ~/${ROS_ROOT}/ && \
    colcon build --symlink-install