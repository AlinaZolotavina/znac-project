# ---------- Stage 1 ----------
FROM node:22-alpine AS builder

WORKDIR /app

COPY znac/package*.json ./

RUN npm ci

COPY znac/ .

ARG REACT_APP_API_URL
ENV REACT_APP_API_URL=$REACT_APP_API_URL

RUN npm run build

# ---------- Stage 2 ----------
FROM nginx:alpine

COPY nginx/nginx.conf /etc/nginx/nginx.conf

COPY --from=builder /app/build /usr/share/nginx/html