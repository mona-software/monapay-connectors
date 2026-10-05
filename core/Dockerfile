# Build from the repository root: docker build -f core/Dockerfile -t monapay-connectors .
FROM node:20-alpine

WORKDIR /app
COPY --chown=node:node package.json ./
RUN npm install --omit=dev --no-audit --no-fund
COPY --chown=node:node core ./core
COPY --chown=node:node adapters ./adapters

ENV NODE_ENV=production \
    HOST=127.0.0.1 \
    PORT=8787

USER node
EXPOSE 8787
CMD ["node", "core/server.js"]
