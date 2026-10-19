
# Use the official lightweight Nginx image based on Alpine Linux.
FROM nginx:stable-alpine

# curl is required by the Docker Compose healthcheck.
RUN apk add --no-cache curl

# Replace the default Nginx configuration with our proxy configuration.
COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
