# STEP 1: Dockerfile for the app-service Spring Boot microservice

# Base image: Java 17 runtime (JRE is enough to run a packaged JAR)
FROM eclipse-temurin:17-jre

# Working directory inside the container
WORKDIR /app

# Copy the executable Spring Boot JAR produced by "mvn clean package"
# (the name is fixed by <finalName>app-service</finalName> in pom.xml)
COPY target/app-service.jar app.jar

# Document the port the application listens on
EXPOSE 8080

# Command that starts the application when the container starts
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
