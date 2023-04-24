# Copyright Confidential Containers Contributors
#
# SPDX-License-Identifier: Apache-2.0
#
# Builds pod vm image inside container
#
# syntax=docker/dockerfile:1.3
ARG BUILDER_IMG="quay.io/confidential-containers/podvm-builder-ubuntu"
ARG BINARIES_IMG="quay.io/confidential-containers/podvm-binaries-ubuntu-amd64"

FROM ${BINARIES_IMG} AS podvm_binaries
FROM ${BUILDER_IMG} AS podvm_builder

ARG CLOUD_PROVIDER=azure
ARG PODVM_DISTRO=ubuntu
# If not provided, uses system architecture
ARG ARCH=x86_64
ARG CAA_SRC=""
ARG CAA_SRC_REF=""

ENV CLOUD_PROVIDER ${CLOUD_PROVIDER}
ENV PODVM_DISTRO ${PODVM_DISTRO}

RUN if [ -n "${CAA_SRC}" ]; then \
      rm -rf cloud-api-adaptor && \
      git clone ${CAA_SRC} cloud-api-adaptor;\
    fi && \
    if [ -n "${CAA_SRC_REF}" ]; then \
      cd cloud-api-adaptor && \
      git fetch origin ${CAA_SRC_REF} && \
      git checkout FETCH_HEAD -b ${CAA_SRC_REF} ;\
    fi

# Copy the binaries to podvm/files folder
COPY --from=podvm_binaries /podvm-binaries.tar.gz /src/cloud-api-adaptor/podvm/files
RUN tar xvf /src/cloud-api-adaptor/podvm/files/podvm-binaries.tar.gz -C /src/cloud-api-adaptor/podvm/files

# Copy the pause_bundle to podvm/files folder
COPY --from=podvm_binaries /pause-bundle.tar.gz /src/cloud-api-adaptor/podvm/files
RUN tar xvf /src/cloud-api-adaptor/podvm/files/pause-bundle.tar.gz -C /src/cloud-api-adaptor/podvm/files

ARG AZURE_REGION=eastus
ARG VM_SIZE=Standard_A2_v2
ARG AZURE_RESOURCE_GROUP
ARG IMAGE_NAME=peer-pod-vmimage
ARG SSH_USERNAME=peerpod
ARG PUBLISHER=Canonical
ARG OFFER=0001-com-ubuntu-minimal-jammy
ARG SKU=minimal-22_04-lts
ARG PLAN_NAME
ARG PLAN_PRODUCT
ARG PLAN_PUBLISHER

ENV AZURE_REGION ${AZURE_REGION}
ENV AZURE_RESOURCE_GROUP ${AZURE_RESOURCE_GROUP}
ENV VM_SIZE ${VM_SIZE}
ENV IMAGE_NAME ${IMAGE_NAME}
ENV PUBLISHER ${PUBLISHER}
ENV OFFER ${OFFER}
ENV SKU ${SKU}
ENV PLAN_NAME ${PLAN_NAME}
ENV PLAN_PRODUCT ${PLAN_PRODUCT}
ENV PLAN_PUBLISHER ${PLAN_PUBLISHER}

RUN --mount=type=secret,id=AZURE_SUBSCRIPTION_ID \
    --mount=type=secret,id=AZURE_CLIENT_ID \
    --mount=type=secret,id=AZURE_CLIENT_SECRET \
    --mount=type=secret,id=AZURE_TENANT_ID \
    export AZURE_SUBSCRIPTION_ID=$(cat /run/secrets/AZURE_SUBSCRIPTION_ID) && \
    export AZURE_CLIENT_ID=$(cat /run/secrets/AZURE_CLIENT_ID) && \
    export AZURE_CLIENT_SECRET=$(cat /run/secrets/AZURE_CLIENT_SECRET) && \
    export AZURE_TENANT_ID=$(cat /run/secrets/AZURE_TENANT_ID) && \
    cd cloud-api-adaptor/azure/image && \
     BINARIES= PAUSE_BUNDLE= CLOUD_PROVIDER=azure PODVM_DISTRO=$PODVM_DISTRO make image