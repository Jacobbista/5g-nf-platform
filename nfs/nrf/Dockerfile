ARG UPSTREAM_TAG=v2.7.5
ARG NF_NAME=nrf

FROM ubuntu:22.04 AS builder
RUN apt-get update && apt-get install -y \
    build-essential git cmake flex bison \
    libsctp-dev libgnutls28-dev libgcrypt-dev libssl-dev \
    libidn11-dev libmongoc-dev libbson-dev libmicrohttpd-dev \
    libnghttp2-dev libcurl4-gnutls-dev libtins-dev libtalloc-dev \
    libyaml-dev python3-pip python3-setuptools python3-wheel \
    meson ninja-build pkg-config \
    && rm -rf /var/lib/apt/lists/*

ARG UPSTREAM_TAG
RUN git clone --depth 1 --branch ${UPSTREAM_TAG} \
    https://github.com/open5gs/open5gs /src/open5gs

COPY patches/ /patches/
RUN cd /src/open5gs && \
    for p in $(ls /patches/*.patch 2>/dev/null | sort); do \
      git apply "$p"; \
    done

ARG NF_NAME
RUN cd /src/open5gs && \
    meson build --prefix=/install && \
    ninja -C build install

FROM ubuntu:22.04
RUN apt-get update && apt-get install -y \
    libsctp1 libgnutls30 libgcrypt20 libssl3 libidn2-0 \
    libmongoc-1.0-0 libbson-1.0-0 libyaml-0-2 libtalloc2 \
    libnghttp2-14 libmicrohttpd12 libtins4.0 \
    libcurl4 curl \
    && rm -rf /var/lib/apt/lists/*

ARG NF_NAME=nrf
COPY --from=builder /install/bin/open5gs-${NF_NAME}d /usr/local/bin/
COPY --from=builder /install/lib/ /usr/local/lib/
COPY --from=builder /install/etc/open5gs/ /install/etc/open5gs/
RUN mkdir -p /install/var/log/open5gs /install/var/run
COPY --from=builder /usr/lib/x86_64-linux-gnu/libcurl-gnutls.so.4* /usr/lib/x86_64-linux-gnu/
COPY --from=builder /usr/lib/x86_64-linux-gnu/libidn.so.12* /usr/lib/x86_64-linux-gnu/
RUN ldconfig

HEALTHCHECK --interval=10s --timeout=5s --start-period=10s --retries=3 \
    CMD curl -sf http://127.0.0.1:9090/ || exit 1

ENTRYPOINT ["/bin/sh"]
