ARG base_img
FROM ${base_img}
LABEL maintainer="Ettore Landini"

# Create user:
ARG username
# USER root
# RUN useradd -l -u 33334 -G sudo -md /home/${username} -s /bin/bash -p ${username} ${username} && \
#     # passwordless sudo for users in the 'sudo' group
#     sed -i.bkp -e 's/%sudo\s\+ALL=(ALL\(:ALL\)\?)\s\+ALL/%sudo ALL=NOPASSWD:ALL/g' /etc/sudoers

# FOLDER ACCESS ###############################################################################
ARG uid
ARG gid

USER root
RUN groupmod -g ${gid} ${username} && \
    usermod -u ${uid} -g ${gid} -G adm,users,sudo,root ${username} && \
    find / -uid 33334 -exec chown -h ${uid} '{}' \; 2> /dev/null || true && \
    find / -gid 33334 -exec chgrp ${gid} '{}' \; 2> /dev/null || true && \
    chown -R ${username}: /home/${username} && \
    mkdir -p /run/user/${uid} && \
    chown ${username}: /run/user/${uid} && \
    chmod 700 /run/user/${uid} && \
    mkdir -p /run/user/${uid}/dconf && \
    chown ${username}: /run/user/${uid}/dconf && \
    chmod 700 /run/user/${uid}/dconf && \
    mkdir -p /home/${username}/.ros && \
    mkdir -p /home/${username}/.config/yarp && \
    mkdir -p /home/${username}/.local/share/yarp && \
    chown -R ${username}: /home/${username}/

###############################################################################################

RUN apt-get update && apt-get install -y terminator

USER ${username}