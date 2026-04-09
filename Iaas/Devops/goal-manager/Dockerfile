# Stage 1: build
FROM node:20-alpine AS build
WORKDIR /app

# Install security updates
RUN apk update && apk upgrade && rm -rf /var/cache/apk/*

COPY package.json package-lock.json* ./

# Install dependencies
RUN npm install

COPY . .

# Build the application
RUN npm run build && \
    npm prune --production

# Stage 2: runtime with nginx
FROM nginx:stable-alpine

# Setup security context (nginx user already exists in base image)
RUN apk update && apk upgrade && rm -rf /var/cache/apk/*

# Copy built application
COPY --from=build --chown=nginx:nginx /app/dist /usr/share/nginx/html

# Copy and configure nginx
RUN rm /etc/nginx/conf.d/default.conf
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Set permissions
RUN chmod -R 755 /usr/share/nginx/html && \
    chown -R nginx:nginx /usr/share/nginx/html

# Add metadata
LABEL maintainer="Goal Manager Team" \
      version="0.0.1" \
      description="Goal Manager - Task management application"

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD wget --quiet --tries=1 --spider http://localhost/ || exit 1

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
