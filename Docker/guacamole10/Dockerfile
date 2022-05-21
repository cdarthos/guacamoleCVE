FROM tomcat:9.0.62-jre11-openjdk

ENV ARCH=amd64 \
  GUAC_VER=1.0.0 \
  GUACAMOLE_HOME=/app/guacamole


# Apply the s6-overlay
RUN curl -SLO "https://github.com/just-containers/s6-overlay/releases/download/v1.20.0.0/s6-overlay-${ARCH}.tar.gz" \
  && tar -xzf s6-overlay-${ARCH}.tar.gz -C / \
  && tar -xzf s6-overlay-${ARCH}.tar.gz -C /usr ./bin \
  && rm -rf s6-overlay-${ARCH}.tar.gz \
  && mkdir -p ${GUACAMOLE_HOME} \
    ${GUACAMOLE_HOME}/lib \
    ${GUACAMOLE_HOME}/extensions

WORKDIR ${GUACAMOLE_HOME}

# Install dependencies
RUN echo 'deb  http://ftp.de.debian.org/debian/  oldoldstable main contrib non-free' >> /etc/apt/sources.list
RUN echo 'deb-src  http://ftp.de.debian.org/debian/  oldoldstable main contrib non-free' >> /etc/apt/sources.list
RUN apt-get update && apt-get install -y \
    curl \
    wget \
    libcairo2-dev 	\
    libjpeg62-turbo-dev \
    libpng-dev \
    libossp-uuid-dev \
    libpango1.0-dev \
    libssh2-1-dev \
    libssl-dev \
    uuid-dev \
    make \
    gcc-6 \
    openssh-server \
    && rm -rf /var/lib/apt/lists/*


RUN ln -s /usr/bin/gcc-6 /usr/bin/gcc

# Link FreeRDP to where guac expects it to be
RUN [ "$ARCH" = "amd64" ] && ln -s /usr/local/lib/freerdp /usr/lib/x86_64-linux-gnu/freerdp || exit 0

# Install guacamole-server
RUN curl -SLO "http://apache.org/dyn/closer.cgi?action=download&filename=guacamole/${GUAC_VER}/source/guacamole-server-${GUAC_VER}.tar.gz" \
  && tar -xzf guacamole-server-${GUAC_VER}.tar.gz \
  && cd guacamole-server-${GUAC_VER} \
  && ./configure --enable-allow-freerdp-snapshots \
  && make -j$(getconf _NPROCESSORS_ONLN) \
  && make install \
  && cd .. \
  && rm -rf guacamole-server-${GUAC_VER}.tar.gz guacamole-server-${GUAC_VER} \
  && ldconfig

# Install guacamole-client
RUN set -x \
  && rm -rf ${CATALINA_HOME}/webapps/ROOT \
  && curl -SLo ${CATALINA_HOME}/webapps/ROOT.war "http://apache.org/dyn/closer.cgi?action=download&filename=guacamole/${GUAC_VER}/binary/guacamole-${GUAC_VER}.war"


WORKDIR /config

COPY root /



EXPOSE 8080

ENTRYPOINT [ "/init" ]