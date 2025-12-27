FROM debian:bullseye

RUN apt-get update && apt-get -y install \
	wget \
	curl \
	bash \
	php \
	php-cgi \
	php-mysql \
	php-fpm \
	php-pdo \
	php-cli \
	php-mbstring \
	&& rm -rf /var/lib/apt/lists/*

# Copy the .conf in the html directory (PHP 7.4 on bullseye)
COPY ./conf/www.conf /etc/php/7.4/fpm/pool.d/

#Create the folder to enable php start
RUN mkdir /run/php

#we copy the script, give the right to launch it in local on our computer
COPY ./tools/create_wordpress.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/create_wordpress.sh
ENTRYPOINT ["/usr/local/bin/create_wordpress.sh"]

#We go the html directory
WORKDIR /var/www/html/

#We expose the 9000 port
EXPOSE 9000

#Launch PHP FPM in foreground and ignore deamonize from conf file (-F)
CMD ["/usr/sbin/php-fpm7.4", "-F"]

