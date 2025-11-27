FROM eclipse-temurin:17-jdk-jammy as builder

WORKDIR /app
COPY target/student-management-0.0.1-SNAPSHOT.war app.war

# Second stage : image finale, plus légère
FROM eclipse-temurin:17-jre-jammy

WORKDIR /app

COPY --from=builder /app/app.war app.war

EXPOSE 8089

ENTRYPOINT ["java", "-jar", "/app/app.war"]
