```dockerfile
FROM $REPOSITORY:5000/php:8.2-apache
RUN docker-php-ext-install mysqli pdo pdo_mysql
RUN a2enmod rewrite
WORKDIR /var/www/
COPY html/ ./html/
COPY app/ ./app/
COPY ci3113/ ./ci3113/
RUN chown -R www-data:www-data /var/www/html \
    && chmod -R 755 /var/www/html
RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf
EXPOSE 80
CMD ["apache2-foreground"]
