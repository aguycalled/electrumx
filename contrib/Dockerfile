# example of Dockerfile that installs spesmilo electrumx 1.16.0
# ENV variables can be overriden on the `docker run` command

FROM python:3.9.4-buster AS builder

WORKDIR /
ADD https://github.com/aguycalled/electrumx/archive/1.14.0.tar.gz /
RUN tar zxvf *.tar.gz

# Install the libs needed by rocksdb (including development headers)
RUN apt-get update \
    && apt-get -y --no-install-recommends install \
        librocksdb-dev libsnappy-dev libbz2-dev libz-dev liblz4-dev \
    && rm -rf /var/lib/apt/lists/*

RUN python -m venv venv \
    && venv/bin/pip install --no-cache-dir e-x[rapidjson,rocksdb]==1.16.0


FROM python:3.9.4-slim-buster

# Install the libs needed by rocksdb (no development headers or statics)
RUN apt-get update \
    && apt-get -y --no-install-recommends install \
        librocksdb5.17 libsnappy1v5 libbz2-1.0 zlib1g liblz4-1 \
    && rm -rf /var/lib/apt/lists/*

ENV SERVICES="tcp://:40001"
ENV COIN=Navcoin
ENV DB_DIRECTORY=/db
ENV DAEMON_URL="http://username:password@hostname:port/"
ENV ALLOW_ROOT=true
ENV DB_ENGINE=rocksdb
ENV MAX_SEND=10000000
ENV BANDWIDTH_UNIT_COST=50000
ENV CACHE_MB=2000
ENV COST_SOFT_LIMIT=0
ENV COST_HARD_LIMIT=0

WORKDIR /usr/src/app
COPY --from=builder /usr/src/app .

VOLUME /var/lib/electrumx

RUN mkdir -p "$DB_DIRECTORY" && ulimit -n 1048576

CMD ["/usr/src/app/venv/bin/python", "/usr/src/app/venv/bin/electrumx_server"]

# build it with eg.: `docker build -t electrumx .`
# run it with eg.:
# `docker run -d --net=host -v /home/electrumx/db/:/var/lib/electrumx -e DAEMON_URL="http://youruser:yourpass@localhost:8332" -e REPORT_SERVICES=tcp://example.com:50001 electrumx`
# for a clean shutdown, send TERM signal to the running container eg.: `docker kill --signal="TERM" CONTAINER_ID`
