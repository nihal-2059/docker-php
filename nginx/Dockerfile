FROM centos:latest
RUN  yum install nginx -y
COPY nginx.conf /etc/nginx/nginx.conf
ADD  script.sh /usr/local/bin/script.sh
RUN  chmod 777 /usr/local/bin/script.sh
CMD  /usr/local/bin/script.sh

