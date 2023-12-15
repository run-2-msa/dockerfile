FROM openjdk:17-ea-17-jdk-slim
VOLUME /tmp
COPY target/naming-server-1.0.jar NamingServer.jar
ENTRYPOINT [ "java", "-jar", "NamingServer.jar"]