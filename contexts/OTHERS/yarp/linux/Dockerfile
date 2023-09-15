ARG base_img
FROM $base_img
LABEL maintainer="Ettore Landini"

ARG yarp_version

# Non-interactive installation mode
ENV DEBIAN_FRONTEND=noninteractive
ENV DOCKER_UPDATED_ON=10_07_2023

# Install essentials
RUN apt-get update && apt-get install -y apt-utils software-properties-common sudo psmisc lsb-release protobuf-compiler libatlas-base-dev \
    tmux nano geany vim wget curl build-essential git gitk cmake cmake-curses-gui autoconf xserver-xorg-video-dummy xserver-xorg-legacy \
    net-tools terminator libjpeg-dev ffmpeg apt-transport-https ca-certificates gnupg libace-dev ycm-cmake-modules locales curl zip unzip tar \
    python3-setuptools python3-pip iproute2 python3-tornado swig lsof iftop iputils-ping gdb bash-completion btop mlocate && rm -rf /var/lib/apt/lists/*

RUN pip3 install numpy bpytop

RUN sed -i 's/allowed_users=console/allowed_users=anybody/' /etc/X11/Xwrapper.config
COPY ./common/xorg.conf /etc/X11/xorg.conf

# Install yarp dependencies
RUN echo "deb http://www.icub.org/ubuntu `lsb_release -cs` contrib/science" > /etc/apt/sources.list.d/icub.list
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 57A5ACB6110576A6
RUN apt-get update && apt-get install -y icub-common libopencv-dev libsqlite3-dev && sudo rm -rf /var/lib/apt/lists/*

# Create user: yarp-user
USER root
RUN useradd -l -u 33334 -G sudo -md /home/yarp-user -s /bin/bash -p yarp-user yarp-user && \
    # passwordless sudo for users in the 'sudo' group
    sed -i.bkp -e 's/%sudo\s\+ALL=(ALL\(:ALL\)\?)\s\+ALL/%sudo ALL=NOPASSWD:ALL/g' /etc/sudoers

# Set ENV
USER yarp-user
RUN mkdir /home/yarp-user/robotology
ENV robotology_install_user yarp-user
ENV yarp_user_home /home/$robotology_install_user
ENV robotology_install_folder $yarp_user_home/robotology


# FOLDER ACCESS ###############################################################################
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


# Build ycm
USER $robotology_install_user
WORKDIR $robotology_install_folder
RUN git clone https://github.com/robotology/ycm.git -b master
RUN cd ycm && mkdir build && cd build && \
    cmake .. \
    -DCMAKE_BUILD_TYPE=Release && \
    make -j11
ENV YCM_DIR=/home/yarp-user/robotology/ycm/build

# Build YARP
USER $robotology_install_user
WORKDIR $robotology_install_folder
RUN git clone https://github.com/robotology/yarp.git -b $yarp_version
RUN cd yarp && mkdir build && cd build && \
    cmake .. \
    -DCMAKE_BUILD_TYPE=Release \
    -DYARP_COMPILE_libYARP_math=ON \
    -DYARP_COMPILE_GUIS=ON \
    -DYARP_COMPILE_DEVICE_PLUGINS=ON \
    -DENABLE_yarpcar_mjpeg=ON \
    -DENABLE_yarpcar_human=ON \
    -DENABLE_yarpcar_segmentationimage=ON \
    -DENABLE_yarpcar_portmonitor=ON \
    -DENABLE_yarpmod_portaudio=ON \
    -DENABLE_yarpmod_portaudioPlayer=ON \
    -DENABLE_yarpmod_portaudioRecorder=ON \
    -DENABLE_yarpmod_fakeBattery=ON  \
    -DENABLE_yarpmod_controlboardwrapper2=ON \
    -DENABLE_yarpmod_laserFromDepth=ON \
    -DENABLE_yarpmod_laserFromExternalPort=ON \
    -DENABLE_yarpmod_laserFromDepth=ON \
    -DENABLE_yarpmod_laserFromPointCloud=ON \
    -DENABLE_yarpmod_laserFromRosTopic=ON \
    -DENABLE_yarppm_soundfilter_resample=ON \
    -DENABLE_yarppm_bottle_compression_zlib=ON \
    -DENABLE_yarppm_depthimage_compression_zlib=ON \
    -DENABLE_yarppm_image_compression_ffmpeg=ON \
    -DENABLE_yarppm_depthimage_to_mono=ON \
    -DENABLE_yarppm_depthimage_to_rgb=ON && \
    make -j11 && cmake .. -DALLOW_IDL_GENERATION=ON && \
    make -j11 && cmake .. -DALLOW_IDL_GENERATION=OFF
ENV YARP_ROOT=$robotology_install_folder/yarp
ENV YARP_DIR=$robotology_install_folder/yarp/build

# YARP bindings
USER $robotology_install_user
WORKDIR $robotology_install_folder
RUN cd yarp && cd bindings &&  mkdir build && cd build && cmake .. -DCREATE_PYTHON=ON && make -j11

# Install YARP completion
RUN sudo ln -s /usr/local/share/bash-completion/completions/yarp /usr/share/bash-completion/completions

# Set environmental variables
USER $robotology_install_user
RUN echo "PS1='\[\e]0;\u \w\a\]\[\033[01;32m\]\u\[\033[00m\] \[\033[01;34m\]\w\[\033[00m\] \$ '" >> /home/$robotology_install_user/.bashrc
ENV PATH=$PATH:$YARP_DIR/bin
ENV DISPLAY=:1
ENV YARP_DATA_DIRS=$YARP_DIR/share/yarp
ENV LD_LIBRARY_PATH=$robotology_install_folder/yarp/build/lib/yarp/
ENV YARP_COLORED_OUTPUT=1
ENV QT_X11_NO_MITSHM=1
ENV PYTHONPATH=$PYTHONPATH:/home/yarp-user/robotology/yarp/bindings/build/lib/python3/

# Manage yarp port
EXPOSE 10000/tcp 10000/udp