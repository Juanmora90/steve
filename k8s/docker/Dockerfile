# This Dockerfile expects to be run AFTER 'mvn clean package' has already finished.
# It simply packages the pre-built WAR and migration files.

FROM eclipse-temurin:21-jre-jammy

ENV LANG=C.UTF-8 LC_ALL=C.UTF-8

# 1. Create a non-root user
RUN groupadd --system appgroup && useradd --system --gid appgroup appuser

# 2. Set up directories
WORKDIR /opt/steve

# 3. Copy the WAR file (Built by GitHub Actions in the previous step)
COPY target/steve.war /opt/steve.war

# 4. Copy the SQL migrations explicitly so the InitContainer can find them
#    (Note: The path is relative to the root of the repo, passed as context)
COPY src/main/resources/db/migration /opt/steve/db/migration

# 5. Set ownership
RUN chown -R appuser:appgroup /opt/steve

# 6. Final Configuration
EXPOSE 8180
USER appuser

# 7. Start the application
CMD ["java", "-Djdk.tls.client.protocols=TLSv1.2,TLSv1.3", "-jar", "/opt/steve.war"]