FROM ubuntu:22.04 as kevs_baseline
MAINTAINER kevleyski

# Pull in build cross compiler tool dependencies using Advanced Package Tool
ARG DEBIAN_FRONTEND=noninteractive
ENV TZ=Australia/Sydney
RUN apt-get update && apt-get install -y tzdata

RUN set -x \
    && DEBIAN_FRONTEND=noninteractive apt-get -y install wget curl autoconf automake build-essential libass-dev libfreetype6-dev \
                                            libsdl1.2-dev libtheora-dev libtool libva-dev libvdpau-dev libvorbis-dev libxcb1-dev libxcb-shm0-dev \
                                            libxcb-xfixes0-dev pkg-config texinfo zlib1g-dev gettext tcl libssl-dev cmake mercurial unzip git \
                                            libdrm-dev valgrind libpciaccess-dev libxslt1-dev geoip-bin libgeoip-dev zlib1g-dev libpcre3 libpcre3-dev \
                                            libbz2-dev ca-certificates libssl-dev nasm v4l-utils libv4l-dev gtk2.0

COPY . kevs

FROM kevs_baseline_zbar AS kevs_baseline_opencv

# NVIDIA CUDA
#wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/cuda-ubuntu1804.pin
#sudo mv cuda-ubuntu1804.pin /etc/apt/preferences.d/cuda-repository-pin-600
#sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/7fa2af80.pub
#sudo add-apt-repository "deb https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/ /"
#sudo apt-get update
#sudo apt-get -y install cuda

# OpenCV
RUN set -x \
    && cd /kevs/opencv \
    && mkdir -p cmake-build-debug \
    && cd cmake-build-debug \
    && cmake -DCMAKE_BUILD_TYPE=RELEASE \
        -DCMAKE_INSTALL_PREFIX:PATH=/usr \
        -DWITH_FFMPEG=NO \
        -DOPENCV_GENERATE_PKGCONFIG=ON \
        -DBUILD_opencv_world=OFF \
        -DBUILD_opencv_gapi=OFF \
        -DBUILD_DOCS=OFF \
        -DWITH_NVCUVID=OFF \
        -DWITH_CUDA=OFF \
        -DBUILD_TESTS=OFF \
        -DBUILD_PERF_TESTS=OFF \
        -DBUILD_EXAMPLES=OFF \
        -DOPENCV_EXTRA_MODULES_PATH="/kevs/opencv_contrib/modules" .. \
    && make -j$(nproc) \
    && make install

### KJSL dependencies above are known to build to this point

FROM kevs_baseline_opencv AS kevs_clion

# KJSL: CLion remote toolchain deps
RUN set -x \
    && apt-get install -y ssh gcc g++ gdb clang rsync tar python

RUN ( \
    echo 'LogLevel DEBUG2'; \
    echo 'PermitRootLogin yes'; \
    echo 'PasswordAuthentication yes'; \
    echo 'Subsystem sftp /usr/lib/openssh/sftp-server'; \
  ) > /etc/ssh/sshd_config_test_clion \
  && mkdir /run/sshd

RUN useradd -m kevdev \
  && yes kevdev | passwd kevdev

RUN usermod -s /bin/bash kevdev

CMD ["/usr/sbin/sshd", "-D", "-e", "-f", "/etc/ssh/sshd_config_test_clion"]

FROM kevs_clion AS kevs_deploy

