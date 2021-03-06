FROM maven:3.6.3-jdk-8 AS builder

WORKDIR /app

COPY pom.xml .
COPY src  ./src
COPY --from=ufcgsaps/common /app/saps-common-0.0.1-SNAPSHOT.jar ./local-maven-repo/saps-common-0.0.1-SNAPSHOT.jar
COPY --from=ufcgsaps/catalog:artefact-latest /app/saps-catalog-0.0.1-SNAPSHOT.jar ./local-maven-repo/saps-catalog-0.0.1-SNAPSHOT.jar

RUN mvn deploy:deploy-file -DgroupId=org.fogbowcloud -DartifactId=saps-common -Dversion=0.0.1-SNAPSHOT -Durl=file:./local-maven-repo/ -DrepositoryId=local-maven-repo -DupdateReleaseInfo=true -Dfile=./local-maven-repo/saps-common-0.0.1-SNAPSHOT.jar -Dpackaging=jar
RUN mvn deploy:deploy-file -DgroupId=org.fogbowcloud -DartifactId=saps-catalog -Dversion=0.0.1-SNAPSHOT -Durl=file:./local-maven-repo/ -DrepositoryId=local-maven-repo -DupdateReleaseInfo=true -Dfile=./local-maven-repo/saps-catalog-0.0.1-SNAPSHOT.jar -Dpackaging=jar
RUN mvn -e -B clean install

FROM openjdk:8

LABEL maintainer="wesley.monte@ccc.ufcg.edu.br"

ENV MAIN_CLASS="saps.dispatcher.core.SubmissionDispatcherMain"

# Create bind mounts for these three entries
ENV SERVICE_CONF_PATH="/etc/saps/dispatcher.conf"
ENV LOG_PROPERTIES_PATH="/etc/saps/log4j.properties"
ENV EXECUTION_TAGS_FILE_PATH="/etc/saps/execution_script_tags.json"
ENV TEMP_STORAGE_DIR="/nfs"

RUN apt-get update && apt-get install -y --no-install-recommends \
    curl jq sed \
    python \
    python-swiftclient \
    python-gdal \
    python-shapely

WORKDIR /dispatcher

COPY --from=builder /app/target/saps-dispatcher-0.0.1-SNAPSHOT.jar dispatcher.jar
COPY --from=builder /app/target/lib lib/
COPY ./resources resources/
COPY ./scripts  scripts/

ENTRYPOINT java -Dlog4j.configuration=file:"${LOG_PROPERTIES_PATH}" -cp dispatcher.jar:lib/* "${MAIN_CLASS}" "${SERVICE_CONF_PATH}" "${EXECUTION_TAGS_FILE_PATH}"