FROM wordpress:latest

USER root

RUN echo "expose_php = Off" > /usr/local/etc/php/conf.d/hide-php.ini

RUN mkdir -p /var/www/html/wp-content/mu-plugins && \
    { \
        echo '<?php'; \
        echo 'remove_action("wp_head", "wp_generator");'; \
        echo 'add_filter("the_generator", "__return_empty_string");'; \
        echo 'function remove_wp_ver_css_js( $src ) {'; \
        echo '    if ( strpos( $src, "ver=" . get_bloginfo( "version" ) ) )'; \
        echo '        $src = remove_query_arg( "ver", $src );'; \
        echo '    return $src;'; \
        echo '}'; \
        echo 'add_filter( "style_loader_src", "remove_wp_ver_css_js", 9999 );'; \
        echo 'add_filter( "script_loader_src", "remove_wp_ver_css_js", 9999 );'; \
        echo 'add_filter("login_errors", function() { return "Invalid credentials."; });'; \
        echo 'add_filter("rest_endpoints", function($endpoints) {'; \
        echo '    if (isset($endpoints["/wp/v2/users"])) unset($endpoints["/wp/v2/users"]);'; \
        echo '    if (isset($endpoints["/wp/v2/users/(?P<id>[\d]+)"])) unset($endpoints["/wp/v2/users/(?P<id>[\d]+)"]);'; \
        echo '    return $endpoints;'; \
        echo '});'; \
        echo 'add_filter("wp_sitemaps_add_provider", function($provider, $name) {'; \
        echo '    if ($name === "users") return false;'; \
        echo '    return $provider;'; \
        echo '}, 10, 2);'; \
        echo 'function disable_feeds() { wp_die("No feed available."); }'; \
        echo 'add_action("do_feed", "disable_feeds", 1);'; \
        echo 'add_action("do_feed_rdf", "disable_feeds", 1);'; \
        echo 'add_action("do_feed_rss", "disable_feeds", 1);'; \
        echo 'add_action("do_feed_rss2", "disable_feeds", 1);'; \
        echo 'add_action("do_feed_atom", "disable_feeds", 1);'; \
    } > /var/www/html/wp-content/mu-plugins/hardening.php

RUN /usr/sbin/a2enmod headers rewrite && \
    sed -i 's/Listen 80/Listen 8080/' /etc/apache2/ports.conf && \
    sed -i 's/<VirtualHost \*:80>/<VirtualHost *:8080>/' /etc/apache2/sites-available/000-default.conf && \
    { \
        echo '<Directory /var/www/html>'; \
        echo '  RewriteEngine On'; \
        echo '  RewriteRule ^(xmlrpc\.php|wp-cron\.php|wp-links-opml\.php|readme\.html|license\.txt|robots\.txt) - [F]'; \
        echo '  RewriteRule ^wp-admin/(install|upgrade)\.php - [F]'; \
        echo '  RewriteCond %{QUERY_STRING} author=([0-9]*) [NC]'; \
        echo '  RewriteRule .* - [F]'; \
        echo '  RewriteCond %{QUERY_STRING} feed=(.*) [NC]'; \
        echo '  RewriteRule .* - [F]'; \
        echo '</Directory>'; \
    } > /etc/apache2/conf-available/wpscan-block.conf && \
    { \
        echo 'ServerTokens Prod'; \
        echo 'ServerSignature Off'; \
        echo 'Header always set X-Content-Type-Options nosniff'; \
        echo 'Header always set X-Frame-Options SAMEORIGIN'; \
        echo 'Header always set X-XSS-Protection "1; mode=block"'; \
    } >> /etc/apache2/apache2.conf && \
    a2enconf wpscan-block

RUN useradd -r -u 1000 -g www-data wpuser && \
    chown -R wpuser:www-data /var/www/html && \
    chown -R wpuser:www-data /var/run/apache2 /var/lock/apache2 /var/log/apache2

USER wpuser
EXPOSE 8080