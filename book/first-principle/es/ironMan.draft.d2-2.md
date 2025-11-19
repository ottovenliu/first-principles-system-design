# Día 2-2 | Confirmación de Requisitos × Punto de Partida del Diseño de Sistemas (2): Límites de Dominio y Confirmación de Requisitos Básicos

Ayer, extrajimos la esencia de los requisitos funcionales de las palabras del usuario y comprendimos el propósito de la existencia del sistema bajo diferentes modelos cognitivos. Pero cuando estamos listos para transformar estas comprensiones abstractas en una arquitectura AWS concreta, encontraremos un problema clave:

**Los requisitos funcionales responden "qué hacer", pero no "hasta qué punto".**

Cuando los usuarios de inversión y trading dicen "me temo que la velocidad lenta me hará perder dinero", ¿qué quieren decir realmente? ¿Cuántos milisegundos de retraso son aceptables? Cuando los usuarios de finanzas familiares dicen "organizar gastos en cualquier momento", ¿cuántos usuarios concurrentes debe soportar el sistema? Cuando los usuarios de monitoreo de salud dicen "no puede afectar el juicio", ¿cuánto tiempo de inactividad es aceptable?

Estas son las áreas de los **requisitos no funcionales**, que determinan los límites técnicos del sistema y la configuración específica de los servicios de AWS.

## La Ontología de los Requisitos No Funcionales: De la Transformación Cualitativa a la Cuantitativa

### Tres Niveles de Límites Técnicos

**Límites de Rendimiento**

- Los límites superior e inferior de solicitudes API por segundo (RPS)
- El valor esperado y rango de tolerancia del tiempo de respuesta
- Los valores pico y promedio del rendimiento de procesamiento de datos

**Límites de Capacidad**

- La curva esperada y planificación de picos del crecimiento de usuarios
- El modelo de crecimiento y retención histórica del almacenamiento de datos
- La expansión elástica y control de costos de los recursos de cómputo

**Límites de Resiliencia**

- Objetivo de Tiempo de Recuperación (RTO) aceptable
- Objetivo de Punto de Recuperación (RPO) tolerable
- El rango de degradación gradual del sistema y aseguramiento de funciones centrales

### Derivando Especificaciones Técnicas desde Eventos de Dominio

Cada dominio tiene su **ritmo técnico** único, que determina las características de rendimiento del sistema:

**Dominio de Trading Financiero**: Ritmo competitivo a nivel de microsegundos

- Actualizaciones de datos de mercado: miles de veces por segundo
- Retraso en decisiones de trading: milisegundos son inaceptables
- Precisión de cálculo de posiciones: 8 decimales

**Dominio de Gestión Familiar**: Ritmo colaborativo de la vida diaria

- Frecuencia de registro de gastos: varias a docenas de veces al día
- Tolerancia al retraso de sincronización: segundos son aceptables
- Requisitos de consistencia de datos: consistencia eventual es suficiente

**Dominio de Monitoreo de Salud**: Ritmo de monitoreo de ciclos fisiológicos

- Sincronización de datos de dispositivos: minutos a horas
- Retraso de alarma de anomalías: los indicadores clave necesitan ser en tiempo real
- Preservación de datos históricos: almacenamiento a largo plazo a nivel de años

## Solicitudes API por Segundo: Del Comportamiento del Usuario a los Indicadores Técnicos

### El Fundamento Filosófico del RPS

El RPS no es un número estimado de la nada, sino un resultado inevitable derivado de los patrones de comportamiento del usuario. Detrás de cada llamada API hay una intención del usuario, y la distribución de frecuencia de las intenciones del usuario determina las características de carga del sistema.

#### Derivación del RPS para el Sistema de Trading de Inversiones

**Análisis de Patrones de Comportamiento del Usuario**:

**Estratificación Conductual**:

- Inversores ordinarios: verifican posiciones 2-5 veces al día, operan 0-2 veces
- Traders activos: verifican posiciones 20-50 veces al día, operan 5-15 veces
- Traders de alta frecuencia: verifican posiciones cada segundo, pueden operar cada minuto

**Distribución Temporal**:

- 30 minutos antes de la apertura del mercado: picos de tráfico, 10 veces lo usual
- Sesión de trading: tráfico alto continuo, 5 veces lo usual
- Después del cierre del mercado: gradualmente regresa a la línea base

**Lógica de Cálculo**:

- Usuarios ordinarios: 5 llamadas API/día ÷ 8 horas = 0.17 RPS/usuario
- Usuarios activos: 50 llamadas API/día ÷ 8 horas = 1.74 RPS/usuario
- Usuarios de alta frecuencia: 3600 llamadas API/hora ÷ 3600 segundos = 60 RPS/usuario
- Asumiendo distribución de usuarios: 80% ordinarios, 15% activos, 5% alta frecuencia
- RPS Total = (0.8 × 0.17 + 0.15 × 1.74 + 0.05 × 60) × usuarios_totales
- = (0.136 + 0.261 + 3) × usuarios_totales
- = 3.397 × usuarios_totales

**Umbrales de RPS para Selección de Servicios AWS**:

- < 100 RPS: API Gateway + Lambda (Serverless primero)
- 100-1000 RPS: ALB + ECS/EC2 (Contenedor o VM)
- 1000-10000 RPS: ALB + Auto Scaling Groups
- > 10000 RPS: Custom Load Balancer + Despliegue Multi-AZ

#### Derivación del RPS para el Sistema Financiero Familiar

**Agregación Temporal del Comportamiento Colaborativo**:

**Modelo de Colaboración Familiar**:

- Promedio de 3.5 personas por familia, con 2 usuarios frecuentes
- Períodos de uso concentrado: después de la cena (19:00-21:00), períodos de compras del fin de semana
- Conflictos de colaboración: probabilidad de que varias personas registren el mismo gasto al mismo tiempo

**Lógica de Cálculo**:

- Noches entre semana: 2 usuarios/familia × 5 registros/2horas = 2.5 RPS/familia/1000familias
- Pico de fin de semana: 2.5 × 3 = 7.5 RPS/1000familias
- Temporada anual de impuestos: 7.5 × 2 = 15 RPS/1000familias

#### Derivación del RPS para el Sistema de Monitoreo de Salud

**Modelo de Envío de Datos de Dispositivos IoT**:

**Tipo de Dispositivo y Frecuencia**:

- Pulsera inteligente: frecuencia cardíaca cada 30 segundos, pasos cada 10 minutos
- Monitor de presión arterial: medición manual, 1-3 veces al día
- Medidor de glucosa en sangre: medición manual, 2-6 veces al día
- Báscula inteligente: medición manual, 2-3 veces a la semana

**Lógica de Cálculo**:

- Pulsera: (120 + 6) llamadas API/día = 126/día = 0.0015 RPS/usuario
- Monitor de presión arterial: 3 llamadas API/día = 0.000035 RPS/usuario
- Medidor de glucosa: 6 llamadas API/día = 0.00007 RPS/usuario
- Báscula: 0.4 llamadas API/día = 0.000005 RPS/usuario
- Total: ≈ 0.002 RPS/usuario
- Pero considerando el procesamiento por lotes de sincronización de datos y reintentos por excepciones:
- RPS Real = 0.002 × factor_reintento × factor_lote ≈ 0.01 RPS/usuario

### La Filosofía del Diseño de API: De la Interfaz al Contrato

Una API no es solo una interfaz técnica, sino una **expresión programática de conceptos de dominio**. Cada endpoint de API debe corresponder a una operación significativa en el dominio.

**Mapeo Semántico de Dominio RESTful**:

**Dominio de Trading de Inversiones**:

- POST /portfolios/{id}/orders - Enviar una orden de trading
- GET /portfolios/{id}/positions - Consultar estado de posiciones
- PATCH /accounts/{id}/risk-preferences - Actualizar preferencia de riesgo
- GET /market/quotes/{symbol} - Obtener cotización de mercado

**Dominio de Finanzas Familiares**:

- POST /families/{id}/expenses - Registrar un gasto
- GET /families/{id}/budget/status - Consultar estado del presupuesto
- PUT /families/{id}/members/{member_id}/permissions - Establecer permisos
- GET /families/{id}/reports/{period} - Generar un informe financiero

**Estrategia de Limitación de AWS API Gateway**:

```yaml
ThrottlingSettings:
  BurstLimit: 5000 # Límite de tráfico en ráfaga
  RateLimit: 2000 # Límite de tráfico en estado estable

UsagePlan:
  BasicTier:
    Throttle: { BurstLimit: 100, RateLimit: 50 }
    Quota: { Limit: 1000, Period: DAY }
  PremiumTier:
    Throttle: { BurstLimit: 1000, RateLimit: 500 }
    Quota: { Limit: 10000, Period: DAY }
```

## Planificación de Capacidad: Pensamiento Sistémico para Patrones de Crecimiento

### Reconocimiento de Patrones de Crecimiento Empresarial

Diferentes negocios tienen diferentes curvas de crecimiento, lo que afecta directamente las estrategias de planificación de capacidad:

**Modelo de Crecimiento Lineal (Gestión Financiera Familiar)**

- Características: Crecimiento estable de usuarios, patrones de uso predecibles
- Función de crecimiento: capacidad(t) = línea_base + tasa_crecimiento × t
- Estrategia AWS: Reserved Instances + Predictable Scaling
- Optimización de costos: Compromiso a largo plazo a cambio de ahorro de costos

**Modelo de Crecimiento Exponencial (Plataforma de Compartición Social)**

- Características: Difusión viral, tráfico extremadamente volátil
- Función de crecimiento: capacidad(t) = línea_base × (1 + tasa_crecimiento)^t
- Estrategia AWS: Serverless + Auto Scaling + Spot Instances
- Optimización de costos: Pago por uso, escalado elástico rápido

**Modelo de Crecimiento Cíclico (Sistema de Trading de Inversiones)**

- Características: Altamente correlacionado con ciclos externos (horarios de apertura del mercado)
- Función de crecimiento: capacidad(t) = línea_base + factor_estacional × sin(2π × t/periodo)
- Estrategia AWS: Scheduled Scaling + Predictive Scaling
- Optimización de costos: Escalado predictivo para evitar picos repentinos de tráfico

### Gestión del Ciclo de Vida de la Capacidad de Almacenamiento

**Decaimiento del Valor Temporal de los Datos**:

Cada tipo de dato tiene su propio significado temporal y patrón de acceso:

**Ciclo de Vida de los Datos de Trading de Inversiones**:

- Datos en tiempo real (dentro de 1 hora): Frecuencia de acceso extremadamente alta → S3 Standard
- Datos del mismo día (dentro de 24 horas): Frecuencia de acceso alta → S3 Standard
- Datos históricos (dentro de 30 días): Frecuencia de acceso media → S3 IA
- Informes trimestrales (dentro de 1 año): Frecuencia de acceso baja → S3 Glacier
- Archivos regulatorios (dentro de 7 años): Frecuencia de acceso extremadamente baja → S3 Deep Archive

**Ciclo de Vida de los Datos de Monitoreo de Salud**:

- Métricas actuales (dentro de 1 día): Usuario consulta diariamente → S3 Standard
- Tendencias periódicas (dentro de 30 días): Análisis periódico → S3 IA
- Registros anuales (dentro de 1 año): Comparación para chequeos anuales → S3 Glacier
- Historial a largo plazo (de por vida): Valor de investigación médica → S3 Deep Archive

**Análisis de Costo-Beneficio de las Clases de Almacenamiento S3**:

**Costo de Almacenamiento (por GB/mes)**:

- Standard: $0.023
- IA: $0.0125 (45% más barato en almacenamiento, pero mayor costo de recuperación)
- Glacier: $0.004 (83% más barato en almacenamiento, pero la recuperación toma horas)
- Deep Archive: $0.00099 (96% más barato en almacenamiento, pero la recuperación toma 12 horas)

**Umbrales de Costo para Patrones de Acceso**:

- >1 acceso/mes → Standard
- <1 pero >0.1 acceso/mes → IA
- <0.1 pero >0.01 acceso/mes → Glacier
- <0.01 acceso/mes → Deep Archive

## Estrategia de Recuperación: La Filosofía de la Resiliencia

### Pensamiento Ontológico sobre Fallos

En un sistema distribuido, el fallo no es una excepción, sino la norma. La actitud filosófica de diferentes dominios hacia el fallo determina el diseño de las estrategias de recuperación ante desastres.

#### Trading Financiero: Una Filosofía de Tolerancia Cero ante Fallos

**Base filosófica**: El tiempo es dinero, y cualquier retraso es una pérdida

- Requisito de RTO: < 30 segundos (los usuarios no pueden tolerar esperas más largas)
- Requisito de RPO: = 0 (cualquier pérdida de datos de transacción es inaceptable)

**Estrategia de implementación AWS**:

- Multi-AZ RDS: Replicación síncrona, conmutación por error automática
- DynamoDB Global Tables: Consistencia fuerte multi-región
- Lambda@Edge: Computación en el borde para reducir latencia
- Route 53: Conmutación por error DNS, cambio dentro de 30 segundos

**Impacto en costos**: Aumento del 200-300% en costos de infraestructura

#### Finanzas Familiares: Una Filosofía Pragmática ante Fallos

**Base filosófica**: La vida debe continuar, la perfección no es necesaria

- Requisito de RTO: < 4 horas (generalmente aceptable esperar)
- Requisito de RPO: < 1 hora (los registros perdidos se pueden volver a ingresar)

**Estrategia de implementación AWS**:

- RDS Automated Backup: Copias de seguridad diarias, recuperación a un punto en el tiempo
- S3 Cross-Region Replication: Replicación entre regiones
- CloudFormation: Infraestructura como Código para reconstrucción rápida
- SNS: Notificación de fallos, establecimiento de expectativas del usuario

**Impacto en costos**: Aumento del 30-50% en costos de infraestructura

#### Monitoreo de Salud: Una Filosofía de Prevención antes que Cura

**Base filosófica**: Los datos de salud son un asunto de vida, pero la continuidad es más importante que la completitud

- Requisito de RTO: < 2 horas (afecta el monitoreo diario pero no es fatal)
- Requisito de RPO: < 30 minutos (la pérdida de datos a corto plazo es aceptable)

**Estrategia de implementación AWS**:

- IoT Device Shadow: Caché local del estado del dispositivo
- DynamoDB Point-in-time Recovery: Copias de seguridad automáticas
- Timestream: Copia de seguridad y compresión automática de datos de series temporales
- CloudWatch: Monitoreo y alarmas, detección proactiva de problemas

**Impacto en costos**: Aumento del 50-80% en costos de infraestructura

### Patrones de Arquitectura de Recuperación ante Desastres

#### Patrón Pilot Light: Respaldo Mínimo Viable

**Filosofía de diseño**: Una luz de respaldo para funciones centrales
**Escenarios aplicables**: Requisitos de RTO medios, sensible a costos

**Implementación**:

- Los datos clave se replican continuamente a una región de respaldo
- Los recursos de cómputo se mantienen en configuración mínima (o completamente apagados)
- En caso de fallo, se inician y escalan rápidamente

**Implementación AWS**:

- RDS Cross-Region Read Replica
- S3 Cross-Region Replication
- AMI + Launch Template preconfiguración
- Route 53 health checks que disparan conmutación por error

**Características de costo**: Bajo costo durante operación normal, tiempo medio de conmutación por error

#### Patrón Warm Standby: El Arte de Equilibrar un Respaldo Cálido

**Filosofía de diseño**: Siempre listo pero no despilfarrador
**Escenarios aplicables**: Requisitos de RTO más altos, costos controlables

**Implementación**:

- El entorno de respaldo se mantiene en un estado de ejecución mínima
- Los componentes clave están iniciados pero con menor carga
- En caso de fallo, se escalan rápidamente a capacidad completa

**Implementación AWS**:

- Las instancias EC2 se mantienen en ejecución pero con tipos de instancia más pequeños
- Auto Scaling Group min=1, max=tamaño_producción
- ELB health checks para cambio automático de tráfico
- Contenedores Lambda precalentados para evitar arranques en frío

**Características de costo**: Costos de ejecución continuos, pero conmutación por error rápida

#### Patrón Multi-Site Active-Active: Redundancia Completa Sin Interrupciones

**Filosofía de diseño**: La sensación de seguridad de nunca estar caído
**Escenarios aplicables**: Requisitos de cero tiempo de inactividad, no sensible a costos

**Implementación**:

- Múltiples regiones proporcionan servicio completo simultáneamente
- La carga se equilibra entre todos los sitios
- Si algún sitio falla, otros sitios toman el control de forma transparente

**Implementación AWS**:

- Route 53 Latency-based Routing
- CloudFront Global Distribution
- DynamoDB Global Tables
- Multi-Region VPC Peering

**Características de costo**: Costo más alto, pero mejor experiencia

## Concretizando Requisitos No Funcionales desde los Seis Casos

### Caso 1: Especificaciones Técnicas Completas para el Sistema de Trading de Inversiones

**Requisitos de Rendimiento**:

**Tiempo de Respuesta de API**:

- Consulta de datos de mercado: < 50ms (P99)
- Envío de operación: < 100ms (P99)
- Consulta de posiciones: < 200ms (P99)

**Requisitos de Concurrencia**:

- Período normal: 500 usuarios concurrentes
- Pico de apertura: 2000 usuarios concurrentes
- Límite del sistema: 5000 usuarios concurrentes

**Rendimiento**:

- Normal: 1000 RPS
- Pico: 5000 RPS
- Emergencia: 10000 RPS (dentro de 5 minutos)

**Planificación de Capacidad**:

**Pronóstico de Crecimiento de Usuarios**:

- Actual: 10,000 usuarios activos
- 6 meses: 25,000 usuarios activos
- 12 meses: 50,000 usuarios activos

**Crecimiento de Datos**:

- Por usuario por día: 100KB de datos de transacción
- Tasa de crecimiento anual: 500GB/año
- Capacidad total de 5 años: 2.5TB + 50% buffer de crecimiento = 3.75TB

**Recursos de Cómputo**:

- Intensivo en CPU: Cálculo de riesgo, análisis técnico
- Intensivo en memoria: Caché de datos en tiempo real
- Intensivo en red: Sincronización de datos de mercado

**Recuperación ante Desastres**:

**Métricas Clave**:

- RTO: 30 segundos
- RPO: 0 segundos
- Disponibilidad: 99.99% (8.76 horas/año de inactividad)

**Estrategia de Implementación**:

- Primario: us-east-1 (Multi-AZ)
- Secundario: us-west-2 (Hot Standby)
- Datos: DynamoDB Global Tables + RDS Read Replica
- DNS: Route 53 Health Check + conmutación por error automática

### Caso 6: Comparación de Especificaciones Técnicas para la Plataforma de Compartir Viajes

**Requisitos de Rendimiento**:

**Tiempo de Respuesta de API (Comparación con Sistema de Trading)**:

- Carga de contenido: < 3 segundos (vs 100ms para trading, 300 veces más tolerante)
- Interacción social: < 1 segundo (vs 50ms para trading, 20 veces más tolerante)
- Navegación de contenido: < 500ms (vs 200ms para trading, similar pero mayor tolerancia)

**Patrones de Concurrencia**:

- Pico de creación: Períodos de viaje de fin de semana
- Pico de navegación: Períodos de compartir en tardes entre semana
- Estacionalidad: Diferencia de 5-10 veces entre temporadas altas y bajas de viaje

**Diferencias de Filosofía de Planificación de Capacidad**:

**Modelo de Crecimiento**:

- Sistema de trading: Crecimiento estable, necesidades de capacidad predecibles
- Sistema de viajes: Crecimiento explosivo, potencial de difusión viral

**Características de Datos**:

- Sistema de trading: Datos estructurados, cálculos precisos
- Sistema de viajes: Datos multimedia, intensivo en almacenamiento

**Sensibilidad a Costos**:

- Sistema de trading: Rendimiento primero, costo segundo
- Sistema de viajes: Sensible a costos, rendimiento moderado

**Filosofía de Recuperación ante Desastres**:

**Valor de Datos**:

- Sistema de trading: Cada transacción es dinero, la pérdida es inaceptable
- Sistema de viajes: El contenido creativo puede recrearse, la pérdida tiene impacto limitado

**Estrategia de Recuperación**:

- Sistema de trading: Multi-AZ + respaldo en tiempo real + RPO cero
- Sistema de viajes: Respaldo entre regiones + snapshots periódicos + RPO por hora

**Expectativas del Usuario**:

- Sistema de trading: Herramienta profesional, operación perfecta es un requisito básico
- Sistema de viajes: Herramienta de estilo de vida, fallos ocasionales son comprensibles

## Vista Previa del Modelado Abstracto de Mañana

A través de la confirmación de límites técnicos de hoy, hemos derivado especificaciones técnicas específicas desde requisitos empresariales vagos. Ahora sabemos:

- Cuántos RPS: Un indicador cuantitativo derivado del comportamiento del usuario
- Cuánta capacidad: Requisitos de almacenamiento predichos desde patrones de crecimiento
- Qué tan alta disponibilidad: Tolerancia a fallos determinada por el valor del negocio

Pero estas especificaciones técnicas siguen siendo indicadores aislados. Mañana entraremos en la etapa de "modelado abstracto" y aprenderemos cómo transformar estas restricciones técnicas en el modelo conceptual, modelo de comportamiento, modelo de datos y modelo arquitectónico del sistema.

**Las preguntas clave incluyen**:

- ¿Cómo mapear el modelado conceptual de DDD a servicios de AWS?
- ¿Cómo diseñar flujos de API con el modelado de comportamiento de EventStorming?
- ¿Cómo usar el modelado de datos para decidir entre SQL vs NoSQL?
- ¿Cómo usar el modelado arquitectónico para sopesar las compensaciones entre Serverless vs Container?

## Resumen de la Filosofía Técnica de Hoy

- Los requisitos no funcionales son los límites cuantitativos de los requisitos funcionales
- El RPS no es una suposición, sino un resultado inevitable derivado del comportamiento del usuario
- La planificación de capacidad es un mapeo técnico de los patrones de crecimiento del negocio
- La estrategia de recuperación ante desastres refleja la actitud filosófica del dominio hacia el riesgo

Recuerda: Cada indicador técnico tiene su significado empresarial. No estamos optimizando indicadores técnicos, sino optimizando la eficiencia y confiabilidad de la transformación del estado de existencia del usuario.
