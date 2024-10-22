# Use an official Nginx image as the base image
FROM httpd:latest

# Copy the index.html to the Nginx HTML directory
COPY index.html /usr/share/nginx/html/

COPY ./httpd.conf /usr/local/apache2/conf/httpd.conf
