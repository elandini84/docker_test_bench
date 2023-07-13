FROM elandini84/r1images:tourCore2_ubuntu22.04_humble_devel as ros2_tour_default

LABEL maintainer="Ettore Landini"

USER user1
WORKDIR /home/user1

RUN sudo apt-get update && sudo apt-get install -y nautilus python3-ipython ipython3 ros-humble-rviz*

RUN cd tour-guide-robot && git checkout ros2 && \
    git fetch origin && git pull origin ros2 && cd build && \
    cmake .. && make -j11

WORKDIR /home/user1/robotology

RUN cd cer-sim && git remote add elandini84 https://github.com/elandini84/cer-sim.git && \
    git fetch elandini84 && git checkout feature/ros2_control

RUN echo "source /opt/ros/humble/setup.bash" >> /home/user1/.bashrc

RUN cd yarp-devices-ros2/ && git remote add elandini84 https://github.com/elandini84/yarp-devices-ros2.git && \
     git fetch elandini84 && git checkout feature/ros2_control/forward_pos

RUN cd navigation/ && git remote add elandini84 https://github.com/elandini84/navigation.git && \
     git fetch elandini84

WORKDIR /home/user1

RUN echo "source ~/robotology/yarp-devices-ros2/ros2_interfaces_ws/install/setup.bash" >> .bashRos2ontrol && \
    echo "source ~/robotology/yarp-devices-ros2/ros2_components_ws/install/setup.bash" >> .bashRos2ontrol && \
    echo "source ~/robotology/cer-sim/colcon_ws/install/setup.bash" >> .bashRos2ontrol

# ROS2 launch file migrator
RUN git clone https://github.com/aws-robotics/ros2-launch-file-migrator && \
    cd ros2-launch-file-migrator && pip3 install -e ./ --user
ENV PATH=$PATH:/home/user1/.local/bin

# ros2_control from source ############################################################################################################################### BUILD TARGET START ####
FROM ros2_tour_default AS ros2_ctrl_default

WORKDIR /home/user1

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
    git clone https://github.com/ros-controls/ros2_control_demos && cd ros2_control_demos && git checkout 1007b1dfe44f4146e64b04666a1164adeaf8ffa1 && cd .. && \
    git clone https://github.com/ros2/ros_testing
RUN cd /home/user1/ros2_control_ws && /bin/bash -c "source /opt/ros/humble/setup.bash; source /home/user1/ws_rosdep_broken/kinematics_interface/install/setup.bash; rosdep install --from-paths src --ignore-src -r -y && colcon build --symlink-install" && \
    echo "source /home/user1/ros2_control_ws/install/setup.bash" >> /home/user1/.bashrc
RUN cd robotology/cer-sim/colcon_ws && \
    /bin/bash -c "source /opt/ros/humble/setup.bash; colcon build"
RUN cd robotology/yarp-devices-ros2/ros2_interfaces_ws && \
    /bin/bash -c "source /opt/ros/humble/setup.bash; colcon build" &&\
    cd ../ros2_components_ws && \
    /bin/bash -c "source /opt/ros/humble/setup.bash; source /home/user1/ros2_control_ws/install/setup.bash; source ~/robotology/yarp-devices-ros2/ros2_interfaces_ws/install/setup.bash; colcon build --packages-skip cb_hw_fw_generic" &&\
    cd ../build && \
    /bin/bash -c "source /opt/ros/humble/setup.bash; source /home/user1/ros2_control_ws/install/setup.bash; source ~/robotology/yarp-devices-ros2/ros2_interfaces_ws/install/setup.bash; make -j11"
RUN echo "source ~/.bashRos2ontrol" >> ~/.bashrc
ENV YARP_DATA_DIRS=$YARP_DATA_DIRS:/home/user1/robotology/yarp-devices-ros2/build/share/yarp-devices-ros2
