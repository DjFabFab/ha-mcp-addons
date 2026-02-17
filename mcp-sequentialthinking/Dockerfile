FROM node:24.12-alpine AS builder

COPY src/src/sequentialthinking /app
COPY src/tsconfig.json /tsconfig.json

WORKDIR /app

RUN --mount=type=cache,target=/root/.npm npm install

RUN --mount=type=cache,target=/root/.npm-production npm ci --ignore-scripts

ARG BUILD_FROM
FROM ${BUILD_FROM:-ghcr.io/hassio-addons/base:20.0.1} AS release

# Copy root filesystem
COPY rootfs /

COPY --from=builder /app/dist /app/dist
COPY --from=builder /app/package.json /app/package.json
COPY --from=builder /app/package-lock.json /app/package-lock.json

RUN apk add --no-cache nodejs==24.13.0-r1 npm==11.6.3-r0

ENV NODE_ENV=production

WORKDIR /app

RUN npm ci --ignore-scripts

EXPOSE 8000
