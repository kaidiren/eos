FROM eosio/builder as builder
ARG branch=master
ARG symbol=EOS

RUN apt-get update -y && apt-get install -y libuv1 libjemalloc1
RUN wget http://downloads.datastax.com/cpp-driver/ubuntu/18.04/cassandra/v2.11.0/cassandra-cpp-driver_2.11.0-1_amd64.deb
RUN apt-get install ./cassandra-cpp-driver_2.11.0-1_amd64.deb -y
RUN rm cassandra-cpp-driver_2.11.0-1_amd64.deb
RUN wget http://downloads.datastax.com/cpp-driver/ubuntu/18.04/cassandra/v2.11.0/cassandra-cpp-driver-dev_2.11.0-1_amd64.deb
RUN apt-get install ./cassandra-cpp-driver-dev_2.11.0-1_amd64.deb -y
RUN rm cassandra-cpp-driver-dev_2.11.0-1_amd64.deb

RUN apt-get update -y && apt-get install -y libcurl4-openssl-dev libusb-1.0-0-dev
RUN git clone -b $branch https://github.com/kaidiren/eos_with_scylladb_cassandra_history_plugin.git eos --recursive \
    && cd eos && echo "$branch:$(git rev-parse HEAD)" > /etc/eosio-version \
    && cmake -H. -B"/tmp/build" -GNinja -DCMAKE_BUILD_TYPE=Release -DWASM_ROOT=/opt/wasm -DCMAKE_CXX_COMPILER=clang++ \
       -DCMAKE_C_COMPILER=clang -DCMAKE_INSTALL_PREFIX=/tmp/build -DBUILD_MONGO_DB_PLUGIN=false -DCORE_SYMBOL_NAME=$symbol \
    && cmake --build /tmp/build --target install


FROM ubuntu:18.04

RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get -y install openssl ca-certificates && rm -rf /var/lib/apt/lists/*
COPY --from=builder /usr/local/lib/* /usr/local/lib/
COPY --from=builder /tmp/build/bin /opt/eosio/bin
COPY --from=builder /eos/Docker/config.ini /
COPY --from=builder /etc/eosio-version /etc
COPY --from=builder /eos/Docker/nodeosd.sh /opt/eosio/bin/nodeosd.sh
ENV EOSIO_ROOT=/opt/eosio
RUN chmod +x /opt/eosio/bin/nodeosd.sh
ENV LD_LIBRARY_PATH /usr/local/lib
ENV PATH /opt/eosio/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
