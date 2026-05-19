# --- ETAPA 1: Compilación ---
FROM maven:3.9.6-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline -B
COPY src ./src
RUN mvn clean package -DskipTests

# --- ETAPA 2: Imagen de producción ---
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app

# Crear usuario de sistema sin privilegios
RUN addgroup --system spring && adduser --system --ingroup spring spring

# Copiar el jar asignando los permisos correspondientes
COPY --from=build --chown=spring:spring /app/target/*.jar app.jar

# Cambiar al usuario no-root
USER spring
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]