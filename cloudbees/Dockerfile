FROM registry.access.redhat.com/ubi8/openjdk-11:1.3 

LABEL maintainer="product-security@cloudbees.com" \
      securitytxt="https://www.cloudbees.com/.well-known/security.txt"

# Passed as build-arg from Kaniko executor
ARG WSS_AGENT_VERSION
ENV X_MODULE_ANALYZER_VERSION 19.9.2.2
ENV WSS_AGENT_URL https://unified-agent.s3.amazonaws.com/wss-unified-agent-${WSS_AGENT_VERSION}.jar
ENV WSS_INIT_CONFIG_FILE https://github.com/whitesource/unified-agent-distribution/raw/${WSS_AGENT_VERSION}/standAlone/wss-unified-agent.config
ENV WSS_ANALYZER_URL https://unified-agent.s3.amazonaws.com/xModuleAnalyzer/xModuleAnalyzer-${X_MODULE_ANALYZER_VERSION}.jar

ENV USER_ID 1000
ENV HOME_DIR /app

# Install Go
# RUN curl -o go-latest-amd64.tar.gz https://dl.google.com/go/go1.13.12.linux-amd64.tar.gz
# RUN tar xzvf go-latest-amd64.tar.gz 
# RUN rm -rf go-latest-amd64.tar.gz
# RUN ln -sf $HOME_DIR/go/ /opt/go

USER root

# Update packages and install nodejs
RUN dnf update -y && \
    dnf module install nodejs:12 -y

#Remove suid
RUN find / -perm /6000 -type f -exec chmod a-s {} \; 2> /dev/null | wc 

# Create wss user 
RUN groupadd -r wss && \
    useradd -m -r -s /bin/false -d $HOME_DIR -u $USER_ID -g wss wss

USER wss 
RUN mkdir -p $HOME_DIR/logs

# Create go env var and link go future download directory to /opt near java install
#ENV PATH="/opt/go/bin:${PATH}"
#ENV GOROOT=/opt/go

VOLUME "/data" "$HOME_DIR/logs"

WORKDIR $HOME_DIR

# Download the whitesource agent
RUN curl -o wss-unified-agent.jar -SL ${WSS_AGENT_URL} && \
    curl -o wss-unified-agent.config -SL ${WSS_INIT_CONFIG_FILE} && \
    curl -o xModuleAnalyzer.jar -SL ${WSS_ANALYZER_URL}

# Take md5sum for info, even though for now, whitesource does not provide anything.
RUN md5sum wss-unified-agent.jar \
    && md5sum xModuleAnalyzer.jar

COPY --chown=wss:wss entrypoint.sh entrypoint.sh
#COPY --chown=wss:wss wss-config-examples configs

ENTRYPOINT [ "/app/entrypoint.sh" ]
