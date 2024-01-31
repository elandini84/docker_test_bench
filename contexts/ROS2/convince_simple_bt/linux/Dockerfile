ARG base_img
FROM ${base_img}

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

RUN sudo apt-get update && sudo apt-get install -y nautilus python3-ipython ipython3 libqt6scxml6 qt6-base-dev qt6-scxml-dev libqt6scxml6-bin

RUN pip install reelay websocket-client jedi prompt_toolkit

RUN /bin/bash -c "source /opt/ros/iron/setup.bash && \
    cd /home/user1 && git clone https://github.com/autonomy-and-verification-uol/ROSMonitoring.git -b ros2 && \
    cd ROSMonitoring/generator/ros2_devel"

RUN /bin/bash -c "source /opt/ros/iron/setup.bash && \
    cd /home/user1 && git clone https://github.com/ste93/convince_simple_bts.git -b ros2 && \
    cd /home/user1/convince_simple_bts/ros2prova/bt_interfaces/ && colcon build && source install/setup.bash && \
    cd /home/user1/convince_simple_bts/ros2prova/other_interfaces && colcon build && source install/setup.bash && \
    cd /home/user1/convince_simple_bts/ros2prova/bt_nodes && colcon build && source install/setup.bash && \
    cd /home/user1/convince_simple_bts/ros2prova/alarm_battery_low_skill && colcon build && source install/setup.bash && \
    cd /home/user1/convince_simple_bts/ros2prova/battery_drainer_component && colcon build && source install/setup.bash && \
    cd /home/user1/convince_simple_bts/ros2prova/battery_drainer_skill && colcon build && source install/setup.bash && \
    cd /home/user1/convince_simple_bts/ros2prova/battery_level_skill && colcon build && source install/setup.bash && \
    cd /home/user1/convince_simple_bts/ros2prova/bt_executable && colcon build && source install/setup.bash"

RUN echo "source /home/user1/convince_simple_bts/ros2prova/bt_interfaces/install/setup.bash" >> ~/.bashrc && \
    echo "source /home/user1/convince_simple_bts/ros2prova/other_interfaces/install/setup.bash" >> ~/.bashrc && \
    echo "source /home/user1/convince_simple_bts/ros2prova/bt_nodes/install/setup.bash" >> ~/.bashrc && \
    echo "source /home/user1/convince_simple_bts/ros2prova/alarm_battery_low_skill/install/setup.bash" >> ~/.bashrc && \
    echo "source /home/user1/convince_simple_bts/ros2prova/battery_drainer_component/install/setup.bash" >> ~/.bashrc && \
    echo "source /home/user1/convince_simple_bts/ros2prova/battery_drainer_skill/install/setup.bash" >> ~/.bashrc && \
    echo "source /home/user1/convince_simple_bts/ros2prova/battery_level_skill/install/setup.bash" >> ~/.bashrc && \
    echo "source /home/user1/convince_simple_bts/ros2prova/bt_executable/install/setup.bash" >> ~/.bashrc