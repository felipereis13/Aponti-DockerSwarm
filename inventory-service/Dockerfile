FROM node:20-alpine AS dependencies
WORKDIR /opt/service

COPY package*.json ./
# Usa o lockfile quando disponivel; este projeto tambem aceita builds sem ele.
RUN if [ -f package-lock.json ]; then \
      npm ci --omit=dev --no-audit --no-fund; \
    else \
      npm install --omit=dev --no-audit --no-fund; \
    fi

FROM node:20-alpine AS runtime
WORKDIR /opt/service

ENV NODE_ENV=production \
    PORT=3002

COPY --from=dependencies /opt/service/package*.json ./
COPY --from=dependencies /opt/service/node_modules ./node_modules
COPY index.js ./
COPY src/ ./src/

USER node
EXPOSE 3002

HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
    CMD ["node", "-e", "fetch('http://127.0.0.1:' + process.env.PORT + '/health', { signal: AbortSignal.timeout(2000) }).then(response => process.exit(response.status === 200 ? 0 : 1)).catch(() => process.exit(1));"]

CMD ["node", "index.js"]
