FROM --platform=$BUILDPLATFORM node:20-bookworm-slim AS builder

ENV PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=true

WORKDIR /src/unfurler
COPY package*.json ./
RUN npm ci
RUN apt-get update && \
    apt-get install -y --no-install-recommends rsync && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

COPY . .
RUN mkdir -p /src/frontend/dist/mempool/browser
COPY --from=frontend_dist browser/resources /src/frontend/dist/mempool/browser/resources
RUN npm run build && npm prune --omit=dev

FROM --platform=$TARGETPLATFORM node:20-bookworm-slim

ENV NODE_ENV=production
ENV PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=true
ENV UNFURLER_CONFIG=/app/config.json

RUN apt-get update && \
    apt-get install -y --no-install-recommends chromium dumb-init fonts-dejavu-core ca-certificates curl && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY --from=builder --chown=1000:1000 /src/unfurler/node_modules ./node_modules
COPY --from=builder --chown=1000:1000 /src/unfurler/dist ./dist
COPY --from=builder --chown=1000:1000 /src/unfurler/views ./views
COPY --from=builder --chown=1000:1000 /src/unfurler/puppeteer.config.json ./puppeteer.config.json

USER 1000
EXPOSE 4201

CMD ["dumb-init", "node", "--max-old-space-size=2048", "dist/index.js"]
