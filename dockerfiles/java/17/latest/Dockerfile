# Use an official Maven image to build the application
FROM maven:3.8.5-openjdk-17-slim AS builder

# Set the working directory inside the container
WORKDIR /app

# Copy the entire project into the container
COPY . /app

# Run Maven to build the project and create the JAR file
RUN mvn clean compile assembly:single

# RUN ls -la target/ && sleep 11

# Use a smaller JDK image to run the application
FROM openjdk:17-jdk-slim

# Set the working directory inside the container
WORKDIR /app


# Copy the JAR file from the builder stage into the final image
COPY --from=builder /app/target/server-service-1.0.0-jar-with-dependencies.jar /app/server-service.jar

# Expose the port the server will listen on
EXPOSE 8081
CMD ["java", "-jar", "server-service.jar"]
