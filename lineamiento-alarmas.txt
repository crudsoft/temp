# Lineamiento para Generación de Alarmas CloudWatch

## 1. Objetivo

Este documento establece el procedimiento estándar para solicitar la creación de alarmas en CloudWatch basadas en métricas derivadas de queries de CloudWatch Logs Insights. El objetivo es mantener un monitoreo proactivo de los servicios y facilitar la detección temprana de incidentes.

## 2. Alcance

Este lineamiento aplica para todas las aplicaciones y servicios que requieran monitoreo mediante alarmas de CloudWatch, independientemente del proyecto o área.

## 3. Responsabilidades

- **Equipo de Desarrollo/Aplicación**: Definir los criterios de la alarma, umbrales, queries y documentación de resolución.
- **Equipo de Infraestructura/DevOps**: Implementar la alarma mediante Terraform y configurar las notificaciones correspondientes.

## 4. Proceso de Solicitud de Alarma

### 4.1. Definición de la Alarma

Antes de solicitar una alarma, el equipo solicitante debe definir claramente los siguientes componentes:

#### 4.1.1. Query de CloudWatch Logs Insights
- Escribir y validar el query en la consola de CloudWatch Logs Insights
- Asegurar que el query retorne un valor numérico que pueda ser usado como métrica
- Documentar el log group sobre el cual se ejecutará el query
- Validar que el query sea eficiente y no genere costos excesivos

#### 4.1.2. Métrica Personalizada
- **Nombre de la métrica**: Seguir nomenclatura estándar (ejemplo: `ErrorCount`, `HighLatencyRequests`, `FailedTransactions`)
- **Namespace**: Definir el namespace de la métrica (ejemplo: `CustomMetrics/OperationsApp`, `CustomMetrics/NombreProyecto`)
- **Dimensiones**: Especificar las dimensiones necesarias (ejemplo: `Environment`, `Service`, `Region`)
- **Unidad**: Definir la unidad de medida (Count, Milliseconds, Percent, etc.)

#### 4.1.3. Configuración de la Alarma
- **Nombre de la alarma**: Descriptivo y siguiendo convención de nombrado (ejemplo: `operations-app-prod-high-error-rate`)
- **Descripción**: Breve descripción del propósito de la alarma
- **Umbral**: Valor numérico que dispara la alarma
- **Condición**: Mayor que (>), menor que (<), mayor o igual (>=), menor o igual (<=)
- **Período de evaluación**: Tiempo en segundos para evaluar la métrica (ejemplo: 300 segundos = 5 minutos)
- **Puntos de datos para alarmar**: Cantidad de períodos consecutivos que deben superar el umbral
- **Tratamiento de datos faltantes**: Definir comportamiento cuando no hay datos (notBreaching, breaching, ignore, missing)

#### 4.1.4. Severidad y Notificaciones
- **Severidad**: Critical, High, Medium, Low
- **Canal de notificación**: Email, SNS Topic, Slack, PagerDuty, etc.
- **Destinatarios**: Lista de emails o canales que recibirán las notificaciones
- **Horario**: 24/7 o solo en horario laboral

#### 4.1.5. Documentación de Resolución (Runbook)
- **Link a documentación**: URL de Confluence, Wiki, GitHub, etc. con el procedimiento de resolución
- **Pasos iniciales de troubleshooting**: Descripción breve de qué revisar cuando se dispara la alarma
- **Acciones correctivas**: Qué hacer para resolver el problema
- **Escalamiento**: A quién contactar si la alarma persiste

### 4.2. Validación Pre-Solicitud

Antes de enviar la solicitud, verificar:

- [ ] El query ha sido probado y retorna valores correctos
- [ ] El umbral es realista basado en datos históricos
- [ ] La documentación de resolución está disponible y actualizada
- [ ] Los destinatarios de la alarma están correctamente identificados
- [ ] Se ha evaluado el impacto de falsos positivos/negativos

## 5. Template de Email de Solicitud

### 5.1. Subject del Email

```
[ALARMA] Solicitud de Nueva Alarma CloudWatch - [NOMBRE_APLICACIÓN] - [SEVERIDAD]
```

**Ejemplos:**
- `[ALARMA] Solicitud de Nueva Alarma CloudWatch - Operations App - CRITICAL`
- `[ALARMA] Solicitud de Nueva Alarma CloudWatch - Payment Service - HIGH`

### 5.2. Cuerpo del Email

```
Asunto: Solicitud de creación de alarma CloudWatch

Estimado equipo de Infraestructura/DevOps,

Solicitamos la creación de una nueva alarma CloudWatch con los siguientes detalles:

---

## INFORMACIÓN GENERAL

**Aplicación/Servicio:** [Nombre del servicio]
**Ambiente:** [Desarrollo/QA/Producción]
**Solicitante:** [Nombre y email]
**Equipo:** [Nombre del equipo]
**Fecha de solicitud:** [DD/MM/YYYY]
**Prioridad:** [Alta/Media/Baja]
**Severidad de la alarma:** [Critical/High/Medium/Low]

---

## CONFIGURACIÓN DEL QUERY

**Log Group:** [Nombre del log group]
**Región:** [us-east-1/us-west-2/etc]

**Query de CloudWatch Logs Insights:**
```
[Pegar aquí el query completo]
```

**Frecuencia de ejecución del query:** [Cada 5 minutos / Cada 1 minuto / etc]

---

## CONFIGURACIÓN DE LA MÉTRICA

**Namespace:** [Ejemplo: CustomMetrics/OperationsApp]
**Nombre de la métrica:** [Ejemplo: HighErrorRate]
**Dimensiones:**
- Environment: [production]
- Service: [operations-app]
- [Otras dimensiones necesarias]

**Unidad:** [Count/Milliseconds/Percent/Bytes/etc]

---

## CONFIGURACIÓN DE LA ALARMA

**Nombre de la alarma:** [Ejemplo: operations-app-prod-high-error-rate]
**Descripción:** [Breve descripción del propósito de la alarma]

**Condición de alarma:**
- Umbral: [Valor numérico]
- Operador: [Mayor que / Menor que / Mayor o igual / Menor o igual]
- Período de evaluación: [300 segundos (5 minutos)]
- Puntos de datos para alarmar: [2 de 2 / 3 de 5 / etc]
- Tratamiento de datos faltantes: [notBreaching/breaching/ignore/missing]

**Ejemplo:** "Alarmar cuando ErrorCount >= 10 durante 2 períodos consecutivos de 5 minutos"

---

## CONFIGURACIÓN DE NOTIFICACIONES

**Canal de notificación:** [SNS Topic / Email directo / Slack / PagerDuty]
**Destinatarios:**
- [email1@dominio.com]
- [email2@dominio.com]
- [Canal de Slack: #alertas-operaciones]

**Horario de notificaciones:** [24/7 / Lunes a Viernes 9-18hs]

---

## DOCUMENTACIÓN Y RUNBOOK

**Link a documentación de resolución:** [URL completa]

**Pasos iniciales de troubleshooting:**
1. [Paso 1: Verificar logs en CloudWatch]
2. [Paso 2: Revisar métricas de la aplicación]
3. [Paso 3: Validar estado del servicio]

**Acciones correctivas:**
- [Descripción de qué hacer para resolver el problema]

**Escalamiento:**
- Primer nivel: [Equipo/Persona - email/teléfono]
- Segundo nivel: [Equipo/Persona - email/teléfono]

---

## JUSTIFICACIÓN

**Motivo de la alarma:**
[Explicar por qué es necesaria esta alarma, qué problema detecta, y cuál es el impacto si no se monitorea]

**Datos históricos (si aplica):**
[Incluir información sobre incidentes previos, frecuencia del problema, etc.]

---

## INFORMACIÓN ADICIONAL

**Comentarios:**
[Cualquier información adicional relevante]

**Dependencias:**
[Si la alarma depende de otras configuraciones o recursos]

---

Quedamos atentos a cualquier consulta.

Saludos cordiales,
[Nombre del solicitante]
[Equipo/Área]
[Contacto]
```

## 6. Ejemplos Prácticos

### 6.1. Ejemplo 1: Alarma por Alta Tasa de Errores

**Subject:**
```
[ALARMA] Solicitud de Nueva Alarma CloudWatch - Operations App - CRITICAL
```

**Query de Logs Insights:**
```
fields @timestamp, @message
| filter @message like /ERROR/
| filter operationType = "TRANSFER"
| stats count() as errorCount by bin(5m)
| filter errorCount > 0
```

**Configuración:**
- Métrica: `ErrorCount`
- Namespace: `CustomMetrics/OperationsApp`
- Umbral: >= 10 errores
- Período: 5 minutos
- Puntos de datos: 2 de 2
- Severidad: CRITICAL

### 6.2. Ejemplo 2: Alarma por Latencia Alta

**Query de Logs Insights:**
```
fields @timestamp, duration
| filter operation = "confirmOperation"
| stats avg(duration) as avgLatency by bin(5m)
| filter avgLatency > 0
```

**Configuración:**
- Métrica: `OperationLatency`
- Namespace: `CustomMetrics/OperationsApp`
- Umbral: >= 5000 milliseconds
- Período: 5 minutos
- Puntos de datos: 3 de 5
- Severidad: HIGH

### 6.3. Ejemplo 3: Alarma por Operaciones Rechazadas

**Query de Logs Insights:**
```
fields @timestamp, status
| filter status = "REJECTED_BY_SEC_VALIDATION"
| stats count() as rejectedCount by bin(1m)
```

**Configuración:**
- Métrica: `RejectedOperationsCount`
- Namespace: `CustomMetrics/OperationsApp`
- Umbral: >= 5 operaciones
- Período: 60 segundos
- Puntos de datos: 2 de 2
- Severidad: HIGH

## 7. Buenas Prácticas

### 7.1. Definición de Umbrales
- Basar los umbrales en datos históricos reales
- Evitar umbrales muy sensibles que generen falsos positivos
- Considerar variaciones normales del tráfico (picos esperados)
- Revisar y ajustar umbrales periódicamente

### 7.2. Queries Eficientes
- Limitar el rango temporal del query
- Usar filtros específicos para reducir el volumen de datos procesados
- Evitar queries que consuman recursos excesivos
- Probar el query en producción antes de crear la métrica

### 7.3. Documentación
- Mantener la documentación de resolución actualizada
- Incluir ejemplos concretos de cómo investigar la alarma
- Documentar falsos positivos conocidos
- Incluir información de contacto actualizada

### 7.4. Nomenclatura
- Usar nombres descriptivos y consistentes
- Incluir el ambiente en el nombre (prod, qa, dev)
- Seguir convenciones de nombrado del equipo
- Ejemplo: `[servicio]-[ambiente]-[descripcion]`

### 7.5. Severidad
- **CRITICAL**: Impacto en servicio de producción, requiere atención inmediata 24/7
- **HIGH**: Impacto significativo, requiere atención en horario laboral extendido
- **MEDIUM**: Degradación de servicio, requiere atención en horario laboral
- **LOW**: Informativa, para seguimiento y análisis

## 8. SLA de Implementación

Una vez enviada la solicitud con toda la información completa:

- **Alarmas CRITICAL**: Implementación en 24-48 horas hábiles
- **Alarmas HIGH**: Implementación en 3-5 días hábiles
- **Alarmas MEDIUM/LOW**: Implementación en 5-10 días hábiles

**Nota:** Los tiempos pueden variar según la complejidad y la carga de trabajo del equipo de infraestructura.

## 9. Seguimiento y Mantenimiento

### 9.1. Revisión Periódica
- Revisar alarmas existentes trimestralmente
- Ajustar umbrales basados en comportamiento real
- Eliminar alarmas obsoletas o no utilizadas
- Actualizar documentación de resolución

### 9.2. Métricas de Alarmas
- Tasa de falsos positivos
- Tiempo promedio de resolución
- Frecuencia de disparo
- Efectividad de la alarma

## 10. Recursos y Referencias

### 10.1. Documentación AWS
- [CloudWatch Logs Insights Query Syntax](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/CWL_QuerySyntax.html)
- [CloudWatch Alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html)
- [CloudWatch Metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/working_with_metrics.html)

### 10.2. Contactos
- **Equipo de Infraestructura/DevOps:** [email-devops@dominio.com]
- **Soporte CloudWatch:** [soporte-cloud@dominio.com]

## 11. Anexos

### 11.1. Checklist de Solicitud

- [ ] Query de Logs Insights probado y validado
- [ ] Umbral definido basado en datos históricos
- [ ] Severidad correctamente asignada
- [ ] Destinatarios de notificaciones confirmados
- [ ] Documentación de resolución creada y accesible
- [ ] Link de runbook incluido en la solicitud
- [ ] Justificación de negocio documentada
- [ ] Revisado por líder técnico o arquitecto

### 11.2. Template de Runbook

```markdown
# Runbook: [Nombre de la Alarma]

## Descripción
[Qué detecta esta alarma]

## Impacto
[Impacto en el negocio/usuarios]

## Severidad
[Critical/High/Medium/Low]

## Pasos de Investigación
1. Verificar logs en CloudWatch: [Link directo al log group]
2. Revisar dashboard de métricas: [Link al dashboard]
3. Validar estado de servicios dependientes
4. [Otros pasos específicos]

## Causas Comunes
- [Causa 1 y cómo detectarla]
- [Causa 2 y cómo detectarla]
- [Causa 3 y cómo detectarla]

## Solución
1. [Paso 1 de resolución]
2. [Paso 2 de resolución]
3. [Paso 3 de resolución]

## Escalamiento
- **L1:** [Equipo/Persona - Contacto]
- **L2:** [Equipo/Persona - Contacto]
- **L3:** [Equipo/Persona - Contacto]

## Historial de Incidentes
- [Links a incidentes previos relacionados]

## Última actualización
[Fecha y responsable]
```

---

**Versión:** 1.0
**Última actualización:** Diciembre 2025
**Responsable:** Equipo de Arquitectura y DevOps
