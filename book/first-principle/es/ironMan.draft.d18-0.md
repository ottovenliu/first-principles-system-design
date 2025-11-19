# Día 18 | Formulación de Criterios de Aceptación del Sistema: De la Lógica de Verificación al Manual de Aceptación Funcional: Diseño del Proceso UAT y Formulación de Estándares de Calidad

Al iniciar el primer día de la fase de "Verificación y Aseguramiento de la Calidad", exploraremos un tema que a menudo se pasa por alto pero que es extremadamente crítico: **"Formulación de Criterios de Aceptación del Sistema"**.

Después de 17 días de discusión, hemos establecido un proceso de desarrollo completo: desde la **confirmación de requisitos**, el **diseño del sistema**, la **planificación técnica** hasta la **integración continua**. Finalmente, nos enfrentamos a una pregunta crítica:

> **"¿Cómo determinamos que el sistema que hemos construido realmente cumple con las expectativas comerciales iniciales?"**

La "finalización" del software no la determinan los desarrolladores, sino los "criterios de aceptación". Si los capítulos anteriores nos enseñaron **"cómo hacer las cosas bien (Do things right)"**, entonces la formulación de los criterios de aceptación asegura que **"hagamos lo correcto (Do the right thing)"**.

¿Recuerdas el tema <Diseño de Colaboración entre Equipos: Documentación Técnica, OpenAPI, Contratos Compartidos> que discutimos el día 13? En ese tema dijimos:

> Después de que la lógica de negocio central se implementa inicialmente, podemos realizar una exploración colaborativa del dominio de negocio basada en la información existente a través de varios métodos como reuniones o talleres. El objetivo es establecer un lenguaje y una comprensión comunes de los procesos de negocio entre los equipos. Esto asegura que **`"estamos construyendo lo correcto"`**, para que al entrar en **`"construir la cosa correctamente"`** no terminemos yendo en direcciones completamente opuestas.

Aunque la formulación inicial del contrato compartido no mencionó específicamente si el equipo de pruebas participa, antes de la entrega del sistema, aún debemos completar un proceso de **Verificación y Aseguramiento de la Calidad**, incluso si estamos seguros de nuestro repertorio de rendimiento, ¿no deberíamos verificar si nuestros instrumentos están afinados antes de la actuación?

Durante los próximos 4 días (incluido hoy), compartiremos y comprenderemos estos cuatro procesos: `Pautas de diseño de reglas de prueba => Pruebas ciegas de usuario real => Pruebas de interfaz de usuario (front-end) => Pruebas de rendimiento de back-end`. En la etapa anterior (Desarrollo de Software e Integración Continua), si hicimos crecer y desarrollamos el sistema de adentro hacia afuera, a continuación verificaremos continuamente si la **lógica de negocio** se ha implementado con éxito de la piel a la carne.

Este es un arte de **"Compromiso"** y **"Confianza"**, la clave para asegurar que nuestro producto final entregado realmente resuelva los problemas del cliente. En la industria, por impresionante que sea la tecnología, si lo que se construye no es lo que los clientes quieren, todo es un esfuerzo desperdiciado. Esto lleva a la distinción fundamental entre dos conceptos centrales: **Verificación** y **Validación**.

- La **Verificación** pregunta: "¿Estamos construyendo el sistema correctamente?" (Are we building the system right?). Se centra en si el sistema cumple con las especificaciones técnicas, los documentos de diseño y los estándares de codificación. Este es un examen interno que asegura la precisión de la ejecución técnica. Las pruebas unitarias y las pruebas de integración pertenecen a esta categoría.

- La **Validación** pregunta: "¿Estamos construyendo el sistema correcto?" (Are we building the right system?). Se centra en si el sistema satisface las necesidades comerciales reales y las expectativas del usuario. Este es un examen externo que asegura el valor comercial y la practicidad del producto.

A continuación, imaginemos que estamos **construyendo una casa** juntos, desde el plano soñado de un cliente (**confirmación de requisitos**) hasta su entrada satisfecha con las llaves en la mano (**entrega del sistema**).

Podemos desglosar todo el proceso en cuatro pasos interconectados:

1. ¿Por qué aceptamos de esta manera? (El Porqué): Lógica de Verificación
2. ¿Qué aceptamos específicamente? (El Qué): Criterios de Aceptación del Sistema
3. ¿Cómo ejecutamos la aceptación? (El Cómo): Manual de Aceptación Funcional (Manual UAT) y Proceso
4. ¿Qué resultados cuentan como "aprobados"? (El Qué Tan Bien): Estándares de Calidad

## Cambio de Mentalidad de "Entrega Funcional" a "Aceptación de Valor"

Este es un cambio de mentalidad fundamental.

Cuando los clientes firman el contrato diciendo que quieren una "casa cómoda, segura y moderna", esto es un **requisito**.

Pero, ¿qué es "cómodo"? ¿Qué es "seguro"?

A través de contratos compartidos, obtuvimos reconocimiento de límites y configuraciones de escenarios para la implementación del negocio. En este proceso, UI/UX también participó y produjo un borrador de diseño que contenía configuraciones de UI y procesos de UX. Parece bien, ¿verdad?

En el pasado, medíamos el éxito del proyecto por "¿cuántas funciones entregamos?". Esta es la mentalidad de **Entrega de Funciones**. Como construir una casa, les decimos a los clientes: "Mira, el contrato especificaba 10 ventanas, 3 habitaciones, 2 baños; construimos todo, ni una más, ni una menos". Nuestro trabajo está hecho.

Pero, ¿están realmente satisfechos los clientes?

Aunque el número de habitaciones es correcto, las habitaciones orientadas al oeste son hornos en verano, completamente inhabitables; aunque el número de ventanas es correcto, al abrirlas solo se ven las paredes de los vecinos sin ninguna vista.

La **Lógica de Verificación** es el proceso de pensamiento para transformar estos conceptos abstractos en verificables. Este es el paso más central, más abstracto y más importante.

> El cliente dice: "Necesito una puerta segura".
>
> Nuestra lógica de verificación debe pensar:
>
> - "Si" una puerta es segura, "entonces" debe satisfacer las siguientes condiciones:
>   1. Debe tener una cerradura que no pueda ser forzada fácilmente.
>   2. Su material debe ser sólido, no rompible con un solo impacto.
>   3. El marco de la puerta y la puerta misma deben encajar perfectamente sin huecos excesivos.

¿Ves este patrón de pensamiento **"Si-Entonces"**? Esta es la lógica de verificación. Su objetivo es establecer relaciones causales y fundamentos de juicio. **No estamos escribiendo casos de prueba**; estamos **definiendo "qué cuenta como correcto"**. Esta lógica se deriva directamente de las necesidades y riesgos del negocio; la lógica de verificación de "seguridad" para una APP bancaria es muy diferente a la de una APP de juegos.

Muchos fracasos de proyectos se originan aquí. Los **equipos de desarrollo** y los **clientes** tienen imaginaciones completamente diferentes de lo que es "seguridad". Solo se descubre en la entrega final que los clientes querían una puerta de bóveda bancaria, pero solo les dimos una puerta de madera ordinaria, o exactamente lo contrario: diseñar puertas de entrada de Disney con especificaciones de 2012. La **Lógica de Verificación** se trata de `alinear la "imaginación"` antes de que comience la construcción. Aunque los PM y los socios de UI/UX nos han ayudado a establecer configuraciones de escenarios y límites de desarrollo tanto como sea posible, la mentalidad de **Aceptación de Valor** aún debe preguntarse continuamente durante el desarrollo y la configuración del tema de prueba:

> "¿El sistema que estamos entregando realmente resuelve los problemas de los usuarios?"
>
> "¿El sistema que estamos entregando realiza la lógica de negocio esperada (contrato compartido)?"

### Mentalidad Tradicional: Orientada a la "Entrega Funcional"

En los modos de desarrollo tradicionales, la "finalización" generalmente se define como:

```
Desarrollador: "¡La funcionalidad de inicio de sesión está lista!"
Tester: "De acuerdo, probaré si hay algún error."
Gerente de Producto: "¿Están los errores corregidos? Si no, podemos salir en vivo."
```

El problema con esta mentalidad es que asume que **"ningún error técnico" equivale a "cumple con los requisitos comerciales"**. Pero la realidad a menudo es cruel:

- El sistema podría funcionar perfectamente, pero la experiencia del usuario es terrible.
- Todas las funciones se implementan de acuerdo con los documentos de requisitos, pero el proceso general no coincide con la lógica de negocio real (comúnmente llamado cambios de requisitos, incluso si los borradores iniciales de diseño de UX fueron confirmados).
- Se cumplen las métricas técnicas, pero el valor comercial no se puede demostrar.

### Mentalidad Moderna: Orientada a la "Aceptación de Valor"

En contraste, la mentalidad orientada a la "Aceptación de Valor" redefine la "finalización" como:

```
Gerente de Producto: "Necesitamos verificar si esta función realmente resuelve los puntos débiles de los usuarios."
Interesados: "Permítanme experimentar personalmente el viaje completo del usuario."
Equipo de Desarrollo: "Confirmemos juntos si esta solución logra los objetivos comerciales esperados."
```

El núcleo de este cambio de mentalidad es la mejora de los criterios de evaluación, pasando de la **"corrección técnica"** a la **"realización del valor comercial (Dominio)"**.