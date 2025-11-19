# Día 16 | Gobernanza y Estrategia de Arquitectura Multi-Entorno Dev / Staging / Prod: Gestión de Configuración y Estrategia de Despliegue Multi-Entorno de AWS

En los dos temas anteriores, <Infraestructura como Código: Codificando y Versionando la Infraestructura con Terraform> e <Implementación Completa de Automatización CI/CD - GitHub Actions × CodePipeline × CodeBuild>, discutimos la importancia de **"hacer la infraestructura predecible, repetible y versionable"** y la proactividad de **"proteger la lógica de negocio existente de ser corrompida por cambios en el código,"** respectivamente. Ambos tienen como objetivo resolver el problema de `no tener un entorno estable y un proceso de validación de lógica de negocio fijo para pruebas y despliegue`, asegurando que nuestra **`validación de lógica de negocio`** pueda crecer fuerte como un árbol imponente en suelo estable (no estoy hablando de cierto banco taiwanés). Después de hablar brevemente sobre la importancia de los entornos y las soluciones a los puntos débiles correspondientes, ahora discutiremos - **`¿Qué entornos necesito?`**

Recientemente, tuve el placer de visitar un restaurante con un amigo, Impromptu by Paul Lee en el Regent Taipei, para probar los sabores de verano de 2025. Era un diseño único que combinaba una cocina abierta con el área de asientos, permitiendo a los comensales ver de manera simple y transparente cómo el chef y su equipo presentan un hermoso y magnífico tapiz del verano de 2025 utilizando ingredientes, especias y vinos seleccionados. Personalmente, me encantó este diseño de menú; se podía sentir una elegancia clara, nítida y directa.

Pero hablando de eso, ¿cómo se diseña un plato con estrella Michelin? Un nuevo concepto es una noticia emocionante para todos los amantes de la comida, pero ¿qué pasa si se hace y se sirve directamente al cliente en el mostrador y falla? Demasiada sal, tiempo de cocción incorrecto: en el mejor de los casos, pierdes tu reputación; en el peor, el cliente tiene dolor de estómago y, al día siguiente, el nombre de tu restaurante está arruinado en los titulares.

Así que demos un paso atrás. ¿Qué tal una prueba en un rincón de la "cocina principal" del restaurante antes de un lanzamiento oficial?

Este es un buen método. Evitamos que los clientes prueben platos experimentales de inmediato. Sin embargo, aunque los clientes no pueden verlo, podríamos interrumpir el funcionamiento normal existente de la cocina. Podríamos ocupar una tabla de cortar que se está utilizando para la preparación, descubrir que otro chef ha usado una salsa cuando la necesitamos, o incluso el humo de nuestro experimento podría afectar los platos cuidadosamente cocinados a nuestro lado.

Dicho esto, si los fondos y el espacio lo permiten, tratar de realizar nuestro nuevo concepto en una "Cocina de Prueba" completamente independiente parece ser la solución óptima. Aquí, tenemos nuestro propio juego completo de ollas, sartenes, ingredientes y hornos, lo que nos permite experimentar, fallar y volver a intentar libremente sin preocuparnos por afectar las operaciones del restaurante en el exterior.

Este es el **Entorno de Desarrollo (Dev)**. Es una caja de arena completamente aislada que permite a cada desarrollador escribir, modificar y probar libremente su propio código sin preocuparse por "romper" nada o afectar a nadie. Nos impide afectar a los colegas que están probando otras características en la **"cocina principal" entre bastidores (entorno compartido)**, y viceversa. Lo más importante es que evita escribir código directamente en el entorno de producción (Producción / Prod) en la mayor medida posible. Cualquier pequeño error, como un comando de base de datos incorrecto, podría provocar la interrupción del servicio para todos los usuarios y la pérdida de datos. **`Este es un comportamiento absolutamente, absolutamente prohibido.`**

Antes de que se sirva oficialmente, hemos creado el nuevo plato en nuestra propia **cocina de I+D (entorno de desarrollo)** y sabe celestial. ¿Que sigue? ¿Ponerlo directamente en el menú y venderlo a los críticos gastronómicos más importantes?

Si no tenemos un chef maestro escondido en un gorro de chef encima de nosotros, y lo que estamos sirviendo es ratatouille (Ratatouille es una gran película, véanla si tienen tiempo), este es un movimiento muy arriesgado.

Antes del lanzamiento oficial, necesitamos un lugar para un **"ensayo final."** Llamamos a este lugar la "Cocina de Staging". Su configuración, estufas, flujo de trabajo e incluso la marca de los platos deben ser idénticos a los de la "cocina principal" oficial. Aquí, necesitamos simular el proceso de servicio real: el camarero tiene que ejecutar todo el proceso de pedido y servicio, el equipo de chefs tiene que simular si pueden producir el plato de manera estable de acuerdo con el proceso previsto bajo la presión de las horas pico y, finalmente, necesitamos probar si este nuevo plato entrará en conflicto con el proceso cuando se sirva con otros platos clásicos.

Esta es la importancia del **Entorno de Staging / Pre-producción**. Debemos tener un entorno para pruebas exhaustivas en varios escenarios para garantizar que nuestra lógica de negocio cumpla con los requisitos y estándares de calidad.

Ahora, podemos abstraer el ciclo de vida del desarrollo de software en una "Progresión" clara:

```python
Dev => Staging => Prod
```

- **Entorno de Desarrollo (Dev): La Cocina de Prueba**
  - **Propósito**: Desarrollo de características, pruebas unitarias, iteración rápida.
  - **Características**: El caos es la norma. Se permiten modificaciones frecuentes, despliegues e incluso derribos completos. Los desarrolladores tienen mayores privilegios. Los datos suelen ser falsos o anonimizados.
  - **Enfoque Central**: Eficiencia del desarrollo.

- **Entorno de Staging: La Cocina de Staging**
  - **Propósito**: Pruebas de integración, Pruebas de Aceptación del Usuario (UAT), pruebas de estrés de rendimiento. Simula todos los escenarios del mundo real.
  - **Características**: El entorno debe ser extremadamente cercano al entorno de producción. La configuración, el sistema operativo, las reglas de red, la versión de la base de datos, etc., deben ser consistentes. El proceso de despliegue también debe ser idéntico al proceso de lanzamiento oficial. Los datos pueden ser una réplica del entorno de producción (después de la desensibilización).
  - **Enfoque Central**: Estabilidad y Consistencia. Esta es la última línea de defensa de la calidad antes de salir en vivo.

- **Entorno de Producción (Prod): El Restaurante con Estrella Michelin**
  - **Propósito**: Servir a usuarios reales y crear valor.
  - **Características**: Extremadamente estable, seguro y de alto rendimiento. Cualquier cambio debe pasar por una estricta aprobación y procesos estandarizados. El control de acceso es el más estricto y, por lo general, solo las herramientas automatizadas o un número muy pequeño de personal autorizado pueden realizar despliegues.
  - **Enfoque Central**: Fiabilidad y Seguridad.

Este flujo unidireccional de **`Dev => Staging => Prod`** es el `flujo de trabajo central` de la garantía de calidad del software. El código, como el agua, solo puede fluir de un entorno inferior a uno superior, nunca al revés. Cada flujo es una verificación en una **`"Puerta de Calidad,"`** ya sean las **`verificaciones automatizadas`** en el proceso de **CI (Integración Continua)** para **`proteger la lógica de negocio existente de ser corrompida por cambios en el código`**, o la verificación de tres vías de la **Puerta de Revisión por Pares**, la **Puerta de Revisión de Negocio** y la **Puerta de Revisión de Calidad** mencionadas en <Estrategia de Control de Versiones (Estrategia de Revisión de PR)>. Una vez que **`no se cumple ninguna condición para la implementación de la lógica de negocio`**, debe devolverse inmediatamente para su reparación de emergencia. Debemos tener un entendimiento profundo y claro - **`un sistema es la implementación de una lógica de negocio abstracta`**.

En el desarrollo de software moderno, la gestión de múltiples entornos es clave para garantizar la estabilidad y fiabilidad de las aplicaciones. A continuación, profundizaremos en cómo construir y gestionar arquitecturas multi-entorno Dev/Staging/Prod en AWS, centrándonos en tecnologías centrales como la contenedorización de Docker, la gestión de clústeres de ECS y la orquestación de EKS/K8s.

> 1. Principios de Diseño de Arquitectura Multi-Entorno - Estrategia de Aislamiento de Entornos y Práctica de IaC
> 2. Estrategia de Contenerización de Docker - Construcciones Multi-etapa y Configuraciones Específicas del Entorno
> 3. Gestión de Clústeres de Amazon ECS - Definición de Servicios y Configuración de Auto-Scaling
> 4. Gestión de Amazon EKS y Kubernetes - Configuración de Clústeres, Despliegue de Aplicaciones y Configuración de Ingress
> 5. Integración de Pipeline CI/CD - Flujos de Trabajo de GitHub Actions y Despliegue Azul/Verde
> 6. Gestión de Configuración y Secretos - AWS Secrets Manager y K8s ConfigMap/Secret
> 7. Gestión de Monitoreo y Registros - Integración de CloudWatch y Prometheus/Grafana
> 8. Mejores Prácticas de Seguridad - Seguridad de Red y Configuración de RBAC
> 9. Estrategia de Optimización de Costos - Gestión Automatizada de Recursos e Integración de Instancias Spot
> 10. Recuperación ante Desastres y Copias de Seguridad - Copias de Seguridad entre Regiones y Failover de Bases de Datos

## 1. Principios de Diseño de Arquitectura Multi-Entorno

### 1.1 Estrategia de Aislamiento de Entornos

Discutimos anteriormente que necesitamos dividir nuestro restaurante (nuestro sistema) en una "Cocina de I+D (Dev)", una "Cocina de Staging" y una "Tienda Insignia (Prod)". Hablemos de cómo "demarcar los límites" y asegurar que los "códigos de construcción" para estos tres terrenos sean consistentes.

```python
Entorno de Producción
├── Configuración de alta disponibilidad
├── Auto-scaling
├── Monitoreo y alertas completos
└── Proceso de despliegue estricto

Entorno de Staging
├── Espejo del entorno de producción
├── Pruebas funcionales completas
├── Entorno de pruebas de rendimiento
└── Validación de pruebas de integración

Entorno de Desarrollo
├── Despliegue rápido
├── Amigable para el desarrollador
├── Optimización de costos de recursos
└── Ajustes de configuración flexibles
```

La forma más sencilla es, por supuesto, cerrar con llave cada cocina y dar llaves exclusivas a las personas con las necesidades funcionales correspondientes. Cierra las puertas primero para evitar que la gente deambule. Ya es bastante malo si nuestra propia gente ajusta accidentalmente los parámetros o comete errores de configuración que causan la parálisis del servicio. Como mínimo, debemos asegurarnos de que los ladrones y los salteadores sin llaves ni siquiera puedan entrar. Así que nuestro primer principio central es simple: **`Minimizar el Radio de Explosión`**. Esto significa que cuando ocurre un desastre en un entorno (por ejemplo, ser pirateado o un error de configuración que causa la parálisis del servicio), no debe afectar en absoluto a otros entornos.

En AWS, el estándar de oro para lograr este objetivo se llama **`Estrategia Multi-Cuentas`**.

**1. AWS Organizations: La Sede Corporativa en la Nube**

**`AWS Organizations`** es nuestra "sede corporativa", a través de la cual podemos gestionar diferentes cuentas de AWS y formular políticas básicas. Así como un grupo tiene diferentes unidades de negocio para finanzas, ventas, legal, RR.HH., desarrollo y mantenimiento, podemos crear una `DevOU` y una `ProdOU` como conceptos para unidades de negocio. `Account-Dev` (departamento) se coloca en `DevOU` (unidad de negocio); `Account-Staging` y `Account-Prod` se colocan en `ProdOU`. Luego podemos establecer **`Políticas de Control de Servicio (SCPs)`** para restringir lo que las cuentas bajo ellas **pueden** o **no pueden** hacer. Esto es como una "orden ejecutiva suprema" emitida por el grupo. Cuando hay un conflicto entre las políticas de la unidad de negocio y la sede del grupo, las regulaciones de las `SCPs` se seguirán preferentemente y de forma obligatoria. Y `AWS Organizations` tiene otro beneficio: todas las facturas y costos de las unidades de negocio se agregan en la sede, lo que es conveniente para el monitoreo y la gestión.

Las `SCPs` previenen el error humano o el comportamiento malicioso desde la raíz, lo que es mucho más efectivo que remediarlo después. Define la "constitución conductual" y las "leyes físicas" de cada entorno. Por ejemplo:

- Para todas las cuentas bajo DevOU:
  - Prohibir la creación de bases de datos conectadas a la red externa (acceso público a RDS = falso).
  - Prohibir la desactivación de CloudTrail (registros de auditoría) para garantizar que todas las operaciones se registren.
  - Restringir la creación de recursos a la región de Asia-Pacífico (por ejemplo, Tokio, Singapur) para evitar que alguien cree accidentalmente recursos en regiones europeas o americanas costosas.
- Para todas las cuentas bajo ProdOU:
  - Excepto para roles de administrador específicos, prohibir a cualquiera eliminar bases de datos o buckets de S3.

**2. Permisos de IAM: "Pases" para Diferentes Identidades**

Después del aislamiento de la cuenta, ¿cómo entra la gente a trabajar? La respuesta son los Roles de IAM, no crear un conjunto de nombres de usuario y contraseñas (Usuario de IAM) para todos en cada cuenta.

- **Concepto**: Cree Usuarios de IAM en nuestra Cuenta de Administración y luego defina diferentes Roles de IAM (por ejemplo, DeveloperRole, QARole, OperatorRole). Luego, deje que estos Usuarios "Asuman el Rol" para realizar tareas en otras cuentas.
- **Analogía**: Somos empleados de la empresa (Usuario de IAM), y la empresa nos da diferentes credenciales de trabajo (Rol de IAM).
  - Con una "Credencial de Desarrollador", podemos pasar a todas las salas de Account-Dev.
  - Con una "Credencial de QA", podemos entrar en Account-Staging, pero solo para observar y registrar, no para modificar el equipo.
  - Solo con una "Credencial de Gerente de Tienda Insignia" se puede ingresar a Account-Prod para operaciones después de la aprobación.

Este enfoque centraliza la gestión de permisos, haciéndola segura y clara.

### 1.2 Infraestructura como Código (IaC)

Hemos asegurado que hay muros fuertes entre los entornos, pero ¿cómo garantizamos que la decoración interior, las tuberías y las estufas de la "Cocina de Staging" y la "Tienda Insignia" sean idénticas? Aquí es donde entra en juego el poder de la `Infraestructura como Código (IaC)`. De acuerdo con el `plano (IaC)`, podemos construir rápidamente una arquitectura y un entorno idénticos. Mencionamos la importancia de los planos en <Infraestructura como Código: Codificando y Versionando la Infraestructura con Terraform>. Hace que la infraestructura sea `predecible`, `repetible` y `evolucionable`. Continuemos con Terraform como ejemplo.

```
terraform/
├── modules/
│   ├── vpc/
│   │   ├── main.tf
│   │   └── variables.tf
│   └── ec2_instance/
│       ├── main.tf
│       └── variables.tf
│
└── environments/
    ├── dev/
    │   ├── main.tf
    │   └── terraform.tfvars
    ├── staging/
    │   ├── main.tf
    │   └── terraform.tfvars
    └── prod/
        ├── main.tf
        └── terraform.tfvars
```

- `modules/` almacena nuestros módulos de construcción estandarizados.
- `environments/` almacena las "instrucciones de construcción" para cada entorno.

En `environments/dev/main.tf`, lo presentamos con código falso:

```yaml
# Referencia al módulo VPC
module "my_vpc" {
  source = "../../modules/vpc"
  cidr_block = var.vpc_cidr
}

# Referencia al módulo EC2
module "web_server" {
  source      = "../../modules/ec2_instance"
  instance_type = var.instance_type
  vpc_id      = module.my_vpc.id
}
```

¿Lo notan? Este archivo `main.tf` se verá casi igual en dev, staging y prod. La verdadera diferencia radica en el archivo de parámetros `terraform.tfvars` que se encuentra al lado:

- `environments/dev/terraform.tfvars`
```
vpc_cidr      = "10.10.0.0/16"
instance_type = "t3.micro"  // Usa máquinas pequeñas en el entorno de desarrollo para ahorrar dinero
```

- `environments/prod/terraform.tfvars`
```
vpc_cidr      = "10.20.0.0/16"
instance_type = "m5.large"  // Usa máquinas grandes en el entorno de producción para manejar el tráfico
```

Cuando queremos desplegar el entorno de Desarrollo, simplemente vamos a la carpeta `environments/dev` y ejecutamos `terraform apply`. Terraform leerá automáticamente los parámetros de `dev`, los aplicará a los módulos compartidos y construirá un entorno que cumpla con las especificaciones de desarrollo. Lo mismo se aplica al despliegue de `Prod`.

Esto logra perfectamente nuestros objetivos de IaC:
1. **Consistencia**: La "arquitectura" de todos los entornos proviene del mismo plano (módulos).
2. **Trazabilidad**: Todos los cambios en la infraestructura deben realizarse modificando el código. Cada cambio se registra en el sistema de control de versiones (como Git), que puede ser revisado y rastreado.
3. **Reproducibilidad**: Incluso si todo el entorno se destruye, podemos reconstruir rápidamente uno idéntico con el mismo código.

## 2. Estrategia de Contenerización de Docker

Ya hemos dibujado los planos arquitectónicos de las tres cocinas utilizando IaC. Ahora, necesitamos estandarizar nuestras "herramientas de cocina". Este es el problema que la estrategia de contenedorización de Docker pretende resolver.

Todos hemos oído, y quizás incluso experimentado, este escenario clásico:
> Desarrollador A: "¡Esta característica funciona bien en mi máquina! ¿Por qué se rompió después del despliegue?"

Este problema de "Funciona en mi máquina" es la enfermedad crónica que Docker pretende curar. La raíz del problema es que existen diferencias sutiles pero fatales en los "entornos" de la computadora del desarrollador, el servidor de pruebas y el servidor de producción:
- Diferentes versiones del sistema operativo (Ubuntu 20.04 vs 22.04)
- Diferentes versiones de bibliotecas dependientes (Python 3.8 vs 3.9, OpenSSL v1 vs v3)
- Diferentes configuraciones de variables de entorno
- Diferentes parámetros de hardware
- Diferentes rutas de sistema o permisos de archivos

Incluso si todo es igual, uno podría pasar la prueba mientras que el otro falla. Con el mismo sistema operativo como Windows (sin prejuicios, pero aquí es donde es más probable que ocurran `diferencias subyacentes`), solo podemos hacer ofrendas, agua bendita o alabar al Omnissiah para rogarle al espíritu de la máquina por una ejecución fluida.

La idea central de **`Docker`** es empaquetar y llevarse la aplicación junto con todo su "entorno de ejecución".

En lugar de darle a un chef una receta de "Boeuf Bourguignon" y dejar que use las ollas, estufas y condimentos de su propia cocina (lo que probablemente resultaría en un sabor diferente debido a las diferencias en cada cocina), bien podríamos darle el producto semiacabado, una salsa especial y una "caja de calentamiento estandarizada" con control de temperatura preciso. Todo lo que tiene que hacer es presionar un botón.

Esta "caja de calentamiento estandarizada" es el **Contenedor de Docker**. Esta caja contiene:
1. Nuestro código de aplicación (El plato en sí)
2. El tiempo de ejecución, por ejemplo, Node.js v18 o Java 17 (La salsa de cocción específica)
3. Todas las bibliotecas y herramientas a nivel de sistema (La sal y pimienta especiales)

No importa si esta "caja" se coloca en el microondas del entorno de `Dev` o en el horno de primera categoría del entorno de `Prod` (siempre que ambos sean Hosts de Docker calificados), el plato que sale después de calentarse tiene la garantía de tener exactamente el mismo sabor. Esta es la `Portabilidad` y `Consistencia` que aporta la contenedorización.

### 2.1 Optimización de la Construcción Multi-etapa

Dividimos el proceso de construcción del Dockerfile en dos etapas, como una línea de ensamblaje:
- **Etapa 1 (Etapa de Constructor):** Una "planta de procesamiento" espaciosa y bien equipada.
- **Etapa 2 (Etapa Final):** Una "caja de entrega" limpia y ligera.

```dockerfile
# Dockerfile.multi-stage
# Etapa de construcción
FROM node:18-alpine AS builder
WORKDIR /app
# Copia solo el package.json necesario para la compilación
COPY package*.json ./
# Instala solo las dependencias necesarias para producción
RUN npm ci --only=production

# Etapa de desarrollo
FROM builder AS development
RUN npm ci
COPY . .
EXPOSE 3000
CMD ["npm", "run", "dev"]

# Etapa de producción
FROM node:18-alpine AS production
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
EXPOSE 3000
USER node
CMD ["npm", "start"]
```

Los beneficios de hacer esto son obvios:
- **Imagen final extremadamente pequeña:** Solo contiene `node_modules` (producción), `dist` y `package.json`. Sin código fuente, sin herramientas de desarrollo.
- **Mayor seguridad:** La superficie de ataque se minimiza. Las noticias de paquetes atacados o incrustados con troyanos no son comunes, pero se han oído. Las herramientas de compilación (como gcc, npm) y las bibliotecas de desarrollo pueden convertirse en vulnerabilidades para los piratas informáticos. Por lo tanto, el entorno de producción solo debe contener los componentes de ejecución `"mínimos necesarios"`.
- **Proceso de construcción claro:** Separa las preocupaciones de "cómo construir" y "cómo ejecutar".

### 2.2 Gestión de Configuración Específica del Entorno

¿Cómo se adapta una caja a diferentes cocinas?

Ya tenemos una "caja de calentamiento" estandarizada, pero ahora hay un nuevo problema:
> En el entorno de Desarrollo, este plato necesita conectarse a `dev-database`; en el entorno de Producción, necesita conectarse a `prod-database`.
> 
> Pero solo tenemos una caja (Imagen de Docker), entonces, ¿qué debemos hacer?

```yaml
# docker-compose.dev.yml
version: '3.8'
services:
  app:
    build:
      context: .
      target: development
    environment:
      - NODE_ENV=development
      - LOG_LEVEL=debug
    volumes:
      - .:/app
      - /app/node_modules
    ports:
      - "3000:3000"

# docker-compose.prod.yml
version: '3.8'
services:
  app:
    build:
      context: .
      target: production
    environment:
      - NODE_ENV=production
      - LOG_LEVEL=error
    restart: unless-stopped
    ports:
      - "80:3000"
```

Primero, debemos recordar cumplir una ley de hierro, una regla absoluta:
> #### **Construir una vez, ejecutar en cualquier lugar**

No podemos simplemente abrir una lonchera sellada debido a las necesidades de diferentes entornos. Esto violaría por completo la **`Inmutabilidad de la Imagen`**. No debemos construir absolutamente diferentes `Imágenes` para diferentes entornos (por ejemplo, `my-app:dev`, `my-app:prod`). Esto destruiría todas las **`garantías de consistencia`** que hemos establecido.

```
La Imagen que pasa las pruebas en Desarrollo debe ser exactamente la misma Imagen, sin modificar, que se despliega en Staging y Producción.
```

Pero, ¿y si realmente tenemos una necesidad de `inyección externa`? ¿Qué debemos hacer?

Entonces, nuestra "caja de calentamiento estándar" tiene algunas ranuras reservadas, como una "ranura de conexión de base de datos" y una "ranura de clave de API". La caja en sí no lleva estas cosas. En cambio, cuando se coloca en un lugar designado en una cocina específica, el "brazo automatizado" de la cocina (sistema de orquestación de contenedores) conectará los enchufes correspondientes.

O, como algunos fideos instantáneos japoneses, la entrada de agua caliente y la salida de agua son agujeros diferentes, cada uno para su propio propósito. Podemos verter y drenar agua caliente, té caliente (que también es bueno, lo he probado), o incluso té con leche caliente (???, lo he visto pero lo respeto), pero el cuerpo principal (los fideos y el condimento) en el interior permanece sin cambios.

Así que, volviendo al ejemplo de YAML, encontramos algunas pistas:

```yaml
# docker-compose.dev.yml
    environment:
      - NODE_ENV=development
      - LOG_LEVEL=debug

# docker-compose.prod.yml
    environment:
      - NODE_ENV=production
      - LOG_LEVEL=error
```

Podemos establecer diferentes interfaces declarando **`variables de entorno`**. Este es el método más estándar y el que mejor se ajusta a la filosofía de la App de 12 Factores: **el código de la aplicación no debe codificar ninguna configuración, sino leerla de las variables de entorno**.

Cuando se inicia el contenedor (gestionado por ECS/Kubernetes), lo ejecutaríamos de la siguiente manera:
- En el entorno de Desarrollo: `docker run -e DATABASE_HOST=dev.db.internal ... my-app`
- En el entorno de Producción: `docker run -e DATABASE_HOST=prod.db.internal ... my-app`

Para configuraciones más complejas (como un `settings.json` completo), el sistema de orquestación de contenedores puede "montar" el archivo de configuración para el entorno correspondiente en una ruta específica dentro del contenedor cuando se inicia (por ejemplo, `/etc/config/settings.json`). Nuestra aplicación solo necesita leer el archivo de esta ruta fija.

Finalmente, hay una forma más avanzada y segura: **`Obtener de un Servicio de Configuración al iniciar`**. Cuando se inicia el contenedor, la aplicación utiliza su identidad asignada (por ejemplo, un Rol de Tarea de AWS ECS) para llamar a un servicio externo (como AWS Secrets Manager o Parameter Store) para obtener la contraseña de la base de datos, las claves de API, etc., que necesita. De esta manera, incluso las variables de entorno no aparecen en texto plano, lo que representa el nivel más alto de seguridad.

## 3. Gestión de Clústeres de Amazon ECS

Ya hemos construido cocinas estandarizadas (IaC) y diseñado "kits de comidas semi-acabadas" estandarizados (Imágenes de Docker). Ahora, necesitamos un sistema de "jefe de cocina" inteligente y eficiente para gestionar toda la operación de back-end. Este sistema necesita saber:
- ¿Cuál es el procedimiento de cocción estándar para un plato (nuestra aplicación)?
- ¿Cuántas estufas deben estar funcionando simultáneamente durante las horas pico (alto tráfico) para hacer este plato?
- Si una estufa se estropea (fallo del servidor), ¿cómo reemplazarla automáticamente por una nueva para seguir cocinando?

Este "sistema de jefe de cocina" es **`Amazon ECS (Elastic Container Service)`**.

### 3.1 Definición de Servicio de ECS

Para entender ECS, primero debemos conocer sus 4 componentes principales: `Clúster de ECS - "la cocina en sí"`, `Definición de Tarea - "la tarjeta de Procedimiento Operativo Estándar (SOP)"`, `Tarea - "el wok en la estufa de fuego alto"` y `Servicio - "el jefe de cocina incansable"`.

1. **Clúster de ECS - "la cocina en sí"**
- **Concepto Central:** Una agrupación lógica utilizada para contener todas nuestras aplicaciones en contenedores. Podemos pensar en ello como `Encimera de Cocina de Desarrollo`, `Encimera de Cocina de Staging`, `Encimera de Cocina de Producción`. Es como decirle a la gente cuántos pies cuadrados tiene el área de trabajo de su cocina, sin entrar en el diseño real del interior. No cuesta nada en sí mismo, es solo un límite de gestión.
- **Punto Clave:** Un clúster necesita "potencia de cálculo" para funcionar, al igual que una cocina necesita estufas. Hay dos modos para la fuente de potencia de cálculo:
  - **Modo EC2:** Compramos un lote de "estufas" (servidores en la nube EC2) nosotros mismos y los registramos en nuestra cocina. Tenemos control total sobre la marca y el modelo de las estufas y podemos ajustarlas libremente, pero también somos responsables de su mantenimiento y gestión.
  - **Modo Fargate (Serverless - la tendencia principal en la industria):** No compramos estufas. Cuando necesitamos cocinar, simplemente le decimos a AWS: "Necesito una estufa con 500W de potencia y un área de 1 metro cuadrado". AWS conjurará automáticamente una estufa que cumpla con nuestros requisitos de su vasto grupo de recursos, y desaparecerá cuando hayamos terminado.
  - Para la gran mayoría de las aplicaciones nuevas, Fargate es la opción preferida. Nos libera de la tediosa gestión de servidores y nos permite centrarnos más en nuestra aplicación.

2. **Definición de Tarea - "la tarjeta de Procedimiento Operativo Estándar (SOP)"**
- **Concepto:** Este es el plano central de ECS. Es un manual de instrucciones en formato JSON que define en detalle "cómo cocinar un plato".
- **Punto Clave:** Es como una tarjeta SOP a prueba de tontos para el chef, que indica con precisión:
  - `image:` Qué versión del "kit de comida semi-acabada" usar (URL de la imagen de Docker).
  - `cpu & memory:` Cuánta "potencia de fuego" y "espacio" asignar a esta tarea.
  - `environment:` Qué "condimentos" inyectar (variables de entorno, por ejemplo, DATABASE_URL).
  - `secrets:` Qué "recetas secretas" recuperar de la "caja fuerte" (AWS Secrets Manager) (clave de API, contraseña de la base de datos).
  - `logConfiguration:` Dónde enviar los registros del proceso de cocción (registros) (por ejemplo, a CloudWatch Logs).
  - `taskRoleArn:` Qué tipo de "tarjeta de permiso" (Rol de IAM) dar a este chef, permitiéndole interactuar con otros servicios de AWS (por ejemplo, leer archivos de S3).

3. **Tarea - "el wok en la estufa de fuego alto"**
- **Concepto:** Una "Instancia en Ejecución" de una definición de tarea. Cuando ECS realmente inicia un contenedor basado en nuestra tarjeta SOP, ese contenedor en ejecución es una Tarea. Podemos decidir cuántos woks están salteando al mismo tiempo estableciendo el número de Tareas.

4. **Servicio - "el jefe de cocina incansable"**
- **Concepto:** El Servicio es el cerebro y el mayordomo de ECS. Su deber es mantener el estado que deseamos.
- **Punto Clave:** Asegurarse de que todo en la cocina se ejecute según el orden de ejecución que queremos.
  - `desiredCount: { N }`: "Quiero que siempre haya `{ N }` porciones de 'Boeuf Bourguignon' cocinándose en el fuego, ni más ni menos".
  - A través de la `Comprobación de Salud`, el jefe de cocina patrulla constantemente todas las estufas para ver si son normales. Si descubre que una de ellas ha fallado debido a una estufa defectuosa (`fallo de HC`), la descartará inmediatamente y encenderá una nueva estufa, asegurándose de que el número de estufas activas sea siempre 3.
  - **Balanceo de Carga:** Cuando se enciende una estufa (se inicia una nueva Tarea), el jefe de cocina notificará automáticamente a los camareros del frente (Application Load Balancer), diciéndoles que ahora hay una apertura de servicio adicional, y pueden enviar los pedidos de los clientes (tráfico).

**Ejemplo de Implementación**

```json
{
  "family": "app-service",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "executionRoleArn": "arn:aws:iam::account:role/ecsTaskExecutionRole",
  "taskRoleArn": "arn:aws:iam::account:role/ecsTaskRole",
  "containerDefinitions": [
    {
      "name": "app",
      "image": "your-registry/app:${ENVIRONMENT}",
      "portMappings": [
        {
          "containerPort": 3000,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {
          "name": "NODE_ENV",
          "value": "${ENVIRONMENT}"
        }
      ],
      "secrets": [
        {
          "name": "DATABASE_URL",
          "valueFrom": "arn:aws:secretsmanager:region:account:secret:${ENVIRONMENT}/database-url"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/${ENVIRONMENT}/app",
          "awslogs-region": "us-west-2",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ]
}
```

#### Práctica de ECS en Multi-Entorno

Después de explicar brevemente el significado de ECS, ¿cómo se manifiesta nuestra estrategia de `Encimera de Cocina de Desarrollo`, `Encimera de Cocina de Staging`, `Encimera de Cocina de Producción` a nivel de ECS?

- **Aislamiento de Clúster:** Siguiendo la estrategia de múltiples cuentas, crearemos `dev-cluster` en `Account-Dev` y `prod-cluster` en `Account-Prod`. Este es el aislamiento más completo.
- **Gestión de Tarjetas SOP (Definición de Tareas):** Seguimos utilizando IaC (Terraform) para gestionar las definiciones de tareas. Tendremos una plantilla `task-definition.tf` compartida y pasaremos diferentes parámetros para diferentes entornos.
  - `dev.tfvars`: `cpu = 256 (1/4 vCPU)`, `memory = 512 (MB)`, `API_KEY_SECRET_ARN = "arn:aws:secretsmanager:ap-northeast-1:DEV_ACCOUNT_ID..."
  - `prod.tfvars`: `cpu = 1024 (1 vCPU)`, `memory = 2048 (MB)`, `API_KEY_SECRET_ARN = "arn:aws:secretsmanager:ap-northeast-1:PROD_ACCOUNT_ID..."
  - Tenga en cuenta que nuestro parámetro `image` apuntará a la **misma Imagen de Docker**, pero la `CPU`, la `memoria` y los `Secretos` inyectados seguirán proviniendo de diferentes configuraciones de entorno, manteniendo nuestra **`Consistencia`** e **`Inmutabilidad`**.
- **Gestión de Comandos del Jefe de Cocina (Servicio):** También se define mediante IaC.
  - El Servicio del entorno de `desarrollo` podría tener un `desiredCount` de solo `1`, y el auto-scaling podría no estar configurado para ahorrar costos.
  - El Servicio del entorno de `producción` podría comenzar con un `desiredCount` de `3` y se configurará con la estrategia de auto-scaling que discutiremos a continuación.

### 3.2 Configuración de Auto-Scaling de ECS

Nuestro restaurante no puede depender solo de un número fijo de 3 chefs. A veces es predecible, como en una tarde de martes, uno podría ser suficiente, pero en una noche de viernes, podríamos necesitar 10 para manejar el flujo interminable de pedidos. A veces, la multitud podría aumentar repentinamente, requiriendo 20 chefs para manejarla.

Agregar o quitar chefs manualmente en este punto es demasiado lento y poco práctico. Por lo tanto, necesitamos configurar un **`"Sistema de Vinculación de Monitoreo de Estufas y Flujo de Clientes" (Auto-Scaling de Servicio de ECS)`** que nos permita aumentar o disminuir automáticamente el personal en función del "flujo de clientes" para optimizar nuestros costos.

- **Idea Central:** Monitorear una métrica y ajustar automáticamente el `desiredCount` del Servicio en función de los cambios en esta métrica.
- **Métricas Más Comunes:**
  - **Utilización de CPU:** "Si el nivel de negocio promedio de los chefs supera el 70%, agregue más personal". Esta es la métrica más común.
  - **Utilización de Memoria:** "Si la ocupación promedio del espacio en la cocina supera el 75%, expanda la cocina (agregue más Tareas)".
  - **Recuento de Solicitudes de ALB por Objetivo:** Esta es la métrica que mejor refleja la presión real del usuario. "Si el número promedio de pedidos que cada apertura de servicio tiene que manejar por minuto supera los 500, agregue inmediatamente más aperturas de servicio".
- **Establecer Umbral:** Quiero que esta métrica `ALBRequestCountPerTarget` se mantenga alrededor de 500.
- **Establecer Límites:**
  - `min_capacity = 2`: Pase lo que pase, debe haber al menos 2 chefs en espera en la cocina.
  - `max_capacity = 20`: El máximo es 20. Más, y mis costos podrían explotar, o la base de datos de back-end no podría manejarlo.

El flujo operativo es aproximadamente el siguiente:
- **Scale Out:** Cuando el tráfico aumenta a las 7 PM de un viernes por la noche, el recuento promedio de solicitudes se dispara a 800. Auto Scaling lo detecta e inmediatamente aumenta el `desiredCount` del Servicio (por ejemplo, de 2->3, 3->5...). ECS luego inicia nuevas Tareas. Las nuevas Tareas se registran automáticamente en el ALB, compartiendo el tráfico hasta que el recuento promedio de solicitudes vuelva a bajar a alrededor de 500.
- **Scale In:** A las 10 PM, a medida que los clientes se van gradualmente, el recuento promedio de solicitudes cae a 100. Auto Scaling lo detecta y reduce gradualmente el `desiredCount`. ECS terminará con gracia las Tareas en exceso, liberando los recursos y ahorrándonos costos.
- **Programación (Período):** Si ya sabemos que hay un **aumento de tráfico fijo** todos los viernes a las 7 PM y que los clientes se han ido en su mayoría a las **11 PM**, podemos establecer un número fijo de Tareas `{ N }` para el período de `19:00-23:00` para manejar el tráfico. Esto puede reducir aún más los costos: el monitoreo también cuesta dinero.

```yaml
# ecs-auto-scaling.yml
Resources:
  ServiceScalingTarget:
    Type: AWS::ApplicationAutoScaling::ScalableTarget
    Properties:
      MaxCapacity: !Ref MaxCapacity
      MinCapacity: !Ref MinCapacity
      ResourceId: !Sub "service/${ClusterName}/${ServiceName}"
      RoleARN: !Sub "arn:aws:iam::${AWS::AccountId}:role/aws-service-role/ecs.application-autoscaling.amazonaws.com/AWSServiceRoleForApplicationAutoScaling_ECSService"
      ScalableDimension: ecs:service:DesiredCount
      ServiceNamespace: ecs

  ServiceScalingPolicy:
    Type: AWS::ApplicationAutoScaling::ScalingPolicy
    Properties:
      PolicyName: !Sub "${ServiceName}-scaling-policy"
      PolicyType: TargetTrackingScaling
      ScalingTargetId: !Ref ServiceScalingTarget
      TargetTrackingScalingPolicyConfiguration:
        PredefinedMetricSpecification:
          PredefinedMetricType: ECSServiceAverageCPUUtilization
        TargetValue: 70.0
```

## 4. Gestión de Amazon EKS y Kubernetes

Los recursos humanos a veces son limitados y las papilas gustativas pueden cansarse. A veces, abrir más **restaurantes Michelin** para atraer a más gente no es necesariamente una mejor opción. Quizás es una cuestión de gusto, o quizás es una cuestión de presupuesto. Cuando nuestro servicio expande su lógica de negocio, agregar continuamente nuevos platos al mismo restaurante aumentará gradualmente los costos y la complejidad de la gestión. Al mismo tiempo, también es posible que después de abrir una nueva tienda, todos apunten a un plato específico (como la hamburguesa con queso de un chef Michelin y un restaurante que explota automáticamente después de comer), y otros servicios no se utilicen en absoluto; esto no es culpa del proveedor, es sin duda un desperdicio de costos.

Si nuestro próximo objetivo es abrir un grupo de restaurantes con docenas de marcas diferentes (microservicios) para diferentes grupos de clientes, y posiblemente incluso abrir sucursales en centros comerciales de Google o Microsoft (multi-nube), entonces necesitamos un "sistema operativo de grupo" estandarizado y estándar de la industria. Este sistema es Kubernetes (a menudo abreviado como K8s).

Amazon EKS (Elastic Kubernetes Service) es el servicio proporcionado por AWS que nos ayuda a gestionar la "oficina de gestión de la sede del grupo" (plano de control) más central y que más consume recursos de este complejo sistema operativo.

### 4.1 Configuración de Clúster de EKS — Construyendo Nuestro Patio de Comidas

Si un Clúster de ECS es la cocina trasera de un restaurante independiente, entonces un Clúster de EKS es el lugar y la infraestructura básica de todo el patio de comidas. Las partes más importantes son la **"Oficina de Gestión del Patio de Comidas" (Plano de Control)** y los **"Espacios para Puestos de Comida" (Plano de Datos / Nodos de Trabajo)**.

- **Plano de Control - "Oficina de Gestión del Patio de Comidas"**
  - **Concepto:** Este es el cerebro de Kubernetes. Es responsable de recibir instrucciones, programar las ubicaciones de apertura de todos los puestos (aplicaciones), monitorear el estado operativo de todos los puestos, almacenar el diseño de toda la plaza, etc. Esta parte es extremadamente compleja y crucial, y debe estar altamente disponible las 24 horas del día, los 7 días de la semana.
  - **Valor de EKS:** No tenemos que construir esta oficina de gestión nosotros mismos, ni tenemos que contratar guardias de seguridad o personal de TI para mantenerla. AWS EKS nos proporciona una oficina de gestión altamente disponible, segura y actualizada automáticamente. Esta es la razón principal por la que pagamos por EKS. Solo necesitamos centrarnos en operar nuestros puestos de comida.

- **Plano de Datos / Nodos de Trabajo - "Espacios para Puestos de Comida"**
  - **Concepto:** Aquí es donde ocurre la "cocina" real, es decir, los servidores donde se ejecutan realmente los contenedores de nuestra aplicación.
  - En EKS, hay principalmente dos formas:
    - **Grupos de Nodos Administrados de EC2:** Similar al modo EC2 de ECS. Alquilamos una fila de "puestos estándar" (instancias EC2) de la administración de la plaza. Podemos elegir las especificaciones de los puestos (por ejemplo, `m5.large`), pero la energía, la red, la protección contra incendios, etc., de los puestos son administrados por AWS, como actualizaciones y reemplazos automatizados de nodos. Este es el modo más común.
    - **Perfiles de Fargate:** Igual que el modo Fargate de ECS. Ni siquiera tenemos que alquilar un puesto. Simplemente le decimos a la oficina de administración: "Quiero abrir un puesto de takoyaki temporal y necesito un espacio con 1vCPU y 2GB de memoria". EKS creará automáticamente un espacio que cumpla con nuestros requisitos en un rincón de la plaza y se lo llevará cuando terminemos. Es muy adecuado para aplicaciones sin estado o escenarios que necesitan hacer frente a aumentos repentinos de tráfico.

**Estrategia Multi-entorno:** El principio sigue siendo el mismo. Crearemos un `dev-eks-cluster` en `Account-Dev` y un `prod-eks-cluster` en `Account-Prod`. IaC (Terraform) sigue siendo nuestra herramienta para estandarizar la definición de estas configuraciones de clúster.

```yaml
# eks-cluster.yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: app-cluster-${ENVIRONMENT}
  region: us-west-2
  version: "1.28"

nodeGroups:
  - name: worker-nodes
    instanceType: ${INSTANCE_TYPE}
    minSize: ${MIN_SIZE}
    maxSize: ${MAX_SIZE}
    desiredCapacity: ${DESIRED_SIZE}
    privateNetworking: true
    labels:
      environment: ${ENVIRONMENT}
    tags:
      Environment: ${ENVIRONMENT}
      Project: "multi-env-demo"

addons:
  - name: vpc-cni
  - name: coredns
  - name: kube-proxy
  - name: aws-load-balancer-controller
```

### 4.2 Despliegue de Aplicaciones de Kubernetes

En el mundo de Kubernetes, existe un vocabulario estándar y un conjunto de documentos (archivos YAML) para describir "cómo operar un puesto". Esto es más detallado y potente que la Definición de Tareas de ECS.

1. **Pod - "Una Estación de Trabajo de Cocina Independiente"**
   - **Concepto:** La unidad desplegable más pequeña en Kubernetes. Puede contener uno o más contenedores estrechamente acoplados.
   - **Analogía:** Un Pod no es solo la olla (Contenedor); también incluye un pequeño banco de trabajo al lado de la olla, un fregadero y una toma de corriente independientes (dirección IP de red independiente) y un pequeño refrigerador dedicado (Volumen de almacenamiento). Los contenedores dentro de un Pod pueden compartir este pequeño entorno.
   - **Característica Clave:** Los pods son efímeros; pueden ser destruidos y recreados en cualquier momento. Nunca debemos depender directamente de la existencia de un Pod específico.

2. **Deployment - "Manual de Franquicia"**
   - **Concepto:** Un Deployment es un controlador de nivel superior cuya responsabilidad principal es garantizar que se ejecute un número específico de réplicas de Pod idénticas.
   - **Analogía:** Somos los dueños de la marca "Pollo Frito A+". Nuestro YAML de Deployment es nuestro manual de franquicia, que establece:
     - `replicas: 3`: Requiero que siempre haya 3 estaciones de trabajo de cocina "Pollo Frito A+" (Pods) en funcionamiento en el patio de comidas.
     - `template`: El manual detalla las especificaciones de una estación de trabajo estándar (qué imagen de Docker usar, cuánta CPU/memoria se necesita, etc.).
     - **Auto-reparación:** Si el Deployment descubre que solo hay 2 estaciones de trabajo en funcionamiento, creará inmediatamente una tercera.
     - **Actualización Continua:** Cuando queremos lanzar un nuevo sabor (actualizar la Imagen de Docker), el Deployment será muy inteligente: primero abrirá una nueva estación de trabajo -> esperará a que la nueva estación esté lista -> luego cerrará una antigua. Este reemplazo gradual garantiza que nuestro servicio de puesto no se interrumpa.

3. **Service - "Número de Llamada y Línea Interna del Puesto"**
   - **Concepto:** Los pods se crean y destruyen constantemente, y sus direcciones IP siempre están cambiando. Un Servicio proporciona un punto final de red interno estable e inmutable para representar un grupo de Pods funcionalmente idénticos.
   - **Analogía:** Nuestras 3 estaciones de trabajo de pollo frito (Pods) pueden estar en diferentes lugares de la plaza y pueden cambiar de posición en cualquier momento. Es imposible que los clientes las persigan. Por lo tanto, registramos un número de puesto fijo "B-52" (Servicio) en el mostrador de servicio del patio de comidas.
   - **Cómo funciona:** Cuando otros puestos en el patio de comidas (por ejemplo, el Pod "Buenas Bebidas") quieren pedir pollo frito, no necesitan saber la ubicación real de ningún Pod de pollo frito. Solo necesitan llamar a la línea interna "B-52", y el sistema de red de Kubernetes reenviará automáticamente la solicitud a un Pod de pollo frito actualmente saludable. Esto garantiza la estabilidad de la comunicación entre los servicios y es la piedra angular de una arquitectura de microservicios.

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
  namespace: ${ENVIRONMENT}
  labels:
    app: demo-app
    environment: ${ENVIRONMENT}
spec:
  replicas: ${REPLICA_COUNT}
  selector:
    matchLabels:
      app: demo-app
  template:
    metadata:
      labels:
        app: demo-app
        environment: ${ENVIRONMENT}
    spec:
      containers:
        - name: app
          image: your-registry/app:${IMAGE_TAG}
          ports:
            - containerPort: 3000
          env:
            - name: NODE_ENV
              value: ${ENVIRONMENT}
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: database-secret
                  key: url
          resources:
            requests:
              memory: "128Mi"
              cpu: "100m"
            limits:
              memory: "256Mi"
              cpu: "200m"
          livenessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /ready
              port: 3000
            initialDelaySeconds: 5
            periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: app-service
  namespace: ${ENVIRONMENT}
spec:
  selector:
    app: demo-app
  ports:
    - port: 80
      targetPort: 3000
  type: ClusterIP
```

### 4.3 Configuración de Ingress y Balanceador de Carga

Ahora que los puestos están abiertos y el personal interno puede hacer pedidos entre sí, la pregunta más importante es: ¿cómo entran los clientes externos y encuentran nuestros puestos?

Un método simple y burdo es darle a cada puesto una entrada directa al exterior (tipo de Servicio LoadBalancer). Esto crearía un Balanceador de Carga de AWS independiente para cada servicio, lo que sería increíblemente costoso con docenas de servicios.

Un enfoque más elegante y económico es usar **`Ingress`**. `Ingress` es un objeto de API que gestiona el tráfico externo que ingresa al clúster. Proporciona reglas de enrutamiento basadas en HTTP/HTTPS. Es como si todo el patio de comidas tuviera una sola entrada principal, y en la entrada, hay un enorme tablero de directorio electrónico que dice:
- Para ir a "Pollo Frito A+" (`host: food.com, path: /chicken` ) -> Por favor, vaya al Área B, No. 52 (dirige a `chicken-service`)
- Para ir a "Pizzería C" (`host: food.com, path: /pizza` ) -> Por favor, vaya al Área C, No. 03 (dirige a `pizza-service`)

Los dos componentes clave que conforman nuestro sistema de entrada (Ingress) son:
1. **Controlador de Ingress - "El Guardia y Guía en la Entrada"**: Es un programa que se ejecuta realmente en el clúster (él mismo también es un Pod). Es responsable de escuchar las reglas de Ingress que definimos y de configurar realmente el Balanceador de Carga de Aplicaciones (ALB) de AWS en la entrada principal. En un entorno de EKS, el más utilizado es el Controlador de Balanceador de Carga de AWS.
2. **Recurso de Ingress - "Las Reglas del Mapa del Directorio que Escribimos"**: Este es un archivo YAML. Enviamos este archivo a Kubernetes para decirle al Controlador de Ingress las reglas de enrutamiento que queremos.

La gran ventaja de Ingress es que podemos usar un ALB como punto de entrada de tráfico para docenas o incluso cientos de servicios de back-end, y enrutar con precisión el tráfico al Servicio correspondiente a través de diferentes nombres de dominio o rutas de URL. Esto ahorra enormemente los costos y simplifica la gestión del tráfico externo.

```yaml
# k8s/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  namespace: ${ENVIRONMENT}
  annotations:
    kubernetes.io/ingress.class: "alb"
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/certificate-arn: ${CERTIFICATE_ARN}
    alb.ingress.kubernetes.io/ssl-redirect: "443"
spec:
  rules:
    - host: ${ENVIRONMENT}.example.com
      http:
        paths:
          - path: / 
            pathType: Prefix
            backend:
              service:
                name: app-service
                port:
                  number: 80
```

### ECS vs. EKS

Hoy (?), desde una perspectiva macro de la construcción de clústeres hasta la perspectiva micro del despliegue de aplicaciones y la gestión del ingreso de tráfico, hemos recorrido por completo el camino central de Kubernetes. Aunque este sistema es complejo, es el esqueleto que soporta las aplicaciones de microservicios a gran escala de hoy en día.

Finalmente, comparemos brevemente las diferencias entre los dos:
> ### ECS es un restaurante de alta cocina bien organizado:
> AWS ha definido muchas reglas para nosotros. Solo necesitamos centrarnos en los platos (nuestra aplicación). Tiene una alta integración y una curva de aprendizaje suave.

> ### EKS/Kubernetes es un vibrante patio de comidas internacional:
> Proporciona un conjunto de "estándares operativos" extremadamente potentes, flexibles y neutrales al proveedor. Necesitamos aprender todo su conjunto de términos y conceptos (Pod, Deployment, Service, Ingress...), pero el retorno es una flexibilidad, escalabilidad y un ecosistema vibrante sin igual.

## 5. Integración de Pipeline CI/CD

Dado que ya hemos hablado sobre los conceptos de integración y proceso de CI/CD en <Implementación Completa de Automatización CI/CD - GitHub Actions × CodePipeline × CodeBuild>, ahora describiremos brevemente el proceso y organizaremos los trabajos fragmentados, y luego pasaremos a la sección de **`Despliegue Azul/Verde`**, que combina las aplicaciones multi-entorno de <Infraestructura como Código: Codificación y Versionado de Infraestructura basados en Terraform> e <Implementación Completa de Automatización CI/CD - GitHub Actions × CodePipeline × CodeBuild>.

### 5.1 Flujo de Trabajo de GitHub Actions

Para repasar, el concepto más importante del proceso de CI/CD es `establecer un proceso de verificación y entrega de lógica de negocio estable y fijo,` **`protegiendo activamente la lógica de negocio existente de ser corrompida por cambios en el código.`**

Cada vez que se envía código al repositorio de código (por ejemplo, GitHub), el pipeline se inicia automáticamente. Obtiene el código más reciente y ejecuta una serie de "verificaciones de calidad" como compilación, pruebas unitarias y escaneos de calidad de código. Esto es como la parte de "control de calidad" de una fábrica de alimentos. Cada vez que llegan nuevas materias primas, el inspector de calidad toma muestras inmediatamente para asegurarse de que este lote de materiales está bien y no contaminará toda la línea de producción: el objetivo de la CI es **descubrir problemas lo antes posible**. Cuando se pasan todos los pasos de control de calidad de la CI, el pipeline pasa al siguiente paso: empaquetado (creación de una imagen de Docker), envío al repositorio (ECR) y luego activación automática del despliegue en los entornos de Desarrollo -> Staging -> Producción. El objetivo de la CD es hacer de los lanzamientos un evento **rutinario**, **estable** y **rastreable**, al igual que las materias primas aprobadas por calidad se envían automáticamente a la línea de producción, se procesan, se cocinan, se envasan al vacío y, finalmente, se entregan directamente a los camiones mediante una cinta transportadora para su distribución a las principales tiendas.

```yaml
# .github/workflows/deploy.yml
name: Despliegue Multi-Entorno

on:
  push:
    branches: [release, develop]
  pull_request:
    branches: [release]

env:
  AWS_REGION: us-west-2
  ECR_REPOSITORY: demo-app

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      image-tag: ${{ steps.build.outputs.image-tag }}
    steps:
      - uses: actions/checkout@v3

      - name: Configurar credenciales de AWS
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Iniciar sesión en Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v1

      - name: Construir y enviar imagen de Docker
        id: build
        run: |
          IMAGE_TAG=${GITHUB_SHA:0:8}
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          echo "image-tag=$IMAGE_TAG" >> $GITHUB_OUTPUT

  deploy-dev:
    if: github.ref == 'refs/heads/develop'
    needs: build
    runs-on: ubuntu-latest
    environment: development
    steps:
      - name: Desplegar en Desarrollo
        run: |
          # Despliegue de ECS
          aws ecs update-service \
            --cluster dev-cluster \
            --service app-service \
            --task-definition app-service:LATEST

  deploy-staging:
    if: github.ref == 'refs/heads/main'
    needs: build
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - name: Desplegar en Staging
        run: |
          # Despliegue de Kubernetes
          kubectl set image deployment/app-deployment \
            app=$ECR_REGISTRY/$ECR_REPOSITORY:${{ needs.build.outputs.image-tag }} \
            -n staging

  deploy-prod:
    if: github.ref == 'refs/heads/main'
    needs: [build, deploy-staging]
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Desplegar en Producción
        run: |
          # Despliegue Azul-Verde
          ./scripts/blue-green-deploy.sh ${{ needs.build.outputs.image-tag }}
```

Y de acuerdo con lo que dijimos en <Implementación Completa de Automatización CI/CD - GitHub Actions × CodePipeline × CodeBuild>, <Modularización de Trabajos a Nivel Empresarial y Referencia Cruzada de Dominios>, cuando necesitamos desplegar, podemos extraer procesos comunes (Trabajos), canalizarlos para su gestión y realizar un despliegue automatizado basado en el comportamiento de la rama actual.

```yaml
# build
parameters:
  - name: ngCliVersion
    type: string
    default: latest
    values:
      - latest
      - 16
      - 12
      - 11
jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      image-tag: ${{ steps.build.outputs.image-tag }}
    steps:
      - uses: actions/checkout@v3

      - name: Configurar credenciales de AWS
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Iniciar sesión en Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v1

      - name: Construir y enviar imagen de Docker
        id: build
        run: |
          IMAGE_TAG=${GITHUB_SHA:0:8}
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          echo "image-tag=$IMAGE_TAG" >> $GITHUB_OUTPUT
```

```yaml
#  deploy-dev
parameters:
  - name: ngCliVersion
    type: string
    default: latest
    values:
      - latest
      - 16
      - 12
      - 11
jobs:
  deploy-dev:
    if: github.ref == 'refs/heads/develop'
    needs: build
    runs-on: ubuntu-latest
    environment: development
    steps:
      - name: Desplegar en Desarrollo
        run: |
          # Despliegue de ECS
          aws ecs update-service \
            --cluster dev-cluster \
            --service app-service \
            --task-definition app-service:LATEST
```

```yaml
#  deploy-staging
parameters:
  - name: ngCliVersion
    type: string
    default: latest
    values:
      - latest
      - 16
      - 12
      - 11

jobs:
  deploy-staging:
    if: github.ref == 'refs/heads/main'
    needs: build
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - name: Desplegar en Staging
        run: |
          # Despliegue de Kubernetes
          kubectl set image deployment/app-deployment \
            app=$ECR_REGISTRY/$ECR_REPOSITORY:${{ needs.build.outputs.image-tag }} \
            -n staging
```

```yaml
#  deploy-prod
parameters:
  - name: ngCliVersion
    type: string
    default: latest
    values:
      - latest
      - 16
      - 12
      - 11

jobs:
  deploy-prod:
    if: github.ref == 'refs/heads/main'
    needs: [build, deploy-staging]
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Desplegar en Producción
        run: |
          # Despliegue Azul-Verde
          ./scripts/blue-green-deploy.sh ${{ needs.build.outputs.image-tag }}
```

El proceso extraído se vería así:
```yaml
# .github/workflows/deploy.yml
name: Despliegue Multi-Entorno

on:
  push:
    branches: [release, develop]
  pull_request:
    branches: [release]

env:
  AWS_REGION: us-west-2
  ECR_REPOSITORY: demo-app

jobs:
  - template: jobs/build.yml@templates
  - template: jobs/deploy-dev.yml@templates
  - template: jobs/deploy-staging.yml@templates
    parameters:
      Version: ${{ variables.Version }}
      env: $(env)
      hasPreview: true
```

### 5.2 Despliegue Azul/Verde: Instancias Monolíticas (ECS/EC2) y Clústeres (EKS)

#### Despliegue Azul/Verde para Instancias Monolíticas (ECS/EC2)
Después de revisar brevemente el proceso de CI/CD y la división independiente de `Trabajos`, entendamos la aplicación práctica del despliegue multi-entorno - **`Despliegue Azul/Verde`**.

Hay dos estrategias principales en el desarrollo de sistemas actual. Una es la **Actualización Continua**, que se despliega después de que `el tráfico se reduce hasta que el servicio está completamente frío`, y la otra es una estrategia de lanzamiento de `cero tiempo de inactividad` - **Despliegue Azul/Verde**. Una actualización continua es como en una carrera de F1, donde las `reparaciones y actualizaciones rápidas` solo se permiten cuando el coche entra en el pit stop y está completamente parado (`tráfico reducido hasta que el servicio está completamente frío`). En el pit stop, tienes que correr contra el cronómetro. Cualquier error retrasará el tiempo para salir del pit stop. Peor aún, si vuelves a la pista y encuentras un problema con el nuevo neumático, el coche ya está en un estado inestable y tienes que volver a entrar en el pit stop para hacer ajustes; una situación aún peor es tener que **`realizar reparaciones de emergencia en la pista a alta velocidad`**.

El **`Despliegue Azul/Verde`**, por otro lado, ofrece una filosofía más segura y elegante. No trabajamos en el coche de carreras en movimiento (`entorno Azul`). En la pista lateral, preparamos un coche nuevo idéntico con neumáticos nuevos instalados (`entorno Verde`). Dejamos que este coche nuevo arranque, se caliente y comprobamos todos los salpicaderos. Cuando estamos 100% seguros de que el coche nuevo no tiene fallos (por supuesto, esta es la situación ideal, jaja), cambiamos gradualmente la presión al coche nuevo. El coche viejo se va retirando gradualmente. Si hay un problema con el coche nuevo, podemos **`volver al instante`**.

- **Pasos de Ejecución:**
  1. **Estado Inicial:** Supongamos que nuestro entorno de producción actual es V1, al que llamamos entorno azul. Todo el tráfico de usuarios se dirige al Grupo de Destino del entorno azul a través del Application Load Balancer (ALB).
  2. **Desplegar Entorno Verde:**
     - El pipeline de CI/CD comienza.
     - Creará un conjunto de infraestructura completamente nuevo e independiente (por ejemplo, un nuevo Servicio de ECS o un Despliegue de K8s) y desplegará la versión V2 del software en él. Este es el entorno verde. En este punto, el entorno verde está en funcionamiento pero no tiene tráfico de usuarios real.
  3. **Verificar Entorno Verde:**
     - Esta es la ventaja más crítica del despliegue azul-verde. Antes de cambiar el tráfico, podemos realizar pruebas automatizadas completas y aisladas en el entorno verde. Por ejemplo, enviar solicitudes simuladas desde dentro del pipeline al punto final interno del entorno verde para verificar que sus funciones principales son normales y que la API devuelve correctamente.
  4. **Cambio de Tráfico (El Interruptor):**
     - Cuando todas las pruebas hayan pasado (generalmente con un paso de "aprobación manual"), el pipeline de CI/CD ejecutará una única operación atómica: modificar las reglas del Listener del ALB para redirigir un **porcentaje personalizado** (dependiendo de las necesidades del negocio, pero el 20% es una proporción común) de tráfico del Grupo de Destino azul al Grupo de Destino verde. Este cambio es instantáneo para el usuario; no sentirán ninguna interrupción del servicio.
  5. **En Espera y Desmantelamiento:**
     - Después del cambio, el entorno verde toma el control gradualmente y se convierte oficialmente en el nuevo entorno de producción.
     - El antiguo entorno azul no se destruye de inmediato. Permanece en espera como una "copia de seguridad en caliente". Si el sistema de monitoreo detecta un problema grave con la versión V2 unos minutos u horas después de la puesta en marcha, podemos realizar una "reversión con un solo clic", es decir, realizar otro cambio de tráfico para dirigir el tráfico de vuelta al entorno azul, logrando una recuperación ante desastres de segundo nivel.
     - Después de confirmar que la nueva versión ha estado funcionando de forma estable durante un período de tiempo, el pipeline de CI/CD realizará el trabajo de limpieza final, destruyendo el entorno azul para ahorrar costos.

AWS CodeDeploy está profundamente integrado con ECS y EC2 y puede automatizar el complejo proceso de despliegue azul-verde descrito anteriormente. Solo necesitamos hacer una configuración simple.

El modelo de división del trabajo y colaboración de **`"gestionar el estado del entorno con IaC, gestionar el ciclo de vida del despliegue con CI/CD y verificar gradualmente las diferencias del entorno"`** que acabamos de discutir es actualmente la **Implementación Óptima** y el **Estándar de Oro** más comentados y reconocidos en el campo Cloud-Native.

"Óptimo" no significa que sea la solución "más simple" o "más rápida de construir", sino que logra el equilibrio más elegante entre varios objetivos de ingeniería clave, a menudo en conflicto. Primero, equilibra **"Estabilidad"** y **"Agilidad."** En el pasado, para hacer un sistema estable significaba reducir los cambios, con ciclos de despliegue de posiblemente varios meses. Querer iterar rápidamente (ágil) a menudo sacrificaba la estabilidad, lo que provocaba frecuentes errores en línea. El enfoque moderno nativo de la nube utiliza IaC para garantizar la piedra angular de la "estabilidad". Nuestra infraestructura ya no es una caja negra. Cada cambio tiene un registro, es revisado (Revisión de Código) y es rastreable. Esto elimina la mayoría de los problemas de estabilidad causados por "errores manuales" o "entornos inconsistentes": la infraestructura se vuelve tan sólida como una roca. CI/CD proporciona el motor para la "agilidad". Sobre esta base sólida, el pipeline automatizado hace que el despliegue de aplicaciones sea extremadamente rápido y de bajo riesgo. Los desarrolladores pueden desplegar docenas de veces al día sin preocuparse por bloquear la infraestructura subyacente. Estrategias como el despliegue azul-verde proporcionan una red de seguridad de alto nivel para esta iteración de alta velocidad.

En segundo lugar, a menudo hay un abismo entre el equipo de requisitos (Require) y el equipo de operaciones (Maintain). Los requirentes quieren entregar rápidamente nuevas características, mientras que los mantenedores quieren estabilizar las funciones existentes. La culpa mutua y la ineficiencia son las normas. Pero `"gestionar el estado del entorno con IaC, gestionar el ciclo de vida del despliegue con CI/CD y verificar gradualmente las diferencias del entorno"` logra un equilibrio entre **"Responsabilidades Claras"** y **"Colaboración en Equipo."** A través de un Límite Claro, nuestro modelo de división del trabajo proporciona un "contrato" extremadamente claro:
- **Equipo de Plataforma/Operaciones:** Su producto principal es una plataforma de infraestructura estable, confiable y estandarizada (definida por Terraform). Son como urbanistas, responsables de construir tuberías de agua, electricidad, gas y cimientos de edificios estandarizados.
- **Equipo de Aplicación/Desarrollo:** Su producto principal son aplicaciones de alta calidad (encapsuladas en Docker) y un proceso de despliegue seguro y confiable (definido por el Flujo de Trabajo de GitHub Actions). Son como equipos de construcción que construyen casas sobre cimientos estándar, solo necesitan centrarse en la casa en sí sin preocuparse de dónde vienen el agua y la electricidad.

Una vez que el equipo de la plataforma proporciona una plataforma estable, el equipo de la aplicación puede realizar "despliegues de autoservicio" a través del pipeline de CI/CD sin tener que molestar al equipo de la plataforma cada vez, lo que mejora en gran medida la autonomía y la eficiencia del desarrollo.

Finalmente, se logra un equilibrio entre **"Controlabilidad"** y **"Complejidad."** Los sistemas modernos en la nube son realmente muy complejos, a menudo involucran cientos de recursos y configuraciones. Aunque hay una curva de aprendizaje, IaC es la única forma efectiva de gestionar esta complejidad. Transforma vastos recursos en la nube en texto estructurado legible por humanos y ejecutable por máquinas. Podemos revisar, probar y modularizar nuestra infraestructura como cualquier software. Al mismo tiempo, CI/CD oculta la complejidad. Para la mayoría de los desarrolladores, no necesitan comprender los complejos detalles del cambio de reglas del ALB detrás del despliegue azul-verde. Solo necesitan saber que "cuando fusiono el código en la rama principal y hago clic en el botón 'aprobar' en el Pipeline, mi nueva característica se desplegará de forma segura en línea": el pipeline de CI/CD encapsula el complejo proceso de despliegue en un proceso automatizado simple y confiable.

#### Despliegue Azul/Verde para Clústeres (EKS)
Ahora que conocemos la "filosofía" del despliegue azul-verde y su aplicación a nivel de ECS y EC2, veamos cómo Kubernetes, el "gerente del patio de comidas", utiliza sus herramientas para realizar esta precisa operación de cambio de tráfico.

A diferencia de ECS, Kubernetes no tiene un recurso pre-hecho llamado `blue-green-deployment`. Sin embargo, proporciona una serie de componentes básicos más flexibles y potentes (que llamamos primitivas) que nos permiten construir procesos de despliegue azul-verde más flexibles que AWS CodeDeploy, como ensamblar Legos. Nos centraremos en los dos métodos de implementación más convencionales y que mejor encarnan la filosofía de diseño de Kubernetes en la industria: **`cambio de capa de Servicio`** y **`cambio de capa de Ingress`**.

Para repasar, una idea clave en Kubernetes es: **`Deployments/Pods (instancias de aplicación)`** son los "puestos" que se están ejecutando realmente; son **cambiables** y **efímeros**. Y **`Services/Ingresses (entradas de servicio)`** son los "números de puesto fijos" o "entradas del patio de comidas" proporcionados a los clientes internos y externos; deben ser **estables e inmutables**. La esencia del despliegue azul-verde de K8s es mantener la "entrada de servicio" sin cambios y solo cambiar la "instancia de aplicación" a la que apunta.

##### Cambio de Selector de Servicio (el enfoque más clásico de K8s)
Este es el método que mejor encarna el concepto central de Kubernetes, al modificar el selector del Servicio para cambiar instantáneamente el tráfico. Según el caso del patio de comidas que usamos al discutir EKS, el "dispositivo de llamada B-52" (Servicio) del patio de comidas no se mueve. Simplemente, en su sistema de back-end, cambiamos instantáneamente al personal de servicio del "equipo de uniforme azul" al "equipo de uniforme verde".

Pasos Ilustrativos:
1. **Estado Inicial (entorno Azul en ejecución):**
   - Tenemos un `Deployment-Blue` que gestiona Pods de la versión `v1`. Todos estos Pods están etiquetados con `version: blue`.
   ```yaml
   # deployment-blue.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: myapp-blue
   spec:
     replicas: 3
     template:
       metadata:
         labels:
           app: myapp
           version: blue # Etiqueta clave
       spec:
         containers:
         - name: myapp
           image: my-app:v1
   ```
   - Tenemos un `Servicio` estable. Su `selector` solo encontrará `Pods` con la etiqueta `version: blue` y les reenviará el tráfico.
   ```yaml
   # service.yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: myapp-service # Este es el punto de entrada estable e inmutable
   spec:
     selector:
       app: myapp
       version: blue # Actualmente apunta a azul
     ports:
     - protocol: TCP
       port: 80
       targetPort: 8080
   ```
   En este punto, todo el tráfico fluye a los Pods `v1`.

2. **Desplegar Entorno Verde:**
   - El pipeline de CI/CD creará un `Deployment-Green` completamente nuevo que gestiona `Pods` de la versión `v2`, y estos `Pods` tienen la etiqueta `version: green`.
   ```yaml
   # deployment-green.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: myapp-green
   spec:
     replicas: 3
     template:
       metadata:
         labels:
           app: myapp
           version: green # Etiqueta clave
       spec:
         containers:
         - name: myapp
           image: my-app:v2
   ```
   - Una vez completado el despliegue, los Pods v2 se ejecutan de forma saludable en el clúster, pero como el selector de `myapp-service` sigue siendo `version: blue`, el entorno verde no recibe ningún tráfico oficial.

3. **Verificar Entorno Verde:**
   - En este momento, podemos crear un Servicio de prueba solo interno, como `myapp-green-test-service`, cuyo selector apunte a `version: green`. El pipeline de CI/CD puede utilizar la dirección de este Servicio interno para realizar pruebas automatizadas completas en el entorno verde.

4. **Ejecutar Cambio de Tráfico (El Interruptor):**
   - Cuando todo esté listo, el pipeline de CI/CD ejecutará un único comando atómico:
   ```bash
   kubectl patch service myapp-service -p '{"spec":{"selector":{"version":"green"}}}'
   ```
   - La red central de Kubernetes actualizará instantáneamente las reglas de enrutamiento. Todo el tráfico posterior al punto de entrada estable de `myapp-service` se dirigirá inmediatamente a los `Pods` con `version: green`. ¡Cambio completo!

5. **Reversión y Limpieza:**
   - Si se produce un problema con la versión `v2`, la operación de reversión es igual de simple y rápida:
   ```bash
   kubectl patch service myapp-service -p '{"spec":{"selector":{"version":"blue"}}}'
   ```
   - Después de confirmar que `v2` se está ejecutando de forma estable, el pipeline de CI/CD puede eliminar de forma segura el antiguo entorno azul:
   ```bash
   kubectl delete deployment myapp-blue
   ```

##### Cambio de Capa de Ingress (Más adecuado para microservicios y lanzamientos Canary)
Este método va un paso más allá. No toca el Servicio, sino que modifica las reglas de enrutamiento de Ingress. Esta vez, el "equipo azul" y el "equipo verde" tienen sus propios dispositivos de llamada internos independientes (`service-blue` y `service-green`). Modificamos el tablero de directorio (Ingress) en la entrada principal del patio de comidas, cambiando la dirección de "Pollo Frito A+" de "mostrador azul" a "mostrador verde".

Pasos Ilustrativos:
1. **Estado Inicial:**
   - Tenemos `deployment-blue` y `deployment-green`.
   - Hay dos `Servicios`: `service-blue` apunta a `Pods` con `version: blue`, y `service-green` apunta a `Pods` con `version: green`.
   - Hay un recurso de `Ingress` cuyas reglas apuntan a `service-blue`.
   ```yaml
   # ingress.yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: myapp-ingress
   spec:
     rules:
     - host: myapp.example.com
       http:
         paths:
         - path: / 
           pathType: Prefix
           backend:
             service:
               name: service-blue # Apunta al servicio azul
               port:
                 number: 80
   ```

2. **Cambio de Tráfico:**
   - El pipeline de CI/CD ejecuta un comando para modificar el recurso de `Ingress`, cambiando el `backend.service.name` de `service-blue` a `service-green`.
   ```bash
   # Ejemplo simple, en la práctica usarías kubectl patch o apply -f
   # Modificar el nombre del servicio en ingress.yaml y luego ejecutar apply
   kubectl apply -f ingress.yaml
   ```
   - El Controlador de Ingress (por ejemplo, el Controlador de Balanceador de Carga de AWS) detecta el cambio y actualiza las reglas de reenvío del ALB para enrutar el tráfico a `service-green`.

Este modelo es la base para implementar Lanzamientos Canary. Podemos configurar el Ingress para enviar el 95% del tráfico a `service-blue` y el 5% a `service-green` para lograr una verificación en escala de grises a pequeña escala. Además, debido a que hay menos cambios en la capa de Servicio, se ajusta mejor a su posicionamiento como un "punto de entrada estable", y las responsabilidades son más claras.

El modelo mental que hemos dominado ahora, que combina `IaC, contenedorización, CI/CD y estrategias de despliegue avanzadas`, no es solo un conjunto de instrucciones operativas técnicas; es una **filosofía de la ingeniería de software moderna**. Es el secreto central que permite a las principales empresas de tecnología como Google, Netflix y Amazon realizar miles de despliegues por semana mientras mantienen sistemas altamente estables. Porque no resuelve simplemente un problema técnico, sino un problema de ingeniería sistémica. Tiene un profundo entendimiento de que la entrega de software no se trata solo de escribir código, sino que también incluye pruebas, despliegue, operaciones, colaboración en equipo, control de riesgos y una serie de otros eslabones.

En su futura carrera, independientemente de cómo evolucionen las herramientas (de Terraform a OpenTofu, de GitHub Actions a GitLab CI), las ideas centrales detrás `separar estado y proceso` y la `colaboración declarativa e imperativa` seguirán siendo aplicables durante mucho tiempo y se convertirán en una base sólida para construir sistemas de software de alta calidad. Las herramientas evolucionarán con los tiempos, pero la filosofía de diseño es un proceso de deducción lógica y dialéctica, al igual que:
> Xunzi, "Los Efectos del Ru": "Mil movimientos y diez mil cambios, el Camino es uno."

O como prefiero, de Zhuangzi:
> Zhuangzi, "Tianxia": "No apartarse del antepasado, uno es llamado un hombre del Cielo."

## 6. Gestión de Configuración y Secretos
Finalmente, por hoy (?), hablemos de cómo gestionar los `parámetros de entorno`, las `claves públicas de autorización` y los `puertos de conexión`.

Hasta ahora, nuestro patio de comidas automatizado es muy avanzado: tenemos planos de puestos estandarizados (IaC), kits de comidas estandarizados (Docker), un gerente eficiente (ECS/EKS) y un pipeline de entrega totalmente automatizado (CI/CD).

Pero ahora, tenemos que lidiar con un tema extremadamente sensible e importante: **¿Dónde ponemos la "receta secreta exclusiva" (Secretos) de cada puesto?**

No podemos simplemente pegar un trozo de papel con la receta secreta del adobo para el "Pollo Frito A+" en la pared del `Dockerfile` o del repositorio `Git`, ¿verdad? Eso sería un desastre de seguridad. Así que a continuación, necesitamos aprender el espíritu del Sr. Cangrejo y aferrarnos a nuestros datos confidenciales, separando la "Configuración" y los "Secretos" de nuestro código de aplicación, e "inyectándolos" solo cuando sea necesario en la aplicación correcta.

**Soluciones Nativas de K8s — ConfigMap y Secret**
Kubernetes proporciona dos recursos nativos para manejar este problema.

1. **ConfigMap - "Menús y Anuncios Públicos"**
   - **Propósito:** Se utiliza para almacenar datos de configuración no sensibles.
   - **Analogía:** Imagina el tablón de anuncios en el patio de comidas. Tiene todo tipo de información pública publicada en él:
     - `API_URL: https://api.internal.mycompany.com` (la dirección de la API de back-end)
     - `FEATURE_FLAG_NEW_UI: "true"` (una bandera de característica para habilitar la nueva interfaz de usuario)
     - `LOG_LEVEL: "info"` (el nivel de detalle para el registro)
   - **Características:**
     - Almacena datos de texto sin formato en pares clave-valor.
     - Muy flexible, puede ser utilizado por los Pods de dos maneras principales:
       1. **Inyectado como Variables de Entorno:** Esta es la forma más común.
       2. **Montado como un archivo:** Cuando el contenido de la configuración es grande o tiene un formato complejo (por ejemplo, un archivo `nginx.conf`), todo el ConfigMap se puede montar en el sistema de archivos del Pod.
   - **Punto Clave:** ConfigMap es texto sin formato y nunca debe usarse para almacenar información sensible.

2. **Secret - "Receta Secreta Guardada en un Cajón Ordinario"**
   - **Propósito:** Se utiliza para almacenar datos sensibles.
   - **Analogía:** Esto es como el "cuaderno de recetas secretas" que el gerente del puesto guarda en un cajón ordinario de su oficina. Podría decir:
     - `DATABASE_PASSWORD: "mypassword123"`
     - `STRIPE_API_KEY: "sk_live_..."
   - **Características:**
     - También almacena datos en pares clave-valor.
     - La mayor diferencia con ConfigMap es que cuando Kubernetes almacena el valor de un Secret, lo codifica en Base64.
       - **¡Base64 no es cifrado!** Es solo un método de codificación. Cualquiera que obtenga la cadena codificada puede decodificarla fácilmente de nuevo al texto original.
       1. **Inyectado como Variables de Entorno:** Esta es la forma más común.
       2. **Montado como un archivo:** Cuando el contenido de la configuración es grande o tiene un formato complejo (por ejemplo, un archivo `nginx.conf`), todo el Secret se puede montar en el sistema de archivos del Pod.
   - **Punto Clave:** La seguridad de un Secret de Kubernetes se basa principalmente en RBAC (Control de Acceso Basado en Roles). Podemos establecer permisos estrictos para especificar que solo ciertos Pods o administradores tienen derecho a "leer" este objeto Secret. Protege contra los caballeros, no contra los villanos, y principalmente previene la fuga accidental de información.

```yaml
# k8s/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: ${ENVIRONMENT}
data:
  NODE_ENV: ${ENVIRONMENT}
  LOG_LEVEL: ${LOG_LEVEL}
  API_BASE_URL: ${API_BASE_URL}
---
apiVersion: v1
kind: Secret
metadata:
  name: database-secret
  namespace: ${ENVIRONMENT}
type: Opaque
data:
  url: ${DATABASE_URL_BASE64}
```

Pero para un entorno de producción serio, simplemente usar K8s Secret no es suficiente. Como acabamos de decir, `protege contra los caballeros, no contra los villanos, y principalmente previene la fuga accidental de información.` Es como guardar la receta secreta en un cajón que cualquiera puede abrir fácilmente. Todavía nos falta:
- **Cifrado en Reposo:** El secreto en la base de datos (etcd) solo está codificado, no cifrado.
- **Pista de Auditoría:** ¿Quién echó un vistazo a la receta secreta y cuándo? (¿Alguien echó un vistazo?) No tenemos forma de saberlo.
- **Rotación Automática:** Por seguridad, las contraseñas deben cambiarse regularmente. La rotación manual es tediosa y propensa a errores.

**La Caja Fuerte de Primera Categoría de AWS — Secrets Manager**
Para resolver los problemas anteriores, los proveedores de servicios en la nube ofrecen "servicios de gestión de secretos" dedicados. En AWS, este servicio es AWS Secrets Manager.

- **Propósito:** Almacenar, gestionar, auditar y rotar de forma centralizada todos sus secretos sensibles.
- **Analogía:** Esto ya no es un cajón ordinario, sino una caja fuerte de primera categoría, de grado bancario.
- **Ventajas Principales:**
  1. **Cifrado Fuerte:** Todos los secretos se cifran fuertemente utilizando AWS KMS (Key Management Service) cuando se almacenan.
  2. **Control de Permisos de IAM de Grano Fino:** Puede utilizar Políticas de IAM para especificar con precisión "qué aplicación (qué Rol de IAM) solo puede leer qué versión de qué secreto".
  3. **Pista de Auditoría Completa:** Profundamente integrado con CloudTrail. Cada lectura, modificación y eliminación de un secreto se registra en detalle para las auditorías del equipo de seguridad.
  4. **Rotación Automática:** Esta es la carta de triunfo de Secrets Manager. Se puede integrar con servicios como RDS para lograr una rotación regular totalmente automática de las contraseñas de las bases de datos, y su aplicación no necesita modificar el código ni reiniciarse.

```yaml
# terraform/secrets.tf
resource "aws_secretsmanager_secret" "database_url" {
  name = "${var.environment}/database-url"
  description = "Cadena de conexión de la base de datos para ${var.environment}"

  tags = {
    Environment = var.environment
    Project     = "multi-env-demo"
  }
}

resource "aws_secretsmanager_secret_version" "database_url" {
  secret_id     = aws_secretsmanager_secret.database_url.id
  secret_string = jsonencode({
    url = var.database_url
  })
}
```

**Mejor Práctica: Dejar que los Pods de K8s Obtengan Directamente de la Caja Fuerte de AWS**
Ahora, tenemos aplicaciones de K8s (Pods) y una caja fuerte de AWS (Secrets Manager). La pregunta es, ¿cómo puede un Pod obtener de forma segura la receta secreta de la caja fuerte, en lugar de a través de una transmisión manual poco fiable?

Esto requiere una tecnología "puente" clave: Roles de IAM para Cuentas de Servicio (IRSA), y una herramienta importante: Proveedor de Secretos y Configuración de AWS (ASCP).

Así como no le diríamos al chef del puesto (Pod) la contraseña de la caja fuerte. En su lugar, le emitimos a este chef una **"Tarjeta de Empleado Autorizado"** especial (Cuenta de Servicio con un Rol de IAM). Cuando el chef necesita la receta secreta, utiliza esta tarjeta para pasar por una puerta específica en el banco (ASCP). Después de que el sistema bancario verifique los permisos de la tarjeta, pondrá automática y temporalmente la página de la receta secreta necesaria para hoy en una caja dedicada y cerrada con llave frente a él (montada como un archivo en el Pod). La caja se destruye después de que el chef termina.

**Flujo Operativo:**
1. **Configurar IRSA:**
   - En AWS IAM, cree un Rol y autorícelo para leer el Secreto `my-app/db-password`.
   - En EKS, cree una Cuenta de Servicio de Kubernetes.
   - "Vincule" estos dos, diciéndole a EKS: "Cualquier Pod que use esta Cuenta de Servicio puede asumir la identidad de ese Rol de IAM".
2. **Especificar la Cuenta de Servicio en el Despliegue:**
   - En su `deployment.yaml`, especifique explícitamente `spec.template.spec.serviceAccountName` como la Cuenta de Servicio que acaba de crear.
3. **Montar Secretos usando ASCP:**
   1. Primero debe instalar el complemento Proveedor de Secretos y Configuración de AWS en su clúster de EKS.
   2. Luego, en su `deployment.yaml`, defina un `Volumen` y dígale a ASCP:
      1. Quiero montar un secreto.
      2. El nombre del secreto es `my-app/db-password` (el nombre en Secrets Manager).
      3. Por favor, móntelo en la ruta `/etc/secrets/db-password` en el Pod.
4. **La Aplicación Lee el Secreto:**
   - Después de que su aplicación se inicia, ya no lee la contraseña de una variable de entorno, sino directamente del archivo local `/etc/secrets/db-password`.

Las enormes ventajas de esta solución:
- **Máxima Seguridad:** El secreto nunca aparece en texto plano en ningún recurso de Kubernetes (YAML, variables de entorno). Se transmite cifrado desde AWS Secrets Manager y se monta directamente en el sistema de archivos de memoria del Pod.
- **Acoplamiento de Código Cero:** Su aplicación no sabe ni le importa si el secreto proviene de AWS o de otro lugar; solo necesita leer desde una ruta de archivo fija.
- **Actualizaciones Dinámicas:** Cuando actualiza el valor del secreto en Secrets Manager, ASCP puede actualizar el contenido del archivo montado en el Pod casi en tiempo real, y la aplicación puede leer dinámicamente el último valor.

Finalmente, hemos creado una estrategia de gestión de configuración y secretos por capas:
> **ConfigMap:** Se utiliza para almacenar configuraciones de aplicaciones públicas.
> **Secreto de K8s:** Como herramienta básica de gestión de secretos, principalmente protegida por RBAC, adecuada para algunos escenarios menos sensibles.
**AWS Secrets Manager + IRSA + ASCP** es el estándar de oro y la mejor práctica para gestionar secretos altamente sensibles en EKS. Proporciona capacidades de cifrado de extremo a extremo, auditoría y rotación automática, y es una parte necesaria para construir un sistema seguro y confiable.

Dejémoslo en un cliffhanger por ahora. Discutiremos los tres temas de **Gestión de Monitoreo y Registros**, **Mejores Prácticas de Seguridad** y **Recuperación ante Desastres y Copias de Seguridad** en detalle en la futura etapa de **Lanzamiento y Monitoreo Operacional (Seguridad y Cumplimiento)**. Lo que discutimos principalmente hoy (?) es la **Estrategia de Gobernanza y Arquitectura Multi-Entorno de Desarrollo / Staging / Producción - Desarrollo de Software e Integración Continua**. Y mañana, discutiremos el último contenido de **Desarrollo de Software e Integración Continua** - Optimización de la Experiencia del Desarrollador (DX): Diseño de Herramientas Internas y Solución de Problemas.