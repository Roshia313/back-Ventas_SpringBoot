# Backend - Ventas SmartLogix

API REST hecha con Spring Boot para gestionar las ventas de SmartLogix. Se conecta a una base de datos MySQL compartida con el servicio de despachos y se despliega en una EC2 en la subred privada de AWS.

## Tecnologías

- Java 17 + Spring Boot 3
- Maven 3.9
- Spring Data JPA / Hibernate
- MySQL 8
- JUnit + Mockito (tests unitarios)
- SpringDoc (Swagger UI)
- Docker

## Estructura

```
Springboot-API-REST/
├── .github/workflows/deploy.yml
├── src/
│   ├── main/java/com/citt/
│   │   ├── controller/VentaController.java
│   │   ├── persistence/
│   │   │   ├── entity/Venta.java
│   │   │   ├── repository/
│   │   │   └── services/
│   │   ├── config/OpenApiConfing.java
│   │   └── exceptions/
│   └── test/java/persistence/service/VentaServiceTest.java
├── src/main/resources/application.properties
├── Dockerfile
└── pom.xml
```

## Cómo correr el proyecto

### Con docker-compose (recomendado)

Desde la carpeta raíz donde está el `docker-compose.yml`:

```bash
docker-compose up --build
```

El servicio queda en: `http://localhost:8080`

### Solo el contenedor

Necesitas una instancia MySQL corriendo:

```bash
docker build -t smartlogix-back-ventas .

docker run -d --name back-ventas-app -p 8080:8080 \
  -e SPRING_DATASOURCE_URL=jdbc:mysql://<HOST>:3306/smartlogix_db?useSSL=false \
  -e SPRING_DATASOURCE_USERNAME=<usuario> \
  -e SPRING_DATASOURCE_PASSWORD=<contraseña> \
  smartlogix-back-ventas
```

### Correr tests

```bash
mvn test
```

### Swagger UI

```
http://localhost:8080/swagger-ui.html
```

## Variables de entorno

| Variable | Descripción |
|---|---|
| `SPRING_DATASOURCE_URL` | URL de conexión a MySQL |
| `SPRING_DATASOURCE_USERNAME` | Usuario de la base de datos |
| `SPRING_DATASOURCE_PASSWORD` | Contraseña de la base de datos |

Puerto expuesto: `8080`

## Dockerfile

Multi-stage build en dos etapas:

- **Etapa 1:** `maven:3.9.6-eclipse-temurin-17`, compila con `mvn clean package` y genera el `.jar`
- **Etapa 2:** `eclipse-temurin:17-jre-alpine`, copia solo el `.jar`. Crea el usuario `spring` y corre sin privilegios de root

## Persistencia

Los datos van a MySQL. El volumen `mysql_data` del `docker-compose.yml` garantiza que no se pierda información al reiniciar los contenedores.

Se optó por **named volume** porque es manejado directamente por Docker, no depende de rutas del host y es más fácil de manejar en producción que un bind mount.

## Pipeline CI/CD

Se activa con push a la rama `deploy`.

Pasos:
1. Checkout del código
2. Configura credenciales AWS
3. Login en Amazon ECR
4. Build y push de la imagen (repositorio: `back-ventas`, tag: `latest`)
5. SSH a la EC2 del backend
6. Pull y reemplazo del contenedor con las variables de BD

Secrets necesarios:

| Secret | Descripción |
|---|---|
| `AWS_ACCESS_KEY_ID` | Credencial AWS Academy |
| `AWS_SECRET_ACCESS_KEY` | Credencial AWS Academy |
| `AWS_SESSION_TOKEN` | Token de sesión AWS Academy |
| `EC2_BACKEND_HOST` | IP de la EC2 del backend |
| `EC2_SSH_KEY` | Clave privada PEM para SSH |
| `DB_URL` | URL de la base de datos en producción |
| `DB_USERNAME` | Usuario de la BD en producción |
| `DB_PASSWORD` | Contraseña de la BD en producción |

## Notas

- El proceso corre como usuario `spring`, no como root
- Las credenciales van solo en GitHub Secrets, nunca en el código
- El servicio no es accesible desde internet, solo desde el frontend según los Security Groups