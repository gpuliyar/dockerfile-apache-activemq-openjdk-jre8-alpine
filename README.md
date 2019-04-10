# dockerfile-apache-activemq-openjdk-jre8-alpine
The Project builds a Container for Apache ActiveMQ using OpenJDK JRE 8 and Alpine

## What the code means? Let's go step by step:

```
FROM openjdk:8-jre-alpine
```
Creating the image from OpenJDK JRE 8

```
ENV ACTIVEMQ_VERSION 5.14.5
ENV ACTIVEMQ apache-activemq-${ACTIVEMQ_VERSION}
ENV ACTIVEMQ_TCP=61616 ACTIVEMQ_AMQP=5672 ACTIVEMQ_STOMP=61613 ACTIVEMQ_MQTT=1883 ACTIVEMQ_WS=61614 ACTIVEMQ_UI=8161
ENV ACTIVEMQ_HOME /opt/activemq
```
Let's set the environment variables. For my use-case, I chose the version 5.14.5. You can keep the rest of the environment variables as is. You are free to change it as per your need too.

```
RUN set -ex; \
    \
# Add a new user and group. We will use the same user to run the service \
    addgroup -S -g 1000 activemq; \
    adduser -S -D -s /sbin/nologin -G activemq -u 1000 activemq; \
    \
# Install the relevant packages. Install with --virtual option so you can remove them once they are no longer needed. \
    apk --update add --virtual build-dependencies gnupg; \
    \
# Download the binaries and keys to validate and verify the package \
    wget -q http://www.apache.org/dist/activemq/KEYS -O KEYS; \
    wget -q https://www.apache.org/dist/activemq/${ACTIVEMQ_VERSION}/${ACTIVEMQ}-bin.tar.gz.asc -O ${ACTIVEMQ}-bin.tar.gz.asc; \
    wget -q https://archive.apache.org/dist/activemq/${ACTIVEMQ_VERSION}/${ACTIVEMQ}-bin.tar.gz -O ${ACTIVEMQ}-bin.tar.gz; \
    \
# Import the keys and verify the package \
    gpg --import KEYS; \
    gpg --verify ${ACTIVEMQ}-bin.tar.gz.asc ${ACTIVEMQ}-bin.tar.gz; \
    \
# Extract the file to the specific directory and remove the TAR file \
    tar xvz -f ${ACTIVEMQ}-bin.tar.gz -C /opt; \
    rm ${ACTIVEMQ}-bin.tar.gz; \
    \
# Create the symolic link to /opt/activemq \
    ln -s /opt/${ACTIVEMQ} ${ACTIVEMQ_HOME}; \
    \
# Give the relevant permissions to the user on the directories \
    chown -R activemq:activemq /opt/${ACTIVEMQ}; \
    chown -h activemq:activemq ${ACTIVEMQ_HOME}; \
    \
# Remove the packages that are no longer needed and clean the APK cache to reduce the image size \
    apk del build-dependencies; \
    rm -rf /var/cache/apk/*;
```

```
VOLUME [ "/opt/activemq/data", "/opt/activemq/conf", "/opt/activemq/logs" ]
```
Define the volume for the data, logs and configuration

```
WORKDIR ${ACTIVEMQ_HOME}
```
Set the work directory

```
EXPOSE ${ACTIVEMQ_TCP} ${ACTIVEMQ_AMQP} ${ACTIVEMQ_STOMP} ${ACTIVEMQ_MQTT} ${ACTIVEMQ_WS} ${ACTIVEMQ_UI}
```
Expose all the relevant ports

```
USER activemq
```
Set the user to activemq. 

```
ENTRYPOINT [ "/bin/sh", "-c", "bin/activemq console" ]
```
Set the entrypoint
