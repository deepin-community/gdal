# syntax=docker/dockerfile:1

##
# osgeo/gdal:ubuntu-small

# This file is available at the option of the licensee under:
# Public domain
# or licensed under MIT (LICENSE.TXT) Copyright 2019 Even Rouault <even.rouault@spatialys.com>

ARG PROJ_INSTALL_PREFIX=/usr/local
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
            build-essential ca-certificates ccache \
            git make ninja-build cmake wget unzip libtool automake \
            zlib1g-dev${APT_ARCH_SUFFIX} libsqlite3-dev${APT_ARCH_SUFFIX} pkg-config sqlite3 libcurl4-openssl-dev${APT_ARCH_SUFFIX} \
            libtiff-dev${APT_ARCH_SUFFIX} \
            patchelf \
            rsync

# Setup build env for GDAL
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    . /buildscripts/bh-set-envvars.sh \
    && apt-get update -y \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y --fix-missing --no-install-recommends \
       python3-dev${APT_ARCH_SUFFIX} python3-numpy${APT_ARCH_SUFFIX} python3-setuptools${APT_ARCH_SUFFIX} \
       libjpeg-dev${APT_ARCH_SUFFIX} libgeos-dev${APT_ARCH_SUFFIX} \
       libexpat-dev${APT_ARCH_SUFFIX} libxerces-c-dev${APT_ARCH_SUFFIX} \
       libwebp-dev${APT_ARCH_SUFFIX} libpng-dev${APT_ARCH_SUFFIX} \
       libdeflate-dev${APT_ARCH_SUFFIX} \
       libzstd-dev${APT_ARCH_SUFFIX} bash zip curl \
       libpq-dev${APT_ARCH_SUFFIX} libssl-dev${APT_ARCH_SUFFIX} libopenjp2-7-dev${APT_ARCH_SUFFIX} \
       libspatialite-dev${APT_ARCH_SUFFIX} \
       autoconf automake sqlite3 bash-completion swig

ARG PROJ_INSTALL_PREFIX
ARG PROJ_DATUMGRID_LATEST_LAST_MODIFIED
RUN \
    mkdir -p /build_projgrids/${PROJ_INSTALL_PREFIX}/share/proj \
    && curl -LO -fsS http://download.osgeo.org/proj/proj-datumgrid-latest.zip \
    && unzip -q -j -u -o proj-datumgrid-latest.zip  -d /build_projgrids/${PROJ_INSTALL_PREFIX}/share/proj \
    && rm -f *.zip

ARG RSYNC_REMOTE
ARG WITH_CCACHE

# Build PROJ
ARG PROJ_VERSION=master
RUN --mount=type=cache,id=ubuntu-small-proj,target=$HOME/.cache \
    . /buildscripts/bh-set-envvars.sh \
    && mkdir proj \
    && wget -q https://github.com/OSGeo/PROJ/archive/${PROJ_VERSION}.tar.gz -O - \
        | tar xz -C proj --strip-components=1 \
    && export PROJ_DB_CACHE_PARAM="" \
    && cd proj \
    && if test "${RSYNC_REMOTE:-}" != ""; then \
        echo "Downloading cache..."; \
        rsync -ra ${RSYNC_REMOTE}/proj/${GCC_ARCH}/ $HOME/.cache/; \
        echo "Finished"; \
    fi \
    && if [ -n "${WITH_CCACHE:-}" ]; then \
        export CC="ccache ${GCC_ARCH}-linux-gnu-gcc"; \
        export CXX="ccache ${GCC_ARCH}-linux-gnu-g++"; \
        export PROJ_DB_CACHE_PARAM="-DPROJ_DB_CACHE_DIR=$HOME/.cache"; \
        ccache -M 100M; \
    fi \
    && CFLAGS='-DPROJ_RENAME_SYMBOLS -O2' CXXFLAGS='-DPROJ_RENAME_SYMBOLS -DPROJ_INTERNAL_CPP_NAMESPACE -O2' \
        cmake . \
	-G Ninja \
        -DBUILD_SHARED_LIBS=ON \
        -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_INSTALL_PREFIX=${PROJ_INSTALL_PREFIX} \
        -DBUILD_TESTING=OFF \
        $PROJ_DB_CACHE_PARAM \
    && ninja \
    && DESTDIR="/build" ninja install \
    && if test "${RSYNC_REMOTE:-}" != ""; then \
        echo "Uploading cache..."; \
        rsync -ra --delete $HOME/.cache/ ${RSYNC_REMOTE}/proj/${GCC_ARCH}/; \
        echo "Finished"; \
    fi \
    && if [ -n "${WITH_CCACHE:-}" ]; then \
        ccache -s; \
        unset CC; \
        unset CXX; \
    fi \
    && cd .. \
    && rm -rf proj \
    && PROJ_SO=$(readlink -f /build${PROJ_INSTALL_PREFIX}/lib/libproj.so | awk 'BEGIN {FS="libproj.so."} {print $2}') \
    && PROJ_SO_FIRST=$(echo $PROJ_SO | awk 'BEGIN {FS="."} {print $1}') \
    && mv /build${PROJ_INSTALL_PREFIX}/lib/libproj.so.${PROJ_SO} /build${PROJ_INSTALL_PREFIX}/lib/libinternalproj.so.${PROJ_SO} \
    && ln -s libinternalproj.so.${PROJ_SO} /build${PROJ_INSTALL_PREFIX}/lib/libinternalproj.so.${PROJ_SO_FIRST} \
    && ln -s libinternalproj.so.${PROJ_SO} /build${PROJ_INSTALL_PREFIX}/lib/libinternalproj.so \
    && rm /build${PROJ_INSTALL_PREFIX}/lib/libproj.*  \
    && ${GCC_ARCH}-linux-gnu-strip -s /build${PROJ_INSTALL_PREFIX}/lib/libinternalproj.so.${PROJ_SO} \
    && for i in /build${PROJ_INSTALL_PREFIX}/bin/*; do ${GCC_ARCH}-linux-gnu-strip -s $i 2>/dev/null || /bin/true; done \
    && patchelf --set-soname libinternalproj.so.${PROJ_SO_FIRST} /build${PROJ_INSTALL_PREFIX}/lib/libinternalproj.so.${PROJ_SO} \
    && for i in /build${PROJ_INSTALL_PREFIX}/bin/*; do patchelf --replace-needed libproj.so.${PROJ_SO_FIRST} libinternalproj.so.${PROJ_SO_FIRST} $i; done

# Build GDAL
ARG GDAL_VERSION=master
ARG GDAL_RELEASE_DATE
ARG GDAL_BUILD_IS_RELEASE
ARG GDAL_REPOSITORY=OSGeo/gdal

RUN --mount=type=cache,id=ubuntu-small-gdal,target=$HOME/.cache \
    . /buildscripts/bh-set-envvars.sh \
    && if test "${GDAL_VERSION}" = "master"; then \
        export GDAL_VERSION=$(curl -Ls https://api.github.com/repos/${GDAL_REPOSITORY}/commits/HEAD -H "Accept: application/vnd.github.VERSION.sha"); \
        export GDAL_RELEASE_DATE=$(date "+%Y%m%d"); \
    fi \
    && if test "x${GDAL_BUILD_IS_RELEASE:-}" = "x"; then \
        export GDAL_SHA1SUM=${GDAL_VERSION}; \
    fi \
    && mkdir gdal \
    && wget -q https://github.com/${GDAL_REPOSITORY}/archive/${GDAL_VERSION}.tar.gz -O - \
        | tar xz -C gdal --strip-components=1 \
    && cd gdal \
    && if test "${RSYNC_REMOTE:-}" != ""; then \
        echo "Downloading cache..."; \
        rsync -ra ${RSYNC_REMOTE}/gdal/${GCC_ARCH}/ $HOME/.cache/; \
        echo "Finished"; \
    fi \
    && if [ -n "${WITH_CCACHE:-}" ]; then \
        # Little trick to avoid issues with Python bindings
        printf "#!/bin/sh\nccache %s-linux-gnu-gcc \$*" "${GCC_ARCH}" > ccache_gcc.sh; \
        chmod +x ccache_gcc.sh; \
        printf "#!/bin/sh\nccache %s-linux-gnu-g++ \$*" "${GCC_ARCH}" > ccache_g++.sh; \
        chmod +x ccache_g++.sh; \
        export CC=$PWD/ccache_gcc.sh; \
        export CXX=$PWD/ccache_g++.sh; \
        ccache -M 1G; \
    fi \
    && mkdir build \
    && cd build \
    # -Wno-psabi avoid 'note: parameter passing for argument of type 'std::pair<double, double>' when C++17 is enabled changed to match C++14 in GCC 10.1' on arm64
    && CFLAGS='-DPROJ_RENAME_SYMBOLS -O2' CXXFLAGS='-DPROJ_RENAME_SYMBOLS -DPROJ_INTERNAL_CPP_NAMESPACE -O2 -Wno-psabi' \
       cmake .. \
        -G Ninja \
        -DCMAKE_INSTALL_PREFIX=/usr \
        -DGDAL_FIND_PACKAGE_PROJ_MODE=MODULE \
        -DPROJ_INCLUDE_DIR="/build${PROJ_INSTALL_PREFIX-/usr/local}/include" \
        -DPROJ_LIBRARY="/build${PROJ_INSTALL_PREFIX-/usr/local}/lib/libinternalproj.so" \
        -DGDAL_USE_TIFF_INTERNAL=ON \
        -DGDAL_USE_GEOTIFF_INTERNAL=ON \
        -DBUILD_TESTING=OFF \
    && ninja \
    && DESTDIR="/build" ninja install \
    && cd .. \
    && if test "${RSYNC_REMOTE:-}" != ""; then \
        echo "Uploading cache..."; \
        rsync -ra --delete $HOME/.cache/ ${RSYNC_REMOTE}/gdal/${GCC_ARCH}/; \
        echo "Finished"; \
    fi \
    && if [ -n "${WITH_CCACHE:-}" ]; then \
        ccache -s; \
        unset CC; \
        unset CXX; \
    fi \
    && cd .. \
    && rm -rf gdal \
    && mkdir -p /build_gdal_python/usr/lib \
    && mkdir -p /build_gdal_python/usr/bin \
    && mkdir -p /build_gdal_version_changing/usr/include \
    && mv /build/usr/lib/python*            /build_gdal_python/usr/lib \
    && mv /build/usr/lib                    /build_gdal_version_changing/usr \
    && mv /build/usr/include/gdal_version.h /build_gdal_version_changing/usr/include \
    && mv /build/usr/bin/*.py               /build_gdal_python/usr/bin \
    && mv /build/usr/bin                    /build_gdal_version_changing/usr \
    && for i in /build_gdal_version_changing/usr/lib/${GCC_ARCH}-linux-gnu/*; do ${GCC_ARCH}-linux-gnu-strip -s $i 2>/dev/null || /bin/true; done \
    && for i in /build_gdal_python/usr/lib/python3/dist-packages/osgeo/*.so; do ${GCC_ARCH}-linux-gnu-strip -s $i 2>/dev/null || /bin/true; done \
    && for i in /build_gdal_version_changing/usr/bin/*; do ${GCC_ARCH}-linux-gnu-strip -s $i 2>/dev/null || /bin/true; done

# Build final image
FROM $TARGET_BASE_IMAGE AS runner

USER root
RUN date

# Update distro
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    export DEBIAN_FRONTEND=noninteractive \
    && apt-get update -y \
    && apt-get upgrade -y \
    # PROJ dependencies
    && apt-get install -y --no-install-recommends \
        libsqlite3-0 libtiff6 libcurl4 \
        curl unzip ca-certificates \
    # GDAL dependencies
    && apt-get install -y --no-install-recommends \
        bash-completion python3-numpy libpython3.12 \
        libjpeg-turbo8 libgeos3.12.1 libgeos-c1v5 \
        libexpat1 \
        libxerces-c3.2 \
        libwebp7 libpng16-16 \
        libdeflate0 \
        libzstd1 bash libpq5 libssl3 libopenjp2-7 libspatialite8 \
        # pil for antialias option of gdal2tiles
        python3-pil \
    && apt-get install -y python-is-python3

# Order layers starting with less frequently varying ones
# Only used for custom libopenjp2
# COPY --from=builder  /build_thirdparty/usr/ /usr/

COPY --from=builder  /build_projgrids/usr/ /usr/

ARG PROJ_INSTALL_PREFIX
COPY --from=builder  /build${PROJ_INSTALL_PREFIX}/share/proj/ ${PROJ_INSTALL_PREFIX}/share/proj/
COPY --from=builder  /build${PROJ_INSTALL_PREFIX}/include/ ${PROJ_INSTALL_PREFIX}/include/
COPY --from=builder  /build${PROJ_INSTALL_PREFIX}/bin/ ${PROJ_INSTALL_PREFIX}/bin/
COPY --from=builder  /build${PROJ_INSTALL_PREFIX}/lib/ ${PROJ_INSTALL_PREFIX}/lib/

COPY --from=builder  /build/usr/share/bash-completion/ /usr/share/bash-completion/
COPY --from=builder  /build/usr/share/gdal/ /usr/share/gdal/
COPY --from=builder  /build/usr/include/ /usr/include/
COPY --from=builder  /build_gdal_python/usr/ /usr/
COPY --from=builder  /build_gdal_version_changing/usr/ /usr/

RUN ldconfig

RUN echo "source /usr/share/bash-completion/bash_completion" >> /root/.bashrc

CMD ["/bin/bash", "-l"]
