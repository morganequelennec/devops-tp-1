FROM httpd:2.4 AS web-server
COPY ./public-html/ /usr/local/apache2/htdocs/
#RUN echo "ServerName localhost" >> /usr/local/apache2/conf/httpd.conf
COPY httpd.conf /usr/local/apache2/conf/httpd.conf