FROM openjdk:17-ea-17-jdk-slim
VOLUME /tmp
#COPY client.jks client.jks
COPY target/config-server-1.0.jar ConfigServer.jar
ENTRYPOINT [ "java", "-jar", "ConfigServer.jar"]

#ENTRYPOINT [ \
#    "java", \
#    "-jar", \
#    "ConfigServer.jar" \
#]
