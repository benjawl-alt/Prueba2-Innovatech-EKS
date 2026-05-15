# ETAPA 1: Construcción (Build)
FROM maven:3.8.5-openjdk-17-slim AS build
WORKDIR /app
# Copiar solo el pom primero para aprovechar el caché de Docker
COPY pom.xml .
RUN mvn dependency:go-offline

# Copiar el código fuente y compilar
COPY src ./src
RUN mvn clean package -DskipTests

# ETAPA 2: Ejecución (Runtime)
FROM openjdk:17-jdk-slim
WORKDIR /app

# Seguridad: Crear usuario no-root (Requisito IE1)
RUN groupadd -r devops && useradd -r -g devops devops
USER devops

# Copiar solo el archivo JAR resultante de la etapa anterior
COPY --from=build /app/target/*.jar app.jar

EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]