FROM node:22-bookworm-slim AS build

ENV PYTHON=/usr/bin/python3
RUN apt-get update && \
    apt-get install -y --no-install-recommends python3 g++ build-essential libsqlite3-dev && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY . .

RUN corepack enable && yarn install --immutable
RUN APP_BASE_URL=http://localhost BACKEND_BASE_URL=http://localhost yarn build:all

FROM node:22-bookworm-slim AS runner

ENV PYTHON=/usr/bin/python3
ENV NODE_ENV=production
ENV NODE_OPTIONS="--no-node-snapshot"

RUN apt-get update && \
    apt-get install -y --no-install-recommends python3 g++ build-essential libsqlite3-dev && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /app
USER node

COPY --chown=node:node --from=build /app/.yarn ./.yarn
COPY --chown=node:node --from=build /app/.yarnrc.yml ./
COPY --chown=node:node --from=build /app/backstage.json ./
COPY --chown=node:node --from=build /app/yarn.lock ./
COPY --chown=node:node --from=build /app/package.json ./
COPY --chown=node:node --from=build /app/packages/backend/dist/skeleton.tar.gz ./
RUN tar xzf skeleton.tar.gz && rm skeleton.tar.gz

RUN corepack enable && yarn workspaces focus --all --production

COPY --chown=node:node --from=build /app/packages/backend/dist/bundle.tar.gz ./
RUN tar xzf bundle.tar.gz && rm bundle.tar.gz

COPY --chown=node:node --from=build /app/examples ./examples
COPY --chown=node:node --from=build /app/app-config*.yaml ./

CMD ["node", "packages/backend", "--config", "app-config.yaml", "--config", "app-config.production.yaml"]
