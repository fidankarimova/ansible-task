FROM maven:3.9-eclipse-temurin-25 AS build
WORKDIR /app
COPY . .
# download all maven deps offline, then build the project without running tests
RUN ./mvnw dependency:go-offline -B && ./mvnw clean package -DskipTests

FROM eclipse-temurin:25-jre-alpine
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
