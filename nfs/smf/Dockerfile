# Baseline tested with v2.7.5 (version from original manual build).
# v2.7.5 is the pinned default; update upstream-versions.json to change it.
ARG UPSTREAM_TAG=v2.7.5
ARG NF_NAME=smf

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

# WebUI (Node.js) is intentionally excluded. Subscriber management and
# monitoring are handled by the 5g-k3s-kubedge-testbed dashboard.

COPY patches/ /patches/
RUN cd /src/open5gs && \
    for p in $(ls /patches/*.patch 2>/dev/null | sort); do \
      git apply "$p"; \
    done

ARG NF_NAME
RUN cd /src/open5gs && \
    meson build --prefix=/install && \
    ninja -C build install && \
    ldd /install/bin/open5gs-${NF_NAME}d

FROM ubuntu:22.04
RUN apt-get update && apt-get install -y \
    libsctp1 libgnutls30 libgcrypt20 libssl3 libidn2-0 \
    libmongoc-1.0-0 libbson-1.0-0 libyaml-0-2 libtalloc2 \
    libnghttp2-14 libmicrohttpd12 libtins4.0 \
    libcurl4 curl \
    && rm -rf /var/lib/apt/lists/*

ARG NF_NAME=smf
COPY --from=builder /install/bin/open5gs-${NF_NAME}d /usr/local/bin/
COPY --from=builder /install/lib/ /usr/local/lib/
COPY --from=builder /install/etc/open5gs/ /etc/open5gs/
# libs with no matching runtime package on Ubuntu 22.04
COPY --from=builder /usr/lib/x86_64-linux-gnu/libcurl-gnutls.so.4* /usr/lib/x86_64-linux-gnu/
COPY --from=builder /usr/lib/x86_64-linux-gnu/libidn.so.12* /usr/lib/x86_64-linux-gnu/
RUN ldconfig

# All Open5GS NFs expose a management HTTP server on port 9090.
# The healthcheck polls it; a 200 response means the NF is operational.
HEALTHCHECK --interval=10s --timeout=5s --start-period=10s --retries=3 \
    CMD curl -sf http://127.0.0.1:9090/ || exit 1

ENTRYPOINT ["/bin/sh"]
