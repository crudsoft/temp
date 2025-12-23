# Operations App - MiddleAPI

API RESTful para la gestión de operaciones bancarias con autenticación de dos factores y validación de seguridad integrada.

## Descripción General

Operations App es un microservicio desarrollado en Spring Boot que proporciona un sistema robusto para la gestión del ciclo de vida completo de operaciones bancarias. El servicio implementa flujos de autenticación de dos factores (2FA), validación de seguridad y gestión de estados de operaciones.

## Características Principales

- **Gestión de Operaciones**: Creación, consulta, confirmación y rechazo de operaciones bancarias
- **Autenticación de Dos Factores (2FA)**: Integración con sistema de seguridad para envío y validación de códigos
- **Validación de Seguridad**: Integración con servicio externo de validación de seguridad
- **Multi-canal**: Soporte para diferentes canales (ILE, ILE_APP, etc.)
- **Paginación**: Consultas paginadas de operaciones con filtros avanzados
- **Gestión de Estados**: Máquina de estados completa para el ciclo de vida de operaciones
- **Whitelist de Clientes**: Registro de clientes en lista blanca para operaciones bloqueadas

## Arquitectura

### Arquitectura General

```
┌─────────────────┐
│   Controllers   │ ← Capa de presentación (REST API)
└────────┬────────┘
         │
┌────────▼────────┐
│    Services     │ ← Lógica de negocio
└────────┬────────┘
         │
┌────────▼────────┐
│  Repositories   │ ← Capa de acceso a datos
└────────┬────────┘
         │
┌────────▼────────┐
│   PostgreSQL    │ ← Base de datos
└─────────────────┘
```

### Estructura del Proyecto

```
operations-app/
├── app/
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/uy/com/itau/middleapi/operations/
│   │   │   │   ├── controller/       # Controladores REST
│   │   │   │   ├── service/          # Servicios de negocio
│   │   │   │   │   └── impl/         # Implementaciones de servicios
│   │   │   │   ├── repository/       # Repositorios JPA
│   │   │   │   ├── model/            # Entidades JPA
│   │   │   │   ├── dto/              # Objetos de transferencia de datos
│   │   │   │   ├── mapper/           # Mappers (MapStruct)
│   │   │   │   ├── exception/        # Excepciones personalizadas
│   │   │   │   ├── validation/       # Validadores
│   │   │   │   ├── filter/           # Filtros HTTP
│   │   │   │   └── util/             # Utilidades
│   │   │   └── resources/
│   │   │       ├── application.properties
│   │   │       ├── logback.xml
│   │   │       └── parameter_stores.json
│   │   └── test/
│   └── pom.xml
├── bd_scripts/                        # Scripts de base de datos
├── ecs/                               # Configuración ECS
└── Jenkinsfile                        # Pipeline CI/CD
```

### Componentes Principales

#### 1. Controladores

- **OperationsController**: Endpoints principales para gestión de operaciones
- **TwoFactorController**: Endpoints para autenticación de dos factores
- **HealthController**: Endpoint de salud del servicio

#### 2. Servicios

- **OperationService**: Lógica de negocio para operaciones
- **TwoFactorService**: Lógica para autenticación 2FA
- **HealthService**: Verificación de salud del servicio

#### 3. Modelos de Datos

- **Operation**: Entidad principal que representa una operación
- **TwoFactorAuth**: Configuración de autenticación 2FA asociada a una operación
- **BaseEntity**: Entidad base con campos de auditoría

#### 4. Estados de Operación

```java
PENDING                                  // Operación pendiente
CANCELED                                 // Cancelada por nueva operación
TWO_FACTOR_CONFIRMED                     // Confirmada vía 2FA
CONFIRMED                                // Confirmada definitivamente
REJECTED_TOO_MANY_ATTEMPTS              // Rechazada por múltiples intentos
REJECTED_TIMEOUT                        // Rechazada por timeout
REJECTED_BY_DOMAIN                      // Rechazada por dominio
REJECTED_BY_CLIENT                      // Rechazada por cliente
REJECTED_BY_SEC_VALIDATION_RULES        // Rechazada por reglas de seguridad
REJECTED_BY_SEC_VALIDATION              // Rechazada por validación de seguridad
REJECTED_BY_SEC_VALIDATION_INTERNAL_ERROR // Error interno en validación
```

## API Endpoints

### Operaciones

#### Crear Operación
```http
POST /api-operations/v1/operations
Content-Type: application/json

{
  "operationType": "TRANSFER",
  "documentType": 1,
  "documentNumber": "12345678",
  "username": "user123",
  "channel": "ILE_APP",
  "deviceId": "device-uuid",
  "sourceId": "ref-123",
  "initialData": {
    "amount": 1000,
    "currency": "UYU"
  },
  "additionalInformation": {}
}
```

#### Obtener Operación
```http
GET /api-operations/v1/operations/{id}
```

#### Listar Operaciones (con paginación)
```http
GET /api-operations/v1/operations?documentNumber=12345678&documentType=1&from=2024-01-01&to=2024-12-31&operationType=TRANSFER&status=PENDING&page=0&size=20
```

#### Confirmar Operación
```http
PUT /api-operations/v1/operations/{id}/confirmation
Content-Type: application/json

{
  "documentType": 1,
  "documentNumber": "12345678",
  "sourceId": "ref-123",
  "confirmedData": {
    "transactionId": "txn-456"
  }
}
```

#### Verificar Operación
```http
GET /api-operations/v1/operations/{id}/verify?documentType=1&documentNumber=12345678&sourceId=ref-123
```

#### Rechazar Operación
```http
PUT /api-operations/v1/operations/{id}/reject
Content-Type: application/json

{
  "message": "Operación rechazada por el usuario"
}
```

#### Agregar a Whitelist
```http
PUT /api-operations/v1/operations/{id}/whitelist
```

### Autenticación de Dos Factores

#### Enviar Código 2FA
```http
POST /api-operations/v1/operations/{id}/two-factor/send
Content-Type: application/json

{
  "twoFactorAuthType": "SMS"
}
```

#### Validar Código 2FA
```http
POST /api-operations/v1/operations/{id}/two-factor/validate
Content-Type: application/json

{
  "twoFactorAuthType": "SMS",
  "token": "123456"
}
```

### Health Check

```http
GET /api-operations/v1/health
```

## Flujo de Operaciones

### Flujo Normal

```
1. Cliente crea operación
   ↓
2. Sistema valida con servicio de seguridad
   ↓
3. Sistema determina si requiere 2FA
   ├─ No requiere 2FA → Estado: PENDING
   └─ Requiere 2FA → Estado: PENDING + registro de métodos 2FA
      ↓
   4. Cliente solicita envío de código 2FA
      ↓
   5. Cliente valida código 2FA
      ↓
   6. Estado: TWO_FACTOR_CONFIRMED
   ↓
7. Cliente confirma operación
   ↓
8. Estado: CONFIRMED
```

### Flujo con Validación de Seguridad

```
1. Crear operación
   ↓
2. Validación de seguridad (si está habilitada)
   ├─ Aprobada
   │  ├─ Sin 2FA requerido → PENDING
   │  └─ Con 2FA requerido → PENDING + TwoFactorAuth
   └─ Rechazada
      ├─ Por reglas → REJECTED_BY_SEC_VALIDATION_RULES
      ├─ Por validación → REJECTED_BY_SEC_VALIDATION
      └─ Error interno → REJECTED_BY_SEC_VALIDATION_INTERNAL_ERROR
```

### Flujo por Canal

- **Canal ILE/ILE_APP**: No requiere validación de seguridad ni invalidación de operaciones previas
- **Otros canales**: Requiere validación de seguridad y cancela operaciones pendientes previas del mismo usuario

## Base de Datos

### Esquema

#### TB_OPERATIONS
```sql
- ID (UUID, PK)
- DOCUMENT_TYPE (INTEGER)
- DOCUMENT_NUMBER (VARCHAR)
- USERNAME (VARCHAR)
- OPERATION_TYPE (VARCHAR)
- SOURCE_ID (VARCHAR)
- STATUS (VARCHAR)
- INITIAL_DATA (JSON)
- CONFIRMED_DATA (JSON)
- CHANNEL (VARCHAR)
- SEC_VALIDATION_CODE (VARCHAR)
- SEC_VALIDATION_MESSAGE (VARCHAR)
- REJECT_MESSAGE (VARCHAR)
- DEVICE_ID (VARCHAR)
- CREATED_AT (TIMESTAMP)
- UPDATED_AT (TIMESTAMP)
- CREATED_BY (VARCHAR)
- UPDATED_BY (VARCHAR)
```

#### TB_OPERATIONS_TWO_FACTORS_AUTH
```sql
- ID (UUID, PK)
- OPERATION_ID (UUID, FK)
- TYPE (VARCHAR)
- IS_USED (BOOLEAN)
- RETRY_COUNT (INTEGER)
- CREATED_AT (TIMESTAMP)
- UPDATED_AT (TIMESTAMP)
- CREATED_BY (VARCHAR)
- UPDATED_BY (VARCHAR)
```

### Scripts de Migración

Los scripts de base de datos se encuentran en `/bd_scripts`:

1. `1_operations_create_schema.sql` - Creación del esquema
2. `2_operations-tb_operations.sql` - Tabla principal de operaciones
3. `3_operations-tb_operations_two_factor_auth.sql` - Tabla de autenticación 2FA
4. `5_operations_role.sql` - Roles y permisos
5. `6-10_operations_alter_table_operations.sql` - Alteraciones de tabla

## Stack Tecnológico

### Framework y Lenguaje
- **Java 17**
- **Spring Boot 3.3.5**
- **Spring Data JPA**
- **Spring Web**

### Base de Datos
- **PostgreSQL 42.2.20**
- **Hibernate 6.4.4**

### Utilidades y Librerías
- **Lombok 1.18.36** - Reducción de código boilerplate
- **MapStruct 1.6.3** - Mapeo entre entidades y DTOs
- **Jackson** - Serialización/deserialización JSON
- **Logback 1.5.12** - Logging
- **Logstash Logback Encoder 7.4** - Logs estructurados

### Módulos Internos de Itaú
- **aws-rds-postgresql 1.2** - Conectividad a RDS
- **common-util 1.0.17** - Utilidades comunes
- **api-integration 1.2.5** - Integración con APIs
- **traceable 1.4.1** - Trazabilidad de requests
- **aws-parameter-store 1.1.7** - Configuración desde Parameter Store
- **spring-cloud-starter-aws-secrets-manager-config 2.4.4** - Secrets Manager

### Testing
- **JUnit** - Framework de pruebas
- **Mockito** - Mocking
- **Spring Boot Test** - Testing de integración

## Configuración

### Variables de Entorno

Las variables se configuran en `application.properties`:

#### Servidor
```properties
server.servlet.context-path=/api-operations
server.port=8080
```

#### Base de Datos
```properties
# Escritura
spring.datasource.write.url=jdbc-secretsmanager:postgresql://${AURORA_ADDRESS_WRITE}/${DB_ID}
spring.datasource.write.username=${AURORA_USER_PASS}
spring.datasource.write.maximum-pool-size=50
spring.datasource.write.minimum-idle=5

# Lectura
spring.datasource.read.url=jdbc-secretsmanager:postgresql://${AURORA_ADDRESS_READ}/${DB_ID}
spring.datasource.read.username=${AURORA_USER_PASS}
spring.datasource.read.maximum-pool-size=50
spring.datasource.read.minimum-idle=5
```

#### JPA/Hibernate
```properties
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.properties.hibernate.default_schema="OPERATIONS"
```

#### AWS
```properties
aws.region=us-east-1
API_KEY_SM=dev/middleapi/waf_private_cdn_auth_token
```

#### Integración Externa
```properties
API-PRIVATE-CUSTOM_DOMAIN=api-private.middleapi.dev.cloudbiu.com:8081
SEC_VALIDATION_ENABLED=true
```

### Parameter Store

La configuración de parámetros externos se encuentra en:
- `app/src/main/resources/parameter_stores.json`

### Tipos de Operaciones

Los tipos de operaciones soportados se definen en:
- `app/src/main/resources/middleapi/operations/operation_types.json`

## Instalación y Ejecución

### Prerrequisitos

- Java JDK 17
- Maven 3.9+
- PostgreSQL 12+
- Docker (opcional, para despliegue)

### Instalación Local

1. **Clonar el repositorio**
```bash
cd operations-app
```

2. **Configurar base de datos**
```bash
# Crear base de datos
psql -U postgres -c "CREATE DATABASE operations_db;"

# Ejecutar scripts de migración
psql -U postgres -d operations_db -f bd_scripts/1_operations_create_schema.sql
psql -U postgres -d operations_db -f bd_scripts/2_operations-tb_operations.sql
psql -U postgres -d operations_db -f bd_scripts/3_operations-tb_operations_two_factor_auth.sql
# ... ejecutar resto de scripts
```

3. **Configurar variables de entorno**
```bash
# Editar application.properties con configuraciones locales
cd app/src/main/resources
nano application.properties
```

4. **Compilar el proyecto**
```bash
cd app
mvn clean package
```

5. **Ejecutar tests**
```bash
mvn test
```

6. **Ejecutar la aplicación**
```bash
mvn spring-boot:run
```

La aplicación estará disponible en: `http://localhost:8080/api-operations`

### Ejecución con Docker

```bash
# Construir imagen
docker build -t operations-app:latest .

# Ejecutar contenedor
docker run -p 8080:8080 \
  -e AURORA_ADDRESS_WRITE=your-db-host \
  -e AURORA_ADDRESS_READ=your-db-host \
  -e AURORA_USER_PASS=your-db-password \
  -e DB_ID=operations_db \
  operations-app:latest
```

## CI/CD

### Pipeline Jenkins

El proyecto incluye un `Jenkinsfile` completo que implementa:

#### Etapas del Pipeline

1. **Environment** - Configuración de variables de entorno
2. **Printing Variables** - Validación de configuración
3. **Git Clone Scripts** - Clonación de scripts DevOps
4. **Test** - Ejecución de tests unitarios con Maven
5. **Package** - Empaquetado de aplicación
6. **Sonar** - Análisis de calidad de código
7. **Quality Gate** - Validación de umbrales de calidad
8. **Docker - Source to Image** - Construcción de imagen Docker
9. **Tag and Push into Registry** - Publicación en Artifactory
10. **AWS Properties JSON read** - Configuración para ECS
11. **AWS Deploy** - Despliegue en ECS
12. **Deploy Replicated Services** - Despliegue de réplicas

#### Configuración de Despliegue

El despliegue se configura en:
- `deploy_properties.json` - Propiedades por ambiente
- `ecs/properties.json` - Configuración de ECS task
- `ecs/replicas/*.json` - Configuración de servicios replicados

#### Ambientes

- **Develop**: Despliegue automático desde rama principal
- **AWS Account**: 211125757137
- **Region**: us-east-1
- **Stage**: dev
- **Product**: middleapi

### Auto Scaling

El servicio se despliega con auto scaling configurado:
- Políticas de scale-in/scale-out
- Alarmas de CloudWatch
- Métricas de CPU y memoria

## Gestión de Errores

### Excepciones Personalizadas

- `OperationNotFoundException` - Operación no encontrada
- `OperationAlreadyConfirmedException` - Operación ya confirmada
- `OperationNotTwoFactorConfirmedException` - Falta confirmación 2FA
- `OperationNotPendingException` - Operación no está pendiente
- `OperationRejectedException` - Operación rechazada
- `OperationTwoFactorNotAvailable` - Método 2FA no disponible
- `BlockedOperationNotFoundException` - Operación bloqueada no encontrada
- `DocumentNotCorrespondToThisOperation` - Documento no corresponde
- `ReferenceIdMismatchException` - ID de referencia no coincide
- `SecValidationRejectedException` - Validación de seguridad rechazada
- `ValidationException` - Error de validación
- `GenericException` - Error genérico

### Manejador Global de Excepciones

El `GlobalExceptionHandler` gestiona todas las excepciones y retorna respuestas HTTP apropiadas con formato estandarizado:

```json
{
  "errorCode": "OP001",
  "errorMessage": "Descripción del error",
  "timestamp": "2024-01-15T10:30:00"
}
```

## Logging

### Configuración

El logging se configura en `app/src/main/resources/logback.xml` con:

- Formato JSON para integración con sistemas de logging
- Niveles configurables por paquete
- Appenders para consola y archivo
- Integración con Logstash para centralización de logs

### Niveles de Log

```
ERROR - Errores críticos
WARN  - Advertencias importantes
INFO  - Información general de operaciones
DEBUG - Información detallada para debugging
```

## Seguridad

### Integración con Servicios de Seguridad

El servicio se integra con dos APIs de seguridad:

1. **Security Validation API** (`/api-security-validation`)
   - Análisis de riesgo de operaciones
   - Determinación de requisitos 2FA
   - Validación de reglas de seguridad

2. **Security API** (`/api-security`)
   - Envío de códigos 2FA
   - Validación de tokens 2FA

### Autenticación de Dos Factores

Métodos soportados (configurables por tipo de operación):
- SMS
- Email
- Token App
- Otros (definidos en `TwoFactorAuthType`)

### Whitelist de Clientes

Sistema para registrar clientes confiables y evitar validaciones repetitivas en operaciones futuras.

## Monitoreo y Observabilidad

### Health Check

Endpoint `/v1/health` proporciona:
- Estado del servicio
- Conectividad con base de datos
- Estado de dependencias

### Métricas

El servicio se despliega con configuración de observabilidad:
- Alarmas de CloudWatch
- Métricas de ECS (CPU, memoria)
- Auto scaling basado en métricas
- Logs centralizados en CloudWatch Logs

### Trazabilidad

Módulo `traceable` proporciona:
- Tracking de requests
- Correlación de logs
- Headers de trazabilidad

## Desarrollo

### Convenciones de Código

- Uso de Lombok para reducir boilerplate
- MapStruct para mapeo de entidades
- DTOs separados para requests y responses
- Validación en capa de controller y service
- Transacciones explícitas donde corresponda

### Testing

Estructura de tests:
```
app/src/test/java/
└── uy/com/itau/middleapi/operations/
    └── controller/
        └── HealthControllerTest.java
```

Ejecutar tests:
```bash
mvn test
```

### Agregar Nueva Operación

1. Definir tipo de operación en `operation_types.json`
2. Configurar template 2FA si es necesario
3. Actualizar validadores si requiere reglas específicas
4. Implementar lógica de negocio en service
5. Agregar tests unitarios

## Contribución

### Flujo de Trabajo

1. Crear rama feature desde develop
2. Implementar cambios con tests
3. Ejecutar quality gate local
4. Crear merge request
5. Revisión de código
6. Merge a develop
7. Deploy automático a DEV

### Estándares

- Cobertura de tests mínima: según Quality Gate de SonarQube
- Documentación de APIs con comentarios
- Logs informativos en operaciones críticas
- Manejo apropiado de excepciones

## Licencia

Proyecto interno de Itaú - Todos los derechos reservados

## Contacto

Para consultas sobre el proyecto:
- Producto: MiddleAPI
- Ambiente: DEV
- Account: 211125757137

## Solución de Problemas

### Error de Conexión a Base de Datos

Verificar:
- Credenciales en Secrets Manager
- Conectividad de red a Aurora
- Pool de conexiones no saturado

### Error en Validación de Seguridad

Verificar:
- `SEC_VALIDATION_ENABLED` está configurado
- `API-PRIVATE-CUSTOM_DOMAIN` es correcto
- Servicio de seguridad está disponible
- API Key en Secrets Manager es válida

### Operación no Requiere 2FA cuando Debería

Verificar:
- Configuración en `operation_types.json`
- Respuesta del servicio de validación de seguridad
- Canal de la operación (ILE no requiere 2FA)

### Error en Deploy ECS

Verificar:
- Task definition en `ecs/properties.json`
- Límites de CPU/memoria
- Variables de entorno configuradas
- Permisos IAM del service role
