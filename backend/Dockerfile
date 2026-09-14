# builer image
FROM golang:1.27.1-alpine3.24 AS builder

ARG GOOS=linux
ARG GOARCH=amd64

WORKDIR /build

COPY go.mod go.sum ./

RUN go mod download

COPY . .

RUN CGO_ENABLED=0 \
    GOOS="$GOOS" \
    GOARCH="$GOARCH" \
    go build -o app ./cmd/api

# deployment image
FROM alpine:3.24.1 AS runtime

ARG VERSION=0.0.67

LABEL maintainer="maintainer@example.com"
LABEL description="Momo Store Application"
LABEL version="$VERSION"

RUN addgroup -S appgroup && adduser -S appuser -G appgroup

RUN apk upgrade --no-cache && \
    apk add --no-cache curl && \
    mkdir -p /app/config /app/data /app/logs /app/tmp && \
    chown -R appuser:appgroup /app

USER appuser

WORKDIR /app

COPY --from=builder --chown=appuser:appgroup /build/app .

EXPOSE 8081

HEALTHCHECK --interval=30s --timeout=5s --start-period=15s --start-interval=5s --retries=3 \
    CMD curl -fsS http://127.0.0.1:8081/health || exit 1

ENTRYPOINT [ "./app" ]
