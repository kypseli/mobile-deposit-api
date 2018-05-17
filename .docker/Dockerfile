FROM openjdk:8u111-jre-alpine
VOLUME /tmp
ADD ${project.build.finalName}.jar app.jar
RUN ash -c 'touch /app.jar'
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
