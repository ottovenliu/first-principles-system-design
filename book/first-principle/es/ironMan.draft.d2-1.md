# Día 2-1 | Confirmación de Requisitos × Punto de Partida del Diseño de Sistemas (1): Transformación de la Lógica de Negocio

Antes de que cualquier sistema comience a funcionar, tenemos más o menos un propósito comercial principal que queremos alcanzar.

Incluso para el sitio web oficial de una sola página más simple o una página publicitaria, su propósito de existencia es: **permitir que las personas obtengan información rápidamente**.

Por lo tanto, antes de llevar a cabo nuestro diseño de sistema, debemos participar activamente en cada confirmación de requisitos y sincronización de contenido de reuniones, para que podamos tener un concepto aproximado pero claro del sistema que estamos a punto de diseñar y desarrollar desde el principio.

## La Ontología de la Lógica de Negocio: Del Fenómeno a la Esencia

### Tres Niveles de Requisitos

**Nivel de Fenómeno**: Lo que dice el usuario

- "Quiero un software que pueda rastrear logística"
- "Necesito seguimiento en tiempo real, sin demoras"
- "Quiero monitorear la situación en casa en cualquier momento"

**Nivel de Esencia**: La motivación real detrás de la demanda

- El deseo de control del tiempo
- Ansiedad y control sobre los riesgos
- La asunción y compartición de responsabilidad

**Nivel de Existencia**: El estado de vida cambiado por el sistema

- De la espera pasiva al control activo
- De la incertidumbre a la previsibilidad
- De la responsabilidad individual a la colaboración colectiva

### El Código como Encarnación de la Metafísica

El código es una herramienta concreta que convierte la lógica de negocio abstracta en reglas concretas a través del código. Así como no podemos describir directamente la "justicia" abstracta, podemos convertirla en reglas prácticas de conducta a través de disposiciones legales.

Este proceso es "abstracción y reducción filosófica":

- **Abstracción**: Extraer la intencionalidad central de las palabras del usuario
- **Modelado**: Describir la esencia del negocio con conceptos de dominio
- **Reducción**: Hacer que el sistema opere realmente a través de servicios arquitectónicos

Esta es también la habilidad más importante y muy valiosa para que los ingenieros modernos aprendan - **un sentido de excelencia**, cómo desentrañar lentamente conceptos abstractos en un magnífico tapiz.

Por supuesto, nuestras discusiones siempre son hermosas.
Entre ingenieros, los algoritmos y tipos de datos son como nuestra Torre de Babel, estableciendo un entorno básico para la comunicación fluida.

Sin embargo, los no programadores no entienden este lenguaje. Incluso los documentos de aclaración de especificaciones de requisitos generados por IA tendrán consideraciones técnicas que no se tienen en cuenta en algunas de las compensaciones detrás de ellas.

Este tipo de espacio ambiguo sin duda nos está volviendo locos. Una solicitud vaga no es menos que un sacerdote en un templo diciendo repentinamente "los dioses necesitan que ofrezcas coraje".
Hay muchos tipos de coraje: un gran festín de gladiadores, una victoria de conquista triunfal, un laurel olímpico, e incluso admitir los pensamientos en el corazón de uno. Entonces, ¿qué debemos hacer?, ¿cómo sabemos lo que nuestros dioses **realmente** necesitan?

Por lo tanto, necesitamos el contexto de pensamiento filosófico del Diseño Dirigido por Dominio.

## DDD: El Marco Cognitivo del Diseño Dirigido por Dominio

### ¿Por Qué Necesitamos el Pensamiento de Dominio?

Eric Evans enfatizó en "Domain-Driven Design: Tackling Complexity in the Heart of Software": Los ingenieros no solo deben perseguir la elegancia del código, lo más importante es una comprensión profunda de la lógica de negocio.

Pero, ¿qué es la "lógica de negocio"? Desde un punto de vista fenomenológico, cada negocio tiene su **intencionalidad** - apunta a un propósito específico y forma de realizar valor.

**La intencionalidad de un sistema de trading de inversiones**: El control último del tiempo
**La intencionalidad de un sistema financiero familiar**: La gestión coordinada de recursos colectivos
**La intencionalidad de un sistema de monitoreo de salud**: La cognición continua del estado físico

### La Base Filosófica del Modelado de Dominio

Cada dominio tiene su **estructura ontológica** única:

```py
Ontología de Dominio
├── Entidades - Conceptos centrales con identidad
├── Objetos de Valor - Atributos descriptivos
├── Agregados - Límites de consistencia
└── Eventos de Dominio - El significado de los cambios de estado
```

Esto no es solo un patrón técnico, sino también una herramienta cognitiva para comprender la esencia del negocio.

## Event Storming: Haciendo Visibles los Fenómenos de Negocio

### Viendo la Temporalidad del Negocio desde los Eventos

La idea central del Event Storming: **El negocio es esencialmente una secuencia de eventos en el tiempo**.

El análisis tradicional de requisitos pregunta "¿qué debería hacer el sistema?", mientras que Alberto Brandolini, el inventor del Event Storming, señala otra pregunta importante: "¿qué sucedió y por qué es importante?".

**La interacción entre objetos se basa en la ocurrencia de eventos, y la ocurrencia de eventos causa diferencias en los datos de los objetos.**

Este es un concepto muy importante, porque el principio de la existencia de datos se debe al cambio de información (info), y la razón por la cual el cambio de información (info) se registra como datos es para **realizar una lógica de negocio abstracta**.

Precisamente debido a esto, Alberto Brandolini enfatiza la correlación de cada evento y discute los requisitos de aplicación de eventos y eventos.

### Marco Práctico: Event Storming Fenomenológico (Escenario de Trading de Acciones)

**Paso 1: Identificar Eventos Significativos**
No solo registrar "orden creada", sino también entender "qué significa este evento para el usuario":

- Para usuarios orientados al control: El plan comienza a ejecutarse
- Para usuarios orientados a lo social: La expectativa está a punto de cumplirse
- Para usuarios orientados a la seguridad: El compromiso es oficialmente efectivo

**Paso 2: Explorar la Intencionalidad de los Eventos**
¿Cuál es la motivación detrás de cada evento?

- "Alerta de precio activada" → Ansiedad por la pérdida
- "Alarma de sobrepaso de presupuesto" → Miedo a perder el control
- "Notificación de dispositivo fuera de línea" → Preocupación por la responsabilidad

**Paso 3: Mapear a la Implementación Técnica**
Determinar la estrategia técnica basada en la intencionalidad del evento:

- Los eventos impulsados por ansiedad requieren baja latencia (WebSocket + Lambda@Edge)
- Los eventos impulsados por miedo requieren confiabilidad (SQS + DLQ + Retry)
- Los eventos impulsados por preocupación requieren observabilidad (CloudWatch + SNS)

## Ontología de Requisitos: MBTI como Herramienta de Análisis

### ¿Por Qué Elegir MBTI?

MBTI no es solo una prueba psicológica, sino un **sistema de clasificación de modelos cognitivos**. Nos ayuda a entender cómo diferentes usuarios perciben el mundo, toman decisiones y organizan sus vidas.
Creo que MBTI es una herramienta útil para la segmentación de escenarios, principalmente porque se basa en:

- Extroversión o Introversión (E/I)
- Sensación o Intuición (S/N)
- Pensamiento o Sentimiento (T/F)
- Juicio o Percepción (J/P)

Estos cuatro cuadrantes son también las direcciones principales en las que tomamos decisiones.

Creo que debe haber otras herramientas de análisis de personalidad y perfiles de juego de roles de marketing más profesionales, pero por ahora, tratemos de simular las situaciones que podemos encontrar en la vida de la manera más simple y completa posible.

**Correspondencia Técnica de las Cuatro Dimensiones Cognitivas**:

| Dimensión Cognitiva | Impacto en el Diseño del Sistema |
| --- | --- |
| E/I (Dirección de Energía) | Colaboración vs. Personalización |
| S/N (Procesamiento de Información) | Concreto vs. Abstracto |
| T/F (Toma de Decisiones) | Eficiencia vs. Experiencia |
| J/P (Estilo de Vida) | Estructura vs. Flexibilidad |

### Análisis Profundo de Seis Casos Típicos

---

#### Caso 1: Sistema de Trading de Inversiones [I-N-T-J] - Demanda Orientada al Control

**Nivel de Fenómeno**: "Tengo miedo de perder dinero si la velocidad no es lo suficientemente rápida"
**Nivel de Esencia**: La demanda última de control del tiempo
**Nivel de Existencia**: De un receptor pasivo del mercado a un gestor activo del tiempo

**Análisis de Event Storming**:
Actualización de datos del mercado → Activación de análisis de precios → Generación de decisión de trading →
Envío de orden → Confirmación de operación → Actualización de posición → Recálculo de riesgo

El retraso de cada evento puede desencadenar ansiedad, por lo que requiere:

- **Latencia extremadamente baja**: WebSocket + Lambda@Edge + ElastiCache
- **Consistencia de datos**: DynamoDB Transactions
- **Recuperación rápida de fallos**: Multi-AZ + Auto Failover

---

#### Caso 2: Sistema Financiero Familiar [E-N-T-J] - Demanda Orientada a la Coordinación

**Nivel de Fenómeno**: "Permitir que la familia registre en conjunto para evitar desequilibrio presupuestario"
**Nivel de Esencia**: Coordinación y control de la responsabilidad colectiva
**Nivel de Existencia**: De la gestión financiera personal a un sistema de gobernanza familiar

**Evento Central**: "Límite presupuestario alcanzado"

- Este evento significa "el control es desafiado" para los usuarios ENTJ
- El sistema debe proporcionar inmediatamente un camino para "recuperar el control"
- Implementación técnica: Notificación en tiempo real SNS + Ajuste automático de permisos Lambda

---

#### Caso 3: Sistema de Monitoreo de Salud [I-S-T-J] - Demanda de Certeza

**Nivel de Fenómeno**: "Quiero saber a tiempo si los datos del cuerpo se sincronizan correctamente"
**Nivel de Esencia**: La necesidad continua de confirmación del estado físico
**Nivel de Existencia**: De la preocupación pasiva por la salud al monitoreo activo

**Perspectiva Clave**: Lo que más les importa a los usuarios no son los datos en sí, sino "la confiabilidad de los datos"

- **La integridad de los datos** es más importante que el tiempo real
- **La visibilidad del estado de sincronización** es más importante que la riqueza de funciones
- Implementación técnica: IoT Core + Timestream + CloudWatch Dashboards

---

#### Caso 4: Sistema de Monitoreo de Hogar Inteligente [I-S-T-P] - Demanda de Adaptabilidad

**Nivel de Fenómeno**: "Monitorear dispositivos inteligentes en cualquier momento para garantizar la seguridad de ancianos y niños, y puedo aceptar retrasos ocasionales"
**Nivel de Esencia**: Equilibrio flexible al asumir responsabilidad
**Nivel de Existencia**: Transformación de rol de un preocupado a un guardián

**Análisis de Contradicción Central**: "Monitorear en cualquier momento" vs. "Aceptar retrasos"

- El monitoreo de estado diario puede retrasarse (**optimización de costos**)
- Los eventos de emergencia deben ser en tiempo real (**línea base de responsabilidad**)
- Implementación técnica: IoT Device Management + Kinesis + Lambda + SNS

---

#### Caso 5: Sistema de Rastreo de Paquetes [I-S-T-P] - Demanda de Transparencia

**Nivel de Fenómeno**: "Rastrear el progreso del paquete en tiempo real, errores ocasionales son aceptables"
**Nivel de Esencia**: La necesidad gentil de control sobre la incertidumbre
**Nivel de Existencia**: De la espera pasiva al conocimiento activo

**Comparación Filosófica con el Caso 4**:
Ambos son ISTP, pero el grado de responsabilidad es diferente:

- Caso 4: Asumiendo la gran responsabilidad por la seguridad de otros
- Caso 5: Gestionando la ligera responsabilidad de expectativas personales

Esta diferencia afecta directamente la estrategia de tolerancia a fallos del diseño del sistema:

- El monitoreo del hogar requiere un mecanismo de respaldo
- El rastreo de paquetes puede degradarse elegantemente

**La "Filosofía de la Espera" en Event Storming**:
Envío de paquete → Sincronización de estado → Actualización de ubicación →
Detección de anomalías → Ajuste de expectativas → Confirmación de entrega

El evento central es "ajuste de expectativas" en lugar de "actualización de estado"

- Los usuarios ISTP pueden enfrentar retrasos racionalmente, siempre que la información sea transparente
- "Error aceptable" significa que la notificación honesta del sistema es más importante que el rendimiento perfecto

**Diseño de Honestidad de la Implementación Técnica**:

- API Gateway: Integrar fuentes de datos de múltiples partes
- DynamoDB: Registros rastreables de cambios de estado
- EventBridge: Manejo elegante de actualizaciones asincrónicas
- CloudWatch: Transparencia de la salud del sistema

---

#### Caso 6: Plataforma de Compartir Viajes [E-N-F-P] - Demanda Creativa

**Nivel de Fenómeno**: "Registrar inspiración de viaje en cualquier momento, compartir mapas de ruta y transporte"
**Nivel de Esencia**: El deseo de expresión creativa y conexión social
**Nivel de Existencia**: La sublimación de la experiencia personal al valor compartido

**Desafíos Técnicos del Modelo Cognitivo ENFP**:

- **E (Extroversión)**: Requiere mecanismos de compartir e interacción en tiempo real
- **N (Intuición)**: La inspiración es instantánea y requiere registro sin barreras
- **F (Sentimiento)**: Valora la resonancia emocional sobre la perfección funcional
- **P (Percepción)**: Las formas de contenido son variables, y la estructura necesita ser adaptable

**Event Storming del Proceso Creativo**:
Activación de inspiración → Captura de contenido → Etiquetado de contexto →
Procesamiento creativo → Compartir y publicar → Interacción comunitaria → Regeneración de inspiración

**Perspectiva Clave**: La filosofía de diseño de un sistema creativo

- **Capturar es mejor que organizar**: Registrar primero, organizar después
- **Expresión es mejor que perfección**: Poder publicar es más importante que tener funciones completas
- **Conexión es mejor que contenido**: El valor social es más alto que el valor de información

**Arquitectura de Soporte Creativo de AWS**:

- S3: Almacenamiento ilimitado de contenido multimedia
- AppSync: Colaboración en tiempo real para interacción social
- Cognito: Base comunitaria para autenticación de identidad
- Location Service: Contextualización de etiquetado geográfico
- Sincronización offline: La inspiración no se pierde debido a la red

---

### Matriz de Modelo Cognitivo de los Seis Casos

| Caso | Modelo Cognitivo | Demanda Central | Enfoque Técnico | Estrategia AWS |
| --- | --- | --- | --- | --- |
| Trading de Inversiones | I-N-T-J | Control del Tiempo | Baja Latencia | WebSocket + Lambda@Edge |
| Finanzas Familiares | E-N-T-J | Control Colaborativo | Gestión de Permisos | Cognito + DynamoDB |
| Monitoreo de Salud | I-S-T-J | Certeza de Datos | Confiabilidad | IoT Core + Timestream |
| Monitoreo del Hogar | I-S-T-P | Responsabilidad Flexible | Respuesta Escalonada | Kinesis + Lambda |
| Rastreo de Paquetes | I-S-T-P | Transparencia | Integración Multi-fuente | API Gateway + EventBridge |
| Compartir Viajes | E-N-F-P | Expresión Creativa | Interacción Social | AppSync + S3 |

## Transformación de la Lógica de Negocio - Fenómeno x Esencia x Existencia

A través del análisis filosófico de hoy, hemos transformado con éxito las expresiones vagas de los usuarios en requisitos funcionales claros. Pero esto es solo el primer paso.

Cuando comenzamos a diseñar una arquitectura AWS específica, encontraremos que hay problemas más profundos:

- ¿Qué significa "tiempo real" en términos de milisegundos de retraso?
- ¿Cuánta concurrencia necesita soportar "monitorear en cualquier momento"?
- ¿Qué significa "no puede fallar" en términos de nueves de disponibilidad?
- ¿Cuál es el techo presupuestario para "costo controlable"?

Estos son **requisitos no funcionales** - determinan las especificaciones técnicas del sistema y la configuración específica de los servicios AWS.

Mañana discutiremos "Límites de Dominio y Confirmación de Requisitos Básicos", y tendremos una discusión profunda sobre la base filosófica de especificaciones técnicas como diseño de API, planificación de capacidad y estrategias de recuperación ante desastres.

## Marco Cognitivo de Hoy

- **La Ontología de la Lógica de Negocio**: Un análisis de tres capas desde el fenómeno hasta la esencia y la existencia
- **El Valor Cognitivo del DDD**: Los modelos de dominio como herramienta para la comprensión del negocio
- **La Temporalidad del Event Storming**: Viendo la intencionalidad del negocio en los eventos
- **El Mapeo Técnico del MBTI**: El modelo cognitivo de los usuarios centrales determina la elección de la arquitectura

Recuerda: La habilidad central de un arquitecto no es memorizar la lista de servicios AWS, sino entender "por qué los usuarios necesitan este sistema". La tecnología es solo un medio para lograr objetivos, y las necesidades humanas detrás de los objetivos son la base filosófica del diseño.

---

> "Cada sistema tiene su intencionalidad, que apunta a una cierta transformación del estado de existencia del usuario. No estamos diseñando funciones, sino diseñando posibilidades."
