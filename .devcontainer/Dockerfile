# Use the official PHP image based on Alpine
FROM php:8.3-fpm-alpine
RUN set -ex && apk --no-cache add sudo

# Install Nginx, necessary PHP extensions, and tools
RUN apk update && apk add --no-cache \
    nginx \
    supervisor \
    git \
    unzip \
    curl \
    icu-dev \
    mariadb-client \
    postgresql-dev \
    libzip-dev \
    openjdk11-jre \
    && docker-php-ext-install intl pdo pdo_mysql zip

# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Install Symfony CLI
RUN curl -1sLf 'https://dl.cloudsmith.io/public/symfony/stable/setup.alpine.sh' | sh \
    && apk add symfony-cli

# Install Elasticsearch, Logstash, and Kibana
# Version variables for easy management
ENV ELK_VERSION=7.15.0

# Install Elasticsearch
RUN wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-${ELK_VERSION}-linux-x86_64.tar.gz && \
    tar -xzf elasticsearch-${ELK_VERSION}-linux-x86_64.tar.gz && \
    mv elasticsearch-${ELK_VERSION} /usr/share/elasticsearch && \
    rm elasticsearch-${ELK_VERSION}-linux-x86_64.tar.gz

# Install Logstash
RUN wget https://artifacts.elastic.co/downloads/logstash/logstash-${ELK_VERSION}-linux-x86_64.tar.gz && \
    tar -xzf logstash-${ELK_VERSION}-linux-x86_64.tar.gz && \
    mv logstash-${ELK_VERSION} /usr/share/logstash && \
    rm logstash-${ELK_VERSION}-linux-x86_64.tar.gz

# Install Kibana
RUN wget https://artifacts.elastic.co/downloads/kibana/kibana-${ELK_VERSION}-linux-x86_64.tar.gz && \
    tar -xzf kibana-${ELK_VERSION}-linux-x86_64.tar.gz && \
    mv kibana-${ELK_VERSION} /usr/share/kibana && \
    rm kibana-${ELK_VERSION}-linux-x86_64.tar.gz

# Configure Nginx
COPY ./.devcontainer/nginx.conf /etc/nginx/nginx.conf

# Configure Supervisor
COPY ./.devcontainer/supervisord.conf /etc/supervisord.conf

# Add Elasticsearch, Logstash, Kibana to Supervisor
COPY ./.devcontainer/supervisor_elasticsearch.conf /etc/supervisor/conf.d/elasticsearch.conf
COPY ./.devcontainer/supervisor_logstash.conf /etc/supervisor/conf.d/logstash.conf
COPY ./.devcontainer/supervisor_kibana.conf /etc/supervisor/conf.d/kibana.conf

# Create a new user with a shell
RUN adduser -D -s /bin/sh symfony

WORKDIR /var/www/html

# Set permissions
RUN chown -R symfony:www-data /var/www/html

# Set the new user
USER symfony

# Copy current directory to working directory
COPY . /var/www/html

# Expose ports
EXPOSE 80 9200 5601 5044

# Start Supervisor
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisord.conf"]
