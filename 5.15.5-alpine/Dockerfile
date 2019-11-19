FROM gitlab.expertflow.com:9242/general/openjdk:8-jre-alpine-3.10.3

ENV ACTIVEMQ_VERSION 5.15.5
ENV ACTIVEMQ apache-activemq-$ACTIVEMQ_VERSION
ENV ACTIVEMQ_TCP=61616 ACTIVEMQ_AMQP=5672 ACTIVEMQ_STOMP=61613 ACTIVEMQ_MQTT=1883 ACTIVEMQ_WS=61614 ACTIVEMQ_UI=8161
ENV SHA512_VAL=450469a36147cef8dc3ef5f98de7e173dc63e5ca82e090417c112a70132b7a7da87851e330443e9d68f085c375f4e85f2cae34317d09a59295fec05dab8f2d26

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
COPY ./activemq.xml /opt/${ACTIVEMQ_HOME}/conf/
COPY ./jetty.xml /opt/${ACTIVEMQ_HOME}/conf/
COPY activemq_ssl_certificates/. /opt/${ACTIVEMQ_HOME}/conf/
USER 0
RUN echo "user:x:activemq:0::/opt/apache-activemq-5.15.5:" >> /etc/passwd \
    && echo "user:!:$(($(date +%s) / 60 / 60 / 24)):0:99999:7:::" >> /etc/shadow \
    && echo "user:x:activemq:" >> /etc/group \
    && chmod -R 775 /opt/apache-activemq-5.15.5 \
    && chown -R user:root /opt/apache-activemq-5.15.5 \
    && chown -h activemq:activemq /opt/activemq

USER activemq

WORKDIR $ACTIVEMQ_HOME
EXPOSE $ACTIVEMQ_TCP $ACTIVEMQ_AMQP $ACTIVEMQ_STOMP $ACTIVEMQ_MQTT $ACTIVEMQ_WS $ACTIVEMQ_UI

CMD ["/bin/sh", "-c", "bin/activemq console"]
