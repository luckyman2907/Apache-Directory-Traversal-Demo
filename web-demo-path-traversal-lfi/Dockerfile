# Use the official PHP 7.3 Apache image
FROM php:7.3-apache

# Copy the script to clear Apache logs
COPY ./scripts/clearlog.sh /usr/local/bin/clearlog.sh
# Set the script as executable
RUN chmod +x /usr/local/bin/clearlog.sh

# Set up log files
RUN rm /var/log/apache2/access.log /var/log/apache2/error.log 
RUN touch /var/log/apache2/access.log /var/log/apache2/error.log
RUN chmod 777 /var/log/apache2/access.log /var/log/apache2/error.log
RUN chown www-data:www-data /var/log/apache2/access.log /var/log/apache2/error.log

# Enable the Apache headers module and copy configuration files
RUN a2enmod headers
COPY ./configs/apache2.conf /etc/apache2/apache2.conf
COPY ./configs/000-default.conf /etc/apache2/sites-available/000-default.conf

# Copy challenge files to default document root
WORKDIR /var/www/html
COPY ./src/ .

# Set up permissions for owner
RUN chown -R root:www-data .
RUN chmod -R 750 .
RUN find . -type f -exec chmod 640 {} \;
RUN find . -type d -exec chmod 750 {} \;

# Add sticky bit to prevent delete files
RUN chmod +t -R .

# Set message
RUN echo "Message: I love you" > /secret.txt