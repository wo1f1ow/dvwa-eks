ARG VERSION
FROM php:8.1-apache as builder

# Install requirements
RUN set -eux \
	&& apt-get update \
	&& apt-get install -y --no-install-recommends --no-install-suggests \
		git \
		ca-certificates \
	&& update-ca-certificates

# Get DVWA
RUN set -eux \
	&& git clone https://github.com/digininja/DVWA /DVWA \
	&& rm -rf /DVWA/.git \
	&& rm -rf /DVWA/.github \
	&& rm -rf /DVWA/.gitignore \
	&& rm -rf /DVWA/php.ini

# Disable SQLITE
RUN set -eux \
	&& sed -i'' "s/if (\$_DVWA\['SQLI_DB'\]/if ('no'/g" /DVWA/dvwa/includes/dvwaPage.inc.php \
	&& sed -i'' 's/[[:space:]]SQLITE)/"SQLITE")/g' /DVWA/dvwa/includes/dvwaPage.inc.php

# Get Adminer
RUN set -eux \
	&& URL="$( \
		curl -sS --fail -k https://www.adminer.org/ \
		| grep -Eo 'https://github.com/vrana/adminer/releases/download/v[.0-9]+/adminer-[.0-9]+-mysql-en.php' \
	)" \
	&& curl -sS --fail -k -L "${URL}" > /adminer.php


ARG VERSION
FROM php:8.1-apache

# Satisfy PHP requirements
RUN set -eux \
	&& apt-get update \
	&& apt-get install -y --no-install-recommends --no-install-suggests \
		libpng-dev \
		ca-certificates \
	&& update-ca-certificates \
	&& docker-php-ext-install gd \
	&& docker-php-ext-install mysqli \
	&& docker-php-ext-install pdo_mysql \
	&& apt-get clean \
	&& rm -rf /var/lib/apt/lists/*

# Satisfy Application requirements
RUN set -eux \
	&& apt-get update \
	&& apt-get install -y --no-install-recommends --no-install-suggests \
		iputils-ping \
		python3 \
		strace \
		sudo \
	&& apt-get clean \
	&& rm -rf /var/lib/apt/lists/*

# Copy source
COPY --from=builder /DVWA/ /var/www/html/
COPY --from=builder /adminer.php /var/www/html/adminer.php
COPY ./config.inc.php /var/www/html/config/config.inc.php
COPY ./entrypoint.sh/ /entrypoint.sh
COPY ./.htaccess /var/www/html/.htaccess
COPY ./.htpasswd /var/www/html/.htpasswd
COPY ./apache2.conf /etc/apache2/apache2.conf

# Configure PHP
RUN set -eux \
	&& { \
		echo "allow_url_include = on"; \
		echo "allow_url_fopen   = on"; \
		echo "error_reporting   = E_ALL | E_STRICT"; \
		echo "magic_quotes_gpc  = off"; \
	} > /usr/local/etc/php/conf.d/default.ini

# Adjust permissions
RUN set -eux \
	&& chown -R www-data:www-data /var/www/html \
	&& chmod 0775 /var/www/html/config/ \
	&& chmod 0775 /var/www/html/hackable/uploads/

CMD ["/entrypoint.sh"]
