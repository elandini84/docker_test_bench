FROM randaz81/r1slam:ros2
#ADD key for ros
RUN sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
RUN sudo apt update
RUN sudo apt install python3-pip gedit geany telnet python3-numpy -y
RUN sed -i '$ d' .bashrc
RUN printf "\nalias ros2st=\"source /opt/ros/foxy/setup.bash && . /home/user1/yarp-ros2/ros2_interfaces_ws/install/setup.bash\"\n# Other stuff ---------------------------------------------------- START #\n\nparse_git_branch() {\n     git branch 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/(\\\1)/'\n}\n#export PS1=\"\[\\\e[32m\]\u@\h \[\\\e[32m\]\w \[\\\e[91m\]\\\$(parse_git_branch)\[\\\e[00m\]$ \"\nexport PS1=\"\[\\\033[01;32m\]\u\[\\\033[00m\]:\[\\\033[01;34m\]\W \[\\\e[91m\]\\\$(parse_git_branch)\[\\\e[00m\]$ \"\n# Other stuff ------------------------------------------------------ END #\n" >> /home/user1/.bashrc
RUN git clone https://github.com/aws-robotics/ros2-launch-file-migrator.git
RUN cd ros2-launch-file-migrator && pip3 install -e ./ --user && sudo python3 setup.py install
RUN sudo apt install rsync

WORKDIR /home/user1
RUN mkdir robotology && mv robots-configuration robotology

#YCM
WORKDIR /home/user1/ycm
RUN git pull origin master && cd build && cmake .. && make -j8 && sudo make install

#YARP
WORKDIR /home/user1/yarp
#RUN git remote add elandini84 https://www.github.com/elandini84/yarp.git && git fetch elandini84 && git remote set-url origin --push DISABLED && git checkout --track origin/yarp-3.5 && git pull origin yarp-3.5 && cd build && cmake .. && make -j8 && sudo make install
RUN git remote add elandini84 https://www.github.com/elandini84/yarp.git && git fetch elandini84 && git remote set-url origin --push DISABLED && git checkout --track elandini84/fix_frameTransformStorage_start && git pull elandini84 fix_frameTransformStorage_start && cd build && cmake .. && make -j8 && sudo make install

#ICUBMAIN
WORKDIR /home/user1/icub-main
RUN git fetch origin && git checkout devel && git pull origin devel && cd build && cmake .. && make -j8 && sudo make install

#CER
WORKDIR /home/user1/cer
RUN git fetch origin && git checkout devel && git pull origin devel && cd build && cmake .. && make -j8 && sudo make install

#GAZEBO_YARP_PLUGINS
WORKDIR /home/user1/gazebo-yarp-plugins
RUN git fetch origin && git checkout devel && git pull origin devel && cd build && cmake .. -DGAZEBO_YARP_PLUGINS_DISABLE_IMPLICIT_NETWORK_WRAPPERS=ON && make -j8 && sudo make install

#CER-SIM
WORKDIR /home/user1/cer-sim
RUN git fetch origin && git checkout devel && git pull origin devel

#NAVIGATION
WORKDIR /home/user1/navigation
#RUN git remote add elandini84 https://www.github.com/elandini84/navigation.git && git fetch elandini84 && git remote set-url origin --push DISABLED && git pull origin master && cd build && cmake .. && make -j8 && sudo make install
RUN git remote add elandini84 https://www.github.com/elandini84/navigation.git && git fetch elandini84 && git remote set-url origin --push DISABLED && git checkout --track elandini84/refactor_rosLocalizer_yarp3.5 && git pull elandini84 refactor_rosLocalizer_yarp3.5 && cd build && cmake .. && make -j8 && sudo make install
WORKDIR /home/user1
#git clone https://github.com/robotology-playground/tour-guide-robot.git && cd tour-guide-robot && mkdir build && git checkout --track origin/migration-to-new-yarp && cd aux_modules
