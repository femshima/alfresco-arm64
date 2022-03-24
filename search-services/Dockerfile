FROM alfresco/alfresco-search-services:2.0.2.2 as builder


FROM openjdk:11.0.13-oraclelinux8

ENV DIST_DIR /opt/alfresco-search-services
ENV LANG C.UTF-8

# Get values from ENV VARS or use default values if ENV VARS are not specified
ENV SOLR_DATA_DIR_ROOT=${SOLR_DATA_DIR_ROOT:-$DIST_DIR/data}
ENV SOLR_SOLR_MODEL_DIR=${SOLR_SOLR_MODEL_DIR:-$DIST_DIR/data/alfrescoModels}

ARG USERNAME=solr
ARG USERID=33007

COPY --from=builder /opt/ /opt/

RUN set -x \
    && useradd \
    -c "Alfresco ${USERNAME}" \
    -M \
    -s "/bin/bash" \
    -u "${USERID}" \
    -o \
    "${USERNAME}" \
    && microdnf update -y \
    && microdnf install -y lsof ca-certificates procps \
    && microdnf clean all \
    && chown -R ${USERNAME}:${USERNAME} $DIST_DIR \
    && echo '#Docker Setup' >> $DIST_DIR/solr.in.sh \
    && echo 'SOLR_JAVA_STACK_SIZE="-Xss512k" SOLR_OPTS="$SOLR_OPTS -Dsolr.data.dir.root=$SOLR_DATA_DIR_ROOT -Dsolr.solr.model.dir=$SOLR_SOLR_MODEL_DIR"' >> $DIST_DIR/solr.in.sh

COPY --from=builder $DIST_DIR/solr/bin/ $DIST_DIR/solr/bin/
RUN chmod +x $DIST_DIR/solr/bin/search_config_setup.sh

WORKDIR $DIST_DIR

VOLUME $DIST_DIR/data
VOLUME $DIST_DIR/solrhome
# Expose a folder to mount keystores in the host (required for Mutual TLS Auth)
VOLUME $DIST_DIR/keystores

# SOLR Service Port
EXPOSE 8983


COPY --from=builder /usr/local/YourKit-JavaProfiler* /usr/local/
COPY --from=builder /licenses /licenses

# YourKit Profiling Port
EXPOSE 10001

USER ${USERNAME}
CMD $DIST_DIR/solr/bin/search_config_setup.sh "$DIST_DIR/solr/bin/solr start -f"