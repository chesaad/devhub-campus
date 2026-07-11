# ---- Stage 1 : build / install des dépendances ----
FROM node:20-alpine AS build

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci --omit=dev

COPY src/ ./src/

# ---- Stage 2 : image runtime ----
FROM node:20-alpine AS runtime

LABEL org.opencontainers.image.source="https://github.com/chesaad/devhub-campus"

WORKDIR /app

COPY --from=build /app/node_modules ./node_modules
COPY --from=build /app/src ./src
COPY package.json ./

RUN addgroup -g 1001 -S app && adduser -u 1001 -S app -G app
USER 1001

ENV PORT=8080 \
    LOG_LEVEL=info \
    NODE_ENV=production

EXPOSE 8080

CMD ["node", "src/index.js"]
