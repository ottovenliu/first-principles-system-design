# Día 4 | Construyendo Lógica de Negocio con DDD: Del Caso de Uso al Diseño de Agregados

Ayer, completamos los cuatro ámbitos del modelado abstracto, comprendiendo la esencia del sistema desde las cuatro dimensiones de concepto, comportamiento, datos y arquitectura. Pero cuando queremos transformar estas comprensiones abstractas en lógica de negocio ejecutable, enfrentamos un desafío clave:

**¿Cómo organizar los conceptos de dominio en unidades de negocio cohesivas y significativas?**

Este es el problema central de la práctica de DDD: **la transformación sistemática del análisis de casos de uso al diseño de agregados**. Hoy, estableceremos la estructura estática del modelo de dominio para sentar las bases para una discusión en profundidad de los escenarios de operación del usuario mañana.

## Análisis de Casos de Uso: De la Intención a la Identificación de Límites

### Reexaminando los Seis Casos del Día 2

En el Día 2, analizamos seis casos típicos de usuario, cada uno con su **patrón de caso de uso central**. Deconstruyámoslos nuevamente desde una perspectiva DDD:

**Sistema de Trading de Inversiones [I-N-T-J] - Patrón de Caso de Uso Orientado al Control**

```
Caso de Uso Central: Ejecutar decisiones de trading en tiempo real
Actor Principal: Trader Profesional
Objetivo en Contexto: Completar trades de alta calidad en el menor tiempo posible
Alcance: Sistema de gestión de portafolio de inversión personal
Nivel: Nivel de objetivo del usuario

Escenario Principal de Éxito:
1. El trader monitorea datos del mercado
2. Identifica oportunidades de trading
3. Calcula el impacto del riesgo
4. Envía una orden de trading
5. Confirma la ejecución del trade
6. Actualiza el estado del portafolio

Extensiones:
2a. Los datos del mercado están retrasados → El sistema proporciona una advertencia de retraso
3a. El riesgo excede el límite → El sistema requiere confirmación explícita
4a. Fondos insuficientes → El sistema sugiere ajustes o financiamiento
5a. El trade falla → El sistema entra en estado de reintento o procesamiento manual
```

De este análisis de caso de uso, podemos identificar varios **conceptos de dominio centrales**:

- **Trader** - Un sujeto con capacidad de toma de decisiones
- **Portfolio** - Un contenedor de estado para asignación de activos
- **TradeOrder** - Una expresión estructurada de intención de trading
- **MarketData** - Una fuente de información de precios externos
- **RiskAssessment** - Lógica de cálculo para soporte de decisiones

**Sistema Financiero Familiar [E-N-T-J] - Patrón de Caso de Uso Orientado a la Coordinación**

```
Caso de Uso Central: Gestionar gastos colaborativos familiares
Actor Principal: Jefe de finanzas familiar
Objetivo en Contexto: Coordinar el comportamiento de gasto de los miembros de la familia para mantener el equilibrio presupuestario
Alcance: Sistema de gestión financiera familiar
Nivel: Nivel de resumen de negocio

Escenario Principal de Éxito:
1. Establecer presupuesto familiar y permisos
2. Los miembros de la familia registran gastos
3. El sistema verifica el estado del presupuesto
4. Dispara advertencias o ajustes presupuestarios
5. Genera un informe financiero familiar

Complejidad de Interacción de Roles:
- Tomador de decisiones principal: Establece reglas, monitorea la situación general
- Miembros generales: Realizan gastos, siguen las reglas
- Coordinación del sistema: Asegura la ejecución de reglas, proporciona transparencia
```

Identificación de Conceptos de Dominio:

- **Family** - El límite de colaboración y el creador de reglas
- **FamilyMember** - Un participante con permisos específicos
- **Budget** - Una regla de restricción para asignación de recursos
- **Expense** - Un registro factual de consumo de recursos
- **PermissionManagement** - Un mecanismo de control para comportamiento colaborativo

### Características de Dominio de Patrones de Casos de Uso

A través del análisis comparativo de los seis casos, encontramos que diferentes patrones de casos de uso corresponden a diferentes **características de complejidad de dominio**:

| Patrón de Caso de Uso | Complejidad Principal | Desafío Central | Enfoque de Diseño |
| --- | --- | --- | --- |
| Orientado al Control (Trading de Inversiones) | Sensibilidad temporal | Decisiones en milisegundos | Consistencia de estado |
| Orientado a la Coordinación (Finanzas Familiares) | Colaboración multi-rol | Permisos y transparencia | Límites de roles |
| Orientado a la Certeza (Monitoreo de Salud) | Confiabilidad de datos | Monitoreo continuo | Integridad de datos |
| Adaptativo (Hogar Inteligente) | Respuesta flexible | Control progresivo | Degradación elegante |
| Transparente (Seguimiento de Paquetes) | Sincronización de información | Integración de datos multi-fuente | Sincronización de estado |
| Creativo (Compartir Viajes) | Flexibilidad expresiva | Herramientas de soporte creativo | Fluidez de contenido |

Cada patrón afectará nuestra estrategia de diseño de agregados.

## Identificación de Agregados: El Arte de Dividir Límites de Negocio

### Tres Principios de Identificación de Agregados

**Principio 1: Límite de Invariante de Negocio**
El límite de un agregado debe corresponder a un conjunto de conceptos que deben permanecer consistentes en el negocio.

**Análisis de Invariantes del Sistema de Trading de Inversiones**:

```
Invariantes del Agregado Portfolio:
- Valor total de activos = Σ(valor de tenencia) + saldo de efectivo
- Ningún trade puede resultar en un saldo de efectivo negativo
- La exposición al riesgo no puede exceder el límite preestablecido

Invariantes del Agregado Order:
- Los parámetros centrales de una orden no pueden modificarse una vez enviados
- Los registros de ejecución deben corresponder a la instrucción original
- Las transiciones de estado deben cumplir con el proceso de negocio
```

**Principio 2: Correspondencia de Límite de Transacción**
Las modificaciones dentro de un agregado deben poder completarse en una sola transacción.

**Análisis de Transacciones del Sistema Financiero Familiar**:

```
Agregado Family:
✓ Puede completarse en una sola transacción:
  - Establecer monto del presupuesto
  - Ajustar permisos de miembros
  - Actualizar información básica de la familia

✗ No puede completarse en una sola transacción:
  - Contar los gastos mensuales de todos los miembros (a través de múltiples agregados)
  - Generar un informe anual (volumen de datos demasiado grande)
```

**Principio 3: Correspondencia de Lenguaje de Negocio**
El nombre y la estructura de un agregado deben corresponder directamente al lenguaje de los expertos en negocio.

### Comparación de Diseños de Agregados para los Seis Casos

**Caso 1: Sistema de Trading de Inversiones**

```
Diseño de Agregados:
┌─────────────────┐    ┌─────────────────┐
│   Portfolio     │    │      Order      │
│  (Portafolio de Inversión)     │    │    (Orden de Trading)   │
├─────────────────┤    ├─────────────────┤
│ - portfolioId   │    │ - orderId       │
│ - traderId      │    │ - portfolioId   │
│ - holdings[]    │    │ - tradeOrder    │
│ - cashBalance   │    │ - status        │
│ - riskLimit     │    │ - execution[]   │
└─────────────────┘    └─────────────────┘
        │                       │
        └───────────────────────┘
             Comunicación de Eventos de Dominio

Lógica de Diseño:
- Portfolio gestiona el estado de activos y control de riesgo
- Order gestiona el proceso de trading y seguimiento de ejecución
- Los dos mantienen consistencia eventual a través de eventos
```

**Caso 2: Sistema Financiero Familiar**

```
Diseño de Agregados:
┌─────────────────┐    ┌─────────────────┐
│     Family      │    │     Budget      │
│    (Familia)       │    │    (Presupuesto)       │
├─────────────────┤    ├─────────────────┤
│ - familyId      │    │ - budgetId      │
│ - members[]     │    │ - familyId      │
│ - permissions   │    │ - categories[]  │
│ - settings      │    │ - limits        │
└─────────────────┘    │ - period        │
                       └─────────────────┘
              │                 │
              └─────────────────┘
                Coordinación Impulsada por Eventos

┌─────────────────┐
│    Expense      │
│   (Registro de Gasto)    │
├─────────────────┤
│ - expenseId     │
│ - familyId      │
│ - memberId      │
│ - amount        │
│ - category      │
│ - timestamp     │
└─────────────────┘

Lógica de Diseño:
- Family gestiona miembros y estructura de permisos
- Budget gestiona reglas presupuestarias y límites
- Expense registra hechos de gasto reales
- Los tres colaboran para lograr control presupuestario
```

**Caso 3: Sistema de Monitoreo de Salud**

```
Diseño de Agregados:
┌─────────────────┐    ┌─────────────────┐
│   HealthProfile │    │  DeviceReading  │
│   (Perfil de Salud)    │    │   (Datos de Dispositivo)    │
├─────────────────┤    ├─────────────────┤
│ - userId        │    │ - readingId     │
│ - metrics[]     │    │ - userId        │
│ - thresholds    │    │ - deviceType    │
│ - history       │    │ - value         │
└─────────────────┘    │ - timestamp     │
                       │ - status        │
                       └─────────────────┘

Lógica de Diseño:
- HealthProfile gestiona el estado de salud a largo plazo
- DeviceReading gestiona la recopilación de datos en tiempo real
- El diseño separado soporta diferentes ciclos de vida de datos
```

### Patrones de Interacción entre Agregados

Diferentes dominios producirán diferentes patrones de interacción de agregados:

**Patrón 1: Coordinación Impulsada por Eventos (Trading de Inversiones)**

```
Agregado Portfolio → TradeValidatedEvent → Agregado Order
Agregado Order → OrderExecutedEvent → Agregado Portfolio
Agregado Portfolio → PortfolioUpdatedEvent → Agregado RiskManagement
```

**Patrón 2: Colaboración Impulsada por Permisos (Finanzas Familiares)**

```
Agregado Family → Establece reglas de permisos → Agregado Expense verifica permisos
Agregado Budget → Establece reglas de límites → Agregado Expense verifica límites
Agregado Expense → Registra gasto → Agregado Budget actualiza estado
```

**Patrón 3: Agregación Impulsada por Datos (Monitoreo de Salud)**

```
Device → Envía datos → Agregado DeviceReading
Agregado DeviceReading → Validación de datos completa → Agregado HealthProfile
Agregado HealthProfile → Verificación de umbral → Agregado Alert
```

## Servicios de Dominio: Lógica de Negocio Entre Agregados

### Cuándo Identificar Servicios de Dominio

Necesitamos servicios de dominio cuando la lógica de negocio **no pertenece naturalmente a ningún agregado individual**.

**Caso de Servicio de Dominio para el Sistema de Trading de Inversiones**:

```typescript
// Servicio de cálculo de riesgo: requiere colaboración entre Portfolio y MarketData
class RiskCalculationService {
  calculate(portfolio: Portfolio, marketData: MarketData): RiskMetric {
    // Esta lógica no pertenece a Portfolio (porque necesita datos de mercado externos)
    // Ni pertenece a MarketData (porque necesita información específica del portafolio)

    const portfolioValue = portfolio.getTotalValue(marketData);
    const volatility = marketData.calculateVolatility(portfolio.getSymbols());
    const concentration = portfolio.calculateConcentration();

    return new RiskMetric(portfolioValue, volatility, concentration);
  }
}

// Servicio de coordinación de ejecución de trade: coordina agregados Portfolio y Order
class TradeExecutionService {
  async executeTrade(
    portfolio: Portfolio,
    tradeOrder: TradeOrder
  ): Promise<ExecutionResult> {
    // 1. Validar la capacidad de trading del Portfolio
    const validation = portfolio.validateTrade(tradeOrder);
    if (!validation.isValid) {
      return ExecutionResult.rejected(validation.reason);
    }

    // 2. Crear un agregado Order para gestionar el proceso de ejecución
    const order = Order.create(portfolio.getId(), tradeOrder);

    // 3. Enviar al mercado (servicio externo)
    const marketResult = await this.marketService.submit(order);

    // 4. Actualizar el estado de agregados relacionados
    if (marketResult.isSuccessful) {
      portfolio.applyExecution(marketResult);
      order.markAsExecuted(marketResult);
    }

    return ExecutionResult.fromMarket(marketResult);
  }
}
```

### Principios de Diseño para Servicios de Dominio

**Sin Estado**: Los servicios de dominio no mantienen estado, solo coordinan interacciones entre agregados

**Orientados al Negocio**: El nombre e interfaz de un servicio deben reflejar conceptos de negocio, no implementaciones técnicas

**Responsabilidad Mínima**: Cada servicio solo maneja un escenario de negocio claro

## Preparándose para Escenarios de Operación del Usuario

### Del Modelo de Dominio a Roles de Usuario

El diseño de agregados establecido hoy afectará directamente cómo diseñamos escenarios de operación del usuario mañana:

**Correspondencia entre Roles y Agregados**:

```
Sistema de Trading de Inversiones:
- Rol Trader → Propietario del agregado Portfolio
- Administrador del sistema → Monitor del sistema completo
- Especialista en control de riesgo → Operador del agregado RiskManagement

Sistema Financiero Familiar:
- Jefe de familia → Gestor del agregado Family
- Miembro de familia → Creador del agregado Expense
- Gestor de presupuesto → Mantenedor del agregado Budget
```

**Límites de Agregado de Permisos de Operación**:

- Cada agregado define los límites de operación para roles específicos
- Las operaciones entre agregados necesitan coordinarse a través de servicios de dominio
- Los procesos de negocio complejos producirán historias de usuario multi-paso

### Preparación para Escenarios de Estado

El cambio de estado de cada agregado producirá un escenario de operación de usuario correspondiente:

**Transiciones de Estado del Agregado Portfolio**:

```
Portfolio Vacío → [Inicializar fondos] → Portfolio Activo
Portfolio Activo → [Ejecutar trade] → Portfolio Activo (actualización de estado)
Portfolio Activo → [Riesgo excedido] → Portfolio Restringido
Portfolio Restringido → [Ajuste de riesgo] → Portfolio Activo
Portfolio Activo → [Vender todo] → Portfolio Vacío
```

Cada transición de estado corresponde a un escenario de operación del usuario. Mañana, profundizaremos en cómo diseñar estas transiciones en Historias de Usuario intuitivas.

## Registrando Decisiones de Diseño

Al diseñar agregados, tomamos muchas decisiones importantes. Registrar estas decisiones es crucial para el diseño subsecuente de Historias de Usuario:

**Decisión 1: Separar Portfolio y Order**

- Razón: Portfolio se enfoca en el estado de activos, Order se enfoca en el proceso de trading
- Impacto: Los usuarios necesitan ver el estado de activos y el estado de trades en diferentes interfaces
- Impacto en Historia de Usuario: Necesidad de diseñar un flujo de operación entre agregados

**Decisión 2: El Agregado Family Incluye Gestión de Permisos**

- Razón: La configuración de permisos está estrechamente relacionada con la estructura familiar
- Impacto: Los cambios de permisos desencadenarán cambios de estado en el agregado Family
- Impacto en Historia de Usuario: Necesidad de diseñar una interfaz de usuario para gestión de permisos

**Decisión 3: Adoptar Separación Lectura-Escritura para Datos de Salud**

- Razón: La frecuencia de lecturas y escrituras es muy diferente
- Impacto: Las consultas en tiempo real y el análisis histórico usan diferentes fuentes de datos
- Impacto en Historia de Usuario: Necesidad de considerar el retraso temporal de sincronización de datos

## Puente hacia Mañana

A través del diseño de agregados de hoy, hemos establecido la estructura estática de la lógica de negocio. Mañana, profundizaremos en:

1. **Técnicas de Escritura de Historias de Usuario**: Cómo transformar las capacidades de negocio de los agregados en historias de usuario
2. **Diseño de Roles y Permisos**: Cómo diseñar un sistema de roles y permisos basado en límites de agregados
3. **Flujos de Escenarios de Operación**: Cómo diseñar flujos de negocio complejos entre agregados
4. **Gestión de Escenarios de Estado**: Cómo manejar los desafíos de experiencia de usuario provocados por cambios de estado de agregados

## Conclusiones Conceptuales de Hoy

- **Los agregados son los límites de invariantes de negocio**: La implementación técnica debe servir a las reglas de negocio
- **Los servicios de dominio coordinan lógica entre agregados**: La organización de procesos de negocio complejos
- **Diferentes dominios tienen diferentes patrones de agregados**: Inversión, familia y salud cada uno tiene sus propias características
- **El diseño de agregados afecta directamente la experiencia del usuario**: Un buen modelo de dominio es la base de una buena UX

Recuerda: Lo que establecimos hoy no es una arquitectura técnica, sino la organización de la lógica de negocio. Esta organización determinará directamente cómo diseñamos la experiencia de operación del usuario mañana.

---

> "Un agregado no es una agrupación de código, sino una unidad viviente de un concepto de negocio. Cada agregado tiene su propio ciclo de vida único, y cada operación del usuario está participando en la evolución de este ciclo de vida."
