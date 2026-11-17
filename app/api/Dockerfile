
# Stage 1: Install production dependencies
FROM node:22-alpine3.21 AS builder

WORKDIR /app

COPY package*.json ./
RUN npm install --omit=dev --no-audit --no-fund

# Stage 2: Smaller runtime image
FROM alpine:3.21

RUN apk add --no-cache libstdc++ libgcc

COPY --from=builder /usr/local/bin/node /usr/local/bin/node
COPY --from=builder /usr/local/lib/node_modules/npm /usr/local/lib/node_modules/npm
RUN ln -s /usr/local/lib/node_modules/npm/bin/npm-cli.js /usr/local/bin/npm

WORKDIR /app

COPY --from=builder /app/node_modules ./node_modules
COPY --chown=10001:10001 . .

RUN addgroup -g 10001 appgroup && adduser -D -u 10001 -G appgroup appuser

USER appuser

EXPOSE 3000

HEALTHCHECK --interval=15s --timeout=5s --start-period=60s --retries=10 CMD node -e "require('http').get('http://127.0.0.1:3000/api/health',r=>process.exit(r.statusCode===200?0:1)).on('error',()=>process.exit(1))"

CMD ["npm", "start"]

