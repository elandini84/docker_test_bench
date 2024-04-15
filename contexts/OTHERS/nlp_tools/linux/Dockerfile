ARG base_img
FROM ${base_img}
LABEL maintainer="Ettore Landini"

ARG user_name

WORKDIR /home/${user_name}/Downloads
# Install openai c++ dependencies
RUN git clone https://github.com/Microsoft/vcpkg.git && \
    cd vcpkg && ./bootstrap-vcpkg.sh -disableMetrics && \
    ./vcpkg integrate install && ./vcpkg install curl[tool] && \
    ./vcpkg install nlohmann-json

    # Install openai c++ community library
RUN git clone https://github.com/D7EAD/liboai.git && \
    cd liboai && \
    git remote add fbrand-new https://github.com/fbrand-new/liboai.git && \
    git fetch fbrand-new v4.0.0-dev && \
    git checkout fbrand-new/v4.0.0-dev && \
    cd liboai && \
    cmake -B build -S . -DCMAKE_TOOLCHAIN_FILE=/home/${user_name}/Downloads/vcpkg/scripts/buildsystems/vcpkg.cmake \
    -DCMAKE_POSITION_INDEPENDENT_CODE=true && \
    cd build && make -j8 && sudo make install

# Configure openai credentials
RUN echo "set -a" >> /home/${user_name}/.bashrc && \
    echo "source ~/.env/config.env" >> /home/${user_name}/.bashrc && \
    echo "set +a" >> /home/${user_name}/.bashrc

# Install yarpGPT
WORKDIR /home/${user_name}/robotology
RUN git clone https://github.com/robotology/yarp-devices-llm -b feat/aux_module && \
    cd yarp-devices-llm && mkdir build && \
    cmake -B build -S . \
    -DCMAKE_TOOLCHAIN_FILE=/home/${user_name}/Downloads/vcpkg/scripts/buildsystems/vcpkg.cmake \
    -DLIBOAI_INSTALL_PATH=/home/${user_name}/Downloads/liboai && cd build && \
    make -j8
ENV YARP_DEVICES_LLM_DIR=/home/${user_name}/robotology/yarp-devices-llm/build
ENV YARP_DATA_DIRS=${YARP_DATA_DIRS}:${YARP_DEVICES_LLM_DIR}/share/yarp:${YARP_DEVICES_LLM_DIR}/share

ARG tour_branch

RUN git clone https://github.com/hsp-iit/tour-guide-robot -b $tour_branch
RUN cd tour-guide-robot && mkdir build && cd build && cmake ..

ENV TOUR_GUIDE_ROBOT_SOURCE_DIR=/home/${user_name}/tour-guide-robot
ENV YARP_DATA_DIRS=${YARP_DATA_DIRS}:${TOUR_GUIDE_ROBOT_SOURCE_DIR}/build/share/tour-guide-robot