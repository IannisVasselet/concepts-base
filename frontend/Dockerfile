# DockerFile pour le frontend
# Étape 1: Build
FROM node:18 AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Étape 2: Run (Nginx pour servir l'application)
FROM nginx:alpine AS run
COPY --from=build /app/build /usr/share/nginx/html

# Configuration Nginx
COPY nginx.conf /etc/nginx/conf.d/default.conf