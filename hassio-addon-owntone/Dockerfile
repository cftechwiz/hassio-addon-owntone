FROM ghcr.io/cftechwiz/alpine:3.19.1 AS buildstage
############## build stage ##############

ARG DAAPD_RELEASE
ARG TARGETPLATFORM
ARG VERSION
ARG CHANNEL

RUN \
    echo "**** install build packages ****" && \
    apk add -U --update --no-cache \
    alsa-lib-dev \
    autoconf \
    automake \
    avahi-dev \
    bison \
    bsd-compat-headers \
    confuse-dev \
    curl-dev \
    ffmpeg-dev \
    file \
    flac-dev \
    flex \
    g++ \
    gcc \
    gettext-dev \
    gnutls-dev \
    gperf \
    json-c-dev \
    libevent-dev \
    libgcrypt-dev \
    libogg-dev \
    libplist-dev \
    openssl-dev \
    libsodium-dev \
    libtool \
    libunistring-dev \
    libwebsockets-dev \
    make \
    mxml-dev \
    openjdk8-jre-base \
    protobuf-c-dev \
    sqlite-dev \
    taglib-dev && \
    mkdir -p \
    /tmp/source/owntone && \
    echo "**** compile owntone-server ****" && \
    if [ -z ${DAAPD_RELEASE+x} ]; then \
    DAAPD_RELEASE=$(curl -sX GET "https://api.github.com/repos/owntone/owntone-server/releases/latest" \
    | awk '/tag_name/{print $4;exit}' FS='[""]'); \
    fi && \
    curl -o \
    /tmp/source/owntone.tar.gz -L \
    "https://github.com/owntone/owntone-server/archive/${DAAPD_RELEASE}.tar.gz" && \
    tar xf /tmp/source/owntone.tar.gz -C \
    /tmp/source/owntone --strip-components=1 && \
    export PATH="/tmp/source:$PATH" && \
    cd /tmp/source/owntone && \
    autoreconf -i -v && \
    ./configure \
    --build=$CBUILD \
    --enable-chromecast \
    --enable-lastfm \
    --enable-mpd \
    --host=$CHOST \
    --infodir=/usr/share/info \
    --localstatedir=/var \
    --mandir=/usr/share/man \
    --prefix=/usr \
    --sysconfdir=/etc && \
    make && \
    make DESTDIR=/tmp/daapd-build install && \
    mv /tmp/daapd-build/etc/owntone.conf /tmp/daapd-build/etc/owntone.conf.orig && \
    rm -rf /tmp/daapd-build/var
############## runtime stage ##############
FROM ghcr.io/cftechwiz/alpine:3.19.1

# set version label
ARG BUILD_DATE
ARG VERSION
LABEL build_version="Linuxserver.io version:- ${VERSION} Build-date:- ${BUILD_DATE}"
LABEL maintainer="BernsteinA"

RUN \
    echo "**** install runtime packages ****" && \
    apk add -U --update --no-cache \
    avahi \
    confuse \
    dbus \
    ffmpeg \
    gnutls \
    json-c \
    libevent \
    libgcrypt \
    libplist \
    libsodium \
    libunistring \
    libuuid \
    libwebsockets \
    mxml \
    protobuf-c \
    sqlite \
    sqlite-libs && \
    apk add -U --update --no-cache --repository https://dl-cdn.alpinelinux.org/alpine/edge/testing \
    librespot && \
    mkdir -p /music

# copy buildstage and local files
COPY --from=buildstage /tmp/daapd-build/ /
COPY root/ /

# ports and volumes
EXPOSE 3689

VOLUME /config /music

ARG BUILD_ARCH

RUN apk add --no-cache jq

RUN sed -i -e s#"ipv6 = yes"#"ipv6 = no"#g /etc/owntone.conf.orig \
    && sed -i s#/srv/music#/share/owntone/music#g /etc/owntone.conf.orig \
    && sed -i s#/var/cache/owntone/songs3.db#/share/owntone/dbase_and_logs/songs3.db#g /etc/owntone.conf.orig \
    && sed -i s#/var/cache/owntone/cache.db#/share/owntone/dbase_and_logs/cache.db#g /etc/owntone.conf.orig \
    && sed -i s#/var/log/owntone.log#/share/owntone/dbase_and_logs/owntone.log#g /etc/owntone.conf.orig \
    && sed -i "/websocket_port\ =/ s/# *//" /etc/owntone.conf.orig \
    && sed -i "/trusted_networks\ =/ s/# *//" /etc/owntone.conf.orig \
    && sed -i "/pipe_autostart\ =/ s/# *//" /etc/owntone.conf.orig \
    && sed -i "/airplay_shared/ s/# *//" /etc/owntone.conf.orig \
    && sed -i "/control_port\ =/ s/#/ /" /etc/owntone.conf.orig \
    && sed -i "/timing_port\ =/ s/#/ /" /etc/owntone.conf.orig \
    && sed -i "/timing_port/{N;s/\n#/\n/}" /etc/owntone.conf.orig \
    && sed -i "s/\(control_port =\).*/\1 3690/" /etc/owntone.conf.orig \
    && sed -i "s/\(timing_port =\).*/\1 3691/" /etc/owntone.conf.orig \
    && sed -i "/type\ =/ s/#/ /" /etc/owntone.conf.orig \
    && sed -i 's/\(type =\).*/\1 "pulseaudio"/' /etc/owntone.conf.orig

ADD 90-homeassistant /etc/cont-init.d/90-homeassistant

RUN chmod +x /etc/cont-init.d/90-homeassistant
