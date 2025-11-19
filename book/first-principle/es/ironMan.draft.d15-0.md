# Día 15 | Implementación de Automatización CI/CD Completa - GitHub Actions × CodePipeline × CodeBuild: Pipeline de Integración y Despliegue Continuo y Gestión de Segmentación de Tareas

En el desarrollo de software moderno, no solo debemos enfocarnos en la calidad del código, sino también asegurar la fiabilidad de todo el proceso de entrega. ¿Recuerdas lo que mencionamos en <Estrategia de Control de Versiones (Estrategia de Revisión de PR)>: `Imagina que cada PR es como una "propuesta de producto" que debe pasar por un riguroso "proceso de revisión" antes de poder ser incorporada a la línea de productos oficial de la empresa (fusionada en la rama principal)...`?

```mermaid
flowchart TD
    A[Desarrollador Envía PR] --> B{Verificaciones Automatizadas}
    B -->|Falla| C[Corregir Problemas]
    C --> A
    B -->|Pasa| D[Asignar Revisores]

    subgraph "Lado del Desarrollador"
      P1[Auto-verificación del Desarrollador]
      P2[Ejecutar Pruebas Locales]
      P3[Formatear Código]
      P4[Confirmar Convención de Mensajes de Commit]

      P1 --> P2
      P2 --> P3
      P3 --> P4
      P4 --> A
    end

    subgraph "Lado del Revisor"
      D[Asignar Revisores] --> E[Revisión del Líder Técnico]
      D --> F[Revisión del Ingeniero Senior]
      D --> G[Revisión del Product Owner]

      E --> H{Revisión Técnica Aprobada?}
      F --> I{Revisión de Calidad de Código Aprobada?}
      G --> J{Revisión de Lógica de Negocio Aprobada?}

      H -->|No| K[Solicitar Cambios]
      I -->|No| K
      J -->|No| K
      K --> A

      H -->|Sí| L{Todas las Revisiones Aprobadas?}
      I -->|Sí| L
      J -->|Sí| L

      L -->|No| M[Esperando Otras Revisiones]
      L -->|Sí| N[Aprobación Final]
    end

    N --> O{Verificación de Conflictos}
    O -->|Tiene Conflictos| P[Resolver Conflictos]
    P --> A
    O -->|Sin Conflictos| Q[Fusionar a Rama Destino]

    Q --> R[Auto-Despliegue]
    Q --> S[Notificar a Interesados]

    style A fill:#e3f2fd
    style Q fill:#e8f5e8
    style K fill:#ffebee
    style R fill:#f3e5f5
```

En este capítulo, mencionamos brevemente las **Verificaciones Automatizadas y el Auto-Despliegue (CI/CD)** en el diagrama de flujo. Sin embargo, en ese momento, no profundizamos en cómo ejecutar este proceso, sino que nos centramos en analizar Git, o más bien, la gestión de calidad por fases de los resultados del desarrollo. Pero en esta parte del proceso, podemos reducir el tiempo dedicado a cada envío local y aplicar pasos como `pruebas funcionales en el entorno de lanzamiento`, `pruebas de integración entre sistemas` y `pruebas e2e` a un entorno automatizado que se ejecuta de forma independiente para su detección. Esta es una de las funciones principales de la **CI (Integración Continua)**.

Este proceso elimina el costo de ejecutar pruebas localmente cada vez y minimiza las diferencias entre los entornos de desarrollo y lanzamiento tanto como sea posible (en mi experiencia de desarrollo, me he encontrado con situaciones en las que las pruebas fallaban localmente pero funcionaban bien en el lado del producto, y viceversa). Al mismo tiempo, cada Tarea y Trabajo ejecutado es también una **protección activa** de la **implementación de la lógica de negocio**. Así que hoy, profundizaremos en cómo construir un pipeline de automatización CI/CD a nivel empresarial a través de GitHub Actions, AWS CodePipeline y CodeBuild, con un enfoque especial en:

- **Gestión de Fragmentación de Trabajos**: Cómo dividir y organizar razonablemente las tareas de CI/CD.
- **Gestión de Control de Versiones de Tareas**: Cómo controlar las versiones del proceso CI/CD.
- **Gestión de Abstracción de Parámetros**: Cómo gestionar los parámetros de configuración para diferentes entornos.
- **Puerta de Revisión de Negocio**: Cómo añadir un mecanismo de revisión manual al proceso automatizado.

Primero, revisemos las dificultades del despliegue de software en la era pre-CI/CD:

**Escenario 1: El Miedo al Despliegue Manual**

```
Viernes, 5 PM...

Desarrollador: "Estoy a punto de desplegar la nueva característica en producción."
Colegas: "¡Espera! ¡No despliegues un viernes!"
Desarrollador: "¿Por qué?"
Colegas: "Porque si algo sale mal, tendremos que trabajar horas extras durante el fin de semana..."

**Problemas Centrales**:
- El proceso de despliegue depende de operaciones manuales, propenso a errores.
- Falta de un proceso estandarizado, cada despliegue es un riesgo.
- Difícil de revertir, un problema tiene un gran impacto.
- El equipo carece de confianza en el despliegue.
```

**Escenario 2: El Desastre de Entornos Inconsistentes**

```
Desarrollador: "Extraño, funciona bien en mi máquina..."
Tester: "Pero hay un error en el entorno de pruebas."
Ingeniero de Operaciones: "Y el entorno de producción es diferente del entorno de pruebas..."
Gerente de Producto: "¿Cuándo se puede arreglar? ¡Los clientes están esperando!"

**Problemas Centrales**:
- Configuraciones inconsistentes en los entornos de desarrollo, pruebas y producción.
- Gestión caótica de dependencias, frecuentes conflictos de versiones.
- Falta de pruebas automatizadas, los problemas se descubren demasiado tarde.
- Configuración del entorno que consume mucho tiempo, afectando la eficiencia del desarrollo.
```

**Escenario 3: La Caja Negra del Proceso de Despliegue**

```
Nuevo Ingeniero: "¿Cómo despliego mi código?"
Ingeniero Senior: "Hmm... déjame pensar..."
"Primero, necesitas SSH en el servidor..."
"Luego git pull..."
"Luego recompilar..."
"Recuerda reiniciar nginx..."
"Y limpiar la caché..."
"Oh, y no olvides hacer una copia de seguridad de la base de datos..."

Novato: 😵‍💫

**Problemas Centrales**:
- El conocimiento de despliegue se concentra en unas pocas personas.
- El proceso es complejo y carece de documentación.
- Alto costo de aprendizaje para los nuevos miembros del equipo.
- Alto riesgo si el personal clave se va.
```

¿Has notado que estos puntos débiles en la entrega de software son muy similares a los escenarios del capítulo <Infraestructura como Código: Codificando y Controlando Versiones de la Infraestructura con Terraform>? Cuando necesitamos probar y desplegar sin un **entorno estable** y un **proceso fijo de prueba de lógica de negocio**, es fácil que el equipo de desarrollo sienta que camina sobre hielo delgado con cada despliegue. La razón más importante por la que la **CI (Integración Continua)** y la **CD (Despliegue/Entrega Continua)** son necesarias es para establecer un proceso estable y fijo de verificación y entrega de lógica de negocio, para **`proteger activamente la lógica de negocio existente de ser corrompida por cambios en el código`**.

## Conceptos Centrales y Valor de CI/CD

```
Proteger la lógica de negocio existente de ser corrompida por cambios en el código.
```

### ¿Qué es CI/CD?

**CI (Integración Continua)**: Los desarrolladores fusionan frecuentemente los cambios de código en la rama principal, y cada fusión desencadena un proceso automatizado de construcción y prueba.

**CD (Despliegue/Entrega Continua)**: A través de un proceso automatizado, el código que ha pasado las pruebas se despliega automáticamente en varios entornos, e incluso directamente en el entorno de producción.

### El Valor Central de CI/CD

```mermaid
graph TD
    A[Valor Central de CI/CD] --> B[Retroalimentación Rápida]
    A --> C[Reducción de Riesgos]
    A --> D[Aumento de Eficiencia]
    A --> E[Garantía de Calidad]

    B --> B1[Detección temprana de problemas]
    B --> B2[Corrección rápida de defectos]

    C --> C1[Lanzamientos en pequeños lotes]
    C --> C2[Capacidad de reversión rápida]

    D --> D1[Automatización reemplaza el trabajo manual]
    D --> D2[Manejo de procesos paralelos]

    E --> E1[Cobertura de pruebas automatizadas]
    E --> E2[Entornos consistentes]
```

Imagina: `La transformación de un "taller manual" a una "fábrica automatizada".`

El desarrollo de software pasado era como un taller manual:

- **Hecho a mano**: Cada producto (versión de software) requería un artesano (desarrollador) para hacerlo a mano.
- **Calidad Inestable**: Dependía de la habilidad y el estado del artesano ese día.
- **Producción Limitada**: Un artesano solo podía producir un número limitado de productos por día.
- **Conocimiento Concentrado**: Las habilidades estaban en manos de unos pocos artesanos.

Y CI/CD es como construir una línea de producción automatizada:

- **Proceso Estandarizado**: Cada paso tiene estándares claros y verificaciones automatizadas.
- **Garantía de Calidad**: Cada producto pasa por el mismo proceso de inspección de calidad.
- **Alta Eficiencia**: Las máquinas no se cansan y pueden trabajar 24/7.
- **Intercambio de Conocimiento**: El proceso está codificado, por lo que cualquiera puede entenderlo y mejorarlo.

## Práctica Profunda con GitHub Actions

GitHub Actions es una plataforma CI/CD proporcionada por GitHub que nos permite definir y ejecutar flujos de trabajo automatizados directamente en nuestro repositorio de código.

### Conceptos Centrales de GitHub Actions

```mermaid
graph LR
    A[Arquitectura de GitHub Actions] --> B[Flujo de Trabajo]
    A --> C[Trabajo]
    A --> D[Paso]
    A --> E[Acción]

    B --> B1[.github/workflows/*.yml]
    B --> B2[Definición de condición de activación]

    C --> C1[Selección de entorno de ejecutor]
    C --> C2[Soporte de ejecución paralela]

    D --> D1[Comandos de ejecución específicos]
    D --> D2[Configuración de variables de entorno]

    E --> E1[Unidades de código reutilizables]
    E --> E2[Ecosistema de Marketplace]
```

### Estrategia de Gestión de Fragmentación de Trabajos

En un proceso CI/CD a nivel empresarial, la segmentación razonable de los trabajos es crucial. Necesitamos equilibrar la eficiencia de ejecución y la mantenibilidad:

#### 1. Principios Básicos de Segmentación de Trabajos

```yaml
# .github/workflows/ci-cd.yml
name: "Pipeline CI/CD Frontend"

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  workflow_dispatch:

env:
  NODE_VERSION: "18"
  PNPM_VERSION: "8.15.0"

jobs:
  # Primera capa: Verificaciones básicas (fallo rápido)
  code-quality:
    name: "Verificación de Calidad de Código"
    runs-on: ubuntu-latest
    timeout-minutes: 10

    outputs:
      cache-key: ${{ steps.cache-key.outputs.key }}

    steps:
      - name: Checkout del código
        uses: actions/checkout@v4

      - name: Generar clave de caché
        id: cache-key
        run: |
          echo "key=node-${{ env.NODE_VERSION }}-${{ hashFiles('**/pnpm-lock.yaml') }}" >> $GITHUB_OUTPUT

      - name: Configurar Node.js y dependencias
        uses: ./.github/actions/setup-node
        with:
          node-version: ${{ env.NODE_VERSION }}
          pnpm-version: ${{ env.PNPM_VERSION }}
          cache-key: ${{ steps.cache-key.outputs.key }}

      - name: Verificación de Lint y formato
        run: |
          pnpm run lint:check
          pnpm run format:check
          pnpm run type-check

  # Segunda capa: Ejecución de pruebas (procesamiento paralelo)
  unit-tests:
    name: "Pruebas Unitarias"
    runs-on: ubuntu-latest
    needs: code-quality
    timeout-minutes: 15

    strategy:
      matrix:
        node-version: [16, 18, 20]

    steps:
      - name: Checkout del código
        uses: actions/checkout@v4

      - name: Configurar Node.js y dependencias
        uses: ./.github/actions/setup-node
        with:
          node-version: ${{ matrix.node-version }}
          pnpm-version: ${{ env.PNPM_VERSION }}
          cache-key: ${{ needs.code-quality.outputs.cache-key }}

      - name: Ejecutar pruebas unitarias
        run: pnpm run test:unit --coverage

      - name: Subir cobertura
        if: matrix.node-version == '18'
        uses: codecov/codecov-action@v4
        with:
          token: ${{ secrets.CODECOV_TOKEN }}

  integration-tests:
    name: "Pruebas de Integración"
    runs-on: ubuntu-latest
    needs: code-quality
    timeout-minutes: 20

    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
        options: >
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - name: Checkout del código
        uses: actions/checkout@v4

      - name: Configurar Node.js y dependencias
        uses: ./.github/actions/setup-node
        with:
          node-version: ${{ env.NODE_VERSION }}
          pnpm-version: ${{ env.PNPM_VERSION }}
          cache-key: ${{ needs.code-quality.outputs.cache-key }}

      - name: Configurar base de datos de prueba
        run: |
          pnpm run db:migrate:test
          pnpm run db:seed:test
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test_db

      - name: Ejecutar pruebas de integración
        run: pnpm run test:integration
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test_db

  e2e-tests:
    name: "Pruebas E2E"
    runs-on: ubuntu-latest
    needs: code-quality
    timeout-minutes: 30

    strategy:
      matrix:
        browser: [chromium, firefox]
        shard: [1/4, 2/4, 3/4, 4/4]

    steps:
      - name: Checkout del código
        uses: actions/checkout@v4

      - name: Configurar Node.js y dependencias
        uses: ./.github/actions/setup-node
        with:
          node-version: ${{ env.NODE_VERSION }}
          pnpm-version: ${{ env.PNPM_VERSION }}
          cache-key: ${{ needs.code-quality.outputs.cache-key }}

      - name: Instalar Playwright
        run: npx playwright install --with-deps ${{ matrix.browser }}

      - name: Construir aplicación
        run: pnpm run build:test

      - name: Ejecutar pruebas E2E
        run: |
          pnpm run start:test &
          npx wait-on http://localhost:3000
          npx playwright test --project=${{ matrix.browser }} --shard=${{ matrix.shard }}

      - name: Subir resultados de pruebas
        if: failure()
        uses: actions/upload-artifact@v4
        with:
          name: e2e-results-${{ matrix.browser }}-${{ matrix.shard }}
          path: playwright-report/

  # Tercera capa: Verificaciones de seguridad y calidad
  security-scan:
    name: "Escaneo de Seguridad"
    runs-on: ubuntu-latest
    needs: code-quality
    timeout-minutes: 15

    steps:
      - name: Checkout del código
        uses: actions/checkout@v4

      - name: Ejecutar auditoría de seguridad
        run: |
          npx audit-ci --config .audit-ci.json

      - name: Ejecutar escaneo SAST
        uses: github/super-linter@v4
        env:
          DEFAULT_BRANCH: main
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          VALIDATE_JAVASCRIPT_ES: true
          VALIDATE_TYPESCRIPT_ES: true

      - name: Ejecutar escáner de vulnerabilidades Trivy
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: "fs"
          scan-ref: "."
          format: "sarif"
          output: "trivy-results.sarif"

      - name: Subir resultados del escaneo de seguridad
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: "trivy-results.sarif"

  # Cuarta capa: Preparación de construcción y despliegue
  build:
    name: "Construir Aplicación"
    runs-on: ubuntu-latest
    needs: [unit-tests, integration-tests, e2e-tests, security-scan]
    timeout-minutes: 10

    outputs:
      build-version: ${{ steps.version.outputs.version }}

    steps:
      - name: Checkout del código
        uses: actions/checkout@v4

      - name: Generar versión
        id: version
        run: |
          VERSION=$(date +%Y%m%d-%H%M%S)-${GITHUB_SHA:0:7}
          echo "version=$VERSION" >> $GITHUB_OUTPUT
          echo "::notice::Versión de construcción: $VERSION"

      - name: Configurar Node.js y dependencias
        uses: ./.github/actions/setup-node
        with:
          node-version: ${{ env.NODE_VERSION }}
          pnpm-version: ${{ env.PNPM_VERSION }}
          cache-key: ${{ needs.code-quality.outputs.cache-key }}

      - name: Construir aplicación
        run: |
          pnpm run build
          echo ${{ steps.version.outputs.version }} > dist/version.txt
        env:
          VITE_BUILD_VERSION: ${{ steps.version.outputs.version }}
          VITE_BUILD_TIME: ${{ github.event.head_commit.timestamp }}
          VITE_COMMIT_SHA: ${{ github.sha }}

      - name: Validar construcción
        run: |
          # Verificar archivos clave
          test -f dist/index.html || { echo "❌ index.html no encontrado"; exit 1; }
          test -f dist/version.txt || { echo "❌ version.txt no encontrado"; exit 1; }

          # Verificar tamaño de construcción
          BUNDLE_SIZE=$(du -sk dist | cut -f1)
          echo "Tamaño del paquete: ${BUNDLE_SIZE}KB"

          if [ $BUNDLE_SIZE -gt 20480 ]; then
            echo "⚠️ Advertencia de tamaño de paquete: ${BUNDLE_SIZE}KB > 20MB"
          fi

      - name: Subir artefactos de construcción
        uses: actions/upload-artifact@v4
        with:
          name: build-${{ steps.version.outputs.version }}
          path: dist/
          retention-days: 7

  # Quinta capa: Desplegar en entorno de staging
  deploy-staging:
    name: "Desplegar en Staging"
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/develop'
    timeout-minutes: 10

    environment:
      name: staging
      url: https://staging.myapp.com

    steps:
      - name: Descargar artefactos de construcción
        uses: actions/download-artifact@v4
        with:
          name: build-${{ needs.build.outputs.build-version }}
          path: dist/

      - name: Configurar credenciales de AWS
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-northeast-1

      - name: Desplegar en S3
        run: |
          aws s3 sync dist/ s3://${{ secrets.STAGING_S3_BUCKET }} --delete
          aws cloudfront create-invalidation --distribution-id ${{ secrets.STAGING_CLOUDFRONT_ID }} --paths "/*"

      - name: Ejecutar verificación de despliegue
        run: |
          sleep 30
          HTTP_CODE=$(curl -s -o /dev/null -w "%{{http_code}}" https://staging.myapp.com)
          if [ $HTTP_CODE -ne 200 ]; then
            echo "❌ Verificación de despliegue fallida (HTTP $HTTP_CODE)"
            exit 1
          fi
          echo "✅ Despliegue en Staging verificado"

  # Sexta capa: Puerta de revisión de negocio
  approval-gate:
    name: "Aprobación de Producción"
    runs-on: ubuntu-latest
    needs: [build, deploy-staging]
    if: github.ref == 'refs/heads/main'
    timeout-minutes: 1440 # 24 horas de tiempo de espera

    environment:
      name: production-approval

    steps:
      - name: Solicitar aprobación de despliegue en producción
        run: |
          echo "🔍 Aprobación de despliegue en producción solicitada"
          echo "📋 Versión de construcción: ${{ needs.build.outputs.build-version }}"
          echo "🌐 URL de Staging: https://staging.myapp.com"
          echo "⏰ Tiempo de espera de aprobación: 24 horas"

  # Séptima capa: Despliegue en producción
  deploy-production:
    name: "Desplegar en Producción"
    runs-on: ubuntu-latest
    needs: [build, approval-gate]
    timeout-minutes: 15

    environment:
      name: production
      url: https://myapp.com

    steps:
      - name: Descargar artefactos de construcción
        uses: actions/download-artifact@v4
        with:
          name: build-${{ needs.build.outputs.build-version }}
          path: dist/

      - name: Configurar credenciales de AWS
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-northeast-1

      - name: Hacer copia de seguridad de la versión actual
        run: |
          BACKUP_TIMESTAMP=$(date +%Y%m%d-%H%M%S)
          aws s3 sync s3://${{ secrets.PRODUCTION_S3_BUCKET }} s3://${{ secrets.BACKUP_S3_BUCKET }}/${BACKUP_TIMESTAMP}/
          echo "BACKUP_PATH=${BACKUP_TIMESTAMP}" >> $GITHUB_ENV

      - name: Desplegar en producción
        run: |
          aws s3 sync dist/ s3://${{ secrets.PRODUCTION_S3_BUCKET }} --delete
          aws cloudfront create-invalidation --distribution-id ${{ secrets.PRODUCTION_CLOUDFRONT_ID }} --paths "/*"

      - name: Verificación de salud de producción
        run: |
          echo "Esperando propagación de CDN..."
          sleep 60

          for i in {1..5};
          do
            HTTP_CODE=$(curl -s -o /dev/null -w "%{{http_code}}" https://myapp.com)
            if [ $HTTP_CODE -eq 200 ]; then
              echo "✅ Verificación de salud de producción aprobada (intento $i)"
              break
            elif [ $i -eq 5 ]; then
              echo "❌ Verificación de salud de producción fallida después de 5 intentos"
              echo "🔄 Revirtiendo..."
              aws s3 sync s3://${{ secrets.BACKUP_S3_BUCKET }}/${{ env.BACKUP_PATH }}/ s3://${{ secrets.PRODUCTION_S3_BUCKET }} --delete
              aws cloudfront create-invalidation --distribution-id ${{ secrets.PRODUCTION_CLOUDFRONT_ID }} --paths "/*"
              exit 1
            else
              echo "⏳ Intento de verificación de salud $i fallido, reintentando..."
              sleep 30
            fi
          done

      - name: Notificar éxito del despliegue
        if: success()
        uses: 8398a7/action-slack@v3
        with:
          status: success
          channel: "#deployment"
          text: |
            🚀 ¡Despliegue en producción exitoso!
            📦 Versión: ${{ needs.build.outputs.build-version }}
            🌐 URL: https://myapp.com
            👤 Desplegado por: ${{ github.actor }}
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}

      - name: Notificar fallo del despliegue
        if: failure()
        uses: 8398a7/action-slack@v3
        with:
          status: failure
          channel: "#deployment"
          text: |
            ❌ ¡Despliegue en producción fallido!
            📦 Versión: ${{ needs.build.outputs.build-version }}
            🔄 Reversión automática iniciada
            👤 Intentado por: ${{ github.actor }}
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

#### 2. Modularización de Trabajos a Nivel Empresarial y Referencia Cruzada de Dominios

Además de dividir los trabajos dentro de un único archivo de flujo de trabajo, para estándares compartidos a nivel empresarial y mantenibilidad, podemos separar los trabajos que pertenecen a un dominio o función específica en diferentes flujos de trabajo reutilizables. Cuando sea necesario, el flujo de trabajo principal puede referenciar y ejecutar estos trabajos independientes como una biblioteca.

La mayor ventaja de este enfoque es que permite que la lógica de interacción de dominio entre diferentes sistemas se valide mutuamente. Por ejemplo, cuando se actualiza el proceso CI del `user-service`, el proceso CI del `order-service` puede asegurar que este cambio no ha roto las funciones relacionadas con el usuario en el proceso de pedido al referenciar el trabajo de prueba del `user-service`. Esto evita el dilema de "está bien en mi dominio, pero afecta a otros dominios".

**Escenario: Un cambio en el Servicio de Usuario necesita asegurar que el Servicio de Pedidos no se vea afectado.**

Primero, definimos un flujo de trabajo CI reutilizable para el `user-service`, que incluye un trabajo específicamente para validar funciones centrales.

```yaml
# .github/workflows/reusable-user-service-ci.yml
name: "CI Reutilizable del Servicio de Usuario"

on:
  workflow_call:
    inputs:
      node-version:
        required: false
        type: string
        default: "18"
    outputs:
      test-results-summary:
        description: "Un resumen de los resultados de las pruebas del servicio de usuario"
        value: ${{ jobs.validate-user-core.outputs.summary }}

jobs:
  validate-user-core:
    name: "Validar Lógica Central del Servicio de Usuario"
    runs-on: ubuntu-latest
    outputs:
      summary: ${{ steps.test-summary.outputs.summary }}

    steps:
      - name: Checkout del código del Servicio de Usuario
        uses: actions/checkout@v4
        with:
          repository: "my-org/user-service"
          ref: "main"

      # ... (Configurar Node.js, instalar dependencias) ...

      - name: Ejecutar pruebas centrales del servicio de usuario
        id: core-tests
        run: |
          # Ejecutar pruebas de integración para funciones centrales
          pnpm run test:core-integration
          echo "summary=Lógica central del usuario validada exitosamente" >> $GITHUB_OUTPUT

      - name: Crear resumen de pruebas
        id: test-summary
        run: echo "summary=✅ Validación central del Servicio de Usuario aprobada" >> $GITHUB_OUTPUT
```

Luego, en el flujo de trabajo CI/CD principal del `order-service`, podemos referenciar el trabajo anterior para la validación de integración entre dominios.

```yaml
# .github/workflows/order-service-ci.yml
name: "CI/CD del Servicio de Pedidos"

on:
  push:
    branches: [main]
  pull_request:

jobs:
  # Ejecutar las propias pruebas del servicio de pedidos
  test-order-service:
    name: "Probar Servicio de Pedidos"
    runs-on: ubuntu-latest
    steps:
      - name: Checkout del código del Servicio de Pedidos
        uses: actions/checkout@v4
      # ... (Ejecutar las propias pruebas del servicio de pedidos) ...
      - name: Ejecutar pruebas de pedidos
        run: pnpm run test

  # Referenciar el trabajo CI del Servicio de Usuario para validación de integración
  cross-validate-with-user-service:
    name: "Validación Cruzada de Dominio (Servicio de Usuario)"
    needs: test-order-service
    uses: ./.github/workflows/reusable-user-service-ci.yml
    with:
      node-version: "18"

  # Construir solo después de que sus propias pruebas y la validación cruzada de dominio hayan pasado
  build:
    name: "Construir Servicio de Pedidos"
    runs-on: ubuntu-latest
    needs: [test-order-service, cross-validate-with-user-service]
    steps:
      - name: Mostrar resultados de validación
        run: |
          echo "Las pruebas del Servicio de Pedidos pasaron."
          echo "Resumen de validación del Servicio de Usuario: ${{ needs.cross-validate-with-user-service.outputs.test-results-summary }}"
      - name: Checkout del código
        uses: actions/checkout@v4
      # ... (Pasos de construcción) ...
      - name: Construir aplicación
        run: pnpm run build
```

Por supuesto, si desea reducir la granularidad al nivel de `Step`, también es factible.

```yaml
steps:
  - task: SonarCloudAnalyze@3
    inputs:
      jdkversion: "JAVA_HOME_21_X64"
    condition: succeededOrFailed()
  - task: SonarCloudPublish@3
    inputs:
      pollingTimeoutSec: "300"
    condition: succeededOrFailed()
```

A través de este enfoque, elevamos la tarea de CI/CD de "garantía de calidad para un solo proyecto" a "protección de la estabilidad del sistema en proyectos y dominios", lo cual es crucial en arquitecturas de microservicios o sistemas complejos.

#### 3. Creación de Acciones Reutilizables

Para mejorar la reutilización del código, creamos acciones personalizadas:

```yaml
# .github/actions/setup-node/action.yml
name: "Configurar Entorno Node.js"
description: "Configurar Node.js con pnpm y caché de dependencias"

inputs:
  node-version:
    description: "Versión de Node.js"
    required: true
    default: "18"
  pnpm-version:
    description: "Versión de pnpm"
    required: true
    default: "8.15.0"
  cache-key:
    description: "Clave de caché para dependencias"
    required: true

runs:
  using: "composite"
  steps:
    - name: Configurar Node.js
      uses: actions/setup-node@v4
      with:
        node-version: ${{ inputs.node-version }}

    - name: Configurar pnpm
      uses: pnpm/action-setup@v3
      with:
        version: ${{ inputs.pnpm-version }}

    - name: Obtener directorio de la tienda pnpm
      id: pnpm-cache
      shell: bash
      run: echo "STORE_PATH=$(pnpm store path)" >> $GITHUB_OUTPUT

    - name: Configurar caché de pnpm
      uses: actions/cache@v4
      with:
        path: ${{ steps.pnpm-cache.outputs.STORE_PATH }}
        key: ${{ inputs.cache-key }}
        restore-keys: |
          ${{ runner.os }}-pnpm-store-

    - name: Instalar dependencias
      shell: bash
      run: pnpm install --frozen-lockfile
```

### Estrategia de Gestión de Control de Versiones de Tareas

#### 1. Configuración de Versionado de Flujos de Trabajo

```yaml
# .github/workflows/ci-cd-v2.yml
name: "Pipeline CI/CD v2.0"

on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string
      workflow-version:
        required: false
        type: string
        default: "v2.0"

env:
  WORKFLOW_VERSION: ${{ inputs.workflow-version }}
  TARGET_ENVIRONMENT: ${{ inputs.environment }}

jobs:
  version-check:
    name: "Verificación de Versión del Flujo de Trabajo"
    runs-on: ubuntu-latest

    steps:
      - name: Validar versión del flujo de trabajo
        run: |
          echo "🔄 Ejecutando Pipeline CI/CD ${{ env.WORKFLOW_VERSION }}"
          echo "🎯 Entorno Objetivo: ${{ env.TARGET_ENVIRONMENT }}"

          # Verificación de compatibilidad de versiones
          case "${{ env.WORKFLOW_VERSION }}" in
            v1.*)
              echo "⚠️ Usando versión de flujo de trabajo heredada"
              ;;;
            v2.*)
              echo "✅ Usando versión actual de flujo de trabajo"
              ;;;
            *)
              echo "❌ Versión de flujo de trabajo desconocida"
              exit 1
              ;;;
          esac
```

#### 2. Gestión de Archivos de Configuración de Pipeline

```yaml
# .github/pipeline-configs/production.yml
version: "2.0"
environment: "production"

stages:
  code-quality:
    enabled: true
    timeout: 10
    node-versions: [18, 20]

  testing:
    unit-tests:
      enabled: true
      timeout: 15
      parallel-matrix: true
    integration-tests:
      enabled: true
      timeout: 20
      services: [postgres, redis]
    e2e-tests:
      enabled: true
      timeout: 30
      browsers: [chromium, firefox]
      sharding: 4

  security:
    audit: true
    sast: true
    vulnerability-scan: true

  deployment:
    strategy: "blue-green"
    health-check:
      enabled: true
      retries: 5
      interval: 30
    rollback:
      auto: true
      timeout: 300

  notifications:
    slack:
      success: "#deployment"
      failure: "#alerts"
    email:
      - devops@company.com
```

#### 3. Carga Dinámica de Pipeline

```yaml
# .github/workflows/dynamic-pipeline.yml
name: "Cargador Dinámico de Pipeline"

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  load-config:
    name: "Cargar Configuración de Pipeline"
    runs-on: ubuntu-latest

    outputs:
      config: ${{ steps.config.outputs.config }}
      version: ${{ steps.config.outputs.version }}

    steps:
      - name: Checkout del código
        uses: actions/checkout@v4

      - name: Cargar configuración de pipeline
        id: config
        run: |
          # Seleccionar archivo de configuración basado en la rama
          if [[ "${{ github.ref }}" == "refs/heads/main" ]]; then
            CONFIG_FILE=".github/pipeline-configs/production.yml"
          elif [[ "${{ github.ref }}" == "refs/heads/develop" ]]; then
            CONFIG_FILE=".github/pipeline-configs/staging.yml"
          else
            CONFIG_FILE=".github/pipeline-configs/development.yml"
          fi

          # Cargar y mostrar configuración
          CONFIG=$(cat $CONFIG_FILE | yq -o=json)
          VERSION=$(echo $CONFIG | jq -r '.version')

          echo "config=$CONFIG" >> $GITHUB_OUTPUT
          echo "version=$VERSION" >> $GITHUB_OUTPUT
          echo "📄 Configuración cargada desde: $CONFIG_FILE"
          echo "🏷️ Versión de Pipeline: $VERSION"

  execute-pipeline:
    name: "Ejecutar Pipeline"
    needs: load-config
    uses: ./.github/workflows/ci-cd-v2.yml
    with:
      environment: ${{ fromJson(needs.load-config.outputs.config).environment }}
      workflow-version: ${{ needs.load-config.outputs.version }}
    secrets: inherit
```

#### 4. Control de Versiones para Flujos de Trabajo Reutilizables entre Dominios

Cuando modularizamos trabajos en flujos de trabajo reutilizables (como `reusable-user-service-ci.yml`), el control de versiones de estos flujos de trabajo compartidos se vuelve crucial. Esto asegura que cuando el proceso CI de un servicio central (como `user-service`) cambia, no rompa accidentalmente el proceso de construcción de otros servicios que dependen de él (como `order-service`).

**Escenario: El proceso CI para `user-service` se actualiza de v1 a v2, pero `order-service` necesita temporalmente seguir usando v1.**

Podemos usar etiquetas de Git para controlar las versiones de los flujos de trabajo reutilizables.

Primero, el archivo `reusable-user-service-ci.yml` para el propio `user-service` no necesita una etiqueta de versión especial. Su versión se gestiona mediante las etiquetas del repositorio de Git en el que reside. Cuando creemos que `reusable-user-service-ci.yml` ha alcanzado un estado estable, creamos una etiqueta de versión en ese commit, por ejemplo, `v1.0` o `v2.0`.

Luego, en el flujo de trabajo CI de `order-service`, podemos especificar explícitamente la versión del CI de `user-service` a usar.

```yaml
# .github/workflows/order-service-ci.yml
name: "CI/CD del Servicio de Pedidos"

on:
  push:
    branches: [main]
  pull_request:

jobs:
  test-order-service:
    # ... (Ejecutar las propias pruebas del servicio de pedidos) ...
    name: "Probar Servicio de Pedidos"
    runs-on: ubuntu-latest
    steps:
      - name: Checkout del código del Servicio de Pedidos
        uses: actions/checkout@v4
      - name: Ejecutar pruebas de pedidos
        run: pnpm run test

  # Referenciar la v1.0 del CI del Servicio de Usuario para validación
  cross-validate-with-user-service-v1:
    name: "Validación Cruzada de Dominio (Servicio de Usuario @v1.0)"
    needs: test-order-service
    # Usar la sintaxis @<tag> para bloquear la versión del flujo de trabajo reutilizable
    uses: my-org/user-service/.github/workflows/reusable-user-service-ci.yml@v1.0
    with:
      node-version: "18"

  # Cuando el equipo esté listo, se puede crear un nuevo PR para actualizar a v2.0
  # cross-validate-with-user-service-v2:
  #   name: "Validación Cruzada de Dominio (Servicio de Usuario @v2.0)"
  #   needs: test-order-service
  #   uses: my-org/user-service/.github/workflows/reusable-user-service-ci.yml@v2.0
  #   with:
  #     node-version: '20'

  build:
    name: "Construir Servicio de Pedidos"
    runs-on: ubuntu-latest
    needs: [test-order-service, cross-validate-with-user-service-v1]
    steps:
      - name: Mostrar resultados de validación
        run: |
          echo "Las pruebas del Servicio de Pedidos pasaron."
          echo "Resumen de validación del Servicio de Usuario: ${{ needs.cross-validate-with-user-service-v1.outputs.test-results-summary }}"
      - name: Checkout del código
        uses: actions/checkout@v4
      - name: Construir aplicación
        run: pnpm run build
```

Esta estrategia extiende el concepto de "Control de Versiones de Tareas" desde dentro de un solo proyecto a una dimensión a nivel empresarial a través de proyectos y servicios, logrando una verdadera gestión versionada de Pipeline as Code.

### Estrategia de Gestión de Abstracción de Parámetros

#### 1. Gestión por Capas de Variables de Entorno

```yaml
# .github/environments/production.yml
variables:
  # Variables públicas
  NODE_ENV: "production"
  API_VERSION: "v2"
  BUILD_TARGET: "production"
  CACHE_TTL: "3600"

  # Configuración de CDN
  CDN_DOMAIN: "cdn.myapp.com"
  ASSET_PREFIX: "/static"

  # Banderas de características
  FEATURE_NEW_UI: "true"
  FEATURE_ANALYTICS: "true"
  FEATURE_AB_TESTING: "true"

secrets:
  # Información sensible (debe configurarse en GitHub Secrets)
  - AWS_ACCESS_KEY_ID
  - AWS_SECRET_ACCESS_KEY
  - SENTRY_DSN
  - ANALYTICS_API_KEY
  - DATABASE_CONNECTION_STRING
```

#### 2. Inyección Dinámica de Variables de Entorno

```yaml
# .github/workflows/env-management.yml
name: "Gestión de Entorno"

jobs:
  setup-environment:
    name: "Configurar Variables de Entorno"
    runs-on: ubuntu-latest

    outputs:
      env-config: ${{ steps.env-setup.outputs.config }}

    steps:
      - name: Checkout del código
        uses: actions/checkout@v4

      - name: Configurar entorno
        id: env-setup
        run: |
          # Cargar configuración basada en el entorno objetivo
          case "${{ github.ref }}" in
            refs/heads/main)
              ENV_FILE=".github/environments/production.yml"
              ;;;
            refs/heads/develop)
              ENV_FILE=".github/environments/staging.yml"
              ;;;
            *)
              ENV_FILE=".github/environments/development.yml"
              ;;;
          esac

          # Cargar variables de entorno
          ENV_VARS=$(yq '.variables' $ENV_FILE)

          # Salida en formato JSON
          echo "config=$ENV_VARS" >> $GITHUB_OUTPUT

          # Establecer variables de entorno para el trabajo actual
          yq '.variables | to_entries | .[] | "\(.key)=\(.value)"' $ENV_FILE >> $GITHUB_ENV

  build-with-env:
    name: "Construir con Entorno"
    needs: setup-environment
    runs-on: ubuntu-latest

    steps:
      - name: Checkout del código
        uses: actions/checkout@v4

      - name: Aplicar configuración de entorno
        run: |
          # Obtener configuración de entorno del trabajo anterior
          ENV_CONFIG='${{ needs.setup-environment.outputs.env-config }}'
          echo "$ENV_CONFIG" | jq -r 'to_entries | .[] | "\(.key)=\(.value)"' >> $GITHUB_ENV

      - name: Construir aplicación
        run: |
          echo "🏗️ Construyendo con entorno: $NODE_ENV"
          echo "🌐 Versión de API: $API_VERSION"
          echo "🎯 Objetivo de Construcción: $BUILD_TARGET"

          # Crear archivo de información de construcción
          cat > build-info.json << EOF
          {
            "environment": "$NODE_ENV",
            "apiVersion": "$API_VERSION",
            "buildTarget": "$BUILD_TARGET",
            "buildTime": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
            "gitCommit": "$GITHUB_SHA",
            "gitBranch": "$GITHUB_REF_NAME"
          }
          EOF

          # Ejecutar construcción
          npm run build
```

#### 3. Gestión Segura de Parámetros Secretos

```yaml
# .github/workflows/secrets-management.yml
name: "Gestión de Secretos"

jobs:
  validate-secrets:
    name: "Validar Secretos Requeridos"
    runs-on: ubuntu-latest

    steps:
      - name: Verificar secretos requeridos
        env:
          # Referenciar los secretos requeridos
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          SENTRY_DSN: ${{ secrets.SENTRY_DSN }}
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}

        run: |
          # Validar que existan los secretos necesarios
          MISSING_SECRETS=()

          [ -z "$AWS_ACCESS_KEY_ID" ] && MISSING_SECRETS+=("AWS_ACCESS_KEY_ID")
          [ -z "$AWS_SECRET_ACCESS_KEY" ] && MISSING_SECRETS+=("AWS_SECRET_ACCESS_KEY")
          [ -z "$SENTRY_DSN" ] && MISSING_SECRETS+=("SENTRY_DSN")
          [ -z "$SLACK_WEBHOOK_URL" ] && MISSING_SECRETS+=("SLACK_WEBHOOK_URL")

          if [ ${#MISSING_SECRETS[@]} -gt 0 ]; then
            echo "❌ Secretos requeridos faltantes:"
            printf "  - %s\n" "${MISSING_SECRETS[@]}"
            exit 1
          fi

          echo "✅ Todos los secretos requeridos están disponibles"

      - name: Probar credenciales de AWS
        run: |
          # Probar validez de credenciales de AWS
          aws sts get-caller-identity || {
            echo "❌ Las credenciales de AWS no son válidas"
            exit 1
          }
          echo "✅ Las credenciales de AWS son válidas"
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_DEFAULT_REGION: ap-northeast-1
```

## Integración a Nivel Empresarial con AWS CodePipeline y CodeBuild

Para entornos empresariales grandes, necesitamos una infraestructura CI/CD más potente. AWS CodePipeline proporciona características más completas a nivel empresarial.

### Diseño Completo del Proceso de CodePipeline

```yaml
# infrastructure/codepipeline.tf
resource "aws_codepipeline" "main" {
  name     = "${var.project_name}-pipeline"
  role_arn = aws_iam_role.codepipeline_role.arn

  artifact_store {
    location = aws_s3_bucket.pipeline_artifacts.bucket
    type     = "S3"

    encryption_key {
      id   = aws_kms_key.pipeline_key.arn
      type = "KMS"
    }
  }

  # Etapa 1: Fuente
  stage {
    name = "Source"

    action {
      name             = "SourceAction"
      category         = "Source"
      owner            = "ThirdParty"
      provider         = "GitHub"
      version          = "1"
      output_artifacts = ["source_output"]

      configuration = {
        Owner  = var.github_owner
        Repo   = var.github_repo
        Branch = var.github_branch
        OAuthToken = var.github_token
        PollForSourceChanges = false
      }
    }
  }

  # Etapa 2: Calidad de Código
  stage {
    name = "CodeQuality"

    action {
      name             = "LintAndFormat"
      category         = "Test"
      owner            = "AWS"
      provider         = "CodeBuild"
      version          = "1"
      input_artifacts  = ["source_output"]
      output_artifacts = ["quality_output"]

      configuration = {
        ProjectName = aws_codebuild_project.code_quality.name
      }
    }

    action {
      name             = "SecurityScan"
      category         = "Test"
      owner            = "AWS"
      provider         = "CodeBuild"
      version          = "1"
      input_artifacts  = ["source_output"]
      output_artifacts = ["security_output"]

      configuration = {
        ProjectName = aws_codebuild_project.security_scan.name
      }

      run_order = 2
    }
  }

  # Etapa 3: Pruebas
  stage {
    name = "Testing"

    action {
      name             = "UnitTests"
      category         = "Test"
      owner            = "AWS"
      provider         = "CodeBuild"
      version          = "1"
      input_artifacts  = ["source_output"]
      output_artifacts = ["unit_test_output"]

      configuration = {
        ProjectName = aws_codebuild_project.unit_tests.name
      }
    }

    action {
      name             = "IntegrationTests"
      category         = "Test"
      owner            = "AWS"
      provider         = "CodeBuild"
      version          = "1"
      input_artifacts  = ["source_output"]
      output_artifacts = ["integration_test_output"]

      configuration = {
        ProjectName = aws_codebuild_project.integration_tests.name
      }

      run_order = 2
    }

    action {
      name             = "E2ETests"
      category         = "Test"
      owner            = "AWS"
      provider         = "CodeBuild"
      version          = "1"
      input_artifacts  = ["source_output"]
      output_artifacts = ["e2e_test_output"]

      configuration = {
        ProjectName = aws_codebuild_project.e2e_tests.name
      }

      run_order = 3
    }
  }

  # Etapa 4: Construcción
  stage {
    name = "Build"

    action {
      name             = "BuildApplication"
      category         = "Build"
      owner            = "AWS"
      provider         = "CodeBuild"
      version          = "1"
      input_artifacts  = ["source_output"]
      output_artifacts = ["build_output"]

      configuration = {
        ProjectName = aws_codebuild_project.build.name
        EnvironmentVariables = jsonencode([
          {
            name  = "ENVIRONMENT"
            value = "staging"
          },
          {
            name  = "BUILD_NUMBER"
            value = "#{codepipeline.PipelineExecutionId}"
          },
          {
            name  = "COMMIT_SHA"
            value = "#{SourceVariables.CommitId}"
          }
        ])
      }
    }
  }

  # Etapa 5: Desplegar en Staging
  stage {
    name = "DeployStaging"

    action {
      name            = "DeployToS3"
      category        = "Deploy"
      owner           = "AWS"
      provider        = "S3"
      version         = "1"
      input_artifacts = ["build_output"]

      configuration = {
        BucketName = aws_s3_bucket.staging.bucket
        Extract    = "true"
      }
    }

    action {
      name             = "InvalidateCloudFront"
      category         = "Invoke"
      owner            = "AWS"
      provider         = "Lambda"
      version          = "1"

      configuration = {
        FunctionName = aws_lambda_function.invalidate_cache.function_name
        UserParameters = jsonencode({
          distribution_id = aws_cloudfront_distribution.staging.id
          paths = ["/*"]
        })
      }

      run_order = 2
    }

    action {
      name             = "StagingTests"
      category         = "Test"
      owner            = "AWS"
      provider         = "CodeBuild"
      version          = "1"
      input_artifacts  = ["source_output"]

      configuration = {
        ProjectName = aws_codebuild_project.staging_tests.name
        EnvironmentVariables = jsonencode([
          {
            name  = "TEST_URL"
            value = "https://${aws_cloudfront_distribution.staging.domain_name}"
          }
        ])
      }

      run_order = 3
    }
  }

  # Etapa 6: Aprobación de Negocio
  stage {
    name = "ApprovalGate"

    action {
      name     = "ProductionApproval"
      category = "Approval"
      owner    = "AWS"
      provider = "Manual"
      version  = "1"

      configuration = {
        NotificationArn    = aws_sns_topic.deployment_approval.arn
        CustomData         = "Por favor, revise el despliegue en staging y apruebe para producción. URL de Staging: https://${aws_cloudfront_distribution.staging.domain_name}"
        ExternalEntityLink = "https://${aws_cloudfront_distribution.staging.domain_name}"
      }
    }
  }

  # Etapa 7: Desplegar en Producción
  stage {
    name = "DeployProduction"

    action {
      name             = "BackupProduction"
      category         = "Invoke"
      owner            = "AWS"
      provider         = "Lambda"
      version          = "1"

      configuration = {
        FunctionName = aws_lambda_function.backup_production.function_name
        UserParameters = jsonencode({
          source_bucket = aws_s3_bucket.production.bucket
          backup_bucket = aws_s3_bucket.backup.bucket
          timestamp = "#{codepipeline.PipelineExecutionId}"
        })
      }
    }

    action {
      name             = "DeployToProduction"
      category         = "Deploy"
      owner            = "AWS"
      provider         = "S3"
      version          = "1"
      input_artifacts = ["build_output"]

      configuration = {
        BucketName = aws_s3_bucket.production.bucket
        Extract    = "true"
      }

      run_order = 2
    }

    action {
      name             = "InvalidateProductionCache"
      category         = "Invoke"
      owner            = "AWS"
      provider         = "Lambda"
      version          = "1"

      configuration = {
        FunctionName = aws_lambda_function.invalidate_cache.function_name
        UserParameters = jsonencode({
          distribution_id = aws_cloudfront_distribution.production.id
          paths = ["/*"]
        })
      }

      run_order = 3
    }

    action {
      name             = "ProductionHealthCheck"
      category         = "Invoke"
      owner            = "AWS"
      provider         = "Lambda"
      version          = "1"

      configuration = {
        FunctionName = aws_lambda_function.health_check.function_name
        UserParameters = jsonencode({
          url = "https://${aws_cloudfront_distribution.production.domain_name}"
          retries = 5
          interval = 30
        })
      }

      run_order = 4
    }
  }

  # Etapa 8: Verificación Post-Despliegue
  stage {
    name = "PostDeployment"

    action {
      name             = "ProductionTests"
      category         = "Test"
      owner            = "AWS"
      provider         = "CodeBuild"
      version          = "1"
      input_artifacts  = ["source_output"]

      configuration = {
        ProjectName = aws_codebuild_project.production_tests.name
        EnvironmentVariables = jsonencode([
          {
            name  = "TEST_URL"
            value = "https://${aws_cloudfront_distribution.production.domain_name}"
          }
        ])
      }
    }

    action {
      name             = "NotifySuccess"
      category         = "Invoke"
      owner            = "AWS"
      provider         = "Lambda"
      version          = "1"

      configuration = {
        FunctionName = aws_lambda_function.notify_deployment.function_name
        UserParameters = jsonencode({
          status = "success"
          pipeline_execution_id = "#{codepipeline.PipelineExecutionId}"
          commit_id = "#{SourceVariables.CommitId}"
        })
      }

      run_order = 2
    }
  }

  tags = var.tags
}

# Manejo de fallos de pipeline
resource "aws_cloudwatch_event_rule" "pipeline_failure" {
  name = "${var.project_name}-pipeline-failure"

  event_pattern = jsonencode({
    source      = ["aws.codepipeline"]
    detail-type = ["CodePipeline Pipeline Execution State Change"]
    detail = {
      state = ["FAILED"]
      pipeline = [aws_codepipeline.main.name]
    }
  })
}

resource "aws_cloudwatch_event_target" "pipeline_failure_lambda" {
  rule      = aws_cloudwatch_event_rule.pipeline_failure.name
  target_id = "PipelineFailureLambdaTarget"
  arn       = aws_lambda_function.handle_pipeline_failure.arn
}
```

### Configuración Avanzada del Proyecto CodeBuild

#### 1. Proyecto de Construcción Multi-etapa

```yaml
# buildspec/multi-stage-build.yml
version: 0.2

phases:
  install:
    runtime-versions:
      nodejs: 18
      python: 3.9
      docker: 20
    commands:
      - echo "🚀 Iniciando proceso de construcción multi-etapa"
      - echo "📦 Instalando dependencias de construcción..."

      # Instalar herramientas relacionadas con Node.js
      - npm install -g pnpm@8.15.0
      - npm install -g npm-check-updates

      # Instalar herramientas relacionadas con Python
      - pip install --upgrade pip
      - pip install boto3 requests

      # Instalar otras herramientas
      - curl -sSfL https://raw.githubusercontent.com/anchore/syft/main/install.sh | sh -s -- -b /usr/local/bin

  pre_build:
    commands:
      - echo "🔍 Validación previa a la construcción iniciada"

      # Validación de variables de entorno
      - |
        REQUIRED_VARS=("ENVIRONMENT" "BUILD_NUMBER" "COMMIT_SHA")
        for var in "${REQUIRED_VARS[@]}"; do
          if [ -z "${!var}" ]; then
            echo "❌ La variable de entorno requerida $var no está configurada"
            exit 1
          fi
        done
        echo "✅ Todas las variables de entorno requeridas están configuradas"

      # Instalación y caché de dependencias
      - echo "📦 Instalando dependencias..."
      - pnpm install --frozen-lockfile

      # Verificación de seguridad de dependencias
      - echo "🔒 Ejecutando auditoría de seguridad de dependencias..."
      - pnpm audit --audit-level=moderate

      # Generar información de construcción
      - |
        BUILD_INFO=$(cat << EOF
        {
          "buildNumber": "$BUILD_NUMBER",
          "environment": "$ENVIRONMENT",
          "commitSha": "$COMMIT_SHA",
          "buildTime": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
          "nodeVersion": "$(node --version)",
          "pnpmVersion": "$(pnpm --version)",
          "platform": "$(uname -a)"
        }
        EOF
        )
        echo "$BUILD_INFO" > build-info.json
        echo "📄 Información de construcción generada"

  build:
    commands:
      - echo "🏗️ Fase de construcción iniciada"

      # Verificaciones de calidad de código
      - echo "🔍 Ejecutando verificaciones de calidad de código..."
      - pnpm run lint:check
      - pnpm run format:check
      - pnpm run type-check

      # Pruebas unitarias
      - echo "🧪 Ejecutando pruebas unitarias..."
      - pnpm run test:unit --coverage --reporter=junit --outputFile=test-results.xml

      # Construir aplicación
      - echo "📦 Construyendo aplicación..."
      - pnpm run build

      # Validación post-construcción
      - echo "✅ Validación post-construcción..."
      - |
        # Verificar archivos de construcción
        if [ ! -f "dist/index.html" ]; then
          echo "❌ Archivo de entrada principal no encontrado"
          exit 1
        fi

        # Verificar tamaño de construcción
        BUNDLE_SIZE=$(du -sk dist | cut -f1)
        echo "📊 Tamaño del paquete: ${BUNDLE_SIZE}KB"

        if [ $BUNDLE_SIZE -gt 51200 ]; then  # 50MB
          echo "⚠️ El tamaño del paquete es grande: ${BUNDLE_SIZE}KB"
        fi

        # Verificar recursos clave
        ASSET_COUNT=$(find dist -name "*.js" -o -name "*.css" | wc -l)
        echo "📄 Activos generados: $ASSET_COUNT archivos"

      # Generar SBOM (Lista de Materiales de Software)
      - echo "📋 Generando SBOM..."
      - syft packages dir:. -o spdx-json > sbom.spdx.json

      # Construir imagen Docker (si es necesario)
      - |
        if [ -f "Dockerfile" ]; then
          echo "🐳 Construyendo imagen Docker..."
          docker build -t ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${BUILD_NUMBER} .
          docker build -t ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:latest .
        fi

  post_build:
    commands:
      - echo "🎯 Fase post-construcción iniciada"

      # Verificación de cobertura de pruebas
      - |
        if [ -f "coverage/lcov.info" ]; then
          COVERAGE=$(lcov --summary coverage/lcov.info 2>&1 | grep "lines" | cut -d' ' -f4 | cut -d'%' -f1)
          echo "📊 Cobertura de pruebas: ${COVERAGE}%"
          
          if (( $(echo "$COVERAGE < 80" | bc -l) )); then
            echo "⚠️ Cobertura de pruebas por debajo del umbral: ${COVERAGE}% < 80%"
          fi
        fi

      # Subir imagen Docker (si es necesario)
      - |
        if [ -f "Dockerfile" ]; then
          echo "🚢 Subiendo imagen Docker..."
          aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com
          docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${BUILD_NUMBER}
          docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:latest
        fi

      # Preparar paquete de despliegue
      - echo "🎁 Preparando paquete de despliegue..."
      - cp build-info.json dist/
      - tar -czf deployment-package.tar.gz -C dist .

      # Informe de construcción
      - |
        echo "📊 Informe de Construcción" > build-report.txt
        echo "===============" >> build-report.txt
        echo "Número de Construcción: $BUILD_NUMBER" >> build-report.txt
        echo "Entorno: $ENVIRONMENT" >> build-report.txt
        echo "Commit: $COMMIT_SHA" >> build-report.txt
        echo "Tamaño del Paquete: ${BUNDLE_SIZE}KB" >> build-report.txt
        echo "Cantidad de Activos: $ASSET_COUNT" >> build-report.txt
        if [ -n "$COVERAGE" ]; then
          echo "Cobertura de Pruebas: ${COVERAGE}%" >> build-report.txt
        fi
        echo "Tiempo de Construcción: $(date)" >> build-report.txt

      - |
        if [ $CODEBUILD_BUILD_SUCCEEDING -eq 1 ]; then
          echo "✅ Construcción completada exitosamente"
        else
          echo "❌ Construcción fallida"
          exit 1
        fi

artifacts:
  files: 
    - "**/*"
  base-directory: dist
  name: build-$BUILD_NUMBER

secondary-artifacts:
  BuildReport:
    files:
      - build-report.txt
      - build-info.json
      - test-results.xml
      - sbom.spdx.json
    name: build-artifacts-$BUILD_NUMBER

  Coverage:
    files:
      - "coverage/**/*"
    name: coverage-$BUILD_NUMBER

reports:
  unit-tests:
    files:
      - "test-results.xml"
    file-format: "JUNITXML"

coverage-reports:
  coverage-reports:
    files:
      - "coverage/lcov.info"
    file-format: "CLOVERXML"

cache:
  paths:
    - "node_modules/**/*"
    - ".pnpm-store/**/*"
    - "/root/.cache/pip/**/*"
```

#### 2. Configuraciones de Construcción Específicas del Entorno

```terraform
# Proyecto CodeBuild - Configuración específica del entorno
resource "aws_codebuild_project" "build" {
  for_each = var.environments

  name          = "${var.project_name}-build-${each.key}"
  description   = "Proyecto de construcción para el entorno ${each.key}"
  service_role  = aws_iam_role.codebuild_role.arn

  artifacts {
    type = "CODEPIPELINE"
  }

  environment {
    compute_type                = each.value.compute_type
    image                      = "aws/codebuild/standard:7.0"
    type                       = "LINUX_CONTAINER"
    image_pull_credentials_type = "CODEBUILD"
    privileged_mode            = true

    dynamic "environment_variable" {
      for_each = merge(
        var.common_env_vars,
        each.value.env_vars,
        {
          ENVIRONMENT = each.key
          BUILD_TARGET = each.value.build_target
          API_ENDPOINT = each.value.api_endpoint
        }
      )
      content {
        name  = environment_variable.key
        value = environment_variable.value
      }
    }

    # Variables de entorno sensibles
    dynamic "environment_variable" {
      for_each = each.value.secret_env_vars
      content {
        name  = environment_variable.key
        value = environment_variable.value
        type  = "PARAMETER_STORE"
      }
    }
  }

  source {
    type = "CODEPIPELINE"
    buildspec = templatefile("${path.module}/buildspec/${each.value.buildspec_file}", {
      environment = each.key
      node_version = each.value.node_version
      build_commands = each.value.build_commands
    })
  }

  vpc_config {
    vpc_id = each.value.vpc_id
    subnets = each.value.subnet_ids
    security_group_ids = [aws_security_group.codebuild[each.key].id]
  }

  logs_config {
    cloudwatch_logs {
      group_name  = aws_cloudwatch_log_group.codebuild[each.key].name
      stream_name = "build-log"
    }

    s3_logs {
      status   = "ENABLED"
      location = "${aws_s3_bucket.build_logs.bucket}/build-logs/${each.key}"
    }
  }

  tags = merge(var.tags, {
    Environment = each.key
    Purpose     = "Build"
  })
}

# Variables de configuración de entorno
variable "environments" {
  description = "Configuraciones específicas del entorno"
  type = map(object({
    compute_type     = string
    build_target     = string
    api_endpoint     = string
    node_version     = string
    buildspec_file   = string
    vpc_id          = string
    subnet_ids      = list(string)
    env_vars        = map(string)
    secret_env_vars = map(string)
    build_commands  = list(string)
  }))

  default = {
    development = {
      compute_type     = "BUILD_GENERAL1_SMALL"
      build_target     = "development"
      api_endpoint     = "https://api-dev.myapp.com"
      node_version     = "18"
      buildspec_file   = "development.yml"
      vpc_id          = "vpc-dev123"
      subnet_ids      = ["subnet-dev1", "subnet-dev2"]
      env_vars = {
        DEBUG_MODE = "true"
        LOG_LEVEL  = "debug"
      }
      secret_env_vars = {
        DEV_API_KEY = "/myapp/dev/api-key"
      }
      build_commands = [
        "npm run build:dev",
        "npm run test:unit"
      ]
    }

    staging = {
      compute_type     = "BUILD_GENERAL1_MEDIUM"
      build_target     = "staging"
      api_endpoint     = "https://api-staging.myapp.com"
      node_version     = "18"
      buildspec_file   = "staging.yml"
      vpc_id          = "vpc-staging123"
      subnet_ids      = ["subnet-staging1", "subnet-staging2"]
      env_vars = {
        DEBUG_MODE = "false"
        LOG_LEVEL  = "info"
      }
      secret_env_vars = {
        STAGING_API_KEY = "/myapp/staging/api-key"
      }
      build_commands = [
        "npm run build:staging",
        "npm run test:unit",
        "npm run test:integration"
      ]
    }

    production = {
      compute_type     = "BUILD_GENERAL1_LARGE"
      build_target     = "production"
      api_endpoint     = "https://api.myapp.com"
      node_version     = "18"
      buildspec_file   = "production.yml"
      vpc_id          = "vpc-prod123"
      subnet_ids      = ["subnet-prod1", "subnet-prod2"]
      env_vars = {
        DEBUG_MODE = "false"
        LOG_LEVEL  = "warn"
        OPTIMIZE   = "true"
      }
      secret_env_vars = {
        PROD_API_KEY    = "/myapp/production/api-key"
        SENTRY_DSN      = "/myapp/production/sentry-dsn"
        ANALYTICS_KEY   = "/myapp/production/analytics-key"
      }
      build_commands = [
        "npm run build:production",
        "npm run test:unit",
        "npm run test:integration",
        "npm run test:e2e"
      ]
    }
  }
}
```

## Implementación de la Puerta de Revisión de Negocio

Finalmente, en un entorno empresarial, el despliegue en producción debe pasar por una rigurosa **revisión de confirmación de lógica de negocio** antes de que pueda incorporarse al lanzamiento oficial del producto de la empresa. Ahora implementaremos un mecanismo de revisión que combina una **Puerta de Revisión por Pares**, una **Puerta de Revisión de Negocio** y una **Puerta de Revisión de Calidad**.

```mermaid
graph TD
    subgraph "Revisión Inteligente y Notificación"
        A[Solicitud de Despliegue Activada] --> B{Analizar Riesgo y Tipo de Cambio}
        B --> C{Determinar Ruta de Aprobación}
        C --> D[Crear Registro de Aprobación<br/>usando DynamoDB]
        D --> E[Enviar Notificaciones Multi-canal<br/>Slack y Correo Electrónico]
    end

    subgraph "Puertas de Aprobación Multi-Nivel"
        E --> F{Revisión Técnica Requerida?}
        F -->|Sí| G[Revisión del Líder Técnico<br/>vía Slack/UI]
        F -->|No| J{Revisión de Negocio Requerida?}
        G --> H{Aprobado?}
        H -->|No| Z[Aprobación Fallida]
        H -->|Sí| J
        J -->|Sí| K[Revisión del Gerente de Producto<br/>vía Slack/UI]
        J -->|No| P{Revisión de Seguridad Requerida?}
        K --> L{Aprobado?}
        L -->|No| Z
        L -->|Sí| P
        P -->|Sí| Q[Revisión del Equipo de Seguridad<br/>vía Slack/UI]
        P -->|No| T[Todas las Aprobaciones Completas]

        Q --> R{Aprobado?}
        R -->|No| Z
        R -->|Sí| T
    end

    subgraph "Autorización y Ejecución Final"
        T --> U[Autorización Final de Despliegue]
        U --> V[✅ Activar Despliegue en Producción]
        Z --> W[❌ Notificar Fallo y Razón]
    end

    style A fill:#e3f2fd,stroke:#333,stroke-width:2px
    style V fill:#e8f5e9,stroke:#333,stroke-width:2px
    style W fill:#ffebee,stroke:#333,stroke-width:2px
    style Z fill:#ffebee,stroke:#333,stroke-width:2px
```

### 1. Proceso de Aprobación Multi-nivel

```yaml
# .github/workflows/approval-workflow.yml
name: "Aprobación de Despliegue en Producción"

on:
  workflow_call:
    inputs:
      deployment-version:
        required: true
        type: string
      staging-url:
        required: true
        type: string

jobs:
  # Primera capa: Revisión Técnica
  technical-review:
    name: "Revisión Técnica"
    runs-on: ubuntu-latest

    environment:
      name: technical-approval

    steps:
      - name: Validación de lista de verificación técnica
        run: |
          echo "🔍 Lista de Verificación Técnica"
          echo "=============================="
          echo "📦 Versión: ${{ inputs.deployment-version }}"
          echo "🌐 URL de Staging: ${{ inputs.staging-url }}"
          echo ""
          echo "Por favor, verifique:"
          echo "✓ Todas las pruebas pasan"
          echo "✓ Revisión de código completada"
          echo "✓ Escaneo de seguridad aprobado"
          echo "✓ Puntos de referencia de rendimiento alcanzados"
          echo "✓ Documentación actualizada"
          echo ""
          echo "Aprobado por: Líder Técnico"
```

### 2. Reglas de Aprobación Inteligentes

```yaml
# .github/workflows/smart-approval.yml
name: "Lógica de Aprobación Inteligente"

on:
  workflow_call:
    inputs:
      change-type:
        required: true
        type: string
      risk-level:
        required: true
        type: string
      affected-components:
        required: true
        type: string

jobs:
  determine-approval-path:
    name: "Determinar Requisitos de Aprobación"
    runs-on: ubuntu-latest

    outputs:
      requires-technical: ${{ steps.approval-logic.outputs.requires-technical }}
      requires-business: ${{ steps.approval-logic.outputs.requires-business }}
      requires-security: ${{ steps.approval-logic.outputs.requires-security }}
      requires-emergency: ${{ steps.approval-logic.outputs.requires-emergency }}

    steps:
      - name: Analizar requisitos de cambio
        id: approval-logic
        run: |
          CHANGE_TYPE="${{ inputs.change-type }}"
          RISK_LEVEL="${{ inputs.risk-level }}"
          AFFECTED_COMPONENTS="${{ inputs.affected-components }}"

          # Requisitos de aprobación predeterminados
          REQUIRES_TECHNICAL="true"
          REQUIRES_BUSINESS="false"
          REQUIRES_SECURITY="false"
          REQUIRES_EMERGENCY="false"

          echo "📊 Analizando requisitos de cambio..."
          echo "Tipo de Cambio: $CHANGE_TYPE"
          echo "Nivel de Riesgo: $RISK_LEVEL"
          echo "Componentes Afectados: $AFFECTED_COMPONENTS"

          # Juicio del nivel de riesgo
          case "$RISK_LEVEL" in
            "low")
              REQUIRES_BUSINESS="false"
              REQUIRES_SECURITY="false"
              ;;;
            "medium")
              REQUIRES_BUSINESS="true"
              REQUIRES_SECURITY="false"
              ;;;
            "high"|"critical")
              REQUIRES_BUSINESS="true"
              REQUIRES_SECURITY="true"
              ;;;
          esac

          # Juicio del tipo de cambio
          case "$CHANGE_TYPE" in
            "hotfix"|"emergency")
              REQUIRES_EMERGENCY="true"
              REQUIRES_BUSINESS="false"
              REQUIRES_SECURITY="false"
              ;;;
            "security-patch")
              REQUIRES_SECURITY="true"
              ;;;
            "feature")
              REQUIRES_BUSINESS="true"
              ;;;
          esac

          # Juicio de componentes afectados
          if [[ "$AFFECTED_COMPONENTS" == *"authentication"* ]] || \
             [[ "$AFFECTED_COMPONENTS" == *"payment"* ]] || \
             [[ "$AFFECTED_COMPONENTS" == *"user-data"* ]]; then
            REQUIRES_SECURITY="true"
          fi

          if [[ "$AFFECTED_COMPONENTS" == *"database"* ]] || \
             [[ "$AFFECTED_COMPONENTS" == *"infrastructure"* ]]; then
            REQUIRES_SECURITY="true"
            REQUIRES_BUSINESS="true"
          fi

          # Resultados de salida
          echo "requires-technical=$REQUIRES_TECHNICAL" >> $GITHUB_OUTPUT
          echo "requires-business=$REQUIRES_BUSINESS" >> $GITHUB_OUTPUT
          echo "requires-security=$REQUIRES_SECURITY" >> $GITHUB_OUTPUT
          echo "requires-emergency=$REQUIRES_EMERGENCY" >> $GITHUB_OUTPUT

          echo ""
          echo "📋 Requisitos de Aprobación:"
          echo "Revisión Técnica: $REQUIRES_TECHNICAL"
          echo "Revisión de Negocio: $REQUIRES_BUSINESS"
          echo "Revisión de Seguridad: $REQUIRES_SECURITY"
          echo "Proceso de Emergencia: $REQUIRES_EMERGENCY"

  conditional-approvals:
    name: "Ejecutar Aprobaciones Requeridas"
    needs: determine-approval-path
    uses: ./.github/workflows/conditional-approval.yml
    with:
      requires-technical: ${{ needs.determine-approval-path.outputs.requires-technical }}
      requires-business: ${{ needs.determine-approval-path.outputs.requires-business }}
      requires-security: ${{ needs.determine-approval-path.outputs.requires-security }}
      requires-emergency: ${{ needs.determine-approval-path.outputs.requires-emergency }}
```

### 3. Notificación y Seguimiento de Aprobaciones

```python
# scripts/approval-notification.py
import json
import boto3
import requests
from datetime import datetime, timedelta

class ApprovalNotificationManager:
    def __init__(self):
        self.sns = boto3.client('sns')
        self.dynamodb = boto3.resource('dynamodb')
        self.approval_table = self.dynamodb.Table('approval-tracking')

    def create_approval_request(self, deployment_info):
        """Crear una solicitud de aprobación"""
        approval_id = f"approval-${deployment_info['build_number']}-${int(datetime.now().timestamp())}"

        approval_record = {
            'approval_id': approval_id,
            'deployment_version': deployment_info['version'],
            'staging_url': deployment_info['staging_url'],
            'requester': deployment_info['requester'],
            'created_at': datetime.now().isoformat(),
            'expires_at': (datetime.now() + timedelta(hours=24)).isoformat(),
            'status': 'pending',
            'required_approvals': deployment_info['required_approvals'],
            'completed_approvals': [],
            'change_summary': deployment_info['change_summary']
        }

        # Guardar en DynamoDB
        self.approval_table.put_item(Item=approval_record)

        # Enviar notificaciones
        self.send_approval_notifications(approval_record)

        return approval_id

    def send_approval_notifications(self, approval_record):
        """Enviar notificaciones de aprobación"""

        # Notificación por correo electrónico
        self.send_email_notification(approval_record)

        # Notificación de Slack
        self.send_slack_notification(approval_record)

        # Notificación de Teams
        self.send_teams_notification(approval_record)

    def send_slack_notification(self, approval_record):
        """Enviar notificación de Slack"""
        webhook_url = os.environ.get('SLACK_WEBHOOK_URL')

        required_approvals = approval_record['required_approvals']
        approval_buttons = []

        for approval_type in required_approvals:
            approval_buttons.append({
                "type": "button",
                "text": {
                    "type": "plain_text",
                    "text": f"Aprobar {approval_type.title()}"
                },
                "action_id": f"approve_{approval_type}",
                "value": approval_record['approval_id'],
                "style": "primary"
            })

        message = {
            "blocks": [
                {
                    "type": "header",
                    "text": {
                        "type": "plain_text",
                        "text": "🚀 Aprobación de Despliegue a Producción Requerida"
                    }
                },
                {
                    "type": "section",
                    "fields": [
                        {
                            "type": "mrkdwn",
                            "text": f"*Versión:* {approval_record['deployment_version']}"
                        },
                        {
                            "type": "mrkdwn",
                            "text": f"*Solicitante:* {approval_record['requester']}"
                        },
                        {
                            "type": "mrkdwn",
                            "text": f"*URL de Staging:* <{approval_record['staging_url']}|Ver Staging>"
                        },
                        {
                            "type": "mrkdwn",
                            "text": f"*Expira:* {approval_record['expires_at']}"
                        }
                    ]
                },
                {
                    "type": "section",
                    "text": {
                        "type": "mrkdwn",
                        "text": f"*Resumen del Cambio:*
{approval_record['change_summary']}"
                    }
                },
                {
                    "type": "section",
                    "text": {
                        "type": "mrkdwn",
                        "text": f"*Aprobaciones Requeridas:* {', '.join(required_approvals)}"
                    }
                },
                {
                    "type": "actions",
                    "elements": approval_buttons + [
                        {
                            "type": "button",
                            "text": {
                                "type": "plain_text",
                                "text": "Rechazar"
                            },
                            "action_id": "reject_deployment",
                            "value": approval_record['approval_id'],
                            "style": "danger"
                        }
                    ]
                }
            ]
        }

        response = requests.post(webhook_url, json=message)
        return response.status_code == 200

    def process_approval_response(self, approval_id, approval_type, approver, decision):
        """Procesar respuesta de aprobación"""

        # Obtener registro de aprobación
        response = self.approval_table.get_item(Key={'approval_id': approval_id})
        approval_record = response.get('Item')

        if not approval_record:
            return {'error': 'Registro de aprobación no encontrado'}

        if approval_record['status'] != 'pending':
            return {'error': 'Aprobación ya completada o expirada'}

        # Actualizar estado de aprobación
        if decision == 'approve':
            approval_record['completed_approvals'].append({
                'type': approval_type,
                'approver': approver,
                'timestamp': datetime.now().isoformat(),
                'decision': 'approved'
            })
        else:
            approval_record['status'] = 'rejected'
            approval_record['rejected_by'] = approver
            approval_record['rejected_at'] = datetime.now().isoformat()

        # Verificar si todas las aprobaciones necesarias están completas
        required_approvals = set(approval_record['required_approvals'])
        completed_approvals = set([a['type'] for a in approval_record['completed_approvals']])

        if decision == 'approve' and required_approvals.issubset(completed_approvals):
            approval_record['status'] = 'approved'
            approval_record['approved_at'] = datetime.now().isoformat()

            # Activar proceso de despliegue
            self.trigger_deployment(approval_record)

        # Actualizar registro de la base de datos
        self.approval_table.put_item(Item=approval_record)

        # Enviar notificación de actualización de estado
        self.send_status_update(approval_record, approval_type, approver, decision)

        return {'status': approval_record['status']}

    def trigger_deployment(self, approval_record):
        """Activar despliegue a producción"""

        # Activar flujo de trabajo de despliegue a través de la API de GitHub
        github_token = os.environ.get('GITHUB_TOKEN')
        repo_owner = os.environ.get('GITHUB_REPO_OWNER')
        repo_name = os.environ.get('GITHUB_REPO_NAME')

        headers = {
            'Authorization': f'Bearer {github_token}',
            'Accept': 'application/vnd.github.v3+json'
        }

        workflow_data = {
            'ref': 'main',
            'inputs': {
                'deployment_version': approval_record['deployment_version'],
                'approval_id': approval_record['approval_id'],
                'approved_by': ', '.join([a['approver'] for a in approval_record['completed_approvals']])
            }
        }

        url = f"https://api.github.com/repos/{repo_owner}/{repo_name}/actions/workflows/production-deploy.yml/dispatches"
        response = requests.post(url, json=workflow_data, headers=headers)

        return response.status_code == 204

    def send_status_update(self, approval_record, approval_type, approver, decision):
        """Enviar notificación de actualización de estado"""

        status_emoji = "✅" if decision == "approve" else "❌"
        status_text = "aprobado" if decision == "approve" else "rechazado"

        if approval_record['status'] == 'approved':
            message = f"🚀 ¡Despliegue aprobado y activado!\n"
            message += f"Versión: {approval_record['deployment_version']}\n"
            message += f"Todas las aprobaciones requeridas completadas."
        elif approval_record['status'] == 'rejected':
            message = f"❌ Despliegue rechazado por {approver}\n"
            message += f"Versión: {approval_record['deployment_version']}\n"
            message += f"Razón: Aprobación de {approval_type} denegada."
        else:
            remaining_approvals = set(approval_record['required_approvals']) - set([a['type'] for a in approval_record['completed_approvals']])
            message = f"{status_emoji} {approval_type.title()} {status_text} por {approver}\n"
            message += f"Versión: {approval_record['deployment_version']}\n"
            message += f"Aprobaciones restantes necesarias: {', '.join(remaining_approvals)}"

        # Enviar actualización de Slack
        webhook_url = os.environ.get('SLACK_WEBHOOK_URL')
        requests.post(webhook_url, json={"text": message})

# Función manejadora de Lambda
def lambda_handler(event, context):
    manager = ApprovalNotificationManager()

    if event.get('action') == 'create_approval':
        approval_id = manager.create_approval_request(event['deployment_info'])
        return {'approval_id': approval_id}

    elif event.get('action') == 'process_response':
        result = manager.process_approval_response(
            event['approval_id'],
            event['approval_type'],
            event['approver'],
            event['decision']
        )
        return result

    else:
        return {'error': 'Acción desconocida'}
```

## Construyendo un Pipeline de Entrega de Software Moderno

La automatización completa de CI/CD trae cambios fundamentales al desarrollo de software moderno. Partimos de los puntos débiles del despliegue manual tradicional y profundizamos en la construcción de un pipeline de automatización completo, con un enfoque especial en los requisitos a nivel empresarial:

### Reafirmando los Valores Fundamentales

**Transformación Tecnológica**:

- **Gestión de la Fragmentación de Tareas**: Dividir razonablemente las tareas para mejorar la eficiencia de ejecución y la mantenibilidad.
- **Gestión del Control de Versiones de Tareas**: Controlar las versiones del proceso de CI/CD para asegurar la trazabilidad.
- **Gestión de la Abstracción de Parámetros**: Separar las configuraciones de entorno para soportar el despliegue en múltiples entornos.
- **Puerta de Revisión de Negocio**: Integrar la revisión manual en la automatización para equilibrar la eficiencia y el riesgo.

**Impacto Organizacional**:

- **Equilibrio entre Eficiencia y Calidad**: La automatización mejora la eficiencia, mientras que los mecanismos de revisión aseguran la calidad.
- **Agilidad con Riesgo Controlado**: Un equilibrio entre la entrega rápida y el control de riesgos.
- **Colaboración en Equipo Optimizada**: Procesos claros y división de responsabilidades.
- **Cultura de Mejora Continua**: Optimización de procesos basada en datos.

### Factores Clave de Éxito para la Implementación

#### 1. Estrategia de Automatización Incremental

- Comenzar a automatizar con tareas sencillas.
- Aumentar gradualmente la complejidad y la cobertura.
- Establecer mecanismos de monitoreo y alerta.
- Optimizar continuamente la eficiencia del proceso.

#### 2. Mecanismo de Revisión Inteligente

- Revisión dinámica basada en el nivel de riesgo.
- Gestión automatizada del proceso de revisión.
- Notificación y seguimiento multicanal.
- Registro completo del historial de revisiones.

#### 3. Capacidades de Integración a Nivel Empresarial

- Integración perfecta con los sistemas existentes.
- Soporte para entornos multi-nube.
- Consideración de la seguridad y el cumplimiento.
- Diseño arquitectónico escalable.

### La Evolución del Despliegue Manual a la Entrega Inteligente

**Fase 1: Despliegue Manual** → **Fase 2: Pipeline Automatizado** → **Fase 3: Entrega Inteligente**

Ahora estamos entrando en la tercera fase, caracterizada por:

- **Decisiones Inteligentes**: Decisiones automatizadas basadas en datos históricos y monitoreo en tiempo real.
- **Procesos Adaptativos**: Ajuste automático basado en las características del proyecto y los hábitos del equipo.
- **Mantenimiento Predictivo**: Identificación y prevención proactiva de problemas potenciales.
- **Aprendizaje Continuo**: Aprender de cada despliegue y optimizar el proceso.

### Perspectivas Futuras: CI/CD Impulsado por IA

- **Selección Inteligente de Pruebas**: La IA analiza los cambios de código y selecciona inteligentemente las estrategias de prueba.
- **Diagnóstico Automatizado de Problemas**: Los modelos de aprendizaje automático analizan automáticamente las razones de los fallos.
- **Despliegue Predictivo**: Optimización del momento del despliegue basada en la evaluación de riesgos.
- **Gestión Inteligente de Recursos**: Ajustar dinámicamente los recursos informáticos para optimizar los costos.

**CI/CD nos permite ser no solo "operadores que despliegan código", sino "arquitectos que diseñan sistemas de entrega inteligentes."**

Este es el verdadero valor de la automatización completa de CI/CD: no solo hace que la entrega de software sea confiable, rápida y predecible, sino que, lo que es más importante, establece un sistema inteligente capaz de aprendizaje y mejora continuos. En este entorno tecnológico que cambia rápidamente, dominar las prácticas modernas de CI/CD se ha convertido en una competencia central para cada equipo de software.

A través de un pipeline de CI/CD completo, logramos la automatización total desde la subida de código hasta el despliegue en producción, manteniendo la revisión manual necesaria y el control de riesgos. Este equilibrio nos permite disfrutar de las ganancias de eficiencia de la automatización al tiempo que garantizamos la seguridad y estabilidad del negocio. Este es el núcleo de la ingeniería de software moderna: **la fusión perfecta de tecnología y negocio**.