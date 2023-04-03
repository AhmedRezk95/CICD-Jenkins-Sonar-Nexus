FROM tomcat:8-jre11

LABEL "Project"="udemy-Project"
LABEL "Author"="Rizk"

# remove all webapps inside this images
RUN rm -rf /usr/local/tomcat/webapps/*
# copy artifact to tomcat
COPY target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war

EXPOSE 8080
CMD [ "catalina.sh", "run" ]

WORKDIR /usr/local/tomcat/
VOLUME /usr/local/tomcat/webapps
