ARG NODE_VERSION=20
ARG N8N_VERSION=1.83.2
FROM n8nio/base:${NODE_VERSION}

# Redeclare N8N_VERSION so it's available in this build stage
ARG N8N_VERSION

RUN if [ -z "$N8N_VERSION" ] ; then echo "The N8N_VERSION argument is missing!" ; exit 1; fi

LABEL org.opencontainers.image.title="n8n"
LABEL org.opencontainers.image.description="Workflow Automation Tool"
LABEL org.opencontainers.image.source="https://github.com/n8n-io/n8n"
LABEL org.opencontainers.image.url="https://n8n.io"
LABEL org.opencontainers.image.version=${N8N_VERSION}

ENV N8N_VERSION=${N8N_VERSION}
ENV NODE_ENV=production
ENV N8N_RELEASE_TYPE=stable
# Enforce settings file permissions as recommended
ENV N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true
# Enforce settings community codes permissions
ENV N8N_COMMUNITY_PACKAGES_ALLOW_TOOL_USAGE=true
# Enforce settings runner permissions as recommended
ENV N8N_RUNNERS_ENABLED=true
# Ensure both /usr/local/bin and /bin are in PATH
ENV PATH="/usr/local/bin:/bin:${PATH}"

RUN set -eux; \
    # Install n8n globally at version 1.83.2
    npm install -g --omit=dev n8n@${N8N_VERSION} --ignore-scripts && \
    # Install the MCP community node package globally
    npm install -g n8n-nodes-mcp && \
    # (Temporarily skip the MCP build step)
    # npm install -g typescript gulp-cli && \
    # cd /usr/local/lib/node_modules/n8n-nodes-mcp && \
    # tsc && gulp build:icons && \
    # cd - && \
    \
    # Rebuild sqlite3 dependency for n8n
    npm rebuild --prefix=/usr/local/lib/node_modules/n8n sqlite3 && \
    # Remove unnecessary components to reduce image size
    rm -rf /usr/local/lib/node_modules/n8n/node_modules/@n8n/chat && \
    rm -rf /usr/local/lib/node_modules/n8n/node_modules/@n8n/design-system && \
    rm -rf /usr/local/lib/node_modules/n8n/node_modules/n8n-editor-ui/node_modules && \
    # Remove source files and maps to reduce image size
    find /usr/local/lib/node_modules/n8n -type f -name "*.ts" -o -name "*.js.map" -o -name "*.vue" | xargs rm -f && \
    rm -rf /root/.npm

ARG TARGETPLATFORM
ARG LAUNCHER_VERSION=1.1.1
COPY docker/images/n8n/n8n-task-runners.json /etc/n8n-task-runners.json
# Download, verify, then extract the launcher binary
RUN \
    if [[ "$TARGETPLATFORM" = "linux/amd64" ]]; then export ARCH_NAME="amd64"; \
    elif [[ "$TARGETPLATFORM" = "linux/arm64" ]]; then export ARCH_NAME="arm64"; fi; \
    mkdir /launcher-temp && \
    cd /launcher-temp && \
    wget https://github.com/n8n-io/task-runner-launcher/releases/download/${LAUNCHER_VERSION}/task-runner-launcher-${LAUNCHER_VERSION}-linux-${ARCH_NAME}.tar.gz && \
    wget https://github.com/n8n-io/task-runner-launcher/releases/download/${LAUNCHER_VERSION}/task-runner-launcher-${LAUNCHER_VERSION}-linux-${ARCH_NAME}.tar.gz.sha256 && \
    echo "$(cat task-runner-launcher-${LAUNCHER_VERSION}-linux-${ARCH_NAME}.tar.gz.sha256) task-runner-launcher-${LAUNCHER_VERSION}-linux-${ARCH_NAME}.tar.gz" > checksum.sha256 && \
    sha256sum -c checksum.sha256 && \
    tar xvf task-runner-launcher-${LAUNCHER_VERSION}-linux-${ARCH_NAME}.tar.gz --directory=/usr/local/bin && \
    cd - && \
    rm -r /launcher-temp

COPY docker/images/n8n/docker-entrypoint.sh /

RUN \
    mkdir .n8n && \
    chown node:node .n8n
ENV SHELL=/bin/sh
USER node
ENTRYPOINT ["tini", "--", "/docker-entrypoint.sh"]
