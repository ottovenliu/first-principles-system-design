# Día 3 | Metodología de Extracción de Requisitos - Modelado Abstracto

En los primeros tres días, completamos una importante transformación cognitiva: extrayendo la esencia de los requisitos funcionales de las expresiones vagas de los usuarios, y luego derivando límites técnicos específicos a partir de los requisitos funcionales. Sabemos qué debe hacer el sistema y hasta qué punto.

Pero ahora enfrentamos un desafío más profundo: **¿Cómo transformar estas comprensiones en un diseño de sistema alcanzable?**

Cuando sabemos que el sistema de trading de inversiones necesita soportar 2000 RPS, con una latencia menor a 100ms y un RPO de cero, ¿qué tipo de modelo conceptual, flujo de comportamiento, estructura de datos y patrón arquitectónico debemos diseñar?

Este es el problema central que el **modelado abstracto** pretende resolver: **un método de transformación sistemática desde la comprensión de requisitos hasta el diseño de sistemas.**

## Los Cuatro Ámbitos del Modelado Abstracto

El modelado abstracto no es un método técnico único, sino un marco filosófico para **comprender el mismo sistema desde cuatro dimensiones diferentes**. Así como no podemos entender completamente una realidad compleja desde una sola perspectiva, no podemos describir completamente la esencia de un sistema con un solo modelo.

```
                Modelado Abstracto
                   |
┌────────┬─────────────┬────────────┬────────────┐
| Modelado Conceptual | Modelado de Comportamiento | Modelado de Datos | Modelado Arquitectónico |
|---------|--------------|-------------|-------------|
| DDD     | Caso de Uso  | Modelo ER   | Modelo C4   |
| Ontología| EventStorming| Esquema NoSQL| Vista 4+1  |
| ORM     | Máquina de Estados| Normalización| Clean Arch |
```

### El Significado Ontológico de las Cuatro Dimensiones

Cada dimensión responde a una pregunta fundamental en el diseño de sistemas:

- **Modelado Conceptual**: ¿Qué ES? (¿Cuál es la esencia de este sistema?)
- **Modelado de Comportamiento**: ¿Qué HACE? (¿Qué hará este sistema?)
- **Modelado de Datos**: ¿Qué RECUERDA? (¿Qué recuerda este sistema?)
- **Modelado Arquitectónico**: ¿Cómo está ESTRUCTURADO? (¿Cómo está organizado este sistema?)

Del análisis anterior, hemos obtenido restricciones de diseño para las cuatro dimensiones:

**Correspondencia entre Análisis de Requisitos y Restricciones de Modelado**:

- Modelo cognitivo del usuario → Nivel de abstracción del modelado conceptual
- Flujo de eventos de negocio → Lógica temporal del modelado de comportamiento
- Requisitos de RPS y capacidad → Estrategia de almacenamiento del modelado de datos
- Requisitos de disponibilidad y recuperación → Diseño de resiliencia del modelado arquitectónico

## El Primer Ámbito: Modelado Conceptual - La Ontología del Sistema

### DDD: De Conceptos Abstractos a Estructuras Estáticas

La pregunta central del modelado conceptual es: **Después de eliminar todos los detalles técnicos, ¿qué realidad representa este sistema?**

Pero más importante aún: **¿Cómo transformar estos conceptos abstractos en estructuras estáticas que puedan expresarse en un lenguaje de programación?**

Regresando al caso de trading de inversiones, del análisis del usuario INTJ en el Día 2, entendimos que el requisito central es "el control último del tiempo". Esta comprensión abstracta necesita transformarse en conceptos de dominio específicos:

**Transformación de Conceptos Abstractos a Entidades de Dominio**:

```
Capa de Concepto Abstracto:
"El control último del tiempo" → ¿Cuál es su manifestación concreta?

Capa de Concepto de Negocio:
- Sujeto de toma de decisiones: ¿Quién está controlando el tiempo? → Trader
- Objeto de control: ¿Qué está siendo controlado? → Portfolio
- Herramienta de control: ¿Cómo controlar? → TradeOrder
- Dependencia externa: ¿De dónde viene la presión del tiempo? → MarketData

Capa de Entidad de Dominio:
El concepto de programación correspondiente a cada concepto de negocio:
- Trader → Entidad (un sujeto con identidad)
- Portfolio → Raíz de Agregado
- Holding → Entidad (una entidad dentro del agregado)
- TradeOrder → Objeto de Valor
- Price → Objeto de Valor (valor inmutable)
```

**Diseño de Estructura Estática de Entidades de Dominio**:

```
Trader - Entidad
Atributos:
- traderId: String (identificador único)
- riskProfile: RiskProfile
- tradingPreferences: TradingPreferences

Responsabilidades:
- Crear portafolios
- Establecer límites de riesgo
- Realizar órdenes de trading

Invariantes:
- El ID del trader no puede cambiarse una vez creado
- El perfil de riesgo debe estar dentro del rango permitido del sistema

Portfolio - Raíz de Agregado
Atributos:
- portfolioId: String (identificador único)
- traderId: String (propietario)
- holdings: List<Holding>
- totalValue: Money
- lastUpdated: DateTime

Responsabilidades:
- Gestionar todas las tenencias
- Calcular el riesgo general
- Ejecutar operaciones de trading
- Mantener la consistencia de datos

Límite de Agregado:
- Portfolio es la raíz del agregado
- Holding solo puede accederse a través de Portfolio
- Las operaciones dentro del mismo Portfolio deben permanecer consistentes

Invariantes:
- Valor total = Σ(cantidad de tenencia × precio actual)
- La tenencia de efectivo no puede ser negativa
- Los indicadores de riesgo no pueden exceder los límites establecidos
```

**Diseño de Relaciones entre Conceptos**:

```
Análisis de Tipos de Relación:
1. Propiedad
   Trader --posee--> Portfolio
   - Relación uno a muchos
   - Propiedad fuerte, Portfolio no puede existir independientemente
   - Borrado en cascada: Portfolio se elimina cuando se elimina Trader

2. Composición
   Portfolio --contiene--> Holding
   - Relación uno a muchos
   - Composición fuerte, Holding es un componente de Portfolio
   - Vinculación del ciclo de vida: Holding se elimina cuando se elimina Portfolio

3. Referencia
   Holding --referencia--> Asset
   - Relación muchos a uno
   - Referencia débil, Asset existe independientemente
   - Asset es un concepto externo definido por el mercado

4. Historial
   Portfolio --registra--> TradeTransaction
   - Relación uno a muchos
   - Relación de serie temporal, solo aumenta
   - Usado para auditoría y análisis
```

Estos diseños de relaciones sientan directamente las bases para el diseño del diagrama de clases de mañana. Cada tipo de relación tiene su implementación y restricciones específicas.

### Ontología: Dependencia Existencial entre Conceptos

Un modelado conceptual más profundo requiere comprender: **¿Cuál es la relación de dependencia existencial entre estos conceptos?**

**Jerarquía de Dependencia Existencial**:

```
Capa de Existencia Independiente:
- Trader: Sujeto de negocio, puede existir independientemente
- Asset: Concepto de mercado, definido externamente, existe independientemente de nuestro sistema
- MarketData: Flujo de eventos externos, independiente de nuestro sistema

Capa de Existencia Dependiente:
- Portfolio: Depende de Trader para existir, pero tiene su propio ciclo de vida
- Holding: Depende de Portfolio para existir, no tiene significado independiente
- TradeOrder: Depende de Portfolio para existir, expresa intención de trading

Capa de Existencia Relacional:
- Price: La relación entre Asset y tiempo
- Risk: La relación entre Portfolio y Market
- Profit/Loss: La relación del cambio de valor a lo largo del tiempo
```

**Estrategia de Implementación para Dependencia Existencial**:

```python
# Existencia independiente: puede crearse y gestionarse por separado
class Trader:
    def __init__(self, trader_id: str):
        self.trader_id = trader_id
        self.portfolios = {}  # Gestiona pero no posee el ciclo de vida

    def create_portfolio(self) -> Portfolio:
        portfolio = Portfolio(self.trader_id, generate_id())
        self.portfolios[portfolio.id] = portfolio
        return portfolio

# Existencia dependiente: debe crearse a través de la raíz del agregado
class Portfolio:
    def __init__(self, trader_id: str, portfolio_id: str):
        self.trader_id = trader_id  # Dependencia existencial
        self.portfolio_id = portfolio_id
        self.holdings = {}

    def add_holding(self, symbol: str, quantity: int) -> None:
        # Holding solo puede crearse a través de Portfolio
        holding = Holding(self.portfolio_id, symbol, quantity)
        self.holdings[symbol] = holding

    def remove_holding(self, symbol: str) -> None:
        # Controla el ciclo de vida de Holding
        if symbol in self.holdings:
            del self.holdings[symbol]

# Existencia relacional: calculado, no almacenado independientemente
class PortfolioAnalyzer:
    def calculate_risk(self, portfolio: Portfolio, market_data: MarketData) -> RiskMetrics:
        # El riesgo es la relación entre Portfolio y Market, calculado
        return RiskMetrics(
            volatility=self._calculate_volatility(portfolio, market_data),
            var=self._calculate_var(portfolio, market_data)
        )
```

Este análisis de dependencia existencial proporciona la base para el diseño de agregados de mañana y el diseño de interacción entre clases.

## El Segundo Ámbito: Modelado de Comportamiento - Interacción entre Conceptos

### El Comportamiento como Colaboración entre Conceptos

La intuición central del modelado de comportamiento es: **El comportamiento de un sistema no es el comportamiento de un solo objeto, sino la interacción colaborativa entre múltiples conceptos.**

Esta cognición sienta una base importante para el diseño de diagramas de secuencia de mañana: cada caso de uso es una serie de paso de mensajes y cambios de estado entre conceptos.

**Análisis de Interacción Conceptual del Caso de Uso de Trading de Inversiones**:

```
Caso de Uso: Ejecutar un trade rápido
Conceptos Participantes: Trader, Portfolio, Holding, MarketData, TradeOrder

Secuencia de Interacción:
1. Trader.createTradeIntent(symbol, quantity)
   - Trader valida internamente la intención de trade
   - Crea un objeto de valor TradeIntent

2. Portfolio.validateTrade(tradeIntent)
   - Portfolio verifica fondos suficientes
   - Portfolio verifica límites de riesgo
   - Llama a RiskCalculator.assess(portfolio, tradeIntent)

3. MarketData.getCurrentPrice(symbol)
   - Obtiene el precio de mercado en tiempo real
   - Retorna un objeto de valor Price

4. Portfolio.executeTradeOrder(tradeOrder)
   - Crea un TradeOrder
   - Actualiza el Holding correspondiente
   - Registra una TradeTransaction
   - Dispara un evento PortfolioUpdated

5. NotificationService.notify(trader, tradeResult)
   - Servicio externo maneja las notificaciones
```

**Principios de Diseño para Interacción Conceptual**:

```
1. Control de Dirección de Dependencia:
   - Los conceptos de alto nivel pueden depender de conceptos de bajo nivel
   - Las raíces de agregado pueden depender de entidades dentro del agregado
   - Los servicios de dominio pueden coordinar múltiples agregados

2. Patrones de Paso de Mensajes:
   - Llamadas síncronas: Operaciones dentro del mismo agregado
   - Publicación de eventos: Comunicación entre agregados
   - Patrón de comando: Procesos de negocio complejos

3. Control de Cambio de Estado:
   - Solo la raíz del agregado puede modificar el estado dentro del agregado
   - Los cambios de estado entre agregados se implementan a través de eventos
   - Todos los cambios de estado deben mantener invariantes
```

### EventStorming: Modelado Temporal de Interacción Conceptual

EventStorming no es solo una herramienta de análisis de requisitos, sino también un método de modelado para el diseño de interacción conceptual:

**Interacción Conceptual Impulsada por Eventos de Dominio**:

```
Roles Conceptuales en el Flujo de Eventos:
TradeIntentCreated
- Disparador: Trader (crea una intención de trade)
- Participante: Portfolio (recibe y valida)
- Resultado: FundsValidationRequested

FundsValidated
- Disparador: Portfolio (completa verificación de fondos)
- Participante: RiskCalculator (servicio de evaluación de riesgo)
- Resultado: RiskAssessmentRequested

RiskAssessed
- Disparador: RiskCalculator (completa evaluación de riesgo)
- Participante: Portfolio (decide ejecutar)
- Resultado: TradeOrderCreated o TradeRejected

TradeOrderSubmitted
- Disparador: Portfolio (envía la orden)
- Participante: MarketGateway (interfaz de mercado externo)
- Resultado: MarketExecutionRequested

MarketExecutionConfirmed
- Disparador: MarketGateway (confirmación del mercado)
- Participante: Portfolio (actualiza estado)
- Resultado: HoldingUpdated, TransactionRecorded

PortfolioUpdated
- Disparador: Portfolio (el estado ha sido actualizado)
- Participante: NotificationService (servicio de notificación)
- Resultado: TraderNotified
```

Cada evento representa una interacción importante entre conceptos, lo que proporciona un script de interacción detallado para el diagrama de secuencia de mañana.

### Máquina de Estados: Coordinación de Estados Conceptuales

Una máquina de estados no es solo el cambio de estado de un solo objeto, sino más importante aún, la coordinación de estados entre múltiples conceptos:

**Cambio de Estado y Coordinación Conceptual de TradeOrder**:

```
Colaboración Conceptual en Transiciones de Estado:

Created → Validating:
- Portfolio inicia el proceso de validación
- RiskCalculator participa en el cálculo de riesgo
- MarketData proporciona información de precios

Validating → Approved:
- Portfolio confirma que todas las validaciones han pasado
- El estado de TradeOrder se actualiza a Approved
- Portfolio se prepara para enviar al mercado

Approved → Submitted:
- Portfolio llama a MarketGateway
- El estado de TradeOrder se actualiza a Submitted
- Establece una relación con el sistema externo

Submitted → Executed:
- MarketGateway recibe confirmación del mercado
- Portfolio actualiza el estado de Holding
- TradeOrder se marca como Executed
- Dispara procesos de notificación subsecuentes
```

Este diseño de máquina de estados con coordinación multi-concepto sienta las bases para el diagrama de diseño de sistemas de mañana.

## El Tercer Ámbito: Modelado de Datos - Persistencia de Conceptos

### De Conceptos de Dominio a Estructuras de Datos

La clave del modelado de datos es: **¿Cómo mapear fielmente las relaciones de entidades en el modelo conceptual a la estructura de almacenamiento de datos?**

**Diseño de Almacenamiento de Agregados en DynamoDB**:

```
Estrategia de Particionamiento Orientada a Raíces de Agregado:
PK (Partition Key) | SK (Sort Key)        | Tipo de Entidad
TRADER#123        | PROFILE              | Trader
TRADER#123        | PORTFOLIO#001        | Portfolio
TRADER#123        | HOLDING#001#AAPL     | Holding
TRADER#123        | TRADE#001#20240101   | TradeTransaction

Correspondencia entre Patrones de Consulta y Modelo Conceptual:
1. Obtener todos los Portfolios de un Trader:
   PK = TRADER#123, begins_with(SK, "PORTFOLIO")

2. Obtener todos los Holdings de un Portfolio:
   PK = TRADER#123, begins_with(SK, "HOLDING#001")

3. Reconstrucción completa del agregado Portfolio:
   PK = TRADER#123, SK between "PORTFOLIO#001" and "PORTFOLIO#001~"

Correspondencia entre Garantía de Consistencia y Límite de Agregado:
- Operaciones dentro de la misma Partición = Operaciones dentro del mismo agregado
- Transacción de DynamoDB = Operaciones entre agregados
- Publicación de eventos = Comunicación entre agregados
```

### Implementación de Datos de Relaciones Conceptuales

Diferentes relaciones conceptuales requieren diferentes estrategias de implementación de datos:

```python
# Relación de propiedad: Diseño embebido
{
    "PK": "TRADER#123",
    "SK": "PORTFOLIO#001",
    "ownerId": "TRADER#123",  # Identificador de propiedad
    "portfolioData": {
        "totalValue": 100000,
        "createdAt": "2024-01-01",
        "riskProfile": "MODERATE"
    }
}

# Relación de composición: Múltiples registros dentro de la misma partición
{
    "PK": "TRADER#123",
    "SK": "HOLDING#001#AAPL",
    "portfolioId": "PORTFOLIO#001",  # Identificador de relación de composición
    "symbol": "AAPL",
    "quantity": 100,
    "avgPrice": 150.50
}

# Relación de referencia: Clave foránea + caché
{
    "PK": "TRADER#123",
    "SK": "HOLDING#001#AAPL",
    "assetSymbol": "AAPL",  # Referencia a un Asset externo
    # Los detalles del Asset se almacenan en caché vía ElastiCache
}

# Relación de historial: Almacenamiento de series temporales
{
    "PK": "TRADER#123",
    "SK": "TRADE#001#2024-01-01T10:30:00",
    "portfolioId": "PORTFOLIO#001",
    "tradeType": "BUY",
    "symbol": "AAPL",
    "quantity": 100,
    "price": 150.50,
    "timestamp": "2024-01-01T10:30:00Z"
}
```

## El Cuarto Ámbito: Modelado Arquitectónico - Mapeo Técnico de Conceptos

### Mapeo entre Servicios de AWS y Conceptos de Dominio

Cada servicio de AWS debe corresponder a un concepto de dominio o responsabilidad específica:

**Estrategia de Mapeo de Servicios**:

```
Concepto de Dominio → Servicio de AWS → Responsabilidad Arquitectónica

Gestión de identidad del Trader → Cognito User Pool → Autenticación y autorización
Agregado Portfolio → DynamoDB + Lambda → Gestión de estado
Flujo de MarketData → Kinesis Data Streams → Ingesta de eventos externos
Procesamiento de TradeOrder → Step Functions → Procesos de negocio complejos
Publicación de eventos de dominio → EventBridge → Comunicación entre conceptos
Caché de cálculo de precios → ElastiCache → Optimización de rendimiento
Notificación de trades → SNS + SES → Integración externa
```

### Capas Conceptuales en Clean Architecture

```python
# Capa de Dominio: Modelo conceptual puro
class Portfolio:  # Concepto de dominio
    def execute_trade(self, trade_order: TradeOrder) -> TradeResult:
        # Lógica de negocio pura, no depende de ninguna implementación técnica
        pass

# Capa de Aplicación: Coordinación conceptual
class ExecuteTradeUseCase:  # Colaboración entre conceptos
    def __init__(self, portfolio_repo: PortfolioRepository,
                 market_gateway: MarketGateway):
        self.portfolio_repo = portfolio_repo
        self.market_gateway = market_gateway

    def execute(self, trade_intent: TradeIntent) -> TradeResult:
        # Coordina la interacción de conceptos como Portfolio, MarketGateway, etc.
        portfolio = self.portfolio_repo.find_by_id(trade_intent.portfolio_id)
        return portfolio.execute_trade(trade_intent.to_trade_order())

# Capa de Infraestructura: Implementación técnica de AWS
class DynamoDBPortfolioRepository(PortfolioRepository):  # Persistencia de conceptos
    def save(self, portfolio: Portfolio) -> None:
        # Mapea conceptos de dominio a DynamoDB
        pass

class KinesisMarketGateway(MarketGateway):  # Ingesta de conceptos externos
    def submit_order(self, order: TradeOrder) -> OrderStatus:
        # Interactúa con el mercado externo vía Kinesis
        pass
```

## Preparándose para la Inmersión Profunda en DDD de Mañana

A través del modelado abstracto de hoy, hemos establecido los cuatro fundamentos del diseño de sistemas:

**El modelado conceptual** estableció entidades de dominio y sus relaciones
**El modelado de comportamiento** definió los patrones de interacción entre conceptos
**El modelado de datos** diseñó la estrategia de persistencia para conceptos
**El modelado arquitectónico** planificó la implementación técnica de conceptos

Pero estos todavía son marcos abstractos. Mañana profundizaremos en las prácticas específicas de DDD y aprenderemos:

- **¿Cómo transformar los conceptos abstractos de hoy en un diseño concreto de diagrama de clases?**
- **¿Cómo describir con precisión la interacción entre conceptos con un diagrama de secuencia?**
- **¿Cómo determinar e implementar los límites del diseño de agregados?**
- **¿Cuál es el proceso de diseño completo desde el caso de uso hasta el agregado?**

En particular, hoy entendimos la intuición central de que "el comportamiento es la interacción entre diferentes conceptos". Mañana aprenderemos específicamente cómo diseñar estas interacciones y cómo asegurar que la colaboración entre conceptos no solo cumpla con los requisitos de negocio sino que también cumpla con las restricciones técnicas.

## Resumen de la Filosofía de Modelado de Hoy

- **El modelado abstracto es una lente cuádruple para comprender la esencia de un sistema**
- **El modelo conceptual debe proporcionar una guía clara para la implementación concreta**
- **El comportamiento es esencialmente una interacción colaborativa entre conceptos**
- **Cada nivel de abstracción sienta las bases para la implementación del siguiente nivel**

Recuerda: No estamos modelando tecnología, sino modelando la realidad. Cada concepto tiene su razón de existir, y cada interacción tiene su significado de negocio. Mañana veremos cómo estos conceptos abstractos se convierten en un sistema ejecutable bajo la guía de DDD.

---

> "La interacción de conceptos constituye el comportamiento del sistema. No estamos diseñando objetos, sino las relaciones entre objetos."
