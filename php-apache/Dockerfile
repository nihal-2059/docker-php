FROM centos:latest
RUN  yum install php php-mysqlnd httpd -y && mkdir /var/run/php-fpm
ADD  script.sh /usr/local/bin/script.sh
RUN  chmod 777 /usr/local/bin/script.sh
CMD  /usr/local/bin/script.sh
