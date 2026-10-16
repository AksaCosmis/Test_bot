# Multi-stage Docker build for Puls Dostupa (Production Ready)
FROM node:20-alpine AS builder

WORKDIR /app

# Copy package descriptors
COPY package*.json ./
RUN npm install

# Copy source code and config
COPY . .

# Build Vite client and compile Express server to dist/server.cjs
RUN npm run build

# Production runtime image
FROM node:20-alpine AS runner

WORKDIR /app

ENV NODE_ENV=production
ENV PORT=3000
ENV AUTO_RESTART_HOURS=24
ENV TZ=Europe/Moscow

# Git is required for providers that use authenticated Git remotes.
RUN apk add --no-cache curl bash git tzdata

# Copy package files and install production dependencies
COPY package*.json ./
RUN npm install --omit=dev

# Copy compiled artifacts from builder
COPY --from=builder /app/dist ./dist

# Copy entrypoint script with configurable auto-restart supervisor
COPY docker-entrypoint.sh /usr/local/bin/docker-entrypoint.sh
RUN chmod +x /usr/local/bin/docker-entrypoint.sh

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=8s --start-period=15s --retries=3 \
  CMD curl -f "http://localhost:${PORT:-3000}/api/health" || exit 1

ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]
