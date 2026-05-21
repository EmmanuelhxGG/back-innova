FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

FROM eclipse-temurin:17-jre-alpine
# Creación de usuario no-root por seguridad (IE1, IE6)
RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring

WORKDIR /app
# Copiamos el jar generado en la etapa build
COPY --from=build /app/target/*.jar app.jar

# Puerto específico para Despachos
EXPOSE 8082
ENTRYPOINT ["java", "-jar", "app.jar"]