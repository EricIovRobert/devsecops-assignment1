FROM wordpress:latest

USER root

RUN echo "expose_php = Off" > /usr/local/etc/php/conf.d/hide-php.ini

RUN mkdir -p /var/www/html/wp-content/mu-plugins && \
    echo "<?php \
    /* Hide Version from Head & Feeds */ \
    remove_action('wp_head', 'wp_generator'); \
    add_filter('the_generator', '__return_empty_string'); \
    /* Strip version from scripts and styles */ \
    function remove_wp_ver_css_js( \$src ) { \
        if ( strpos( \$src, 'ver=' . get_bloginfo( 'version' ) ) ) \
            \$src = remove_query_arg( 'ver', \$src ); \
        return \$src; \
    } \
    add_filter( 'style_loader_src', 'remove_wp_ver_css_js', 9999 ); \
    add_filter( 'script_loader_src', 'remove_wp_ver_css_js', 9999 ); \
    /* Hide login errors */ \
    add_filter('login_errors', function() { return 'Invalid credentials.'; }); \
    ?>" > /var/www/html/wp-content/mu-plugins/hardening.php

RUN /usr/sbin/a2enmod headers rewrite && \
    sed -i 's/Listen 80/Listen 8080/' /etc/apache2/ports.conf && \
    sed -i 's/<VirtualHost \*:80>/<VirtualHost *:8080>/' /etc/apache2/sites-available/000-default.conf && \
    sed -i '/<VirtualHost \*:8080>/a \
    <Directory /var/www/html>\n \
        RewriteEngine On\n \
        RewriteRule ^(xmlrpc\\.php|wp-cron\\.php|wp-links-opml\\.php|readme\\.html|license\\.txt) - [F]\n \
        RewriteRule ^wp-admin/(install|upgrade)\\.php - [F]\n \
        RewriteCond %{QUERY_STRING} author=([0-9]*) [NC]\n \
        RewriteRule .* - [F]\n \
        RewriteCond %{QUERY_STRING} feed=(.*) [NC]\n \
        RewriteRule .* - [F]\n \
    </Directory>' /etc/apache2/sites-available/000-default.conf && \
    { \
        echo 'ServerTokens Prod'; \
        echo 'ServerSignature Off'; \
        echo 'Header always set X-Content-Type-Options nosniff'; \
        echo 'Header always set X-Frame-Options SAMEORIGIN'; \
        echo 'Header always set X-XSS-Protection "1; mode=block"'; \
    } >> /etc/apache2/apache2.conf

RUN useradd -r -u 1000 -g www-data wpuser && \
    chown -R wpuser:www-data /var/www/html && \
    chown -R wpuser:www-data /var/run/apache2 /var/lock/apache2 /var/log/apache2

USER wpuser
EXPOSE 8080