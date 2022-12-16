FROM elandini84/r1images:r1Sim2 as ros2_lando_default

LABEL maintainer="Ettore Landini"

# DANIELE STUFF ###############################################################################
ARG username
ARG uid
ARG gid

USER root
RUN groupmod -g $gid $username && \
    usermod -u $uid -g $gid -G adm,users,sudo,root $username && \
    find / -uid 33334 -exec chown -h $uid '{}' \; 2> /dev/null || true && \
    find / -gid 33334 -exec chgrp $gid '{}' \; 2> /dev/null || true && \
    chown -R $username: /home/$username && \
    mkdir -p /run/user/$uid && \
    chown $username: /run/user/$uid && \
    chmod 700 /run/user/$uid && \
    mkdir -p /run/user/$uid/dconf && \
    chown $username: /run/user/$uid/dconf && \
    chmod 700 /run/user/$uid/dconf && \
    mkdir -p /home/$username/.ros && \
    mkdir -p /home/$username/.config/yarp && \
    mkdir -p /home/$username/.local/share/yarp && \
    chown -R $username: /home/$username/

###############################################################################################

USER user1
WORKDIR /home/user1

RUN sudo apt-get update && sudo apt-get install -y nautilus python3-ipython ipython3 ros-humble-rviz*

RUN cd tour-guide-robot && git checkout ros2 && \
    git fetch origin && git pull origin ros2 && cd build && \
    cmake .. && make -j11

WORKDIR /home/user1/robotology

ENV RMW_IMPLEMENTATION=

RUN cd cer-sim && git remote add elandini84 https://github.com/elandini84/cer-sim.git && \
    git fetch elandini84 && git checkout --track elandini84/test/ros2_based_control

RUN echo "source /opt/ros/humble/setup.bash" >> /home/user1/.bashrc

RUN cd yarp-devices-ros2/ && git remote add elandini84 https://github.com/elandini84/yarp-devices-ros2.git && \
     git fetch elandini84

RUN cd navigation/ && git remote add elandini84 https://github.com/elandini84/navigation.git && \
     git fetch elandini84

WORKDIR /home/user1

# ROS2 launch file migrator
RUN git clone https://github.com/aws-robotics/ros2-launch-file-migrator && \
    cd ros2-launch-file-migrator && pip3 install -e ./ --user
ENV PATH=$PATH:/home/user1/.local/bin


# ros2_control from apt ################################################################################################################################## BUILD TARGET START ####
FROM ros2_lando_default AS ros2_ctrl_apt
RUN sudo apt-get update && sudo apt-get install -y ros-humble-control* ros-humble-ros2-control ros-humble-ros2-controllers


# moveit2 from apt only ################################################################################################################################## BUILD TARGET START ####
FROM ros2_ctrl_apt AS moveit2_apt
RUN sudo apt-get update && sudo apt-get install -y ros-humble-moveit*

RUN sudo apt-get install -y python3-colcon-common-extensions python3-colcon-mixin && \
     /bin/bash -c "source /opt/ros/humble/setup.bash; colcon mixin add default https://raw.githubusercontent.com/colcon/colcon-mixin-repository/master/index.yaml" && \
     /bin/bash -c "source /opt/ros/humble/setup.bash; colcon mixin update default"


# ros2_control from source ############################################################################################################################### BUILD TARGET START ####
FROM ros2_lando_default AS ros2_ctrl_default

RUN mkdir /home/user1/ws_rosdep_broken && cd ws_rosdep_broken && \
    git clone https://github.com/ros-controls/kinematics_interface && \
    cd kinematics_interface && /bin/bash -c "source /opt/ros/humble/setup.bash; colcon build --packages-select kinematics_interface" && \
    echo "source /home/user1/ws_rosdep_broken/kinematics_interface/install/setup.bash" >> /home/user1/.bashrc
RUN sudo apt-get update && sudo apt-get install -y ros-humble-realtime-tools
RUN mkdir /home/user1/ros2_control_ws && cd ros2_control_ws && mkdir src && \
    cd src && git clone https://github.com/ros-controls/ros2_control && \
    cd ros2_control && git fetch --all --tags && git checkout tags/3.3.0 -b v3.3.0-branch && \
    cd .. && git clone https://github.com/ros-controls/ros2_controllers && \
    #cd ros2_controllers && git fetch --all --tags && git checkout tags/2.3.0 -b v2.3.0-branch && cd .. && \
    cd ros2_controllers && git checkout 34da157213e64dbccbc32f3d478519371dbac54c && cd .. && \
    git clone https://github.com/ros-controls/ros2_control_demos && \
    git clone https://github.com/ros2/ros_testing
RUN cd /home/user1/ros2_control_ws && /bin/bash -c "source /opt/ros/humble/setup.bash; source /home/user1/ws_rosdep_broken/kinematics_interface/install/setup.bash; rosdep install --from-paths src --ignore-src -r -y && colcon build --symlink-install" && \
    echo "source /home/user1/ros2_control_ws/install/setup.bash" >> /home/user1/.bashrc
RUN mkdir -p /home/user1/ws_gazebo-ros2-control/src && cd ws_gazebo-ros2-control/src && \
    git clone https://github.com/ros-controls/gazebo_ros2_control.git
RUN cd /home/user1/ws_gazebo-ros2-control && /bin/bash -c "source /opt/ros/humble/setup.bash; source /home/user1/ros2_control_ws/install/setup.bash; colcon build" && \
    echo "source ~/ws_gazebo-ros2-control/install/setup.bash" >> ~/.bashrc


## MoveIt2 related stuff. Really heavy, build it only if needed ---------------------------------------------------------------------------------- ##
# moveit2 apt and demos from sources ######################################################################################################################## BUILD TARGET START ####
FROM ros2_ctrl_default AS moveit2

RUN sudo apt-get update && sudo apt-get install -y ros-humble-moveit*

RUN sudo apt-get install -y python3-colcon-common-extensions python3-colcon-mixin && \
     /bin/bash -c "source /opt/ros/humble/setup.bash; colcon mixin add default https://raw.githubusercontent.com/colcon/colcon-mixin-repository/master/index.yaml" && \
     /bin/bash -c "source /opt/ros/humble/setup.bash; colcon mixin update default"

RUN mkdir -p /home/user1/ws_moveit2/src && cd ws_moveit2/src && \
    git clone https://github.com/ros-planning/moveit2_tutorials -b main && \
    cd moveit2_tutorials && echo "repositories:" > moveit2_tutorials.repos && \
    echo "  moveit_task_constructor:" >> moveit2_tutorials.repos && \
    echo "    type: git" >> moveit2_tutorials.repos && \
    echo "    url: https://github.com/ros-planning/moveit_task_constructor.git" >> moveit2_tutorials.repos && \
    echo "    version: ros2" >> moveit2_tutorials.repos && \
    echo "  rosparam_shortcuts:" >> moveit2_tutorials.repos && \
    echo "    type: git" >> moveit2_tutorials.repos && \
    echo "    url: https://github.com/PickNikRobotics/rosparam_shortcuts" >> moveit2_tutorials.repos && \
    echo "    version: ros2" >> moveit2_tutorials.repos && cd .. && \
    vcs import < moveit2_tutorials/moveit2_tutorials.repos && cd .. && \
    /bin/bash -c "source /opt/ros/humble/setup.bash; source /home/user1/ros2_control_ws/install/setup.bash; colcon build --mixin release"
RUN echo "source ~/ws_moveit2/install/setup.bash" >> ~/.bashrc
