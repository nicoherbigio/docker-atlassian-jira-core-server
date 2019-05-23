FROM nicoherbigio/adoptopenjdk-openjdk:8u212

LABEL maintainer="Nico Herbig <nico@nicoherbig.io>"


# Install additional required software packages

RUN set -ex \
    && apt-get update \
    && apt-get install -y --no-install-recommends bash ca-certificates curl fontconfig openssl perl procps wget \
    && rm -rf /var/lib/apt/lists/*


# Install Tini

ARG TINI_VERSION=0.18.0
ARG TINI_DOWNLOAD_URL=https://github.com/krallin/tini/releases/download/v${TINI_VERSION}/tini

RUN set -ex \
    && curl -fSL ${TINI_DOWNLOAD_URL} -o /sbin/tini \
    && chmod +x /sbin/tini


# Add Atlassian Jira Core Server user and group to ensure that the IDs (UID and GID)
# are consistently assigned regardless of which additional dependencies are added.

ENV RUN_USER_ID=901 RUN_USER=jira
ENV RUN_GROUP_ID=901 RUN_GROUP=jira

RUN groupadd -r -g ${RUN_GROUP_ID} ${RUN_GROUP} && useradd -r -g ${RUN_GROUP} -u ${RUN_USER_ID} ${RUN_USER}


# Install Atlassian Jira Core Server

ARG JIRA_VERSION=8.1.0
ARG JIRA_DOWNLOAD_URL=https://www.atlassian.com/software/jira/downloads/binary/atlassian-jira-core-${JIRA_VERSION}.tar.gz

ENV JIRA_INSTALL_DIR=/opt/atlassian/jira
ENV JIRA_HOME=/var/atlassian/jira

# Actions to perform:
#   - Create installation directory
#   - Create home directory
#   - Download and unpack Atlassian Jira Core Server
#   - Set permissions for the Jira Core installation directory
#   - Set permissions for the Jira Core home directory
RUN set -ex \
    && mkdir -p ${JIRA_INSTALL_DIR} \
    && mkdir -p ${JIRA_HOME} \
    && curl -fSL ${JIRA_DOWNLOAD_URL} | tar -xz --no-same-owner --strip-components=1 -C ${JIRA_INSTALL_DIR} \
    && sed -i -e 's/JVM_MINIMUM_MEMORY="\([0-9]\+[kmg]\)"/: ${JVM_MINIMUM_MEMORY:=\1}/g' ${JIRA_INSTALL_DIR}/bin/setenv.sh \
    && sed -i -e 's/JVM_MAXIMUM_MEMORY="\([0-9]\+[kmg]\)"/: ${JVM_MAXIMUM_MEMORY:=\1}/g' ${JIRA_INSTALL_DIR}/bin/setenv.sh \
    && sed -i -e 's/port="8080"/port="8080" secure="${catalinaConnectorSecure}" scheme="${catalinaConnectorScheme}" proxyName="${catalinaConnectorProxyName}" proxyPort="${catalinaConnectorProxyPort}"/' ${JIRA_INSTALL_DIR}/conf/server.xml \
    && chown -R ${RUN_USER}:${RUN_GROUP} ${JIRA_INSTALL_DIR} \
    && chown -R ${RUN_USER}:${RUN_GROUP} ${JIRA_HOME}


# Configure Atlassian Jira Core Server

VOLUME ${JIRA_HOME}

COPY assets/docker-entrypoint.sh /usr/local/bin/

RUN set -ex \
    && chmod +x /usr/local/bin/docker-entrypoint.sh \
    && ln -s usr/local/bin/docker-entrypoint.sh /

EXPOSE 8080

WORKDIR ${JIRA_HOME}

CMD ["/docker-entrypoint.sh", "-fg"]
ENTRYPOINT ["/sbin/tini", "--"]
