# Backend - Innovatech Chile (Microservicios de Ventas y Despachos)

Backend Innovatech Chile
Descripción
Repositorio unificado que contiene los dos microservicios del backend de Innovatech Chile: API Ventas y API Despachos, desarrollados en Java con Spring Boot. Incluye la base de datos MySQL, orquestados mediante Docker Compose y desplegados automáticamente en AWS EC2 mediante CI/CD con GitHub Actions.

Tecnologías utilizadas

Java 17 + Spring Boot
Maven (gestión de dependencias y compilación)
MySQL 8.0 (base de datos)
Docker (multi-stage build)
Docker Compose (orquestación)
GitHub Actions (CI/CD)
Amazon ECR (registro de imágenes)
Amazon EC2 (despliegue)
AWS SSM (despliegue remoto)


Estructura del repositorio
backend/
├── .github/
│   └── workflows/
│       └── deploy.yml                    # Pipeline CI/CD
├── back-Ventas_SpringBoot/
│   └── Springboot-API-REST/
│       ├── Dockerfile                    # Imagen Docker Ventas
│       ├── src/                          # Código fuente Java
│       └── pom.xml
├── back-Despachos_SpringBoot/
│   └── Springboot-API-REST-DESPACHO/
│       ├── Dockerfile                    # Imagen Docker Despachos
│       ├── src/                          # Código fuente Java
│       └── pom.xml
├── docker-compose.yml                    # Orquestación completa
└── README.md

Contenedorización (IE1)
Dockerfile — Multi-stage Build (Ventas y Despachos)
Cada microservicio implementa una estrategia de construcción en dos etapas:
Stage 1 — Build con Maven:

Imagen base: maven:3.9-eclipse-temurin-17-alpine
Descarga dependencias con mvn dependency:go-offline
Compila el proyecto con mvn clean package -DskipTests
Genera el archivo .jar ejecutable

Stage 2 — Producción con JRE ligero:

Imagen base: eclipse-temurin:17-jre-alpine (solo JRE, sin JDK ni Maven)
Usuario no-root: se crea usuario spring con permisos limitados
Copia únicamente el .jar generado en el stage anterior
Expone el puerto correspondiente (8081 para Ventas, 8082 para Despachos)

Beneficios del multi-stage:

La imagen final no contiene Maven ni el JDK completo
Reduce el tamaño de la imagen de ~500MB a ~100MB
Minimiza vulnerabilidades al excluir herramientas de compilación

docker-compose.yml
Orquesta los tres servicios del backend de forma conjunta:
yamlservices:
  mysql:       # Base de datos — puerto 3306
  ventas:      # API Ventas — puerto 8081
  despachos:   # API Despachos — puerto 8082
Los servicios de ventas y despachos dependen de mysql mediante depends_on.

Persistencia de datos (IE2)
Volumen utilizado
yamlvolumes:
  mysql_data:
    driver: local
Montado en: /var/lib/mysql (ruta de datos de MySQL)
Tipo: Named Volume
Se eligió Named Volume por sobre Bind Mount por las siguientes razones:
CriterioNamed VolumeBind MountGestiónDocker lo administraDepende del SO hostPermisosSin conflictos con MySQLPuede causar errores de permisos en Linux EC2Rendimiento I/OSuperiorInferior en LinuxPortabilidadIndependiente del SODepende de la ruta del hostRespaldoFácil con docker volumeRequiere acceso al filesystem
Continuidad operativa
Los datos de ventas y despachos persisten aunque se reinicie o elimine el contenedor MySQL. Al ejecutar docker-compose up -d nuevamente, el volumen se reutiliza automáticamente con toda la información histórica intacta.

Variables de entorno
VariableDescripciónDB_HOSTHostname de la base de datos (nombre del servicio: mysql)DB_PORTPuerto MySQL (3306)DB_NAMENombre de la base de datos (innovatech)DB_USERUsuario MySQL (root)DB_PASSWORDContraseña MySQLMYSQL_ROOT_PASSWORDContraseña root de MySQLMYSQL_DATABASEBase de datos a crear automáticamente

Pipeline CI/CD (IE3)
Flujo completo
Push a rama deploy
       ↓
Checkout del código
       ↓
Configurar credenciales AWS
       ↓
Login en Amazon ECR
       ↓
Build imagen Ventas → Push a ECR
       ↓
Build imagen Despachos → Push a ECR
       ↓
Deploy en EC2 vía AWS SSM:
  - docker pull ambas imágenes
  - docker-compose up -d --remove-orphans
Activación
El pipeline se activa únicamente con push a la rama deploy:
yamlon:
  push:
    branches: [ "deploy" ]
GitHub Secrets configurados
SecretDescripciónAWS_ACCESS_KEY_IDCredencial AWS (se renueva cada 4h en Academy)AWS_SECRET_ACCESS_KEYCredencial AWS (se renueva cada 4h en Academy)AWS_SESSION_TOKENToken de sesión AWS (se renueva cada 4h en Academy)AWS_REGIONRegión AWS (us-east-1)ECR_REGISTRYURL base del registro ECRECR_REPO_URL_VENTASURL completa del repositorio ECR de VentasECR_REPO_URL_DESPACHOSURL completa del repositorio ECR de DespachosEC2_BACKEND_INSTANCE_IDID de la instancia EC2 del backend (i-xxxxx)

Infraestructura AWS (IE4)
Instancia EC2 Backend

Tipo: t2.micro — Amazon Linux 2023
Subred: Privada (no accesible directamente desde internet)
Puertos expuestos: 8081 (Ventas), 8082 (Despachos)
Security Group: permite tráfico en 8081 y 8082 únicamente desde el Security Group del frontend

Seguridad de red
Internet → EC2 Frontend (puerto 80) → EC2 Backend (puertos 8081/8082)
                                              ↓
                                        MySQL (puerto 3306, interno)
El puerto 3306 de MySQL no está expuesto al exterior. Solo los contenedores dentro de la red Docker interna pueden acceder a él.

Ejecución local
bash# Clonar el repositorio
git clone https://github.com/TU_USUARIO/backend.git
cd backend

# Crear archivo .env
echo "DB_PASSWORD=admin123" > .env
echo "DB_NAME=innovatech" >> .env

# Levantar todos los servicios
docker-compose up -d

# Verificar que están corriendo
docker ps
Endpoints disponibles

API Ventas: http://localhost:8081
API Despachos: http://localhost:8082


Principios DevOps aplicados (IE8)
PrácticaImplementaciónContenedorizaciónDocker multi-stage, usuario no-root, imagen Alpine ligeraGestión de entornosVariables de entorno para configuración de BDCI/CD automatizadoGitHub Actions construye y despliega ambos servicios en un solo pipelineControl de versionesGit con rama deploy para producciónPersistenciaNamed Volume mysql_data para datos críticos de la empresaSeguridadSecurity Group restrictivo, usuario no-root, secrets en GitHubTrazabilidadImágenes etiquetadas en ECR, logs en GitHub Actions

Justificación técnica
¿Por qué un solo repositorio para Ventas y Despachos?
Ambos microservicios comparten la misma base de datos MySQL y se despliegan en la misma instancia EC2. Unificarlos en un repositorio simplifica la orquestación con un único docker-compose.yml y un único pipeline de despliegue.
¿Por qué multi-stage build en Java?
Maven descarga cientos de dependencias y el JDK completo pesa ~500MB. Con multi-stage, la imagen final solo contiene el JRE y el .jar compilado, reduciendo el tamaño drásticamente y eliminando herramientas innecesarias en producción.
¿Por qué Named Volume para MySQL?
Los Named Volumes son gestionados por Docker, independientes de la estructura de carpetas del sistema operativo host. En Amazon Linux EC2, evita errores críticos de permisos con el usuario nativo de MySQL y garantiza que los datos históricos de ventas y despachos no se pierdan ante reinicios o actualizaciones de los contenedores.
¿Por qué SSM en lugar de SSH?
AWS SSM permite ejecutar comandos remotos sin necesidad de abrir el puerto 22 ni gestionar llaves SSH en el pipeline. En AWS Academy, las IPs públicas cambian con cada reinicio del laboratorio, pero los Instance IDs son permanentes, haciendo SSM más confiable para el despliegue automatizado.
Criticidad del CI/CD para Innovatech Chile
La automatización del pipeline elimina el error humano en los despliegues, garantiza que cada cambio en el código sea probado y desplegado de forma consistente, y reduce el tiempo de inactividad de los servicios de ventas y despachos. Esto es crítico para la continuidad operativa de Innovatech Chile.