ARG base_img
FROM ${base_img}
LABEL maintainer="Ettore Landini"

ARG google_cloud_version

USER yarp-user
WORKDIR /home/yarp-user/
RUN git clone https://github.com/googleapis/google-cloud-cpp.git -b ${google_cloud_version}

WORKDIR /home/yarp-user/google-cloud-cpp
RUN git -C $HOME clone https://github.com/microsoft/vcpkg.git
ENV VCPKG_ROOT=$HOME/vcpkg
RUN $HOME/vcpkg/bootstrap-vcpkg.sh
RUN cmake -S . -B cmake-out/ -DCMAKE_TOOLCHAIN_FILE=$HOME/vcpkg/scripts/buildsystems/vcpkg.cmake
RUN cmake --build cmake-out -- -j $(nproc)
