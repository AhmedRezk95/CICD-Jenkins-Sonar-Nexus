FROM nginx

LABEL "Project"="udemy-Project"
LABEL "Author"="Rizk"

RUN rm -rf /etc/nginx/conf.d/default.conf
# Copy nginx configuration
COPY mynginx.conf /etc/nginx/conf.d/vproapp.conf