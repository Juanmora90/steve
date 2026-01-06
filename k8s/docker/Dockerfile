# -----------------------------------------------------------------------------
# Stage 1: Build the Application
# -----------------------------------------------------------------------------
FROM eclipse-temurin:21-jdk AS builder
ENV LANG=C.UTF-8 LC_ALL=C.UTF-8

# Arguments for database configuration
ARG DB_HOST
ARG DB_USERNAME
ARG DB_PASSWORD
ARG DB_DATABASE
ARG DB_PORT

WORKDIR /code

# Copy project files
ADD /src /code/src
ADD /pom.xml /code/pom.xml
ADD mvnw /code/mvnw
ADD .mvn /code/.mvn

# Inject environment variables into pom.xml (Legacy support)
RUN sed -i 's|${db.ip}|${env.DB_HOST}|g' pom.xml
RUN sed -i 's|${db.port}|${env.DB_PORT}|g' pom.xml
RUN sed -i 's|${db.user}|${env.DB_USERNAME}|g' pom.xml
RUN sed -i 's|${db.password}|${env.DB_PASSWORD}|g' pom.xml
RUN sed -i 's|${db.schema}|${env.DB_DATABASE}|g' pom.xml
RUN sed -i 's|${server.host}|${env.SERVER_HOST}|g' pom.xml

# Build the WAR file
RUN chmod +x mvnw
RUN ./mvnw clean package -Pkubernetes -Djdk.tls.client.protocols="TLSv1.2,TLSv1.3" -DskipTests

# -----------------------------------------------------------------------------
# Stage 2: Runtime Image
# -----------------------------------------------------------------------------
FROM eclipse-temurin:21-jre-jammy

ENV LANG=C.UTF-8 LC_ALL=C.UTF-8

# 1. Create a non-root user
RUN groupadd --system appgroup && useradd --system --gid appgroup appuser

# 2. Set up directories
WORKDIR /opt/steve

# 3. Copy the WAR file from the builder
COPY --from=builder /code/target/steve.war /opt/steve.war

# 4. Copy the SQL migrations explicitly so the InitContainer can find them
COPY --from=builder /code/src/main/resources/db/migration /opt/steve/db/migration

# 5. Set ownership so appuser can access them
RUN chown -R appuser:appgroup /opt/steve

# 6. Final Configuration
EXPOSE 8180
USER appuser

# 7. Start the application
CMD ["java", "-Djdk.tls.client.protocols=TLSv1.2,TLSv1.3", "-jar", "/opt/steve.war"]