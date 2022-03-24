FROM alfresco/alfresco-share:7.1.1.2 as share-base

FROM ghcr.io/femshima/alfresco-base-tomcat-multiarch:v1.0.2


ARG KEYCLOAK_URL
ARG KEYCLOAK_REALM
ARG KEYCLOAK_SHARE_SECRET


ARG TOMCAT_DIR=/usr/local/tomcat

COPY --from=share-base ${TOMCAT_DIR}/webapps ${TOMCAT_DIR}/webapps
COPY --from=share-base ${TOMCAT_DIR}/alfresco-mmt/ ${TOMCAT_DIR}/alfresco-mmt/
COPY --from=share-base ${TOMCAT_DIR}/amps_share ${TOMCAT_DIR}/amps_share
COPY --from=share-base ${TOMCAT_DIR}/shared/classes/alfresco/ ${TOMCAT_DIR}/shared/classes/alfresco/
COPY --from=share-base ${TOMCAT_DIR}/conf/ ${TOMCAT_DIR}/conf/

RUN chmod +x ${TOMCAT_DIR}/shared/classes/alfresco/substituter.sh
#/usr/local/tomcat/shared/classes/alfresco/web-extension/share-config-custom.xml
COPY share-config-custom.xml ${TOMCAT_DIR}/shared/classes/alfresco/web-extension/share-config-custom.append.xml
RUN cat ${TOMCAT_DIR}/shared/classes/alfresco/web-extension/share-config-custom.append.xml |\
    sed \
    -e 's/<auth-server-url><\/auth-server-url>/<auth-server-url>'"$(echo $KEYCLOAK_URL|sed 's/\//\\\//g')"'<\/auth-server-url>/' \
    -e 's/<realm><\/realm>/<realm>'"${KEYCLOAK_REALM}"'<\/realm>/' \
    -e 's/<secret><\/secret>/<secret>'"${KEYCLOAK_SHARE_SECRET}"'<\/secret>/'\
    -e '1d' |\
    sed -i \
    -e '/<\/alfresco-config>/r /dev/stdin' \
    -e '$d' \
    ${TOMCAT_DIR}/shared/classes/alfresco/web-extension/share-config-custom.xml
COPY de.acosix.alfresco.utility.core.share-1.3.0.amp ${TOMCAT_DIR}/amps_share/

# apply amps
RUN java -jar ${TOMCAT_DIR}/alfresco-mmt/alfresco-mmt*.jar install \
              ${TOMCAT_DIR}/amps_share ${TOMCAT_DIR}/webapps/share -directory -nobackup -force


COPY de.acosix.alfresco.keycloak.share-1.1.0-rc7-withBouncy.amp ${TOMCAT_DIR}/amps_share/
#COPY de.acosix.alfresco.keycloak.share-1.1.0-rc7.amp ${TOMCAT_DIR}/amps_share/

# apply amps
RUN java -jar ${TOMCAT_DIR}/alfresco-mmt/alfresco-mmt*.jar install \
              ${TOMCAT_DIR}/amps_share ${TOMCAT_DIR}/webapps/share -directory -nobackup -force

RUN ls ${TOMCAT_DIR}/amps_share
### Install AMP on share
# RUN java -jar ${TOMCAT_DIR}/alfresco-mmt/alfresco-mmt*.jar install \
#               ${TOMCAT_DIR}/amps_share/alfresco-governance-services-community-share-*.amp ${TOMCAT_DIR}/webapps/share -nobackup
RUN java -jar ${TOMCAT_DIR}/alfresco-mmt/alfresco-mmt*.jar install \
              ${TOMCAT_DIR}/amps_share/*.amp ${TOMCAT_DIR}/webapps/share -nobackup

ENTRYPOINT ["/usr/local/tomcat/shared/classes/alfresco/substituter.sh", "catalina.sh run"]

EXPOSE 8000