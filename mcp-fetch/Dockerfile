# Use a Python image with uv pre-installed
FROM ghcr.io/astral-sh/uv:python3.12-bookworm-slim AS uv

# Install the project into `/app`
WORKDIR /app

# Enable bytecode compilation
ENV UV_COMPILE_BYTECODE=1

# Copy from the cache instead of linking since it's a mounted volume
ENV UV_LINK_MODE=copy

# Install the project's dependencies using the lockfile and settings
COPY ./src/src/fetch/ .
RUN uv sync --locked --no-install-project --no-dev --no-editable

# Then, add the rest of the project source code and install it
# Installing separately from its dependencies allows optimal layer caching
COPY ./src/src/fetch/src/mcp_server_fetch/ /app
COPY ./src/tsconfig.json /tsconfig.json

RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --locked --no-dev --no-editable

ARG BUILD_FROM
FROM ${BUILD_FROM:-ghcr.io/hassio-addons/base:20.0.1} AS release

# Copy root filesystem
COPY rootfs /

COPY package.json /app/package.json
COPY --from=uv /root/.local /root/.local
COPY --from=uv --chown=app:app /app/.venv /app/.venv

RUN apk add --no-cache nodejs==24.13.0-r1 npm==11.6.3-r0 python3=3.12.12-r0

ENV NODE_ENV=production

WORKDIR /app

RUN --mount=type=cache,target=/root/.npm npm install
RUN --mount=type=cache,target=/root/.npm-production npm ci --ignore-scripts

# Place executables in the environment at the front of the path
ENV PATH="/app/.venv/bin:$PATH"

EXPOSE 8000