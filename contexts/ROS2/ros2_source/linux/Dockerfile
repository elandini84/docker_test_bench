ARG base_img
FROM ${base_img} as ros2_lando_default

ARG ros2_distro

LABEL maintainer="Ettore Landini"

USER root

RUN apt install -y software-properties-common && add-apt-repository -y universe
RUN apt update && sudo apt install curl -y
RUN curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg

RUN echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null

RUN apt update && sudo apt install -y \
    python3-flake8-docstrings \
    python3-pip \
    python3-pytest-cov \
    python3-flake8-blind-except \
    python3-flake8-builtins \
    python3-flake8-class-newline \
    python3-flake8-comprehensions \
    python3-flake8-deprecated \
    python3-flake8-import-order \
    python3-flake8-quotes \
    python3-pytest-repeat \
    python3-pytest-rerunfailures \
    ros-dev-tools libboost-program-options-dev \
    graphicsmagick libgraphicsmagick++-q16-12 \
    libgraphicsmagick++1-dev libxtensor-dev \
    libceres-dev libcv-bridge-dev libompl-dev \
    libgeographic-dev

USER yarp-user
WORKDIR /home/yarp-user

RUN mkdir -p ros2_${ros2_distro}/src && cd ros2_${ros2_distro} && \
    vcs import --input https://raw.githubusercontent.com/ros2/ros2/${ros2_distro}/ros2.repos src

RUN cd ros2_${ros2_distro} sudo apt upgrade -y && sudo rosdep init && rosdep update && \
    rosdep install --from-paths src --ignore-src -y --skip-keys "fastcdr rti-connext-dds-6.0.1 urdfdom_headers"

WORKDIR /home/yarp-user/ros2_${ros2_distro}
RUN colcon build --symlink-install && echo "source /home/yarp-user/ros2_$ros2_distro/install/setup.bash" > /home/yarp-user/.ros2_source

# Possibly temporary ROS2 network middleware change
RUN /bin/bash -c "source /home/yarp-user/ros2_$ros2_distro/install/setup.bash && sudo apt-get update && sudo apt-get install -y cyclonedds-dev cyclonedds-tools \
    ros-$ros2_distro-cyclonedds-dbgsym ros-$ros2_distro-cyclonedds ros-$ros2_distro-rmw-cyclonedds-cpp \
    ros-$ros2_distro-rmw-cyclonedds-cpp-dbgsym libcycloneddsidl0 && \
    sudo apt-get clean && sudo rm -rf /var/lib/apt/lists/*"
ENV RMW_IMPLEMENTATION=rmw_cyclonedds_cpp

WORKDIR /home/yarp-user
RUN git clone https://github.com/BehaviorTree/BehaviorTree.CPP -b v3.8 && \
    cd BehaviorTree.CPP && mkdir build && cmake .. && make -j11 && sudo make install

# Navigation2 dependencies
RUN mkdir -p nav2_dependencies_ws/src
WORKDIR /home/yarp-user/nav2_dependencies_ws
RUN cd src && git clone https://github.com/ros/angles -b ros2 && \
    git clone https://github.com/ros-geographic-info/geographic_info -b ros2 && \
    git clone https://github.com/ros/bond_core -b ros2 && \
    git clone https://github.com/ros/diagnostic -b ros2 && \
    git clone https://github.com/cra-ros-pkg/robot_localization && \
    cd .. && /bin/bash -c "source /home/yarp-user/ros2_$ros2_distro/install/setup.bash && colcon build --symlink-install" && \
    echo "source /home/yarp-user/nav2_dependencies_ws/install/setup.bash" >> /home/yarp-user/.ros2_source

# Navigation2
WORKDIR /home/yarp-user
RUN mkdir -p nav2_ws/src
WORKDIR /home/yarp-user/nav2_ws
RUN cd src && git clone https://github.com/elandini84/navigation2.git --branch ${ros2_distro}
RUN /bin/bash -c "source /home/yarp-user/.ros2_source && rosdep install -y --from-paths ./src --ignore-src && colcon build --symlink-install" && \
    echo "source /home/yarp-user/nav2_ws/install/setup.bash" >> /home/yarp-user/.ros2_source && \
    echo "source /home/yarp-user/.ros2_source" >> /home/yarp-user/.bashrc
