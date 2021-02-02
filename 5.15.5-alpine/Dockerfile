FROM adoptopenjdk/openjdk8-openj9:alpine-slim

ENV ACTIVEMQ_VERSION 5.15.5
ENV ACTIVEMQ apache-activemq-$ACTIVEMQ_VERSION
ENV ACTIVEMQ_TCP=61616 ACTIVEMQ_AMQP=5672 ACTIVEMQ_STOMP=61613 ACTIVEMQ_MQTT=1883 ACTIVEMQ_WS=61614 ACTIVEMQ_UI=8161
ENV SHA512_VAL=450469a36147cef8dc3ef5f98de7e173dc63e5ca82e090417c112a70132b7a7da87851e330443e9d68f085c375f4e85f2cae34317d09a59295fec05dab8f2d26
ENV ACTIVEMQ_HOME /opt/activemq


#=========for prometheus logs, JVM Exporter========#
ENV AGENT_VERSION=0.15.0
ENV AGENT_SHA512=fccad536103503e21df12c6a16e3870924432418e7eef1e5730f9bc81789d7b662f4a8b66baf1fcdc63ba26c171047aab046a7b0b23957def26ca8e1790062cf
ENV AGENT_BASE=jmx_prometheus_javaagent
ENV AGENT=$AGENT_BASE-$AGENT_VERSION.jar
ENV AGENT_CONFIG=config.yaml
ENV AGENT_PORT=8080
ENV AGENT_OPTS="-javaagent:/$AGENT=$AGENT_PORT:/config.yaml"
ADD https://repo1.maven.org/maven2/io/prometheus/jmx/$AGENT_BASE/$AGENT_VERSION/$AGENT /
USER root
RUN chmod 0644 /$AGENT && if [ "$AGENT_SHA512" != "$(cd / && sha512sum $AGENT | awk '{print($1)}')" ]; then \
        echo "Agent sha512 value doesn't match! exiting."  && exit 1; \
    fi
COPY $AGENT_CONFIG /

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
RUN apk update
RUN apk upgrade
RUN apk add --no-cache tzdata unzip
RUN apk add --no-cache curl
COPY ./activemq.xml /opt/apache-activemq-5.15.5/conf/
COPY ./jetty.xml /opt/apache-activemq-5.15.5/conf/
COPY activemq_ssl_certificates/. /opt/apache-activemq-5.15.5/conf/

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
#added $AGENT_PORT in expose
EXPOSE $ACTIVEMQ_TCP $ACTIVEMQ_AMQP $ACTIVEMQ_STOMP $ACTIVEMQ_MQTT $ACTIVEMQ_WS $ACTIVEMQ_UI $AGENT_PORT

#added below lines for prometheus
ENV ACTIVEMQ_OPTS_MEMORY="-Xms64M -Xmx1G"
ENV ACTIVEMQ_CONF=$ACTIVEMQ_HOME/conf
ENV ACTIVEMQ_OPTS="$AGENT_OPTS $ACTIVEMQ_OPTS_MEMORY -Djava.util.logging.config.file=logging.properties -Djava.security.auth.login.config=$ACTIVEMQ_CONF/login.config"
CMD ["/bin/sh", "-c", "bin/activemq console"]
