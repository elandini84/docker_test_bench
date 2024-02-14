ARG base_img
FROM ${base_img} as nav2_source_img

ARG ros2_distro

LABEL maintainer="Ettore Landini"

USER root

RUN apt install -y software-properties-common && add-apt-repository -y universe
RUN apt update && sudo apt install curl -y
RUN curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg

RUN echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null

RUN apt update && apt upgrade -y && sudo apt install -y \
    python3-pip \
    python3-pytest-cov \
    python3-flake8-blind-except \
    python3-flake8-class-newline \
    python3-flake8-deprecated \
    python3-pytest-repeat \
    python3-pytest-rerunfailures \
    ros-dev-tools && \
    apt update && apt upgrade -y && apt install -y ros-${ros2_distro}-desktop && \
    echo "source /opt/ros/${ros2_distro}/setup.bash" >> /home/yarp-user/.ros2_source

USER yarp-user
WORKDIR /home/yarp-user

# Possibly temporary ROS2 network middleware change
RUN sudo apt-get update && sudo apt-get install -y cyclonedds-dev cyclonedds-tools \
    ros-$ros2_distro-cyclonedds-dbgsym ros-$ros2_distro-cyclonedds ros-$ros2_distro-rmw-cyclonedds-cpp \
    ros-$ros2_distro-rmw-cyclonedds-cpp-dbgsym libcycloneddsidl0 && \
    sudo apt-get clean && sudo rm -rf /var/lib/apt/lists/*
ENV RMW_IMPLEMENTATION=rmw_cyclonedds_cpp

# Navigation2
WORKDIR /home/yarp-user
RUN mkdir -p nav2_ws/src
WORKDIR /home/yarp-user/nav2_ws
RUN cd src && git clone https://github.com/elandini84/navigation2.git --branch ${ros2_distro}
RUN /bin/bash -c "source /home/yarp-user/.ros2_source && sudo rosdep init && rosdep update"

RUN sudo apt update && \
    /bin/bash -c "source /home/yarp-user/.ros2_source && rosdep install -y \
    --from-paths ./src \
    --ignore-src && \
    colcon build \
    --symlink-install"
RUN echo "source /home/yarp-user/nav2_ws/install/setup.bash" >> /home/yarp-user/.ros2_source && \
    echo "source /home/yarp-user/.ros2_source" >> /home/yarp-user/.bashrc
