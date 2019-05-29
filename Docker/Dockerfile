FROM ubuntu:18.04

LABEL author="xiaobo <peterwillcn@gmail.com>" maintainer="Xiaobo <peterwillcn@gmail.com> Huang-Ming Huang <huangh@objectcomputing.com>" version="0.1.1" \
  description="This is a base image for building eosio/eos"

RUN echo 'APT::Install-Recommends 0;' >> /etc/apt/apt.conf.d/01norecommends \
  && echo 'APT::Install-Suggests 0;' >> /etc/apt/apt.conf.d/01norecommends \
  && apt-get update \
  && DEBIAN_FRONTEND=noninteractive apt-get install -y sudo wget curl net-tools ca-certificates unzip gnupg

RUN echo "deb http://apt.llvm.org/bionic/ llvm-toolchain-bionic main" >> /etc/apt/sources.list.d/llvm.list \
  && wget -O - https://apt.llvm.org/llvm-snapshot.gpg.key|sudo apt-key add - \
  && apt-get update \
  && DEBIAN_FRONTEND=noninteractive apt-get install -y git-core automake autoconf libtool build-essential pkg-config libtool \
     mpi-default-dev libicu-dev python-dev python3-dev libbz2-dev zlib1g-dev libssl-dev libgmp-dev \
     clang-4.0 lldb-4.0 lld-4.0 llvm-4.0-dev libclang-4.0-dev ninja-build \
  && rm -rf /var/lib/apt/lists/*

RUN update-alternatives --install /usr/bin/clang clang /usr/lib/llvm-4.0/bin/clang 400 \
  && update-alternatives --install /usr/bin/clang++ clang++ /usr/lib/llvm-4.0/bin/clang++ 400

RUN wget https://cmake.org/files/v3.9/cmake-3.9.6-Linux-x86_64.sh \
    && bash cmake-3.9.6-Linux-x86_64.sh --prefix=/usr/local --exclude-subdir --skip-license \
    && rm cmake-3.9.6-Linux-x86_64.sh

ENV CC clang
ENV CXX clang++

RUN wget https://dl.bintray.com/boostorg/release/1.67.0/source/boost_1_67_0.tar.bz2 -O - | tar -xj \
    && cd boost_1_67_0 \
    && ./bootstrap.sh --prefix=/usr/local \
    && echo 'using clang : 4.0 : clang++-4.0 ;' >> project-config.jam \
    && ./b2 -d0 -j$(nproc) --with-thread --with-date_time --with-system --with-filesystem --with-program_options \
       --with-signals --with-serialization --with-chrono --with-test --with-context --with-locale --with-coroutine --with-iostreams toolset=clang link=static install \
    && cd .. && rm -rf boost_1_67_0

RUN wget https://github.com/mongodb/mongo-c-driver/releases/download/1.10.2/mongo-c-driver-1.10.2.tar.gz -O - | tar -xz \
    && cd mongo-c-driver-1.10.2 \
    && mkdir cmake-build && cd cmake-build \
    && cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local -DENABLE_BSON=ON \
		-DENABLE_SSL=OPENSSL -DENABLE_AUTOMATIC_INIT_AND_CLEANUP=OFF -DENABLE_STATIC=ON .. \
    && make -j$(nproc) \
    && make install \
    && cd ../../ && rm -rf mongo-c-driver-1.10.2

RUN git clone --depth 1 --single-branch --branch release_40 https://github.com/llvm-mirror/llvm.git \
    && git clone --depth 1 --single-branch --branch release_40 https://github.com/llvm-mirror/clang.git llvm/tools/clang \
    && cd llvm \
    && cmake -H. -Bbuild -GNinja -DCMAKE_INSTALL_PREFIX=/opt/wasm -DLLVM_TARGETS_TO_BUILD= -DLLVM_EXPERIMENTAL_TARGETS_TO_BUILD=WebAssembly -DCMAKE_BUILD_TYPE=Release  \
    && cmake --build build --target install \
    && cd .. && rm -rf llvm

RUN git clone --depth 1 -b releases/v3.3 https://github.com/mongodb/mongo-cxx-driver \
    && cd mongo-cxx-driver/build \
    && cmake -DBUILD_SHARED_LIBS=OFF -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local .. \
    && make -j$(nproc) \
    && make install \
    && cd ../../ && rm -rf mongo-cxx-driver

ARG branch=master
ARG symbol=EOS

RUN chmod 777 -R /tmp
RUN apt-get update -y && apt-get install -y libuv1 libjemalloc1
RUN wget http://downloads.datastax.com/cpp-driver/ubuntu/18.04/cassandra/v2.11.0/cassandra-cpp-driver_2.11.0-1_amd64.deb \
    && apt-get install ./cassandra-cpp-driver_2.11.0-1_amd64.deb -y && rm cassandra-cpp-driver_2.11.0-1_amd64.deb
RUN wget http://downloads.datastax.com/cpp-driver/ubuntu/18.04/cassandra/v2.11.0/cassandra-cpp-driver-dev_2.11.0-1_amd64.deb \
    && apt-get install ./cassandra-cpp-driver-dev_2.11.0-1_amd64.deb -y && rm cassandra-cpp-driver-dev_2.11.0-1_amd64.deb

RUN apt-get update -y && apt-get install -y libcurl4-openssl-dev libusb-1.0-0-dev
RUN git clone -b $branch https://github.com/kaidiren/eos_with_scylladb_cassandra_history_plugin.git eos --recursive \
    && cd eos && echo "$branch:$(git rev-parse HEAD)" > /etc/eosio-version \
    && cmake -H. -B"/tmp/build" -GNinja -DCMAKE_BUILD_TYPE=Release -DWASM_ROOT=/opt/wasm -DCMAKE_CXX_COMPILER=clang++ \
       -DCMAKE_C_COMPILER=clang -DCMAKE_INSTALL_PREFIX=/tmp/build -DBUILD_MONGO_DB_PLUGIN=false -DCORE_SYMBOL_NAME=$symbol \
    && cmake --build /tmp/build --target install

RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get -y install openssl ca-certificates && rm -rf /var/lib/apt/lists/*
RUN mkdir -p /opt/eosio/bin
RUN cp -r /tmp/build/bin /opt/eosio
RUN cp -r /eos/Docker/config.ini /
RUN cp -r /eos/Docker/nodeosd.sh /opt/eosio/bin/nodeosd.sh
RUN rm -rf /tmp/build && rm -rf /eos
ENV EOSIO_ROOT=/opt/eosio
RUN chmod +x /opt/eosio/bin/nodeosd.sh
ENV LD_LIBRARY_PATH /usr/local/lib
ENV PATH /opt/eosio/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
