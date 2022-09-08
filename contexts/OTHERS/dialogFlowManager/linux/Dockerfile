FROM ubuntu:22.04
LABEL maintainer="Ettore Landini"

# Non-interactive installation mode
ENV DEBIAN_FRONTEND=noninteractive

# Install essentials
RUN apt-get update && apt-get install -y git sudo cmake build-essential libssl-dev zlib1g-dev terminator locate nautilus apt-utils software-properties-common sudo psmisc lsb-release \
    tmux nano geany vim wget curl build-essential gitk cmake-curses-gui autoconf xserver-xorg-video-dummy xserver-xorg-legacy \
    net-tools terminator libjpeg-dev ffmpeg apt-transport-https ca-certificates gnupg libace-dev ycm-cmake-modules locales \
    python3-setuptools python3-pip iproute2 python3-tornado swig lsof iftop iputils-ping gdb bash-completion btop mlocate && rm -rf /var/lib/apt/lists/*

# Create user: user1
USER root
RUN useradd -l -u 33334 -G sudo -md /home/user1 -s /bin/bash -p user1 user1 && \
    # passwordless sudo for users in the 'sudo' group
    sed -i.bkp -e 's/%sudo\s\+ALL=(ALL\(:ALL\)\?)\s\+ALL/%sudo ALL=NOPASSWD:ALL/g' /etc/sudoers

USER user1
WORKDIR /home/user1
RUN mkdir git_repos && cd git_repos && git clone --recursive -b v1.43.2 https://github.com/grpc/grpc.git && cd grpc && mkdir build && \
cd build && cmake .. -DBUILD_SHARED_LIBS=ON -DCMAKE_BUILD_TYPE=Release \
-DgRPC_SSL_PROVIDER=package -DgRPC_ZLIB_PROVIDER=package && sudo make -j4 install && sudo ldconfig
RUN cd git_repos && git clone https://github.com/googleapis/googleapis.git && cd googleapis && git checkout f78745822aadedb3c165d51372732d672b634db2 && sudo make -j4

#cd git_repos && git clone https://github.com/hsp-iit/dialogflow-interface.git && \
#cd dialogflow-interface && mkdir build && cd build && \
#cmake .. -G "Unix Makefiles" -Dgoogleapis_INCLUDE_DIR=/home/user1/git_repos/googleapis/gens -DCMAKE_CXX_COMPILER=/usr/bin/g++ -DCMAKE_C_COMPILER=/usr/bin/gcc -DCMAKE_MAKE_PROGRAM=/usr/bin/make; make -j11
