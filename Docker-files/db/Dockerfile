FROM mysql:5.7.25

LABEL "Project"="udemy-Project"
LABEL "Author"="Rizk"

ENV MYSQL_ROOT_PASSWORD="vprodbpass"
ENV MYSQL_DATABASE="accounts"

# add backup file to the new db instance container
ADD db_backup.sql docker-entrypoint-initdb.d/db_backup.sql