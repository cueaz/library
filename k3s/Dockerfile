# syntax=docker/dockerfile:1
# renovate: datasource=github-releases depName=k3s packageName=k3s-io/k3s
ARG K3S_VERSION=v1.33.1+k3s1
# renovate: datasource=rpm registryUrl=https://nvidia.github.io/libnvidia-container/stable/rpm/aarch64/repodata depName=nvidia-container-toolkit packageName=nvidia-container-toolkit
ARG NVIDIA_CONTAINER_TOOLKIT_VERSION=1.17.8-1
ARG UBI_VERSION=9

FROM "registry.access.redhat.com/ubi$UBI_VERSION/ubi-minimal"

ARG K3S_VERSION
ARG NVIDIA_CONTAINER_TOOLKIT_VERSION

RUN set -eux; \
    ls -l;

ENV NVIDIA_VISIBLE_DEVICES=all
ENV NVIDIA_DRIVER_CAPABILITIES=compute,utility
