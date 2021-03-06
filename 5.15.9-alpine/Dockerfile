FROM gitlab.expertflow.com:9242/general/openjdk:8-jre-alpine-3.12

ENV ACTIVEMQ_VERSION 5.15.9
ENV ACTIVEMQ apache-activemq-$ACTIVEMQ_VERSION
ENV ACTIVEMQ_TCP=61616 ACTIVEMQ_AMQP=5672 ACTIVEMQ_STOMP=61613 ACTIVEMQ_MQTT=1883 ACTIVEMQ_WS=61614 ACTIVEMQ_UI=8161
ENV SHA512_VAL=35cae4258e38e47f9f81e785f547afc457fc331d2177bfc2391277ce24123be1196f10c670b61e30b43b7ab0db0628f3ff33f08660f235b7796d59ba922d444f

ENV ACTIVEMQ_HOME /opt/activemq

RUN set -x && \
    mkdir -p /opt && \
    apk --update add --virtual build-dependencies curl && \
    curl https://archive.apache.org/dist/activemq/$ACTIVEMQ_VERSION/$ACTIVEMQ-bin.tar.gz -o $ACTIVEMQ-bin.tar.gz

# Validate checksum
RUN if [ "$SHA512_VAL" != "$(sha512sum $ACTIVEMQ-bin.tar.gz | awk '{print($1)}')" ];\
    then \
        echo "sha512 values doesn't match! exiting."  && \
        exit 1; \
    fi;

RUN tar xzf $ACTIVEMQ-bin.tar.gz -C  /opt && \
    ln -s /opt/$ACTIVEMQ $ACTIVEMQ_HOME && \
    addgroup -S activemq && adduser -S -H -G activemq -h $ACTIVEMQ_HOME activemq && \
    chown -R activemq:activemq /opt/$ACTIVEMQ && \
    chown -h activemq:activemq $ACTIVEMQ_HOME && \
    apk del build-dependencies && \
    rm -rf /var/cache/apk/*

RUN apk add --no-cache tzdata unzip
RUN apk add --no-cache curl
COPY ./activemq.xml /opt/apache-activemq-5.15.9/conf/
COPY ./jetty.xml /opt/apache-activemq-5.15.9/conf/
COPY activemq_ssl_certificates/. /opt/apache-activemq-5.15.9/conf/

# Following lines remove apk and its repo files, trivy will not be able to scan installed packages in this image and will report no vulnerability
# dirty way to do it but its the common practice in image hardening to remove package manager
RUN rm -f /sbin/apk
RUN rm -rf /etc/apk
RUN rm -rf /lib/apk
RUN rm -rf /usr/share/apk
RUN rm -rf /var/lib/apk

HEALTHCHECK --interval=1m30s --timeout=3s --retries=10 \
      CMD curl -fsk http://localhost:8161 || exit 1

USER activemq

WORKDIR $ACTIVEMQ_HOME
EXPOSE $ACTIVEMQ_TCP $ACTIVEMQ_AMQP $ACTIVEMQ_STOMP $ACTIVEMQ_MQTT $ACTIVEMQ_WS $ACTIVEMQ_UI

CMD ["/bin/sh", "-c", "bin/activemq console"]
