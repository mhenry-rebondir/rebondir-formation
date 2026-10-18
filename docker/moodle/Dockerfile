FROM php:8.3-fpm-bookworm

# Installation des dépendances et de Nginx
RUN apt-get update && apt-get install -y --no-install-recommends \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libpng-dev \
    libicu-dev \
    libxml2-dev \
    libzip-dev \
    libpq-dev \
    git \
    unzip \
    nginx \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) \
        gd intl opcache pgsql pdo_pgsql soap xml zip exif \
    && rm -rf /var/lib/apt/lists/*

# Téléchargement de Moodle
ENV MOODLE_VERSION=503
WORKDIR /var/www/html
RUN curl -SL https://packaging.moodle.org/stable${MOODLE_VERSION}/moodle-latest-${MOODLE_VERSION}.tgz | tar xz --strip-components=1 \
    && chown -R www-data:www-data /var/www/html

# Configuration Nginx basique pour Moodle et PHP-FPM
    # server {
    #     listen 80;
    #     server_name moodle.caminade.fr;
    #     root /var/www/html/public;

    #     index index.php index.html;

    #     # Gestion propre des fichiers statiques pour éviter les 404 / redirections HTML
    #     location ~* \.(ico|css|js|gif|jpe?g|png|woff2?|eot|ttf|svg)$ {
    #         expires 7d;
    #         access_log off;
    #         try_files $uri =404;
    #     }

    #     location / {
    #         try_files $uri $uri/ /index.php?$query_string;
    #     }

    #     location ~ \.php$ {
    #         include fastcgi_params;
    #         fastcgi_pass 127.0.0.1:9000;
    #         fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    #     }
    # }
RUN echo 'server { \
    listen 80; \
    root /var/www/html; \
    index index.php index.html index.htm; \
    location / { \
        try_files $uri $uri/ /index.php?$args; \
    } \
    location ~ \.php$ { \
        include fastcgi_params; \
        fastcgi_pass 127.0.0.1:9000; \
        fastcgi_index index.php; \
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name; \
    } \
}' > /etc/nginx/sites-available/default

RUN { \
    echo 'opcache.memory_consumption=256'; \
    echo 'opcache.interned_strings_buffer=16'; \
    echo 'opcache.max_accelerated_files=10000'; \
    echo 'opcache.revalidate_freq=2'; \
    echo 'opcache.fast_shutdown=1'; \
    echo 'max_input_vars = 5000'; \
} > /usr/local/etc/php/conf.d/moodle-recommended.ini


EXPOSE 80

# Script de lancement combinant Nginx et PHP-FPM
RUN echo '#!/bin/sh' > /entrypoint.sh && \
    echo 'nginx' >> /entrypoint.sh && \
    echo 'exec php-fpm' >> /entrypoint.sh && \
    chmod +x /entrypoint.sh

CMD ["/entrypoint.sh"]