# use latest WordPress image
FROM wordpress:latest

# root to make system changes
USER root

RUN rm -f /var/www/html/readme.html \
          /var/www/html/license.txt \
          /var/www/html/wp-config-sample.php

#  apache security headers - xss protection (hides server version)
RUN /usr/sbin/a2enmod headers && \
    sed -i 's/Listen 80/Listen 8080/' /etc/apache2/ports.conf && \
    sed -i 's/<VirtualHost \*:80>/<VirtualHost *:8080>/' /etc/apache2/sites-available/000-default.conf && \
    echo "ServerTokens Prod" >> /etc/apache2/apache2.conf && \
    echo "ServerSignature Off" >> /etc/apache2/apache2.conf && \
    echo "Header always set X-Content-Type-Options nosniff" >> /etc/apache2/apache2.conf && \
    echo "Header always set X-Frame-Options SAMEORIGIN" >> /etc/apache2/apache2.conf && \
    echo "Header always set X-XSS-Protection '1; mode=block'" >> /etc/apache2/apache2.conf

# disable xml-rpc
RUN echo '<Files "xmlrpc.php">' >> /etc/apache2/apache2.conf && \
    echo '  Require all denied' >> /etc/apache2/apache2.conf && \
    echo '</Files>' >> /etc/apache2/apache2.conf

# non-root user for running
RUN useradd -r -u 1000 -g www-data wpuser && \
    chown -R wpuser:www-data /var/www/html && \
    chown -R wpuser:www-data /var/run/apache2 /var/lock/apache2 /var/log/apache2

# switch to non-root user
USER wpuser

EXPOSE 8080