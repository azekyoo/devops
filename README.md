# DevOps

#### Why should we run the container with a flag -e to give the environment variables?

Using the `-e` flag lets you pass environment variables to containers, making configuration flexible, portable, and secure without modifying the container's code. It ensures consistency across environments while simplifying automation and scaling.

#### Why do we need a volume to be attached to our postgres container?

A volume is needed for a PostgreSQL container to persist data, so that the database remains intact across container restarts or removals. Without a volume, all data would be lost when the container stops.

#### Database documentation

Essential commands: 

```docker build -t <YOUR_USERNAME>/database```

```docker network create app-network```

```docker run --name my-db-container --network app-network -v ${PWD}/postgresql/data:/var/lib/postgresql/data -d <YOUR_USERNAME>/database```

```docker run -p "8090:8080" --net=app-network --name=adminer -d adminer```

Dockerfile:
```
FROM postgres:14.1-alpine

ENV POSTGRES_DB=db \
   POSTGRES_USER=usr \
   POSTGRES_PASSWORD=pwd

COPY ./my-init-scripts/ /docker-entrypoint-initdb.d/
```

#### Why do we need a multistage build? And explain each step of this dockerfile.

A multistage build is used in Docker to reduce the size of the final image and to separate the build process from the runtime environment. It allows to use different images for different stages (like building and running the application), ensuring that only the essential files and dependencies are included in the final image.

### 1. `FROM maven:3.8.6-amazoncorretto-17 AS myapp-build`
- **Purpose**: This specifies the base image for the build stage. The `maven:3.8.6-amazoncorretto-17` image includes both Maven (v3.8.6) and Amazon Corretto (a distribution of OpenJDK 17). This is used for compiling the source code and packaging the application into a JAR file.
- **`AS myapp-build`**: Names this stage of the Docker build process as `myapp-build`, allowing us to refer to this stage in later parts of the Dockerfile.

### 2. `ENV MYAPP_HOME /opt/myapp`
- **Purpose**: Defines an environment variable `MYAPP_HOME` that points to the directory `/opt/myapp`. This variable will be used to simplify path management throughout the Dockerfile.

### 3. `WORKDIR $MYAPP_HOME`
- **Purpose**: Sets the working directory to `/opt/myapp`. All subsequent commands will be run in this directory inside the container.

### 4. `COPY pom.xml .`
- **Purpose**: Copies the `pom.xml` file from your local machine into the container's current working directory (`/opt/myapp`). The `pom.xml` file is required for Maven to resolve dependencies and build the project.

### 5. `COPY src ./src`
- **Purpose**: Copies the entire `src` directory (which contains the Java source code) from your local machine into the container under `/opt/myapp/src`.

### 6. `RUN mvn package -DskipTests`
- **Purpose**: Runs the Maven `package` command to build the project and create a JAR file. The `-DskipTests` option skips running unit tests during the build process, which can speed up the build. The resulting JAR file will be stored in the `target/` directory.

### 7. `FROM amazoncorretto:17`
- **Purpose**: Starts the second stage of the Docker build, using the `amazoncorretto:17` image. This is a lightweight JRE (Java Runtime Environment) image, which contains only the necessary tools to run the application, excluding Maven and other build tools.

### 8. `ENV MYAPP_HOME /opt/myapp`
- **Purpose**: Once again defines the `MYAPP_HOME` environment variable, pointing to the same directory `/opt/myapp` as in the build stage.

### 9. `WORKDIR $MYAPP_HOME`
- **Purpose**: Sets the working directory in the second stage to `/opt/myapp`, where the JAR file will be copied and executed.

### 10. `COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar`
- **Purpose**: Copies the built JAR file from the first build stage (`myapp-build`). The `--from=myapp-build` option tells Docker to fetch files from the `myapp-build` stage. It copies the JAR file (located in `/opt/myapp/target/*.jar`) into the current stage’s working directory as `myapp.jar`.

### 11. `ENTRYPOINT java -jar myapp.jar`
- **Purpose**: Sets the command to run when the container starts. It uses the `java -jar` command to run the Spring Boot application from the `myapp.jar` file.