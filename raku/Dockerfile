# syntax=docker/dockerfile:1

# renovate: datasource=github-releases depName=rakudo packageName=rakudo/star
ARG RAKUDO_VERSION=2024.12

ARG DEBIAN_VERSION=12
ARG DEBIAN_RELEASE=bookworm

FROM "buildpack-deps:$DEBIAN_RELEASE" AS builder

ARG RAKUDO_VERSION

WORKDIR /work

RUN set -eux; \
    url="https://github.com/rakudo/star/releases/download/$RAKUDO_VERSION/rakudo-star-$RAKUDO_VERSION.tar.gz"; \
    curl -fsSL "$url" -o rakudo.tar.gz; \
    mkdir rakudo; \
    tar -xzf rakudo.tar.gz --strip-components=1 -C rakudo; \
    RSTAR_DEBUG=1 bash rakudo/bin/rstar install -p /rootfs/usr/local core; \
    rm -r /rootfs/usr/local/include /rootfs/usr/local/share/pkgconfig; \
    machine="$(gcc -dumpmachine)"; \
    mkdir -p "/rootfs/usr/lib/$machine"; \
    cp "/usr/lib/$machine"/libzstd.so.* "/rootfs/usr/lib/$machine"; \
    find /rootfs;

FROM "gcr.io/distroless/base-nossl-debian$DEBIAN_VERSION:nonroot" AS runtime

COPY --link --from=builder /rootfs/ /

ENTRYPOINT ["raku"]

FROM runtime AS check

RUN ["raku", "--version"]
