# syntax=docker/dockerfile:1
FROM alpine:3.21

RUN apk add --no-cache curl tar git

ARG TARGETARCH
ENV TARGETARCH=${TARGETARCH}

WORKDIR /app

RUN set -eux; \
    # choose the right asset for this build platform
    case "${TARGETARCH}" in \
        amd64) SUFFIX="linux-x64.tgz"  ;; \
        arm64) SUFFIX="linux-arm64.tgz" ;; \
        *)     echo "unsupported arch ${TARGETARCH}" >&2; exit 1 ;; \
    esac; \
    # fetch release metadata once
    RELEASE_JSON=$(curl -fsSL https://api.github.com/repos/mongodb/kingfisher/releases/latest); \
    LATEST_URL=$(printf '%s' "$RELEASE_JSON" | grep -Eo "https://[^\"]*${SUFFIX}\"" | head -1 | tr -d '"'); \
    if [ -z "$LATEST_URL" ]; then \
        echo "Failed to fetch the latest release URL for ${SUFFIX}" >&2; \
        exit 1; \
    fi; \
    # derive the checksum file URL (same base name, .txt extension)
    CHECKSUM_URL=$(printf '%s' "$RELEASE_JSON" | grep -Eo "https://[^\"]*CHECKSUM-[^\"]*\.txt\"" | head -1 | tr -d '"'); \
    curl -fsSL "$LATEST_URL" -o kingfisher.tgz; \
    # verify checksum when the release provides one
    if [ -n "$CHECKSUM_URL" ]; then \
        curl -fsSL "$CHECKSUM_URL" -o CHECKSUM.txt; \
        sha256sum -c CHECKSUM.txt --ignore-missing; \
        rm -f CHECKSUM.txt; \
    else \
        echo "WARNING: no checksum file found for this release – skipping integrity check" >&2; \
    fi; \
    tar -xzf kingfisher.tgz; \
    rm -f kingfisher.tgz; \
    # locate the binary (pattern covers kingfisher-linux-x64 / kingfisher-linux-arm64)
    KF_PATH=$(find . -type f -name 'kingfisher*' -executable -print -quit); \
    if [ -z "$KF_PATH" ]; then echo "No executable kingfisher binary found" >&2; exit 1; fi; \
    install -m 0755 "$KF_PATH" /usr/local/bin/kingfisher; \
    # optional cleanup to keep the image small
    rm -rf /app/*

# quick smoke-test so the build fails early if something’s wrong
RUN kingfisher --version

ENTRYPOINT ["kingfisher"]
