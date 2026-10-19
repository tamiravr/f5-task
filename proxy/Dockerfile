# Proxy image inherited from the previous team.

FROM ubuntu:24.04

RUN apt-get update && \
    apt-get install -y nginx curl htop vim git

# Install Nginx before copying our config to avoid a package configuration
# conflict and an interactive prompt during the image build.
COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
