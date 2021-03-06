# BUILD redisfab/redisearch-${OSNICK}:${VERSION}-${ARCH}

ARG REDIS_VER=5.0.7

# OSNICK=bionic|stretch|buster
ARG OSNICK=buster

# OS=debian:buster-slim|debian:stretch-slim|ubuntu:bionic
ARG OS=debian:buster-slim

# ARCH=x64|arm64v8|arm32v7
ARG ARCH=x64

ARG GIT_DESCRIBE_VERSION

#----------------------------------------------------------------------------------------------
FROM redisfab/redis:${REDIS_VER}-${ARCH}-${OSNICK} AS redis
FROM ${OS} AS builder

ARG OSNICK
ARG OS
ARG ARCH
ARG REDIS_VER
ARG GIT_DESCRIBE_VERSION

RUN echo "Building for ${OSNICK} (${OS}) for ${ARCH}"

WORKDIR /build
COPY --from=redis /usr/local/ /usr/local/

ADD . /build

RUN ./deps/readies/bin/getpy2
RUN ./system-setup.py
RUN /usr/local/bin/redis-server --version
RUN make fetch SHOW=1
RUN make build SHOW=1 CMAKE_ARGS="-DGIT_DESCRIBE_VERSION=${GIT_DESCRIBE_VERSION}"

# ARG PACK=0
ARG TEST=0

# RUN if [ "$PACK" = "1" ]; then make pack; fi
RUN if [ "$TEST" = "1" ]; then TEST= make test; fi

#----------------------------------------------------------------------------------------------
FROM redisfab/redis:${REDIS_VER}-${ARCH}-${OSNICK}

ARG OSNICK
ARG OS
ARG ARCH
ARG REDIS_VER
# ARG PACK

WORKDIR /data

ENV LIBDIR /usr/lib/redis/modules
RUN mkdir -p "$LIBDIR";

COPY --from=builder /build/build/redisearch.so  "$LIBDIR"

CMD ["redis-server", "--loadmodule", "/usr/lib/redis/modules/redisearch.so"]
