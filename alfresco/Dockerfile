
FROM alfresco/alfresco-content-repository-community:7.1.1.2 as acs-base

# Fetch image based on Tomcat 9.0, Java 11 and Centos 8
# More infos about this image: https://github.com/Alfresco/alfresco-docker-base-tomcat
FROM ghcr.io/femshima/alfresco-base-tomcat-multiarch:v1.0.2

# Set default user information
ARG GROUPNAME=Alfresco
ARG GROUPID=1000
ARG IMAGEUSERNAME=alfresco
ARG USERID=33000

# Set default environment args
ARG TOMCAT_DIR=/usr/local/tomcat

COPY --from=acs-base ${TOMCAT_DIR}/webapps ${TOMCAT_DIR}/webapps
COPY --from=acs-base ${TOMCAT_DIR}/lib/ ${TOMCAT_DIR}/lib/
COPY --from=acs-base ${TOMCAT_DIR}/alfresco-mmt/ ${TOMCAT_DIR}/alfresco-mmt/
COPY --from=acs-base /licenses/ /licenses/
COPY --from=acs-base ${TOMCAT_DIR}/shared/classes/alfresco/extension/keystore ${TOMCAT_DIR}/shared/classes/alfresco/extension/keystore
COPY --from=acs-base ${TOMCAT_DIR}/amps/ ${TOMCAT_DIR}/amps/
COPY --from=acs-base ${TOMCAT_DIR}/conf/ ${TOMCAT_DIR}/conf/

RUN microdnf install -y fontconfig \
                   dejavu-fonts-common \
                   fontpackages-filesystem \
                   freetype \
                   libpng \
                   dejavu-sans-fonts && \
    microdnf clean all

# The standard configuration is to have all Tomcat files owned by root with group GROUPNAME and whilst owner has read/write privileges, 
# group only has restricted permissions and world has no permissions.
RUN microdnf update -y && \
    microdnf install -y findutils && \
    microdnf clean all && \
    mkdir -p ${TOMCAT_DIR}/conf/Catalina/localhost && \
    mkdir -p ${TOMCAT_DIR}/alf_data && \
    groupadd -g ${GROUPID} ${GROUPNAME} && \
    useradd -u ${USERID} -G ${GROUPNAME} ${IMAGEUSERNAME} && \
    chgrp -R ${GROUPNAME} ${TOMCAT_DIR} && \
    chmod g+rx ${TOMCAT_DIR}/conf && \
    chmod -R g+r ${TOMCAT_DIR}/conf && \
    find ${TOMCAT_DIR}/webapps -type d -exec chmod 0750 {} \; && \
    find ${TOMCAT_DIR}/webapps -type f -exec chmod 0640 {} \; && \
    chmod -R g+r ${TOMCAT_DIR}/webapps && \
    chmod g+r ${TOMCAT_DIR}/conf/Catalina && \
    chmod g+rwx ${TOMCAT_DIR}/alf_data && \
    chmod g+rwx ${TOMCAT_DIR}/logs && \
    chmod o-w ${TOMCAT_DIR}/logs && \
    chmod g+rwx ${TOMCAT_DIR}/temp && \
    chmod g+rwx ${TOMCAT_DIR}/work && \
    chmod o-w ${TOMCAT_DIR}/work && \
    chmod 664 ${TOMCAT_DIR}/alfresco-mmt/alfresco-mmt-*.jar && \
    find /licenses -type d -exec chmod 0755 {} \; && \
    find /licenses -type f -exec chmod 0644 {} \;

EXPOSE 10001

# For remote debug
EXPOSE 8000




COPY de.acosix.alfresco.utility.repo-1.3.0.amp ${TOMCAT_DIR}/amps/

# Install amps on alfresco.war
RUN java -jar ${TOMCAT_DIR}/alfresco-mmt/alfresco-mmt*.jar install \
              ${TOMCAT_DIR}/amps \
              ${TOMCAT_DIR}/webapps/alfresco -directory -nobackup

#COPY de.acosix.alfresco.keycloak.repo-1.1.0-rc7.amp ${TOMCAT_DIR}/amps/
COPY de.acosix.alfresco.keycloak.repo-1.1.0-rc7-20220123.141234-4.amp ${TOMCAT_DIR}/amps/
# Install amps on alfresco.war
RUN java -jar ${TOMCAT_DIR}/alfresco-mmt/alfresco-mmt*.jar install \
              ${TOMCAT_DIR}/amps \
              ${TOMCAT_DIR}/webapps/alfresco -directory -nobackup




# The standard configuration is to have all Tomcat files owned by root with group GROUPNAME and whilst owner has read/write privileges,
# group only has restricted permissions and world has no permissions.
RUN chgrp -R ${GROUPNAME} ${TOMCAT_DIR}/webapps ${TOMCAT_DIR}/amps ${TOMCAT_DIR}/lib && \
    find ${TOMCAT_DIR}/webapps -type d -exec chmod 0750 {} \; && \
    find ${TOMCAT_DIR}/webapps -type f -exec chmod 0640 {} \; && \
    chmod -R g+r ${TOMCAT_DIR}/webapps && \
    chmod 664 ${TOMCAT_DIR}/amps/*

USER ${IMAGEUSERNAME}