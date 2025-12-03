```dockerfile
FROM $REPOSITORY:5000/php:8.2-apache

# Install PHP extensions
RUN docker-php-ext-install mysqli pdo pdo_mysql

# Enable Apache rewrite module
RUN a2enmod rewrite

# Set working directory
WORKDIR /var/www/

# Copy application files
COPY html/ ./html/
COPY app/ ./app/
COPY ci3113/ ./ci3113/

# Set permissions
RUN chown -R www-data:www-data /var/www/html \
    && chmod -R 755 /var/www/html

# Configure Apache
RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf

# Expose HTTP port
EXPOSE 80

# Start Apache
CMD ["apache2-foreground"]
