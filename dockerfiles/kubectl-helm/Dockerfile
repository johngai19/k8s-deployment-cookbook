FROM alpine:3.21@sha256:a8560b36e8b8210634f77d9f7f9efd7ffa463e380b75e2e74aff4511df3ef88c

# Install necessary tools + mysql client
RUN apk add --no-cache \
      curl tar ca-certificates bash python3 py3-pip \
      mariadb-client

# Fetch the latest stable kubectl version and install it
ARG KUBECTL_VERSION=$(curl -sL https://dl.k8s.io/release/stable.txt)
RUN curl -sL "https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl" \
    -o /usr/local/bin/kubectl && \
    chmod +x /usr/local/bin/kubectl

# Fetch the latest Helm version and install it 
ARG HELM_VERSION=$(curl -sL https://api.github.com/repos/helm/helm/releases/latest \
    | grep '"tag_name":' | sed -E 's/.*"([^"]+)".*/\1/')
RUN curl -sL "https://get.helm.sh/helm-${HELM_VERSION}-linux-amd64.tar.gz" \
    -o helm.tar.gz && \
    tar -zxvf helm.tar.gz && \
    mv linux-amd64/helm /usr/local/bin/helm && \
    rm -rf helm.tar.gz linux-amd64

# Install MinIO client for OSS (S3/MinIO) operations
RUN curl -sL https://dl.min.io/client/mc/release/linux-amd64/mc \
    -o /usr/local/bin/mc && chmod +x /usr/local/bin/mc

# Expose a mount-point for external Helm‐chart files
RUN mkdir -p /charts
VOLUME ["/charts"]

CMD ["bash"]
