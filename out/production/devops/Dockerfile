# Use the official OpenJDK JRE image
FROM openjdk:17-jre-slim

# Set the working directory in the container
WORKDIR /app

# Copy the compiled Java class into the container
COPY Main.java .

RUN javac Main.java

# Command to run the Java program
CMD ["java", "Main"]
