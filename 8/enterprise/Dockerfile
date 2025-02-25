FROM alpine:3.12

ENV JAVA_VERSION="jdk-11.0.10+9" \
    LANG='en_US.UTF-8' \
    LANGUAGE='en_US:en' \
    LC_ALL='en_US.UTF-8'

#
# glibc setup
#
RUN set -eux; \
    apk add --no-cache tzdata --virtual .build-deps curl binutils zstd; \
    GLIBC_VER="2.32-r0"; \
    ALPINE_GLIBC_REPO="https://github.com/sgerrand/alpine-pkg-glibc/releases/download"; \
    GCC_LIBS_URL="https://archive.archlinux.org/packages/g/gcc-libs/gcc-libs-10.1.0-2-x86_64.pkg.tar.zst"; \
    GCC_LIBS_SHA256="f80320a03ff73e82271064e4f684cd58d7dbdb07aa06a2c4eea8e0f3c507c45c"; \
    ZLIB_URL="https://archive.archlinux.org/packages/z/zlib/zlib-1%3A1.2.11-3-x86_64.pkg.tar.xz"; \
    ZLIB_SHA256=17aede0b9f8baa789c5aa3f358fbf8c68a5f1228c5e6cba1a5dd34102ef4d4e5; \
    curl -LfsS https://alpine-pkgs.sgerrand.com/sgerrand.rsa.pub -o /etc/apk/keys/sgerrand.rsa.pub; \
    SGERRAND_RSA_SHA256="823b54589c93b02497f1ba4dc622eaef9c813e6b0f0ebbb2f771e32adf9f4ef2"; \
    echo "${SGERRAND_RSA_SHA256} */etc/apk/keys/sgerrand.rsa.pub" | sha256sum -c - ; \
    curl -LfsS ${ALPINE_GLIBC_REPO}/${GLIBC_VER}/glibc-${GLIBC_VER}.apk > /tmp/glibc-${GLIBC_VER}.apk; \
    apk add --no-cache /tmp/glibc-${GLIBC_VER}.apk; \
    curl -LfsS ${ALPINE_GLIBC_REPO}/${GLIBC_VER}/glibc-bin-${GLIBC_VER}.apk > /tmp/glibc-bin-${GLIBC_VER}.apk; \
    apk add --no-cache /tmp/glibc-bin-${GLIBC_VER}.apk; \
    curl -Ls ${ALPINE_GLIBC_REPO}/${GLIBC_VER}/glibc-i18n-${GLIBC_VER}.apk > /tmp/glibc-i18n-${GLIBC_VER}.apk; \
    apk add --no-cache /tmp/glibc-i18n-${GLIBC_VER}.apk; \
    /usr/glibc-compat/bin/localedef --inputfile en_US --charmap UTF-8 "$LANG" || true ;\
    echo "export LANG=$LANG" > /etc/profile.d/locale.sh; \
    curl -LfsS ${GCC_LIBS_URL} -o /tmp/gcc-libs.tar.zst; \
    echo "${GCC_LIBS_SHA256} */tmp/gcc-libs.tar.zst" | sha256sum -c - ; \
    mkdir /tmp/gcc; \
    zstd -d /tmp/gcc-libs.tar.zst --output-dir-flat /tmp; \
    tar -xf /tmp/gcc-libs.tar -C /tmp/gcc; \
    mv /tmp/gcc/usr/lib/libgcc* /tmp/gcc/usr/lib/libstdc++* /usr/glibc-compat/lib; \
    strip /usr/glibc-compat/lib/libgcc_s.so.* /usr/glibc-compat/lib/libstdc++.so*; \
    curl -LfsS ${ZLIB_URL} -o /tmp/libz.tar.xz; \
    echo "${ZLIB_SHA256} */tmp/libz.tar.xz" | sha256sum -c - ;\
    mkdir /tmp/libz; \
    tar -xf /tmp/libz.tar.xz -C /tmp/libz; \
    mv /tmp/libz/usr/lib/libz.so* /usr/glibc-compat/lib; \
    apk del --purge .build-deps glibc-i18n; \
    rm -rf /tmp/*.apk /tmp/gcc /tmp/gcc-libs.tar* /tmp/libz /tmp/libz.tar.xz /var/cache/apk/*;

#
# AdoptOpenJDK/openjdk11 setup
#
RUN set -eux; \
    apk add --no-cache --virtual .fetch-deps curl; \
    ARCH="$(apk --print-arch)"; \
    case "${ARCH}" in \
       aarch64|arm64) \
         ESUM='5f9a894bd694f598f2befa4a605169685ac8bcb8ec68d25e587e8db4d2307b74'; \
         BINARY_URL='https://github.com/AdoptOpenJDK/openjdk11-binaries/releases/download/jdk-11.0.10%2B9/OpenJDK11U-jre_aarch64_linux_hotspot_11.0.10_9.tar.gz'; \
         ;; \
       armhf|armv7l) \
         ESUM='2f2da2149c089c84f00b0eda63c31b77c8b51a1c080e18a70ecb5a78ba40d8c6'; \
         BINARY_URL='https://github.com/AdoptOpenJDK/openjdk11-binaries/releases/download/jdk-11.0.10%2B9/OpenJDK11U-jre_arm_linux_hotspot_11.0.10_9.tar.gz'; \
         ;; \
       ppc64el|ppc64le) \
         ESUM='d269b646af32eb41d74b3a5259f634921a063c67642ab5c227142463824b2a6d'; \
         BINARY_URL='https://github.com/AdoptOpenJDK/openjdk11-binaries/releases/download/jdk-11.0.10%2B9/OpenJDK11U-jre_ppc64le_linux_hotspot_11.0.10_9.tar.gz'; \
         ;; \
       s390x) \
         ESUM='2c9ec28b10bf1628b20a157c746988f323e0dcbf1053b616aa6593923e3a70df'; \
         BINARY_URL='https://github.com/AdoptOpenJDK/openjdk11-binaries/releases/download/jdk-11.0.10%2B9/OpenJDK11U-jre_s390x_linux_hotspot_11.0.10_9.tar.gz'; \
         ;; \
       amd64|x86_64) \
         ESUM='25fdcf9427095ac27c8bdfc82096ad2e615693a3f6ea06c700fca7ffb271131a'; \
         BINARY_URL='https://github.com/AdoptOpenJDK/openjdk11-binaries/releases/download/jdk-11.0.10%2B9/OpenJDK11U-jre_x64_linux_hotspot_11.0.10_9.tar.gz'; \
         ;; \
       *) \
         echo "Unsupported arch: ${ARCH}"; \
         exit 1; \
         ;; \
    esac; \
    curl -LfsSo /tmp/openjdk.tar.gz ${BINARY_URL}; \
    echo "${ESUM} */tmp/openjdk.tar.gz" | sha256sum -c -; \
    mkdir -p /opt/java/openjdk; \
    cd /opt/java/openjdk; \
    tar -xf /tmp/openjdk.tar.gz --strip-components=1; \
    apk del --purge .fetch-deps; \
    rm -rf /var/cache/apk/*; \
    rm -rf /tmp/openjdk.tar.gz;

#
# SonarQube setup
#
ARG SONARQUBE_VERSION=8.7.1.42226
ARG SONARQUBE_ZIP_URL=https://binaries.sonarsource.com/CommercialDistribution/sonarqube-enterprise/sonarqube-enterprise-${SONARQUBE_VERSION}.zip
ENV JAVA_HOME=/opt/java/openjdk \
    PATH="/opt/java/openjdk/bin:$PATH" \
    SONARQUBE_HOME=/opt/sonarqube \
    SONAR_VERSION="${SONARQUBE_VERSION}" \
    SQ_DATA_DIR="/opt/sonarqube/data" \
    SQ_EXTENSIONS_DIR="/opt/sonarqube/extensions" \
    SQ_LOGS_DIR="/opt/sonarqube/logs" \
    SQ_TEMP_DIR="/opt/sonarqube/temp"

RUN set -eux; \
    addgroup -S -g 1000 sonarqube; \
    adduser -S -D -u 1000 -G sonarqube sonarqube; \
    apk add --no-cache --virtual build-dependencies gnupg unzip curl; \
    apk add --no-cache bash su-exec ttf-dejavu; \
    # pub   2048R/D26468DE 2015-05-25
    #       Key fingerprint = F118 2E81 C792 9289 21DB  CAB4 CFCA 4A29 D264 68DE
    # uid                  sonarsource_deployer (Sonarsource Deployer) <infra@sonarsource.com>
    # sub   2048R/06855C1D 2015-05-25
    sed --in-place --expression="s?securerandom.source=file:/dev/random?securerandom.source=file:/dev/urandom?g" "${JAVA_HOME}/conf/security/java.security"; \
    for server in $(shuf -e ha.pool.sks-keyservers.net \
                            hkp://p80.pool.sks-keyservers.net:80 \
                            keyserver.ubuntu.com \
                            hkp://keyserver.ubuntu.com:80 \
                            pgp.mit.edu) ; do \
        gpg --batch --keyserver "${server}" --recv-keys F1182E81C792928921DBCAB4CFCA4A29D26468DE && break || : ; \
    done; \
    mkdir --parents /opt; \
    cd /opt; \
    curl --fail --location --output sonarqube.zip --silent --show-error "${SONARQUBE_ZIP_URL}"; \
    curl --fail --location --output sonarqube.zip.asc --silent --show-error "${SONARQUBE_ZIP_URL}.asc"; \
    gpg --batch --verify sonarqube.zip.asc sonarqube.zip; \
    unzip -q sonarqube.zip; \
    mv "sonarqube-${SONARQUBE_VERSION}" sonarqube; \
    rm sonarqube.zip*; \
    rm -rf ${SONARQUBE_HOME}/bin/*; \
    chown -R sonarqube:sonarqube ${SONARQUBE_HOME}; \
    # this 777 will be replaced by 700 at runtime (allows semi-arbitrary "--user" values)
    chmod -R 777 "${SQ_DATA_DIR}" "${SQ_EXTENSIONS_DIR}" "${SQ_LOGS_DIR}" "${SQ_TEMP_DIR}"; \
    apk del --purge build-dependencies;
    
COPY --chown=sonarqube:sonarqube run.sh sonar.sh ${SONARQUBE_HOME}/bin/

WORKDIR ${SONARQUBE_HOME}
EXPOSE 9000
ENTRYPOINT ["bin/run.sh"]
CMD ["bin/sonar.sh"]
