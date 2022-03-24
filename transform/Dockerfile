# Image provides an all-in-one (AIO) container in which to run core transformations for Alfresco Content Services.

# Tika is from Apache. See the license at http://www.apache.org/licenses/LICENSE-2.0.
# LibreOffice is from The Document Foundation. See the license at https://www.libreoffice.org/download/license/ or in /libreoffice.txt.
# ImageMagick is from ImageMagick Studio LLC. See the license at http://www.imagemagick.org/script/license.php or in /ImageMagick-license.txt.
# alfresco-pdf-renderer uses the PDFium library from Google Inc. See the license at https://pdfium.googlesource.com/pdfium/+/master/LICENSE or in /pdfium.txt.
FROM alfresco/alfresco-transform-core-aio:2.5.6 as builder

FROM debian:bullseye as pdf-renderer

RUN apt-get update\
    &&apt-get install -y curl git cmake g++ zlib1g-dev libpng-dev\
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /opt

RUN git clone https://github.com/femshima/pdf-renderer.git

WORKDIR /opt/pdf-renderer

RUN curl -OL https://github.com/bblanchon/pdfium-binaries/releases/latest/download/pdfium-linux-arm64.tgz\
    &&mkdir -p pdfium\
    &&tar xzvf pdfium-linux-arm64.tgz -C pdfium\
    &&rm pdfium-linux-arm64.tgz

RUN PDFium_DIR=/opt/pdf-renderer/pdfium\
    cmake -B ./build -DCMAKE_BUILD_TYPE=Release
RUN cmake --build ./build --config Release


FROM alpine:3.15.0

RUN apk update \
    && apk --no-cache add \
    openjdk11-jre-headless \
    libreoffice \
    imagemagick imagemagick-dev \
    exiftool \
    zlib \
    && rm -rf /var/cache/apk/*

RUN apk --no-cache add ca-certificates wget\
    &&wget -q -O /etc/apk/keys/sgerrand.rsa.pub https://alpine-pkgs.sgerrand.com/sgerrand.rsa.pub\
    &&wget https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.28-r0/glibc-2.28-r0.apk\
    &&apk add glibc-2.28-r0.apk\
    &&apk del ca-certificates wget\
    &&rm -rf /var/cache/apk/*

ENV JAVA_OPTS=""

# Set default user information
ARG GROUPNAME=Alfresco
ARG GROUPID=1000
ARG AIOUSERNAME=transform-all-in-one
ARG USERID=33017

ENV LIBREOFFICE_HOME /usr/lib/libreoffice

ENV IMAGEMAGICK_ROOT /usr/lib/ImageMagick-7.1.0
ENV IMAGEMAGICK_DYN /usr/lib/ImageMagick-7.1.0
ENV IMAGEMAGICK_EXE /usr/bin/convert

ENV PDFRENDERER_EXE /usr/bin/pdf-renderer

COPY --from=pdf-renderer /opt/pdf-renderer/build/src/pdf-renderer /usr/bin

COPY --from=builder /usr/bin/alfresco-transform-core-aio-boot.jar /usr/bin
COPY --from=builder /licenses /licenses

RUN addgroup -g ${GROUPID} ${GROUPNAME} && \
    adduser -S -u ${USERID} -G ${GROUPNAME} ${AIOUSERNAME} && \
    chgrp -R ${GROUPNAME} /usr/bin/alfresco-transform-core-aio-boot.jar

EXPOSE 8090

USER ${AIOUSERNAME}

ENTRYPOINT java $JAVA_OPTS -jar /usr/bin/alfresco-transform-core-aio-boot.jar