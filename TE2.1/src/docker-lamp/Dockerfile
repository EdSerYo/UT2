# Imagen base de trabajo
FROM php:8.0.0-apache

# Instalacion de driver de MySQL
RUN docker-php-ext-install mysqli

# Instalacion de librerías para MySQL
RUN apt-get update \
    && apt-get install -y sendmail libpng-dev \
    && apt-get install -y libzip-dev \
    && apt-get install -y zlib1g-dev \
    && apt-get install -y libonig-dev \
    && rm -rf /var/lib/apt/lists/* \
    && docker-php-ext-install zip

RUN docker-php-ext-install mbstring
RUN docker-php-ext-install zip
RUN docker-php-ext-install gd

# habilitamos el módulo rewrite de apache
RUN a2enmod rewrite