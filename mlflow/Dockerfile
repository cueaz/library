# syntax=docker/dockerfile:1
# renovate: datasource=pypi depName=mlflow packageName=mlflow
ARG MLFLOW_VERSION=2.18.0

ARG DEBIAN_VERSION=12
ARG DEBIAN_RELEASE=bookworm

FROM "buildpack-deps:$DEBIAN_RELEASE" AS builder

ARG MLFLOW_VERSION

# ref: https://docs.docker.com/reference/dockerfile/#example-cache-apt-packages
RUN set -eux; \
    rm -f /etc/apt/apt.conf.d/docker-clean; \
    echo 'Binary::apt::APT::Keep-Downloaded-Packages "true";' > /etc/apt/apt.conf.d/keep-cache

RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    set -eux; \
    apt update; \
    apt install -y python3-pip;

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PIP_CACHE_DIR=/var/cache/pip

RUN --mount=type=cache,target="$PIP_CACHE_DIR" \
    set -eux; \
    pip install --prefix /rootfs/usr --no-compile mlflow=="$MLFLOW_VERSION" boto3; \
    mkdir /rootfs/etc; \
    printf '\
exec: mlflow server --host 0.0.0.0 --port 8000 --backend-store-uri sqlite:////data/db --serve-artifacts\n\
dbs:\n\
  - path: /data/db\n\
    replicas:\n\
      - url: $MLFLOW_ARTIFACTS_DESTINATION/db\n\
        endpoint: $MLFLOW_S3_ENDPOINT_URL\n\
    ' > /rootfs/etc/litestream.yml; \
    # ref: https://github.com/benbjohnson/litestream-docker-example/blob/main/scripts/run.sh
    printf '#!/usr/bin/python3\n\
import os\n\
import subprocess\n\
subprocess.run(["litestream", "restore", "-if-db-not-exists", "-if-replica-exists", "/data/db"], check=True)\n\
os.execlp("litestream", "litestream", "replicate")\n\
    ' > /rootfs/usr/local/bin/start; \
    chmod +x /rootfs/usr/local/bin/start; \
    mkdir /data;

FROM "gcr.io/distroless/python3-debian$DEBIAN_VERSION:nonroot" AS runtime

COPY --link --from=builder /rootfs /
# --link is not compatible with --chown, ci failed
# ref: https://github.com/moby/buildkit/issues/2987#issuecomment-1396753289
#COPY -from=builder --chown=nonroot:nonroot /data /data
COPY --link --from=builder --chown=65532:65532 /data /data

COPY --link --from=litestream/litestream /usr/local/bin/litestream /usr/local/bin/litestream

VOLUME /data

EXPOSE 8000

ENTRYPOINT ["start"]
