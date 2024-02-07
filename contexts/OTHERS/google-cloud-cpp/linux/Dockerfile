ARG base_img
FROM ${base_img}
LABEL maintainer="Ettore Landini"

ARG google_cloud_version

USER yarp-user
WORKDIR /home/yarp-user/

### Packaging google-cloud-cpp instructions ############################################# START ####
RUN mkdir Downloads
WORKDIR /home/yarp-user/Downloads
RUN sudo apt-get update && \
    sudo apt-get --no-install-recommends install -y apt-transport-https apt-utils \
    automake build-essential cmake ca-certificates curl git \
    gcc g++ libc-ares-dev libc-ares2 libcurl4-openssl-dev libre2-dev \
    libssl-dev m4 make pkg-config tar wget zlib1g-dev

RUN mkdir -p abseil-cpp
RUN cd abseil-cpp && curl -fsSL https://github.com/abseil/abseil-cpp/archive/20230125.3.tar.gz | \
    tar -xzf - --strip-components=1 && \
    sed -i 's/^#define ABSL_OPTION_USE_\(.*\) 2/#define ABSL_OPTION_USE_\1 0/' "absl/base/options.h" && \
    cmake \
      -DCMAKE_BUILD_TYPE=Release \
      -DABSL_BUILD_TESTING=OFF \
      -DBUILD_SHARED_LIBS=yes \
      -S . -B cmake-out && \
    cmake --build cmake-out -- -j ${NCPU:-4} && \
    sudo cmake --build cmake-out --target install -- -j ${NCPU:-4} && \
    sudo ldconfig

RUN mkdir -p protobuf
RUN cd protobuf && \
    curl -fsSL https://github.com/protocolbuffers/protobuf/archive/v23.4.tar.gz | \
    tar -xzf - --strip-components=1 && \
    cmake \
        -DCMAKE_BUILD_TYPE=Release \
        -DBUILD_SHARED_LIBS=yes \
        -Dprotobuf_BUILD_TESTS=OFF \
        -Dprotobuf_ABSL_PROVIDER=package \
        -S . -B cmake-out && \
    cmake --build cmake-out -- -j ${NCPU:-4} && \
    sudo cmake --build cmake-out --target install -- -j ${NCPU:-4} && \
    sudo ldconfig

RUN mkdir -p grpc
RUN cd grpc && \
    curl -fsSL https://github.com/grpc/grpc/archive/v1.56.2.tar.gz | \
        tar -xzf - --strip-components=1 && \
    cmake \
        -DCMAKE_BUILD_TYPE=Release \
        -DBUILD_SHARED_LIBS=yes \
        -DgRPC_INSTALL=ON \
        -DgRPC_BUILD_TESTS=OFF \
        -DgRPC_ABSL_PROVIDER=package \
        -DgRPC_CARES_PROVIDER=package \
        -DgRPC_PROTOBUF_PROVIDER=package \
        -DgRPC_RE2_PROVIDER=package \
        -DgRPC_SSL_PROVIDER=package \
        -DgRPC_ZLIB_PROVIDER=package \
        -S . -B cmake-out && \
    cmake --build cmake-out -- -j ${NCPU:-4} && \
    sudo cmake --build cmake-out --target install -- -j ${NCPU:-4} && \
    sudo ldconfig

RUN mkdir -p crc32c
RUN cd crc32c && \
    curl -fsSL https://github.com/google/crc32c/archive/1.1.2.tar.gz | \
    tar -xzf - --strip-components=1 && \
    cmake \
        -DCMAKE_BUILD_TYPE=Release \
        -DBUILD_SHARED_LIBS=yes \
        -DCRC32C_BUILD_TESTS=OFF \
        -DCRC32C_BUILD_BENCHMARKS=OFF \
        -DCRC32C_USE_GLOG=OFF \
        -S . -B cmake-out && \
    cmake --build cmake-out -- -j ${NCPU:-4} && \
    sudo cmake --build cmake-out --target install -- -j ${NCPU:-4} && \
    sudo ldconfig

RUN mkdir -p json
RUN cd json && \
    curl -fsSL https://github.com/nlohmann/json/archive/v3.11.2.tar.gz | \
    tar -xzf - --strip-components=1 && \
    cmake \
      -DCMAKE_BUILD_TYPE=Release \
      -DBUILD_SHARED_LIBS=yes \
      -DBUILD_TESTING=OFF \
      -DJSON_BuildTests=OFF \
      -S . -B cmake-out && \
    sudo cmake --build cmake-out --target install -- -j ${NCPU:-4} && \
    sudo ldconfig

WORKDIR /home/yarp-user
RUN git clone https://github.com/googleapis/google-cloud-cpp.git

ENV PREFIX="/home/yarp-user/google-cloud-cpp-installed"
#ENV PREFIX="/usr/local"
RUN cd google-cloud-cpp && git checkout ${google_cloud_version} && \
    cmake -S . -B cmake-out \
        -DCMAKE_INSTALL_PREFIX="${PREFIX}" \
        -DCMAKE_POSITION_INDEPENDENT_CODE=ON \
        -DBUILD_TESTING=OFF \
        -DGOOGLE_CLOUD_CPP_ENABLE_EXAMPLES=OFF \
        -DGOOGLE_CLOUD_CPP_ENABLE=__ga_libraries__ && \
    cmake --build cmake-out -- -j "$(nproc)" && \
#    sudo cmake --build cmake-out --target install
    cmake --build cmake-out --target install
ENV google_cloud_cpp_common_DIR=/home/yarp-user/google-cloud-cpp-installed/lib/cmake/google_cloud_cpp_common
ENV google_cloud_cpp_googleapis_DIR=/home/yarp-user/google-cloud-cpp-installed/lib/cmake/google_cloud_cpp_googleapis
ENV google_cloud_cpp_grpc_utils_DIR=/home/yarp-user/google-cloud-cpp-installed/lib/cmake/google_cloud_cpp_grpc_utils
ENV google_cloud_cpp_texttospeech_DIR=/home/yarp-user/google-cloud-cpp-installed/lib/cmake/google_cloud_cpp_texttospeech
ENV google_cloud_cpp_speech_DIR=/home/yarp-user/google-cloud-cpp-installed/lib/cmake/google_cloud_cpp_speech
ENV google_cloud_cpp_dialogflow_cx_DIR=/home/yarp-user/google-cloud-cpp-installed/lib/cmake/google_cloud_cpp_dialogflow_cx
### Packaging google-cloud-cpp instructions ############################################### END ####

ENV GOOGLE_APPLICATION_CREDENTIALS=/home/yarp-user/.config/google-credential/hsp_google.json
ENV BUILD_SUFFIX=build

RUN sudo apt install -y libsqlite3-0 libsqlite3-dev

WORKDIR ${robotology_install_folder}
RUN cd yarp/build && cmake ..
RUN git clone https://github.com/robotology/yarp-devices-google-cloud && \
    cd yarp-devices-google-cloud && mkdir build && cd build && \
    cmake .. && make -j11
ENV YARP_DATA_DIRS=${YARP_DATA_DIRS}:/home/yarp-user/robotology/yarp-devices-google-cloud/build/share/yarp:/home/yarp-user/robotology/yarp-devices-google-cloud/build/share/
