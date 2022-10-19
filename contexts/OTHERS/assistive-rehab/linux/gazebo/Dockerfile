FROM icubteamcode/fdg-basic:v2022.05.2
LABEL maintainer="Ettore Landini"

# Non-interactive installation mode
ENV DEBIAN_FRONTEND=noninteractive

# Install essentials
RUN apt-get update && apt-get install -y git sudo cmake build-essential libssl-dev zlib1g-dev terminator locate nautilus apt-utils software-properties-common sudo psmisc lsb-release \
    tmux nano geany vim wget curl build-essential gitk cmake-curses-gui autoconf xserver-xorg-video-dummy xserver-xorg-legacy \
    net-tools terminator libjpeg-dev ffmpeg apt-transport-https ca-certificates gnupg libace-dev locales \
    python3-setuptools python3-pip iproute2 python3-tornado swig lsof iftop iputils-ping gdb bash-completion mlocate && rm -rf /var/lib/apt/lists/*
