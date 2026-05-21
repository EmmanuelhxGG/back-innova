# Etapa 1: Construcción con Maven
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn clean package -DskipTests

# Etapa 2: Imagen de ejecución ligera
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app

# Crear usuario no-root por seguridad
RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring

# Copiar el JAR desde la etapa de construcción
COPY --from=build /app/target/*.jar app.jar

# Puerto diferenciado para el segundo microservicio
EXPOSE 8081
ENTRYPOINT ["java", "-jar", "app.jar"]
