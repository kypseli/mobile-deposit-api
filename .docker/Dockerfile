FROM openjdk:8u111-jre-alpine
VOLUME /tmp
COPY ${project.build.finalName}.jar app.jar
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
