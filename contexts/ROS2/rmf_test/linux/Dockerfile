## docker build -t elandini84/ros2Images:rmf .
FROM ubuntu:22.04
LABEL maintainer="Ettore Landini"

# Non-interactive installation mode
ENV DEBIAN_FRONTEND=noninteractive
ENV DOCKER_UPDATED_ON=28_09_2022

# Install essentials
RUN apt-get update && apt-get install -y apt-utils software-properties-common sudo psmisc lsb-release protobuf-compiler libatlas-base-dev \
    tmux nano geany vim wget curl build-essential git gitk cmake cmake-curses-gui autoconf xserver-xorg-video-dummy xserver-xorg-legacy \
    net-tools terminator libjpeg-dev ffmpeg apt-transport-https ca-certificates gnupg libace-dev locales qtbase5-dev qt5-qmake \
    python3-setuptools python3-pip iproute2 python3-tornado swig lsof iftop iputils-ping gdb bash-completion mlocate && rm -rf /var/lib/apt/lists/*

# Locale
RUN apt-get update && apt-get install locales -y && \
    locale-gen en_US en_US.UTF-8 && \
    update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8 && \
    export LANG=en_US.UTF-8

RUN pip3 install numpy bpytop

RUN sed -i 's/allowed_users=console/allowed_users=anybody/' /etc/X11/Xwrapper.config
COPY ./xorg.conf /etc/X11/xorg.conf

# ROS2 Installation
RUN apt-get install software-properties-common && add-apt-repository universe && \
    apt-get update && apt-get install curl gnupg lsb-release && \
    curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/ros2.list > /dev/null
    #echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(source /etc/os-release && echo focal) main" | tee /etc/apt/sources.list.d/ros2.list > /dev/null
RUN apt-get update && apt-get install ros-humble-ros-base -y


# Gazebo
RUN apt-get update && apt-get install -y wget && \
    sh -c 'echo "deb http://packages.osrfoundation.org/gazebo/ubuntu-stable `lsb_release -cs` main" > /etc/apt/sources.list.d/gazebo-stable.list' && \
    wget https://packages.osrfoundation.org/gazebo.key -O - | apt-key add -

# RMF
RUN apt-get install ros-humble-rmf* -y


# Create user: user1
USER root
RUN useradd -l -u 33334 -G sudo -md /home/user1 -s /bin/bash -p user1 user1 && \
    # passwordless sudo for users in the 'sudo' group
    sed -i.bkp -e 's/%sudo\s\+ALL=(ALL\(:ALL\)\?)\s\+ALL/%sudo ALL=NOPASSWD:ALL/g' /etc/sudoers

RUN apt-get install -y python3-rosdep python3-vcstool && rosdep init

# ROS2 Installation (part2)
USER user1
RUN sudo rosdep fix-permissions && rosdep update

RUN python3 -m pip install flask-socketio && sudo apt-get install -y python3-colcon*
RUN sudo apt-get install python3-vcstool -y

# Demo rmf
RUN ros2 run rmf_building_map_tools model_downloader rmf_demos_maps -s airport_terminal


# Set ENV
USER user1
