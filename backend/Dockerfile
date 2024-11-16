# DockerFile pour le backend
# Étape 1: Build
FROM node:18 AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
# Étape 2: Run (utilisation d'une image plus légère)
FROM node:18-slim AS run
WORKDIR /app
ARG NODE_ENV=production
ENV NODE_ENV=${NODE_ENV}
COPY --from=build /app /app
CMD ["npm", "start"]