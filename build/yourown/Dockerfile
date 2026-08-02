# syntax=docker/dockerfile:1.6
# Reproducible YourOwn.Chat RTCD image. The build context is this repository at
# the immutable source commit recorded in the OCI labels.
FROM golang:1.25.12-bookworm@sha256:ea341baa9bd5ba6784f6d7161ace70544349a6242d54d34a0fbfd2c4d51c9d58 AS builder

ARG TARGETOS=linux
ARG TARGETARCH=amd64
ARG RTCD_SOURCE_COMMIT
ARG RTCD_BUILD_VERSION
ARG RTCD_BUILD_DATE

ENV CGO_ENABLED=0
WORKDIR /src
COPY . .

RUN test -n "$RTCD_SOURCE_COMMIT" && \
    printf '%s\n' "$RTCD_SOURCE_COMMIT" | grep -Eq '^[0-9a-f]{40}$' && \
    test -n "$RTCD_BUILD_VERSION" && \
    test -n "$RTCD_BUILD_DATE" && \
    go mod verify && \
    go test -mod=readonly -run '^$' ./... && \
    GOOS="$TARGETOS" GOARCH="$TARGETARCH" \
    go build -mod=readonly -trimpath \
      -ldflags="-s -w \
        -X github.com/mattermost/rtcd/service.buildHash=${RTCD_SOURCE_COMMIT} \
        -X github.com/mattermost/rtcd/service.buildVersion=${RTCD_BUILD_VERSION} \
        -X github.com/mattermost/rtcd/service.buildDate=${RTCD_BUILD_DATE} \
        -X github.com/mattermost/rtcd/service.goVersion=1.25.12" \
      -o /out/rtcd ./cmd/rtcd

FROM gcr.io/distroless/static@sha256:d6fa9db9548b5772860fecddb11d84f9ebd7e0321c0cb3c02870402680cc315f

ARG RTCD_SOURCE_COMMIT
ARG RTCD_BUILD_VERSION
ARG RTCD_BUILD_DATE

LABEL org.opencontainers.image.title="YourOwn.Chat RTCD" \
      org.opencontainers.image.version="${RTCD_BUILD_VERSION}" \
      org.opencontainers.image.revision="${RTCD_SOURCE_COMMIT}" \
      org.opencontainers.image.created="${RTCD_BUILD_DATE}" \
      org.opencontainers.image.source="https://github.com/pilprod/yourown-chat-rtcd/tree/${RTCD_SOURCE_COMMIT}" \
      org.opencontainers.image.licenses="LicenseRef-Mattermost-Commercial-OR-AGPL-3.0"

COPY --from=builder /out/rtcd /opt/rtcd/bin/rtcd
COPY --from=builder /src/build/group /src/build/passwd /etc/
COPY --from=builder /src/LICENSE.txt /licenses/rtcd/LICENSE.txt

USER calls
WORKDIR /opt/rtcd/bin
ENTRYPOINT ["./rtcd"]
