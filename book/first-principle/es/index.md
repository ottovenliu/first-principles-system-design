# Diseño de Sistemas desde los Primeros Principios - Guía Completa de Aprendizaje de 30 Días

> **Del Pensamiento Filosófico a la Práctica de Producción, Construyendo Sistemas que Valga la Pena Ejecutar**

¡Bienvenido a este profundo viaje de diseño de sistemas! Esta no es solo una guía técnica, sino también una exploración filosófica del "por qué".

---

## Descripción General del Camino de Aprendizaje

Este viaje de aprendizaje de 30 días está dividido en **6 fases principales**, cada una de las cuales te llevará a comprender el diseño de sistemas desde una dimensión diferente:

| Fase | Tema | Días | Pregunta Central |
|---|---|---|---|
| **Fase 1** | Fundamento Filosófico | Día 1-7 | ¿Por qué diseñarlo así? |
| **Fase 2** | Arquitectura Técnica | Día 8-16 | ¿Cómo lograr los objetivos del sistema? |
| **Fase 3** | Calidad y Pruebas | Día 17-20 | ¿Cómo asegurar la calidad? |
| **Fase 4** | Preparación para Producción | Día 21-25 | ¿Cómo ejecutarlo de forma estable? |
| **Fase 5** | Datos y Optimización | Día 26-28 | ¿Cómo mejorar continuamente? |
| **Fase 6** | Futuro y Resumen | Día 29-30 | ¿Cómo evolucionar continuamente? |

---

## Lista Completa de Artículos

### Fase 1: Fundamento Filosófico y Modelado de Dominio (Día 1-7)

**Pregunta Central: ¿Cuál es el propósito del sistema? ¿Cómo dividir los límites del dominio?**

<details open>
<summary><strong>Expandir para ver todos los artículos ▼</strong></summary>

#### [Día 1 | Introducción y Guía de la Serie: Construyendo un Diseño de Sistema Entregable desde Cero - Usando AWS como Ejemplo](./ironMan.draft.d1-0.md)

**Conceptos Clave**: Pensamiento de primeros principios, DDD vs BDD, la organicidad y propósito de los sistemas, el valor filosófico de los arquitectos

**Aprenderás**:

- ¿Por qué los arquitectos necesitan pensamiento filosófico?
- La esencia del diseño de sistemas: orientado a propósitos y dirigido por el dominio
- Cómo DDD ayuda a los arquitectos a evitar perderse en la tecnología
- La competitividad central de los arquitectos en la era de la IA

---

#### [Día 2-1 | Confirmación de Requisitos × Punto de Partida del Diseño de Sistemas (1): Transformación de la Lógica de Negocio](./ironMan.draft.d2-1.md)

**Conceptos Clave**: Extracción de requisitos, modelos cognitivos de usuarios, la naturaleza de los requisitos funcionales

**Aprenderás**:

- Cómo extraer requisitos reales del discurso del usuario
- Análisis del propósito del sistema bajo diferentes modelos cognitivos
- Métodos para transformar la lógica de negocio en arquitectura técnica

---

#### [Día 2-2 | Confirmación de Requisitos × Punto de Partida del Diseño de Sistemas (2): Límites de Dominio y Confirmación de Requisitos Básicos](./ironMan.draft.d2-2.md)

**Conceptos Clave**: Requisitos no funcionales, límites de rendimiento, planificación de capacidad, recuperación ante desastres

**Aprenderás**:

- Cómo derivar RPS de API del comportamiento del usuario
- Modelos de planificación de capacidad para diferentes negocios
- Pensamiento filosófico sobre estrategias de recuperación ante desastres
- Análisis de puntos críticos para la selección de servicios de AWS

---

#### [Día 3 | Metodología de Extracción de Requisitos - Modelado Abstracto](./ironMan.draft.d3-0.md)

**Conceptos Clave**: Modelado abstracto, modelo conceptual, modelo de comportamiento, modelo de datos

**Aprenderás**:

- Los cuatro niveles del modelado abstracto
- Cómo transformar conceptos de dominio en diseño de sistemas
- Métodos de validación e iteración de modelos

---

#### [Día 4 | Construyendo Lógica de Negocio con DDD: De Casos de Uso a Diseño de Agregados](./ironMan.draft.d4-0.md)

**Conceptos Clave**: Agregado, Entidad, Objeto de Valor, Evento de Dominio

**Aprenderás**:

- Aplicación práctica de patrones tácticos de DDD
- Cómo identificar raíces de agregados y diseñar límites de agregados
- Diseño y temporización del uso de eventos de dominio

---

#### [Día 5 | Contexto de Operación del Sistema del Usuario - Historia de Usuario y Flujo de Escenarios](./ironMan.draft.d5-0.md)

**Conceptos Clave**: Historia de Usuario, recorrido del usuario, diseño de flujo de escenarios

**Aprenderás**:

- Cómo escribir Historias de Usuario efectivas
- Métodos para dibujar mapas de recorrido del usuario
- Mapeo del flujo de escenarios al diseño de API

---

#### [Día 6 | El Arte del Control de Costos en las Compensaciones: Selección de Instancias en Arquitectura de Nube](./ironMan.draft.d6-0.md)

**Conceptos Clave**: Compensaciones arquitectónicas, optimización de costos, selección de Instancias de AWS

**Aprenderás**:

- Pensamiento de compensaciones en decisiones arquitectónicas
- Escenarios aplicables para diferentes tipos de Instancias de AWS
- El arte de equilibrar costo y rendimiento

---

#### [Día 7 | Dibujando tu Primer Plano de Sistema: Selección y Diseño de Arquitectura](./ironMan.draft.d7-0.md)

**Conceptos Clave**: Diagrama de arquitectura de sistema, Modelo C4, documentación de arquitectura

**Aprenderás**:

- Cómo dibujar un diagrama de arquitectura de sistema claro
- Expresión de arquitectura jerárquica con el Modelo C4
- Mejores prácticas para escribir documentación de arquitectura

</details>

---

### Fase 2: Diseño de Arquitectura Técnica (Día 8-16)

**Pregunta Central: ¿Cómo mapear el modelo de dominio a la implementación técnica?**

<details open>
<summary><strong>Expandir para ver todos los artículos ▼</strong></summary>

#### [Día 8 | Sistematización del Diseño de Módulos de Componentes UI: Introduciendo Sistemas de Diseño y Arquitectura Atómica](./ironMan.draft.d8-0.md)

**Conceptos Clave**: Diseño Atómico, sistema de diseño, arquitectura front-end

**Aprenderás**:

- Los cinco niveles del Diseño Atómico
- Cómo construir un sistema de diseño mantenible
- Combinación de arquitectura front-end con DDD

---

#### [Día 9 | Alta Concurrencia y Diseño de Limitación de Tasa: Cómo Evitar Cuellos de Botella de Recursos](./ironMan.draft.d9-0.md)

**Conceptos Clave**: Estrategias de limitación de tasa, algoritmo de cubo de tokens, algoritmo de cubo con fugas, control de concurrencia

**Aprenderás**:

- Algoritmos comunes de limitación de tasa y sus escenarios de aplicación
- Configuración de limitación de tasa en AWS API Gateway
- Cómo diseñar un sistema de alta concurrencia

---

#### [Día 10 | La Filosofía de las Estrategias de Caché: El Arte de Equilibrar Tiempo, Espacio y Consistencia](./ironMan.draft.d10-0.md)

**Conceptos Clave**: Estrategias de caché, Cache-Aside, Write-Through, invalidación de caché

**Aprenderás**:

- Escenarios aplicables para diferentes patrones de caché
- Pensamiento de compensaciones sobre consistencia de caché
- Estrategias para usar Redis y ElastiCache

---

#### [Día 11 | Filosofía del Diseño de Base de Datos: Análisis de Requisitos, Selección de Tecnología y Estrategias de Diseño de Schema](./ironMan.draft.d11-0.md)

**Conceptos Clave**: Relacional vs NoSQL, normalización, diseño de Schema

**Aprenderás**:

- Cómo elegir el tipo correcto de base de datos
- Diseño de normalización para bases de datos relacionales
- Decisiones de selección para DynamoDB y RDS

---

#### [Día 12 | Estrategia de Control de Versiones × Git Flow × Introducción a Lint](./ironMan.draft.d12-0.md)

**Conceptos Clave**: Git Flow, Code Review, Linting

**Aprenderás**:

- Estrategia de gestión de ramas de Git Flow
- Mejores prácticas para PR Review
- Verificación automatizada de calidad de código

---

#### [Día 13 | Diseño de Colaboración entre Equipos: Documentos Técnicos, OpenAPI, Contratos Compartidos](./ironMan.draft.d13-0.md)

**Conceptos Clave**: Documentación de API, Especificación OpenAPI, pruebas de contrato

**Aprenderás**:

- Cómo escribir especificaciones OpenAPI
- Cómo establecer contratos de API entre equipos
- Estrategias de implementación para pruebas de contrato

---

#### [Día 14 | Infraestructura como Código: Codificación de Infraestructura con Terraform y Control de Versiones](./ironMan.draft.d14-0.md)

**Conceptos Clave**: IaC, Terraform, control de versiones de infraestructura

**Aprenderás**:

- El valor central de Infraestructura como Código
- Sintaxis básica y diseño de módulos de Terraform
- Gestión de versiones y colaboración para infraestructura

---

#### [Día 15 | Implementación de CI/CD Totalmente Automatizado - GitHub Actions × CodePipeline](./ironMan.draft.d15-0.md)

**Conceptos Clave**: CI/CD, despliegue automatizado, diseño de Pipeline

**Aprenderás**:

- Diseño completo de Pipeline CI/CD
- Integración de GitHub Actions y AWS CodePipeline
- Estrategias de pruebas y despliegue automatizados

---

#### [Día 16 | Gobernanza y Estrategia de Arquitectura Multi-Entorno Dev / Staging / Prod](./ironMan.draft.d16-0.md)

**Conceptos Clave**: Gestión multi-entorno, gestión de configuración, aislamiento de entornos

**Aprenderás**:

- Estrategias de gobernanza para múltiples entornos
- Diseño de arquitectura multi-cuenta de AWS
- Métodos de gestión de configuración entre entornos

</details>

---

### Fase 3: Aseguramiento de Calidad y Pruebas (Día 17-20)

**Pregunta Central: ¿Cómo verificar que el sistema ha logrado su propósito previsto?**

<details open>
<summary><strong>Expandir para ver todos los artículos ▼</strong></summary>

#### [Día 17 | Optimización de la Experiencia del Desarrollador (DX): Herramientas Internas y Diseño de Solución de Problemas](./ironMan.draft.d17-0.md)

**Conceptos Clave**: Experiencia del Desarrollador, herramientas internas, diseño de depuración

**Aprenderás**:

- Cómo mejorar la experiencia del desarrollador
- Principios de diseño para herramientas internas
- Estrategias efectivas de depuración y registro

---

#### [Día 18 | Formulación de Criterios de Aceptación del Sistema: De la Lógica de Verificación al Manual de Aceptación Funcional](./ironMan.draft.d18-0.md)

**Conceptos Clave**: UAT, criterios de aceptación, aceptación funcional

**Aprenderás**:

- Cómo formular criterios de aceptación claros
- Diseño y ejecución del proceso UAT
- Métodos para escribir documentación de aceptación

---

#### [Día 19 | Pruebas de UX y Validación de Usabilidad: De Observar el Comportamiento del Usuario a Corregir el Diseño](./ironMan.draft.d19-0.md)

**Conceptos Clave**: Pruebas de usabilidad, investigación de usuarios, optimización de UX

**Aprenderás**:

- Métodos para realizar pruebas de usabilidad
- Cómo observar y analizar el comportamiento del usuario
- De los resultados de las pruebas a las mejoras de diseño

---

#### [Día 20 | Pensamiento de Diseño para Sistemas Probables: Una Guía Completa desde Componentes hasta Pruebas de API](./ironMan.draft.d20-0.md)

**Conceptos Clave**: Pirámide de pruebas, pruebas unitarias, pruebas de integración, pruebas E2E

**Aprenderás**:

- Principios de diseño de la pirámide de pruebas
- Estrategias para diferentes niveles de pruebas
- Cómo diseñar una arquitectura de sistema probable

</details>

---

### Fase 4: Preparación para Producción y Confiabilidad (Día 21-25)

**Pregunta Central: ¿Está el sistema saludable y seguro en el entorno de producción?**

<details open>
<summary><strong>Expandir para ver todos los artículos ▼</strong></summary>

#### [Día 21 | Pruebas de Rendimiento y Pruebas de Carga y Estrés](./ironMan.draft.d21-0.md)

**Conceptos Clave**: Pruebas de rendimiento, pruebas de estrés, análisis de cuellos de botella

**Aprenderás**:

- Tipos y métodos de pruebas de rendimiento
- Cómo realizar pruebas de estrés
- Identificación y optimización de cuellos de botella de rendimiento

---

#### [Día 22 | La Piedra Angular de la Seguridad Moderna "Arquitectura de Confianza Cero"](./ironMan.draft.d22-0.md)

**Conceptos Clave**: Confianza Cero, IAM, VPC, principio de mínimo privilegio

**Aprenderás**:

- Principios fundamentales de la arquitectura de Confianza Cero
- Diseño de mínimo privilegio con AWS IAM
- Aislamiento de red VPC y configuración de grupos de seguridad

---

#### [Día 23 | Los Tres Pilares de la Observabilidad: Del Monitoreo a Responder Preguntas Desconocidas](./ironMan.draft.d23-0.md)

**Conceptos Clave**: Logs, Métricas, Trazas, observabilidad

**Aprenderás**:

- La diferencia entre observabilidad y monitoreo
- Práctica integrada de logs, métricas y trazas
- Estrategias para usar CloudWatch y X-Ray

---

#### [Día 24 | Definición y Medición de Confiabilidad: Método SRE y Práctica de Presupuesto de Errores](./ironMan.draft.d24-0.md)

**Conceptos Clave**: SLI, SLO, SLA, presupuesto de errores, SRE

**Aprenderás**:

- Conceptos fundamentales y prácticas de SRE
- Cómo definir SLI, SLO, SLA
- Uso y gestión de presupuestos de errores

---

#### [Día 25 | Verificación Proactiva de Resiliencia: Ingeniería del Caos](./ironMan.draft.d25-0.md)

**Conceptos Clave**: Ingeniería del Caos, inyección de fallos, pruebas de resiliencia

**Aprenderás**:

- Los principios y el valor de la Ingeniería del Caos
- AWS Fault Injection Simulator en acción
- Cómo diseñar experimentos de pruebas de resiliencia

</details>

---

### Fase 5: Impulsado por Datos y Optimización Continua (Día 26-28)

**Pregunta Central: ¿Cómo hacer que el sistema evolucione continuamente?**

<details open>
<summary><strong>Expandir para ver todos los artículos ▼</strong></summary>

#### [Día 26 | Decisiones de Producto Impulsadas por Datos: De las Pruebas A/B a las Métricas Estrella del Norte](./ironMan.draft.d26-0.md)

**Conceptos Clave**: Pruebas A/B, análisis de datos, métrica Estrella del Norte

**Aprenderás**:

- Cómo diseñar pruebas A/B efectivas
- Un marco para la toma de decisiones impulsadas por datos
- Selección y seguimiento de métricas Estrella del Norte

---

#### [Día 27 | Gestión de Costos Intangibles: Identificación de Deuda Técnica y Estrategias de Pago](./ironMan.draft.d27-0.md)

**Conceptos Clave**: Deuda técnica, cuadrante de deuda técnica, regla del Boy Scout

**Aprenderás**:

- Tipos y métodos de identificación de deuda técnica
- Aplicación del cuadrante de deuda técnica
- Estrategias de pago para la deuda técnica

---

#### [Día 28 | Gobernanza de Datos y Protección de Privacidad: Diseño de Cumplimiento GDPR](./ironMan.draft.d28-0.md)

**Conceptos Clave**: Gobernanza de datos, GDPR, protección de privacidad, ciclo de vida de datos

**Aprenderás**:

- Requisitos fundamentales para el cumplimiento de GDPR
- Gestión del ciclo de vida de los datos
- Patrones de diseño para la protección de privacidad

</details>

---

### Fase 6: Evolución Arquitectónica y Perspectiva Futura (Día 29-30)

**Pregunta Central: ¿Cómo evoluciona y crece el sistema de manera elegante?**

<details open>
<summary><strong>Expandir para ver todos los artículos ▼</strong></summary>

#### [Día 29-1 | El Concierto de la Evolución Arquitectónica: Combinando el Patrón Strangler Fig y BFF para Lograr una Transformación Elegante de un Sistema Monolítico](./ironMan.draft.d29-1.md)

**Conceptos Clave**: Patrón Strangler Fig, BFF, migración a microservicios

**Aprenderás**:

- Escenarios de aplicación del Patrón Strangler Fig
- Diseño de Backend for Frontend (BFF)
- Migración gradual de monolito a microservicios

---

#### [Día 29-2 | Sócrates se Encuentra con la Arquitectura de Sistemas: Una Técnica de Aumento con IA para Diseñadores de Sistemas](./ironMan.draft.d29-2.md)

**Conceptos Clave**: Diseño asistido por IA, pensamiento filosófico, marco de diseño de sistemas

**Aprenderás**:

- Cómo mejorar las capacidades de diseño de sistemas con IA
- Aplicación del método socrático
- La mentalidad de un arquitecto en la era de la IA

---

#### [Día 30 | Final de la Serie: Un Camino de Aprendizaje de Diseño de Sistemas y Reflexiones para Futuros Ingenieros](./ironMan.draft.d30-0.md)

**Conceptos Clave**: Camino de aprendizaje, crecimiento continuo, filosofía del ingeniero

**Aprenderás**:

- Camino de aprendizaje recomendado para el diseño de sistemas
- Cómo mejorar continuamente las habilidades arquitectónicas
- Reflexiones filosóficas para ingenieros

</details>

---

## Cómo Usar Esta Guía

### Camino de Aprendizaje Completo (Recomendado para Principiantes)

1. Comienza desde el Día 1 y lee en orden hasta el Día 30
2. Piensa cada día en cómo aplicarlo a proyectos reales
3. Resume y reflexiona después de completar cada fase

### Aprendizaje Temático en Profundidad (Adecuado para Personas con Experiencia)

**Tema de Diseño de Arquitectura**:

- Día 1 (Fundamento Filosófico)
- Día 6 (El Arte de las Compensaciones)
- Día 7 (Plano de Arquitectura)
- Día 14 (IaC)
- Día 16 (Gobernanza Multi-Entorno)
- Día 29 (Evolución Arquitectónica)

**Tema de DevOps y CI/CD**:

- Día 12 (Control de Versiones)
- Día 14 (Infraestructura como Código)
- Día 15 (CI/CD)
- Día 16 (Gestión Multi-Entorno)

**Tema de Seguridad y Confiabilidad**:

- Día 22 (Arquitectura de Confianza Cero)
- Día 23 (Observabilidad)
- Día 24 (SRE)
- Día 25 (Ingeniería del Caos)

**Tema de Pruebas y Calidad**:

- Día 17 (Experiencia del Desarrollador)
- Día 18 (Criterios de Aceptación)
- Día 19 (Pruebas de UX)
- Día 20 (Estrategia de Pruebas)
- Día 21 (Pruebas de Rendimiento)

**Tema de Datos y Optimización**:

- Día 26 (Decisiones Impulsadas por Datos)
- Día 27 (Gestión de Deuda Técnica)
- Día 28 (Gobernanza de Datos)

---

## Sugerencias de Aprendizaje

### Prácticas Recomendadas

- Dedica 30-60 minutos a la lectura profunda cada día
- Combina con proyectos reales para pensar en escenarios de aplicación
- Toma notas y resume tu propia comprensión
- Discute y comparte experiencias de aprendizaje con tu equipo

### Prácticas a Evitar

- Leer superficialmente sin pensamiento profundo
- Solo ver conclusiones sin entender los principios
- No practicar, solo ver la teoría
- Aprender en aislamiento sin discutir con otros

---

## Otras Versiones en Idiomas

- [繁體中文 (chino tradicional)](../zh-TW/index.md) - Completado ✓
- [English (inglés)](../en-US/index.md) - Completado ✓
- [日本語 (japonés)](../ja/index.md) - Completado ✓
- [Español (español)](../es/index.md) - Completado ✓

---

## Recursos Relacionados

- [Índice de Dominio](../index.md) - Volver a la página principal del dominio
- [Todos los Recursos del Libro](../../index.md) - Ver todos los temas de dominio
- [README del Proyecto](../../../README.md) - Descripción general del proyecto

---

## 💬 Retroalimentación y Contribución

¿Encontraste un error o tienes sugerencias de mejora? Te damos la bienvenida a:

- 📝 Enviar un Issue
- 🔄 Enviar un Pull Request
- 💭 Compartir tu experiencia de aprendizaje

---

**¡Comienza tu viaje de diseño de sistemas! 🚀**

> **"El sistema no examinado no vale la pena ejecutar."**

**© 2025 Diseño de Sistemas desde los Primeros Principios**