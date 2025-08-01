# syntax=docker/dockerfile:1.4

# NOTE: This Dockerfile can only be built using BuildKit. BuildKit is used by
# default when running `docker buildx build` or when DOCKER_BUILDKIT=1 is set
# in environment variables.

FROM --platform=$BUILDPLATFORM grafana/agent-build-image:0.44.4 as build
ARG BUILDPLATFORM
ARG TARGETPLATFORM
ARG TARGETOS
ARG TARGETARCH
ARG TARGETVARIANT
ARG RELEASE_BUILD=1
ARG VERSION

COPY . /src/agent
WORKDIR /src/agent

RUN --mount=type=cache,target=/root/.cache/go-build \
    --mount=type=cache,target=/go/pkg/mod \
    GOOS=$TARGETOS GOARCH=$TARGETARCH GOARM=${TARGETVARIANT#v} GO_TAGS=netgo \
    RELEASE_BUILD=${RELEASE_BUILD} VERSION=${VERSION} \
    make operator

FROM public.ecr.aws/ubuntu/ubuntu:noble

LABEL org.opencontainers.image.source="https://github.com/grafana/agent"

# Install dependencies needed at runtime.
RUN <<EOF
  apt-get update
  apt-get install -qy libsystemd-dev tzdata ca-certificates
  rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
EOF

COPY --from=build /src/agent/build/grafana-agent-operator /bin/grafana-agent-operator
ENTRYPOINT ["/bin/grafana-agent-operator"]
