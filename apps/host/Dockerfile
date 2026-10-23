# Strata's web host in a container: the Node server in server/ plus the built page from src/.
# It talks to a Strata API worker that runs somewhere else; it does not carry an engine.
#
#   docker build -t strata-host:0.2.2 apps/host
#   docker run -d --name strata-host -p 127.0.0.1:8791:8791 \
#     -e STRATA_SERVER_URL=http://strata-api:8080/v1 -e STRATA_SERVER_KEY=... \
#     -e STRATA_REAL_ENGINE=1 -e STRATA_NO_BUNDLED_ENGINE=1 -v strata-host-home:/home/node/.strata strata-host:0.2.2
#
# The page is built here (vite); the runtime image carries dist/ and the production dependencies only.

FROM node:24-bookworm-slim AS build
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci --no-audit --no-fund
COPY . .
RUN npm run typecheck && npm run build

FROM node:24-bookworm-slim AS runtime
ENV NODE_ENV=production \
    STRATA_PORT=8791 \
    STRATA_BIND=0.0.0.0 \
    STRATA_DIST=/app/dist \
    HOME=/home/node
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci --omit=dev --no-audit --no-fund && npm cache clean --force
COPY --from=build /app/dist ./dist
COPY server ./server
RUN mkdir -p /home/node/.strata /workspace && chown node:node /home/node/.strata /workspace
USER node
EXPOSE 8791
HEALTHCHECK --interval=30s --timeout=5s --start-period=20s --retries=3 \
  CMD node -e "fetch('http://127.0.0.1:'+(process.env.STRATA_PORT||8791)+'/').then(r=>process.exit(r.ok?0:1),()=>process.exit(1))"
CMD ["node", "server/index.mjs", "--dist", "dist"]
