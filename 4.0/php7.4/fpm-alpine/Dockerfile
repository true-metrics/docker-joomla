#
# NOTE: THIS DOCKERFILE IS GENERATED VIA "apply-templates.sh"
#
# PLEASE DO NOT EDIT IT DIRECTLY.
#

# from https://downloads.joomla.org/technical-requirements
FROM php:7.4-fpm-alpine
LABEL maintainer="Llewellyn van der Merwe <llewellyn.van-der-merwe@community.joomla.org> (@Llewellynvdm), Harald Leithner <harald.leithner@community.joomla.org> (@HLeithner)"

# Disable remote database security requirements.
ENV JOOMLA_INSTALLATION_DISABLE_LOCALHOST_CHECK=1
# entrypoint.sh dependencies
RUN apk add --no-cache \
	bash
# Install the PHP extensions
RUN set -ex; \
	\
	apk add --no-cache --virtual .build-deps \
		$PHPIZE_DEPS \
		autoconf \
		bzip2-dev \
		gmp-dev \
		icu-dev \
		libjpeg-turbo-dev \
		libmcrypt-dev \
		libmemcached-dev \
		libpng-dev \
		libzip-dev \
		openldap-dev \
		pcre-dev \
		postgresql-dev \
	; \
	\
	docker-php-ext-configure gd --with-jpeg; \
	docker-php-ext-configure ldap; \
	docker-php-ext-install -j "$(nproc)" \
		bz2 \
		gd \
		gmp \
		intl \
		ldap \
		mysqli \
		pdo_mysql \
		pdo_pgsql \
		pgsql \
		zip \
	; \
	\
# pecl will claim success even if one install fails, so we need to perform each install separately
	pecl install APCu-5.1.21; \
	pecl install mcrypt-1.0.4; \
	pecl install memcached-3.1.5; \
	pecl install redis-5.3.4; \
	\
	docker-php-ext-enable \
		apcu \
		mcrypt \
		memcached \
		redis \
	; \
	rm -r /tmp/pear; \
	\
	runDeps="$( \
		scanelf --needed --nobanner --format '%n#p' --recursive /usr/local/lib/php/extensions \
		| tr ',' '\n' \
		| sort -u \
		| awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
		)"; \
	apk add --virtual .joomla-phpext-rundeps $runDeps; \
	apk del .build-deps

VOLUME /var/www/html

# Define Joomla version and expected SHA512 signature
ENV JOOMLA_VERSION 4.0.6
ENV JOOMLA_SHA512 a35f3181c594ef0c30e4f8ec122f32e2ebe1795ccd31f55656be0f214b1f6eed54ef7961aed0be772b2c4a6cc5d0e6454b9213f1ba8833dd52317c6d216d9fd8

# Download package and extract to web volume
RUN set -ex; \
	curl -o joomla.tar.bz2 -SL https://github.com/joomla/joomla-cms/releases/download/${JOOMLA_VERSION}/Joomla_${JOOMLA_VERSION}-Stable-Full_Package.tar.bz2; \
	echo "$JOOMLA_SHA512 *joomla.tar.bz2" | sha512sum -c -; \
	mkdir /usr/src/joomla; \
	tar -xf joomla.tar.bz2 -C /usr/src/joomla; \
	rm joomla.tar.bz2; \
	chown -R www-data:www-data /usr/src/joomla

# Copy init scripts and custom .htaccess
COPY docker-entrypoint.sh /entrypoint.sh
COPY makedb.php /makedb.php

ENTRYPOINT ["/entrypoint.sh"]

CMD ["php-fpm"]

# vim:set ft=dockerfile:
