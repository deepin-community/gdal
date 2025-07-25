# syntax=docker/dockerfile:1

##
# osgeo/gdal:ubuntu-full

# This file is available at the option of the licensee under:
# Public domain
# or licensed under MIT (LICENSE.TXT) Copyright 2019 Even Rouault <even.rouault@spatialys.com>

ARG BASE_IMAGE=ubuntu:24.04
ARG TARGET_BASE_IMAGE=ubuntu:24.04

FROM $BASE_IMAGE AS builder

# Derived from osgeo/proj by Howard Butler <howard@hobu.co>
LABEL maintainer="Even Rouault <even.rouault@spatialys.com>"

ENV HOME="/root"

ARG TARGET_ARCH=
RUN echo ${TARGET_ARCH}
COPY ./bh-set-envvars.sh /buildscripts/bh-set-envvars.sh

RUN rm -f /etc/apt/apt.conf.d/docker-clean \
    && echo 'Binary::apt::APT::Keep-Downloaded-Packages "true";' > /etc/apt/apt.conf.d/keep-cache \
    && echo 'Acquire::Retries "10";' > /etc/apt/apt.conf.d/80-retries
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    . /buildscripts/bh-set-envvars.sh \
    && if test "${TARGET_ARCH}" != ""; then \
    rm -f /etc/apt/sources.list /etc/apt/sources.list.d/ubuntu.sources \
    && echo "deb [arch=amd64] http://us.archive.ubuntu.com/ubuntu/ noble main restricted universe" >> /etc/apt/sources.list \
    && echo "deb [arch=amd64] http://us.archive.ubuntu.com/ubuntu/ noble-updates main restricted universe" >> /etc/apt/sources.list \
    && echo "deb [arch=amd64] http://us.archive.ubuntu.com/ubuntu/ noble-backports main restricted universe" >> /etc/apt/sources.list \
    && echo "deb [arch=amd64] http://security.ubuntu.com/ubuntu noble-security main restricted universe" >> /etc/apt/sources.list \
    && echo "deb [arch=${TARGET_ARCH}] http://ports.ubuntu.com/ubuntu-ports/ noble main restricted universe" >> /etc/apt/sources.list \
    && echo "deb [arch=${TARGET_ARCH}] http://ports.ubuntu.com/ubuntu-ports/ noble-updates main restricted universe" >> /etc/apt/sources.list \
    && echo "deb [arch=${TARGET_ARCH}] http://ports.ubuntu.com/ubuntu-ports/ noble-security main restricted universe" >> /etc/apt/sources.list \
    && dpkg --add-architecture ${TARGET_ARCH} \
    && apt-get update -y \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y g++-13-${GCC_ARCH}-linux-gnu \
    && ln -s ${GCC_ARCH}-linux-gnu-gcc-13 /usr/bin/${GCC_ARCH}-linux-gnu-gcc \
    && ln -s ${GCC_ARCH}-linux-gnu-g++-13 /usr/bin/${GCC_ARCH}-linux-gnu-g++; \
    fi

# Setup build env for PROJ
USER root
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    . /buildscripts/bh-set-envvars.sh \
    && apt-get update -y \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y --fix-missing --no-install-recommends \
            build-essential ca-certificates ccache\
            git make ninja-build cmake wget unzip libtool automake \
            zlib1g-dev${APT_ARCH_SUFFIX} libsqlite3-dev${APT_ARCH_SUFFIX} pkg-config sqlite3 libcurl4-openssl-dev${APT_ARCH_SUFFIX} \
            libtiff-dev${APT_ARCH_SUFFIX} \
            patchelf \
            rsync

ARG JAVA_VERSION=17
# Setup build env for GDAL
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    . /buildscripts/bh-set-envvars.sh \
    && apt-get update -y \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y --fix-missing --no-install-recommends \
       libopenjp2-7-dev${APT_ARCH_SUFFIX} libcairo2-dev${APT_ARCH_SUFFIX} \
       python3-dev${APT_ARCH_SUFFIX} python3-numpy${APT_ARCH_SUFFIX} python3-setuptools${APT_ARCH_SUFFIX} \
       libpng-dev${APT_ARCH_SUFFIX} libjpeg-dev${APT_ARCH_SUFFIX} libgif-dev${APT_ARCH_SUFFIX} liblzma-dev${APT_ARCH_SUFFIX} libgeos-dev${APT_ARCH_SUFFIX} \
       curl libxml2-dev${APT_ARCH_SUFFIX} libexpat-dev${APT_ARCH_SUFFIX} libxerces-c-dev${APT_ARCH_SUFFIX} \
       libnetcdf-dev${APT_ARCH_SUFFIX} libpoppler-dev${APT_ARCH_SUFFIX} libpoppler-private-dev${APT_ARCH_SUFFIX} \
       libspatialite-dev${APT_ARCH_SUFFIX} librasterlite2-dev${APT_ARCH_SUFFIX} swig ant libhdf4-alt-dev${APT_ARCH_SUFFIX} libhdf5-serial-dev${APT_ARCH_SUFFIX} \
       libfreexl-dev${APT_ARCH_SUFFIX} unixodbc-dev${APT_ARCH_SUFFIX}  mdbtools-dev${APT_ARCH_SUFFIX} libwebp-dev${APT_ARCH_SUFFIX} \
       liblcms2-2 libpcre3-dev${APT_ARCH_SUFFIX} libcrypto++-dev${APT_ARCH_SUFFIX} libfyba-dev${APT_ARCH_SUFFIX} \
       libkml-dev${APT_ARCH_SUFFIX} libmysqlclient-dev${APT_ARCH_SUFFIX} libogdi-dev${APT_ARCH_SUFFIX} \
       libcfitsio-dev${APT_ARCH_SUFFIX} openjdk-"$JAVA_VERSION"-jdk${APT_ARCH_SUFFIX} libzstd-dev${APT_ARCH_SUFFIX} \
       libpq-dev${APT_ARCH_SUFFIX} libssl-dev${APT_ARCH_SUFFIX} libboost-dev${APT_ARCH_SUFFIX} \
       autoconf automake bash-completion libarmadillo-dev${APT_ARCH_SUFFIX} \
       libopenexr-dev${APT_ARCH_SUFFIX} libheif-dev${APT_ARCH_SUFFIX} \
       libdeflate-dev${APT_ARCH_SUFFIX} libblosc-dev${APT_ARCH_SUFFIX} liblz4-dev${APT_ARCH_SUFFIX} libbz2-dev${APT_ARCH_SUFFIX} \
       libbrotli-dev${APT_ARCH_SUFFIX} \
       libarchive-dev${APT_ARCH_SUFFIX} \
       libaec-dev${APT_ARCH_SUFFIX} \
       libavif-dev${APT_ARCH_SUFFIX}

ARG RSYNC_REMOTE
ARG WITH_CCACHE

# Build libkea
ARG KEA_VERSION=1.5.2
RUN --mount=type=cache,id=ubuntu-full-kealib,target=$HOME/.cache \
    . /buildscripts/bh-set-envvars.sh \
    && wget -q https://github.com/ubarsc/kealib/archive/kealib-${KEA_VERSION}.zip \
    && unzip -q kealib-${KEA_VERSION}.zip \
    && rm -f kealib-${KEA_VERSION}.zip \
    && if [ -n "${RSYNC_REMOTE:-}" ]; then \
        echo "Downloading cache..."; \
        rsync -ra "${RSYNC_REMOTE}/kealib/${GCC_ARCH}/" "$HOME/.cache/"; \
        echo "Finished"; \
    fi \
    && if [ -n "${WITH_CCACHE:-}" ]; then \
        ccache -M 100M; \
    fi \
    && cd kealib-kealib-${KEA_VERSION} \
    && cmake . -G Ninja $CCACHE_PARAM -DBUILD_SHARED_LIBS=ON -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_INSTALL_PREFIX=/usr -DHDF5_INCLUDE_DIR=/usr/include/hdf5/serial \
        -DHDF5_LIB_PATH=/usr/lib/${GCC_ARCH}-linux-gnu/hdf5/serial -DLIBKEA_WITH_GDAL=OFF \
    && ninja \
    && DESTDIR="/build_thirdparty" ninja install \
    && ninja install \
    && if [ -n "${RSYNC_REMOTE:-}" ]; then \
        echo "Uploading cache..."; \
        rsync -ra --delete "$HOME/.cache/" "${RSYNC_REMOTE}/kealib/${GCC_ARCH}/"; \
        echo "Finished"; \
    fi \
    && if [ -n "${WITH_CCACHE:-}" ]; then \
        ccache -s; \
    fi \
    && cd .. \
    && rm -rf kealib-kealib-${KEA_VERSION} \
    && for i in /build_thirdparty/usr/lib/*; do strip -s $i 2>/dev/null || /bin/true; done \
    && for i in /build_thirdparty/usr/bin/*; do strip -s $i 2>/dev/null || /bin/true; done

# Build mongo-c-driver
ARG MONGO_C_DRIVER_VERSION=1.24.4
RUN --mount=type=cache,id=ubuntu-full-mongo-c-driver,target=$HOME/.cache \
    . /buildscripts/bh-set-envvars.sh \
    && mkdir mongo-c-driver \
    && wget -q https://github.com/mongodb/mongo-c-driver/releases/download/${MONGO_C_DRIVER_VERSION}/mongo-c-driver-${MONGO_C_DRIVER_VERSION}.tar.gz -O - \
        | tar xz -C mongo-c-driver --strip-components=1 \
    && if [ -n "${RSYNC_REMOTE:-}" ]; then \
        echo "Downloading cache..."; \
        rsync -ra "${RSYNC_REMOTE}/mongo-c-driver/${GCC_ARCH}/" "$HOME/.cache/"; \
        echo "Finished"; \
    fi \
    && if [ -n "${WITH_CCACHE:-}" ]; then \
        ccache -M 100M; \
    fi \
    && cd mongo-c-driver \
    && mkdir build_cmake \
    && cd build_cmake \
    && cmake .. -G Ninja $CCACHE_PARAM -DCMAKE_INSTALL_PREFIX=/usr -DENABLE_TESTS=NO -DCMAKE_BUILD_TYPE=Release \
    && ninja \
    && DESTDIR="/build_thirdparty" ninja install \
    && ninja install \
    && if [ -n "${RSYNC_REMOTE:-}" ]; then \
        echo "Uploading cache..."; \
        rsync -ra --delete "$HOME/.cache/" "${RSYNC_REMOTE}/mongo-c-driver/${GCC_ARCH}/"; \
        echo "Finished"; \
    fi \
    && if [ -n "${WITH_CCACHE:-}" ]; then \
        ccache -s; \
    fi \
    && cd ../.. \
    && rm -rf mongo-c-driver \
    && rm /build_thirdparty/usr/lib/${GCC_ARCH}-linux-gnu/*.a \
    && for i in /build_thirdparty/usr/lib/${GCC_ARCH}-linux-gnu/*; do strip -s $i 2>/dev/null || /bin/true; done \
    && for i in /build_thirdparty/usr/bin/*; do strip -s $i 2>/dev/null || /bin/true; done

# Build mongocxx
ARG MONGOCXX_VERSION=3.8.1
RUN --mount=type=cache,id=ubuntu-full-mongo-cxx-driver,target=$HOME/.cache \
    . /buildscripts/bh-set-envvars.sh \
    && mkdir mongocxx \
    && wget -q https://github.com/mongodb/mongo-cxx-driver/archive/r${MONGOCXX_VERSION}.tar.gz -O - \
        | tar xz -C mongocxx --strip-components=1 \
    && if [ -n "${RSYNC_REMOTE:-}" ]; then \
        echo "Downloading cache..."; \
        rsync -ra "${RSYNC_REMOTE}/mongo-cxx-driver/${GCC_ARCH}/" "$HOME/.cache/"; \
        echo "Finished"; \
    fi \
    && if [ -n "${WITH_CCACHE:-}" ]; then \
        ccache -M 100M; \
    fi \
    && cd mongocxx \
    && mkdir build_cmake \
    && cd build_cmake \
    && cmake .. -G Ninja $CCACHE_PARAM -DCMAKE_INSTALL_PREFIX=/usr -DBSONCXX_POLY_USE_BOOST=ON -DENABLE_TESTS=OFF -DMONGOCXX_ENABLE_SLOW_TESTS=NO -DCMAKE_BUILD_TYPE=Release -DBUILD_VERSION=${MONGOCXX_VERSION} \
    && ninja \
    && DESTDIR="/build_thirdparty" ninja install \
    && ninja install \
    && if [ -n "${RSYNC_REMOTE:-}" ]; then \
        echo "Uploading cache..."; \
        rsync -ra --delete "$HOME/.cache/" "${RSYNC_REMOTE}/mongo-cxx-driver/${GCC_ARCH}/"; \
        echo "Finished"; \
    fi \
    && if [ -n "${WITH_CCACHE:-}" ]; then \
        ccache -s; \
    fi \
    && cd ../.. \
    && rm -rf mongocxx \
    && for i in /build_thirdparty/usr/lib/${GCC_ARCH}-linux-gnu/*; do strip -s $i 2>/dev/null || /bin/true; done \
    && for i in /build_thirdparty/usr/bin/*; do strip -s $i 2>/dev/null || /bin/true; done

# Build tiledb
ARG TILEDB_VERSION=2.23.0
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    . /buildscripts/bh-set-envvars.sh \
    && apt-get update -y \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y \
        libspdlog-dev${APT_ARCH_SUFFIX} libmagic-dev${APT_ARCH_SUFFIX}

COPY ./tiledb-FindLZ4_EP.cmake.patch /buildscripts/tiledb-FindLZ4_EP.cmake.patch
COPY ./tiledb-FindOpenSSL_EP.cmake.patch /buildscripts/tiledb-FindOpenSSL_EP.cmake.patch
# Commit from TileDB 2.27.
COPY ./tiledb-cmake-ccache.patch /buildscripts/tiledb-cmake-ccache.patch
RUN --mount=type=cache,id=ubuntu-full-tiledb,target=$HOME/.cache \
    . /buildscripts/bh-set-envvars.sh \
    && if [ -n "${RSYNC_REMOTE:-}" ]; then \
        echo "Downloading cache..."; \
        rsync -ra "${RSYNC_REMOTE}/tiledb/${GCC_ARCH}/" "$HOME/.cache/"; \
        echo "Finished"; \
    fi \
    && if [ -n "${WITH_CCACHE:-}" ]; then \
        export CCACHE_PARAM="-DTILEDB_CCACHE=ON"; \
        ccache -M 100M; \
    fi \
    && mkdir tiledb \
    && wget -q https://github.com/TileDB-Inc/TileDB/archive/${TILEDB_VERSION}.tar.gz -O - \
        | tar xz -C tiledb --strip-components=1 \
    && cd tiledb \
    && patch -p0 < /buildscripts/tiledb-FindLZ4_EP.cmake.patch \
    && patch -p0 < /buildscripts/tiledb-FindOpenSSL_EP.cmake.patch \
    && patch -p1 < /buildscripts/tiledb-cmake-ccache.patch \
    && mkdir build_cmake \
    && cd build_cmake \
    && cmake .. -G Ninja $CCACHE_PARAM -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr -DTILEDB_WERROR=OFF -DTILEDB_SUPERBUILD=OFF  -DTILEDB_TESTS=OFF -DCOMPILER_SUPPORTS_AVX2=FALSE  -DOPENSSL_INCLUDE_DIR=/usr/include -DOPENSSL_CRYPTO_LIBRARY=/usr/lib/${GCC_ARCH}-linux-gnu/libcrypto.so -DOPENSSL_SSL_LIBRARY=/usr/lib/${GCC_ARCH}-linux-gnu/libssl.so \
    && ninja \
    && DESTDIR="/build_thirdparty" ninja install \
    && ninja install \
    && if [ -n "${RSYNC_REMOTE:-}" ]; then \
        echo "Uploading cache..."; \
        rsync -ra --delete "$HOME/.cache/" "${RSYNC_REMOTE}/tiledb/${GCC_ARCH}/"; \
        echo "Finished"; \
    fi \
    && if [ -n "${WITH_CCACHE:-}" ]; then \
        ccache -s; \
    fi \
    && cd ../.. \
    && rm -rf tiledb \
    && for i in /build_thirdparty/usr/lib/${GCC_ARCH}-linux-gnu/*; do strip -s $i 2>/dev/null || /bin/true; done \
    && for i in /build_thirdparty/usr/bin/*; do strip -s $i 2>/dev/null || /bin/true; done

# Build libOpenDRIVE
ARG OPENDRIVE_VERSION=0.6.0-gdal
RUN --mount=type=cache,id=ubuntu-full-libopendrive,target=$HOME/.cache \
    . /buildscripts/bh-set-envvars.sh \
    && if test "${OPENDRIVE_VERSION}" != ""; then ( \
    wget -q https://github.com/DLR-TS/libOpenDRIVE/archive/refs/tags/${OPENDRIVE_VERSION}.tar.gz \
    && tar xzf ${OPENDRIVE_VERSION}.tar.gz \
    && rm -f ${OPENDRIVE_VERSION}.tar.gz \
    && if [ -n "${RSYNC_REMOTE:-}" ]; then \
        echo "Downloading cache..."; \
        rsync -ra "${RSYNC_REMOTE}/libopendrive/${GCC_ARCH}/" "$HOME/.cache/"; \
        echo "Finished"; \
    fi \
    && if [ -n "${WITH_CCACHE:-}" ]; then \
        ccache -M 100M; \
    fi \
    && cd libOpenDRIVE-${OPENDRIVE_VERSION} \
    && cmake . -G Ninja $CCACHE_PARAM -DBUILD_SHARED_LIBS=ON -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/usr/ \
    && ninja \
    && ninja install \
    && if [ -n "${RSYNC_REMOTE:-}" ]; then \
        echo "Uploading cache..."; \
        rsync -ra --delete "$HOME/.cache/" "${RSYNC_REMOTE}/libopendrive/${GCC_ARCH}/"; \
        echo "Finished"; \
    fi \
    && if [ -n "${WITH_CCACHE:-}" ]; then \
        ccache -s; \
    fi \
    && mkdir -p /build_thirdparty/usr/lib \
    && cp -P /usr/lib/libOpenDrive*.so* /build_thirdparty/usr/lib \
    && for i in /build_thirdparty/usr/lib/*; do strip -s $i 2>/dev/null || /bin/true; done \
    && cd .. \
    && rm -rf libOpenDRIVE-${OPENDRIVE_VERSION} \
    ); fi

#Build File Geodatabase

ARG WITH_FILEGDB=
RUN . /buildscripts/bh-set-envvars.sh \
  && if echo "$WITH_FILEGDB" | grep -Eiq "^(y(es)?|1|true)$"  ; then ( \
  wget -q https://github.com/Esri/file-geodatabase-api/raw/master/FileGDB_API_1.5.2/FileGDB_API-RHEL7-64gcc83.tar.gz \
  && tar -xzf FileGDB_API-RHEL7-64gcc83.tar.gz \
  && chown -R root:root FileGDB_API-RHEL7-64gcc83 \
  && mv FileGDB_API-RHEL7-64gcc83 /usr/local/FileGDB_API \
  && rm -rf /usr/local/FileGDB_API/lib/libstdc++* \
  && cp /usr/local/FileGDB_API/lib/* /build_thirdparty/usr/lib/x86_64-linux-gnu \
  && cp /usr/local/FileGDB_API/include/* /usr/include \
  && rm -rf FileGDB_API-RHEL7-64gcc83.tar.gz \
  ) ; fi

# Build libqb3
RUN --mount=type=cache,id=ubuntu-full-libqb3,target=$HOME/.cache \
    . /buildscripts/bh-set-envvars.sh \
    && git clone https://github.com/lucianpls/QB3.git \
    && if [ -n "${RSYNC_REMOTE:-}" ]; then \
        echo "Downloading cache..."; \
        rsync -ra "${RSYNC_REMOTE}/libqb3/${GCC_ARCH}/" "$HOME/.cache/"; \
        echo "Finished"; \
    fi \
    && if [ -n "${WITH_CCACHE:-}" ]; then \
        ccache -M 100M; \
    fi \
    && cd QB3/QB3lib \
    && mkdir build \
    && cd build \
    && cmake .. -G Ninja $CCACHE_PARAM ${CMAKE_EXTRA_ARGS} -DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_BUILD_TYPE=Release \
    && ninja \
    && ninja install \
    && DESTDIR="/build_thirdparty" ninja install \
    && if [ -n "${RSYNC_REMOTE:-}" ]; then \
        echo "Uploading cache..."; \
        rsync -ra --delete "$HOME/.cache/" "${RSYNC_REMOTE}/libqb3/${GCC_ARCH}/"; \
        echo "Finished"; \
    fi \
    && if [ -n "${WITH_CCACHE:-}" ]; then \
        ccache -s; \
    fi \
    && cd ../../.. \
    && rm -rf QB3

ARG WITH_PDFIUM=yes
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
  if echo "$WITH_PDFIUM" | grep -Eiq "^(y(es)?|1|true)$"  ; then ( \
  wget -q https://github.com/rouault/pdfium_build_gdal_3_10/releases/download/pdfium_6677_v1/install-ubuntu2004-rev6677.tar.gz \
  && tar -xzf install-ubuntu2004-rev6677.tar.gz \
  && chown -R root:root install \
  && mv install/lib/* /usr/lib/ \
  && mv install/include/* /usr/include/ \
  && rm -rf install-ubuntu2004-rev6677.tar.gz install \
  && apt-get update -y \
  && apt-get install -y --fix-missing --no-install-recommends liblcms2-dev${APT_ARCH_SUFFIX} \
  ) ; fi

# Build libjxl
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    . /buildscripts/bh-set-envvars.sh \
    && apt-get update -y \
    && apt-get install -y --fix-missing --no-install-recommends libgflags-dev${APT_ARCH_SUFFIX}
RUN --mount=type=cache,id=ubuntu-full-libjxl,target=$HOME/.cache \
    . /buildscripts/bh-set-envvars.sh \
    && git clone https://github.com/libjxl/libjxl.git --recursive \
    && if [ -n "${RSYNC_REMOTE:-}" ]; then \
        echo "Downloading cache..."; \
        rsync -ra "${RSYNC_REMOTE}/libjxl/${GCC_ARCH}/" "$HOME/.cache/"; \
        echo "Finished"; \
    fi \
    && if [ -n "${WITH_CCACHE:-}" ]; then \
        ccache -M 100M; \
    fi \
    && cd libjxl \
    && mkdir build \
    && cd build \
    && cmake -G Ninja $CCACHE_PARAM -DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_BUILD_TYPE=Release -DJPEGXL_ENABLE_JPEGLI_LIBJPEG=OFF -DBUILD_TESTING=OFF -DJPEGXL_ENABLE_TOOLS=OFF -DJPEGXL_ENABLE_BENCHMARK=OFF .. \
    && ninja \
    && ninja install \
    && DESTDIR="/build_thirdparty" ninja install \
    && if [ -n "${RSYNC_REMOTE:-}" ]; then \
        echo "Uploading cache..."; \
        rsync -ra --delete "$HOME/.cache/" "${RSYNC_REMOTE}/libjxl/${GCC_ARCH}/"; \
        echo "Finished"; \
    fi \
    && if [ -n "${WITH_CCACHE:-}" ]; then \
        ccache -s; \
    fi \
    && rm -f /lib/${GCC_ARCH}-linux-gnu/libjxl*.a \
    && rm -f /build_thirdparty/lib/${GCC_ARCH}-linux-gnu/libjxl*.a \
    && cd ../.. \
    && rm -rf libjxl

# Install Arrow C++
ARG ARROW_VERSION=16.1.0-1
# ARROW_SOVERSION to be updated in the "Build final image" section too
ARG ARROW_SOVERSION=1600
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    . /buildscripts/bh-set-envvars.sh \
    && apt-get update -y \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y -V ca-certificates lsb-release wget \
    && curl -LO -fsS https://apache.jfrog.io/artifactory/arrow/$(lsb_release --id --short | tr 'A-Z' 'a-z')/apache-arrow-apt-source-latest-$(lsb_release --codename --short).deb \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y -V ./apache-arrow-apt-source-latest-$(lsb_release --codename --short).deb \
    && apt-get update \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y -V libarrow${ARROW_SOVERSION}${APT_ARCH_SUFFIX} \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y -V libparquet${ARROW_SOVERSION}${APT_ARCH_SUFFIX} \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y -V libarrow-dataset${ARROW_SOVERSION}${APT_ARCH_SUFFIX} \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y -V libarrow-dev${APT_ARCH_SUFFIX}=${ARROW_VERSION} \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y -V libparquet-dev${APT_ARCH_SUFFIX}=${ARROW_VERSION} \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y -V libarrow-acero-dev${APT_ARCH_SUFFIX}=${ARROW_VERSION} \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y -V libarrow-dataset-dev${APT_ARCH_SUFFIX}=${ARROW_VERSION} \
    && rm apache-arrow-apt-source-latest-$(lsb_release --codename --short).deb

ARG WITH_DEBUG_SYMBOLS=no

# Refresh grids
ARG PROJ_DATUMGRID_LATEST_LAST_MODIFIED
ARG PROJ_INSTALL_PREFIX=/usr/local
COPY ./bh-proj.sh /buildscripts/bh-proj.sh
# We build PROJ a first time, just to build the projsync utility if grids need
# to be refreshed. We will rebuild just afterwards, but depending on the
# PROJ_VERSION argument
# We want 2 separate steps for incremental builds where grids are refreshed
# only when the content of cdn.proj.org has changed, independently if PROJ master
# itself has changed.
RUN --mount=type=cache,id=ubuntu-full-proj,target=$HOME/.cache \
    . /buildscripts/bh-set-envvars.sh \
    && DESTDIR=/build_tmp_proj /buildscripts/bh-proj.sh \
     && LD_LIBRARY_PATH=/build_tmp_proj/usr/local/lib /build_tmp_proj/usr/local/bin/projsync --target-dir /tmp/proj_grids --all \
     && rm -rf /build_tmp_proj

# Build PROJ
ARG PROJ_VERSION=master
RUN --mount=type=cache,id=ubuntu-full-proj,target=$HOME/.cache \
    . /buildscripts/bh-set-envvars.sh \
    && /buildscripts/bh-proj.sh

# Build GDAL
ARG GDAL_VERSION=master
ARG GDAL_RELEASE_DATE
ARG GDAL_BUILD_IS_RELEASE
ARG GDAL_REPOSITORY=OSGeo/gdal

COPY ./bh-gdal.sh /buildscripts/bh-gdal.sh
RUN --mount=type=cache,id=ubuntu-full-gdal,target=$HOME/.cache \
    . /buildscripts/bh-set-envvars.sh \
    && /buildscripts/bh-gdal.sh

# Build final image
FROM $TARGET_BASE_IMAGE AS runner

USER root
RUN date
ARG JAVA_VERSION=17
ARG ARROW_SOVERSION=1600

ARG TARGET_ARCH=
# Update distro
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    export DEBIAN_FRONTEND=noninteractive \
    && apt-get update -y \
    && apt-get upgrade -y \
    # PROJ dependencies
    && apt-get install -y \
        libsqlite3-0 libtiff6 libcurl4 \
        wget curl unzip ca-certificates \
    # GDAL dependencies
    && apt-get install -y \
        bash-completion libopenjp2-7 libcairo2 python3-numpy \
        libpng16-16 libjpeg-turbo8 libgif7 liblzma5 libgeos3.12.1 libgeos-c1v5 \
        libxml2 libexpat1 \
        libxerces-c3.2 libnetcdf-c++4-1 netcdf-bin libpoppler134 libspatialite8 librasterlite2-1 gpsbabel \
        libhdf4-0-alt libhdf5-103-1 libhdf5-cpp-103-1 poppler-utils libfreexl1 unixodbc mdbtools libwebp7 \
        liblcms2-2 libpcre3 libcrypto++8 libfyba0 \
        libkmlbase1 libkmlconvenience1 libkmldom1 libkmlengine1 libkmlregionator1 libkmlxsd1 \
        libmysqlclient21 libogdi4.1 libcfitsio10 \
        libzstd1 bash bash-completion libpq5 \
        libarmadillo12 libpython3.12 libopenexr-3-1-30 libheif1 \
        libdeflate0 libblosc1 liblz4-1 \
        libbrotli1 \
        libarchive13 \
        libaec0 \
        libavif16 \
        libspdlog1.12 libmagic1t64 \
        python-is-python3 \
        # pil for antialias option of gdal2tiles
        python3-pil \
    # Install JRE with --no-install-recommends, otherwise it draws default-jre, which draws systemd, which fails to install when running the arm64v8/ubuntu:24.04 image on a 64bit host
    && apt-get install -y --no-install-recommends openjdk-"$JAVA_VERSION"-jre \
    # Worksround OGDI packaging bug for Ubuntu 24.04
    && ln -s /usr/lib/$(uname -p)-linux-gnu/ogdi/4.1/libvrf.so /usr/lib/$(uname -p)-linux-gnu \
    # Install Arrow C++
    && apt-get install -y -V ca-certificates lsb-release wget \
    && curl -LO -fsS https://apache.jfrog.io/artifactory/arrow/$(lsb_release --id --short | tr 'A-Z' 'a-z')/apache-arrow-apt-source-latest-$(lsb_release --codename --short).deb \
    && apt-get install -y -V ./apache-arrow-apt-source-latest-$(lsb_release --codename --short).deb \
    && apt-get update \
    && apt-get install -y -V libarrow${ARROW_SOVERSION} \
    && apt-get install -y -V libparquet${ARROW_SOVERSION} \
    && apt-get install -y -V libarrow-dataset${ARROW_SOVERSION}

# Attempt to order layers starting with less frequently varying ones

COPY --from=builder  /build_thirdparty/usr/ /usr/

ARG PROJ_INSTALL_PREFIX=/usr/local
COPY --from=builder  /tmp/proj_grids/* ${PROJ_INSTALL_PREFIX}/share/proj/

COPY --from=builder  /build${PROJ_INSTALL_PREFIX}/share/proj/ ${PROJ_INSTALL_PREFIX}/share/proj/
COPY --from=builder  /build${PROJ_INSTALL_PREFIX}/include/ ${PROJ_INSTALL_PREFIX}/include/
COPY --from=builder  /build${PROJ_INSTALL_PREFIX}/bin/ ${PROJ_INSTALL_PREFIX}/bin/
COPY --from=builder  /build${PROJ_INSTALL_PREFIX}/lib/ ${PROJ_INSTALL_PREFIX}/lib/

COPY --from=builder  /build/usr/share/bash-completion/ /usr/share/bash-completion/
COPY --from=builder  /build/usr/share/java /usr/share/java
COPY --from=builder  /build/usr/share/gdal/ /usr/share/gdal/
COPY --from=builder  /build/usr/include/ /usr/include/
COPY --from=builder  /build_gdal_python/usr/ /usr/
COPY --from=builder  /build_gdal_version_changing/usr/ /usr/

RUN ldconfig

RUN echo "source /usr/share/bash-completion/bash_completion" >> /root/.bashrc

CMD ["/bin/bash", "-l"]
