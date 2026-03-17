FROM golang:1.25.1-bookworm as builder

ARG GCLUSTER_VERSION=1.1

RUN apt-get update && apt-get install -y \
    git \
    make \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app
RUN git clone https://github.com/GoogleCloudPlatform/cluster-toolkit.git .
RUN [ -z "$GCLUSTER_VERSION" ] || git checkout "$GCLUSTER_VERSION"

RUN make gcluster

FROM gcr.io/cloud-builders/gcloud

# Define version and architecture
ARG TERRAFORM_VERSION=1.12.2
ARG PACKER_VERSION=1.12.0
ARG ARCH=amd64

# RUN apt-get update && apt-get install -y gnupg software-properties-common curl wget         \
#     && install -m 0755 -d /etc/apt/keyrings         \
#     && curl -fsSL https://apt.releases.hashicorp.com/gpg | gpg --dearmor -o /etc/apt/keyrings/hashicorp.gpg         \
#     && chmod a+r /etc/apt/keyrings/hashicorp.gpg         \
#     && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/hashicorp.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/hashicorp.list > /dev/null         \
#     && apt-get update \
#     && apt-get install -y terraform=${TERRAFORM_VERSION:-} packer         \
#     && rm -rf /var/lib/apt/lists/*

# Install dependencies, download, unzip, and clean up
RUN apt-get update && apt-get install -y curl unzip && \
    curl -sLO https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_${ARCH}.zip && \
    unzip terraform_${TERRAFORM_VERSION}_linux_${ARCH}.zip && \
    mv terraform /usr/local/bin/ && \
    rm terraform_${TERRAFORM_VERSION}_linux_${ARCH}.zip && \
    apt-get clean && rm -rf /var/lib/apt/lists/*


# Install dependencies, download, unzip, and move to path
RUN curl -sLO https://releases.hashicorp.com/packer/${PACKER_VERSION}/packer_${PACKER_VERSION}_linux_${ARCH}.zip && \
    unzip packer_${PACKER_VERSION}_linux_${ARCH}.zip && \
    mv packer /usr/local/bin/ && \
    rm packer_${PACKER_VERSION}_linux_${ARCH}.zip && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

COPY --from=builder /app/gcluster /usr/local/bin/gcluster

WORKDIR /workspace

COPY entrypoint.bash /builder/entrypoint.bash
ENTRYPOINT ["/builder/entrypoint.bash"]
