FROM linuxserver/daapd:28.9.20240417

ARG BUILD_ARCH

RUN apk add --no-cache jq

RUN sed -i -e s#"ipv6 = yes"#"ipv6 = no"#g /config/owntone.conf.orig \
    && sed -i s#/srv/music#/share/owntone/music#g /config/owntone.conf.orig \
    && sed -i s#/var/cache/owntone/songs3.db#/share/owntone/dbase_and_logs/songs3.db#g /config/owntone.conf.orig \
    && sed -i s#/var/cache/owntone/cache.db#/share/owntone/dbase_and_logs/cache.db#g /config/owntone.conf.orig \
    && sed -i s#/var/log/owntone.log#/share/owntone/dbase_and_logs/owntone.log#g /config/owntone.conf.orig \
    && sed -i "/websocket_port\ =/ s/# *//" /config/owntone.conf.orig \
    && sed -i "/trusted_networks\ =/ s/# *//" /config/owntone.conf.orig \
    && sed -i "/pipe_autostart\ =/ s/# *//" /config/owntone.conf.orig \
    && sed -i "/airplay_shared/ s/# *//" /config/owntone.conf.orig \
    && sed -i "/control_port\ =/ s/#/ /" /config/owntone.conf.orig \
    && sed -i "/timing_port\ =/ s/#/ /" /config/owntone.conf.orig \
    && sed -i "/timing_port/{N;s/\n#/\n/}" /config/owntone.conf.orig \
    && sed -i "s/\(control_port =\).*/\1 3690/" /config/owntone.conf.orig \
    && sed -i "s/\(timing_port =\).*/\1 3691/" /config/owntone.conf.orig \
    && sed -i "/type\ =/ s/#/ /" /config/owntone.conf.orig \
    && sed -i 's/\(type =\).*/\1 "pulseaudio"/' /config/owntone.conf.orig

ADD 90-homeassistant /config/cont-init.d/90-homeassistant

RUN chmod +x /config/cont-init.d/90-homeassistant
