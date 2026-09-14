FROM node:22.14.0-alpine3.21 AS builder

WORKDIR /build

COPY package.json package-lock.json ./

RUN --mount=type=cache,target=/root/.npm \
    npm ci --cache /root/.npm

COPY . .

ARG VUE_APP_API_URL=http://localhost:8081
ENV VUE_APP_API_URL=${VUE_APP_API_URL}

RUN NODE_OPTIONS=--openssl-legacy-provider \
    npm run build && rm -rf ./node_modules

# deployment image
FROM nginxinc/nginx-unprivileged:1.31-alpine3.24 AS runtime

ARG VERSION=0.0.67

LABEL maintainer="maintainer@example.com"
LABEL description="Momo Store Frontend"
LABEL version="$VERSION"

COPY --from=builder --chown=101:101 /build/dist/ /usr/share/nginx/html/momo-store/
COPY --chown=101:101 nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 8080

USER 101:101

HEALTHCHECK --interval=30s --timeout=5s --start-period=15s --start-interval=5s --retries=3 \
    CMD curl -fsS http://127.0.0.1:8080/health || exit 1

ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["nginx", "-g", "daemon off;"]
