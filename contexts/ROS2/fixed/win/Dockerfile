#FROM randaz81/r1slam:ros2
FROM randaz81/r1slam:ros2galactic
#ADD key for ros
#RUN sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
RUN sudo apt update -qq
RUN sudo apt install python3-pip gedit geany telnet ipython3 -y
RUN sed -i '$ d' .bashrc
RUN printf "\nalias ros2st=\"source /opt/ros/galactic/setup.bash && . /home/user1/yarp-ros2/ros2_interfaces_ws/install/setup.bash\"\n# Other stuff ---------------------------------------------------- START #\n\nparse_git_branch() {\n     git branch 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/(\\\1)/'\n}\n#export PS1=\"\[\\\e[32m\]\u@\h \[\\\e[32m\]\w \[\\\e[91m\]\\\$(parse_git_branch)\[\\\e[00m\]$ \"\nexport PS1=\"\[\\\033[01;32m\]\u\[\\\033[00m\]:\[\\\033[01;34m\]\W \[\\\e[91m\]\\\$(parse_git_branch)\[\\\e[00m\]$ \"\n# Other stuff ------------------------------------------------------ END #\n" >> /home/user1/.bashrc
RUN git clone https://github.com/aws-robotics/ros2-launch-file-migrator.git
RUN cd ros2-launch-file-migrator && pip3 install -e ./ --user && sudo python3 setup.py install
RUN sudo apt install rsync
RUN sudo apt update
# Install extra packages
RUN sudo apt-get remove --purge -y \
        ros-foxy-* \
        && \
    sudo apt-get install -y \
        ros-galactic-ros1-bridge \
        ros-galactic-nav2-amcl \
        ros-galactic-navigation2 \
        ros-galactic-cyclonedds \
        ros-galactic-control-msgs \
        ros-galactic-depth-image-proc \
        ros-galactic-image-proc \
        ros-galactic-image-publisher \
        ros-galactic-image-view \
        ros-galactic-joint-state-publisher \
        ros-galactic-test-msgs

#YCM
WORKDIR /home/user1/ycm
RUN git pull origin master && \
    cd build && \
    cmake .. && \
    make -j8 && \
    sudo make install

#YARP
WORKDIR /home/user1/yarp
RUN git remote add elandini84 https://www.github.com/elandini84/yarp.git && \
    git fetch elandini84 && \
    git remote set-url origin --push DISABLED && \
    git checkout master && \
    git pull origin master && \
    cd build && \
    cmake .. && \
    make -j8 && \
    sudo make install

#ICUBMAIN
WORKDIR /home/user1/icub-main
RUN git fetch origin && \
    git checkout devel && \
    git pull origin devel && \
    cd build && \
    cmake .. && \
    make -j8 && \
    sudo make install

#CER
WORKDIR /home/user1/cer
RUN git fetch origin && \
    git checkout devel && \
    git pull origin devel && \
    cd build && cmake .. && \
    make -j8 && \
    sudo make install

#GAZEBO_YARP_PLUGINS
WORKDIR /home/user1/gazebo-yarp-plugins
RUN git fetch origin && \
    git checkout devel && \
    git pull origin devel && \
    cd build && \
    cmake .. -DGAZEBO_YARP_PLUGINS_DISABLE_IMPLICIT_NETWORK_WRAPPERS=ON && \
    make -j8 && \
    sudo make install

#CER-SIM
#WORKDIR /home/user1/cer-sim
#RUN git fetch origin && git checkout ros2 && git pull origin ros2
RUN cd /home/user1/cer-sim && \
    git fetch --all --prune && \
    git checkout ros2 && \
    git remote set-url origin --push DISABLED && \
    git reset --hard origin/ros2 && \
    cd /home/user1/cer-sim/catkin_ws/src && \
    bash -c " \
        source /opt/ros/noetic/setup.bash && \
        catkin_init_workspace" && \
    cd /home/user1/cer-sim/catkin_ws && \
    bash -c " \
        source /opt/ros/noetic/setup.bash && \
        catkin_make" && \
    cd /home/user1/cer-sim/colcon_ws && \
    bash -c " \
        source /opt/ros/galactic/setup.bash && \
        colcon build"

#NAVIGATION
WORKDIR /home/user1/navigation
#RUN git remote add elandini84 https://www.github.com/elandini84/navigation.git && git fetch elandini84 && git remote set-url origin --push DISABLED && git pull origin master && cd build && cmake .. && make -j8 && sudo make install
RUN git remote add elandini84 https://www.github.com/elandini84/navigation.git && \
    git fetch elandini84 && \
    git remote set-url origin --push DISABLED && \
    git pull origin master && \
    cd build && \
    cmake .. && \
    make -j8 && \
    sudo make install

#YARP-ROS2  ---- Mine ends at line 99
WORKDIR /home/user1/yarp-ros2
RUN git remote add elandini84 https://www.github.com/elandini84/yarp-ros2.git && \
    git fetch elandini84 && \
    git remote set-url origin --push DISABLED && \
    git pull origin master && \
    cd ros2_interfaces_ws && \
    bash -c " \
        source /opt/ros/galactic/setup.bash && \
        colcon build --packages-select map2d_nws_ros2_msgs" && \
    cd .. && \
    mkdir build && \
    cd build && \
    bash -c " \
        source /opt/ros/galactic/setup.bash && \
        source /home/user1/yarp-ros2/ros2_interfaces_ws/install/setup.bash && \
        cmake .. -DCMAKE_BUILD_TYPE=Debug -DCMAKE_EXPORT_COMPILE_COMMANDS:BOOL=ON" && \
    make -j4 && \
    sudo make install && \
    ln -s compile_commands.json ..

WORKDIR /home/user1
