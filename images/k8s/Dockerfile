# Slim image focused on deploying to Kubernetes clusters (cluster_from_env + deploy).
# Build context must be the repository root: docker build -f images/k8s/Dockerfile .
FROM node:22-alpine3.22

# Keep the npm package versions in sync with the root Dockerfile (full image).
RUN npm i -g ejs-cli ts-node@10.9.2 typescript@5.9.3 @cubos/kube-templates@1.1.0 @types/node@~24 && npm cache clean --force

RUN apk add --no-cache bash curl jq gettext ca-certificates kubectl python3 py3-pip py3-yaml && \
    pip install --no-cache-dir --break-system-packages yq

WORKDIR /

COPY kubectl/* /bin/
COPY images/k8s/github_action /bin/github_action
