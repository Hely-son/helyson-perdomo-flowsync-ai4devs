# Prompts

Aquí van **todos los prompts que lanzaste** para hacer el ejercicio, en el orden en que los
lanzaste, con el modelo y la herramienta de cada uno.

Esto no es papeleo. Lo que se revisa es **cómo pediste las cosas**, no solo lo que salió: un
resultado flojo con un prompt bueno y un resultado flojo con un prompt vago necesitan feedback
distinto, y sin este archivo no se distinguen.

## Cómo rellenarlo

- Un apartado `## Prompt N` por cada prompt.
- **Pega el prompt tal cual lo lanzaste**, dentro del bloque de código, aunque ocupe diez líneas
  y aunque tenga faltas. No lo reescribas para que quede bien: el que arreglaste mentalmente
  después no es el que lanzaste.
- Incluye también los que **no funcionaron**. Suelen ser los más útiles de leer.
- `Modelo` y `Herramienta` en todos. Si cambiaste de una a otra a mitad, se nota aquí.

Borra el ejemplo de abajo cuando escribas el primero.

---

## Prompt 1

**Modelo:** Opus 1M xHigh
**Herramienta:** Claude Code

```
Analiza el proyecto completo y explícame su arquitectura actual.

Quiero entender cómo está estructurado, cómo se ejecuta localmente, cómo funciona la persistencia de datos, cómo se ejecutan las pruebas y qué herramientas utiliza actualmente para desarrollo y testing.

No modifiques ningún archivo.

Al finalizar, indícame:
- estructura relevante del proyecto
- tecnología utilizada
- cómo se configura la base de datos
- cómo se ejecutan las migraciones
- cómo se ejecutan las pruebas
- archivos de configuración relevantes
- posibles puntos de atención que debería conocer antes de realizar cambios
```

**Qué salió:** funcionó a la primera: AdonisJS 7 + Lucid 22 sobre SQLite (`tmp/db.sqlite3`), 4 migraciones, 23 tests functional en verde y typecheck en verde como línea base; avisó de que dev y tests compartían el mismo fichero SQLite.

## Prompt 2

**Modelo:** Opus 1M xHigh
**Herramienta:** Claude Code

```
Quiero entender mejor cómo funciona actualmente la persistencia de datos.

Analiza la configuración de la base de datos, el esquema, las migraciones existentes y la forma en que la aplicación y las pruebas se conectan a ella.

No modifiques ningún archivo.

Quiero saber específicamente:

- qué motor de base de datos utiliza actualmente
- dónde se configura
- cómo se inicializa
- cómo se ejecutan las migraciones
- cómo se configura la base utilizada por las pruebas
- si desarrollo y pruebas utilizan la misma base o bases independientes
- qué archivos intervienen en esta configuración

También identifica cualquier aspecto de la configuración actual que pueda ser importante si en el futuro se cambiara el motor de base de datos.
```

**Qué salió:** funcionó a la primera: una sola conexión `sqlite` sin override por entorno (`.env.test` solo cambia la sesión) y señaló `schema_rules.ts`, que fija `due_date` como `string`, como lo que más importa al cambiar de motor.

## Prompt 3

**Modelo:** Opus 1M xHigh
**Herramienta:** Claude Code

```
Necesito cambiar la persistencia actual para utilizar PostgreSQL ejecutándose mediante Docker.

Antes de realizar cambios, analiza qué tendría que modificarse en el proyecto para conseguirlo.

Quiero mantener separadas la base utilizada durante el desarrollo y la utilizada durante las pruebas.

Analiza:

- configuración de la aplicación
- configuración de pruebas
- migraciones
- variables de entorno
- Docker
- Makefile
- posibles cambios en dependencias
- posibles impactos sobre el código existente

No implementes todavía.

Propón una estrategia de cambio y señala los riesgos o incompatibilidades que encuentres.
```

**Qué salió:** funcionó a la primera: estrategia `pg` por variables de entorno, `.env.test` para la base de pruebas, cambiar `better-sqlite3` por `pg`, y como riesgo que el driver `pg` devuelve las columnas `date` como `Date` y no como texto.


## Prompt 4

**Modelo:** Opus 1M xHigh
**Herramienta:** Claude Code

```
Quiero implementar la solución propuesta anteriormente, pero hay algunas restricciones que debes respetar.

El entorno Docker debe cumplir:

- El fichero de Compose debe llamarse `compose.yaml`.
- Los servicios deben llamarse `db` y `db-test`.
- Ambos deben utilizar `pgvector/pgvector:pg17`.
- El puerto de desarrollo debe ser `54410`.
- El puerto de pruebas debe ser `54411`.
- La base de pruebas debe ser efímera y no utilizar un volumen persistente.
- Ambos servicios deben tener healthcheck.
- El arranque debe esperar a que los servicios de PostgreSQL estén saludables.
- No debe utilizarse la clave `version:` de Compose.

Además:

- La configuración de pruebas debe utilizar su propio fichero de entorno.
- Deben existir comandos en el Makefile para levantar y detener las bases.
- Deben existir comandos para migrar ambas bases.
- Debe existir un comando para ejecutar las pruebas.

Antes de modificar nada, revisa nuevamente las migraciones existentes.

Es especialmente importante que ninguna migración existente sea modificada.

Si detectas que alguna migración existente tendría que modificarse para conseguir el cambio, no la modifiques. Detente y explícame el problema.
```

**Qué salió:** funcionó a la primera: revisó las 4 migraciones y ninguna necesitaba cambios para PostgreSQL (`unsigned()` se ignora, `timestamp` pasa a `timestamptz`), así que no hubo que detenerse.


## Prompt 5

**Modelo:** Sonnet Medium
**Herramienta:** Claude Code

```
Implementa los cambios necesarios para migrar el proyecto de SQLite a PostgreSQL siguiendo las restricciones acordadas.

Antes de comenzar:

1. Verifica el estado actual del repositorio.
2. Identifica las migraciones existentes.
3. Confirma cuáles archivos vas a modificar.

Durante la implementación:

- no modifiques migraciones existentes
- crea nuevas migraciones solamente si son necesarias
- mantén separadas las bases de desarrollo y pruebas
- utiliza Docker para PostgreSQL
- respeta los nombres, puertos y configuración definidos anteriormente
- configura correctamente el entorno de pruebas
- agrega los comandos necesarios al Makefile

Después de implementar:

1. levanta las bases
2. verifica su estado
3. ejecuta las migraciones
4. ejecuta las pruebas
5. comprueba que no se hayan modificado migraciones existentes

No ocultes errores ni los soluciones modificando migraciones históricas.
Si algo falla, explícame exactamente qué ocurrió.
```

**Qué salió:** funcionó, con un tropiezo: se denegó leer `backend/.env` por llevar APP_KEY y se añadieron las claves `DB_*` sin leer sus valores; bases sanas en 54410/54411, las dos migradas, 23/23 en verde contra `db-test` (probado con `db` parada) y 0 cambios en migraciones.

## Prompt 5

**Modelo:** Sonnet Medium
**Herramienta:** Claude Code

```
Implementa los cambios necesarios para migrar el proyecto de SQLite a PostgreSQL siguiendo las restricciones acordadas.

Antes de comenzar:

1. Verifica el estado actual del repositorio.
2. Identifica las migraciones existentes.
3. Confirma cuáles archivos vas a modificar.

Durante la implementación:

- no modifiques migraciones existentes
- crea nuevas migraciones solamente si son necesarias
- mantén separadas las bases de desarrollo y pruebas
- utiliza Docker para PostgreSQL
- respeta los nombres, puertos y configuración definidos anteriormente
- configura correctamente el entorno de pruebas
- agrega los comandos necesarios al Makefile

Después de implementar:

1. levanta las bases
2. verifica su estado
3. ejecuta las migraciones
4. ejecuta las pruebas
5. comprueba que no se hayan modificado migraciones existentes

No ocultes errores ni los soluciones modificando migraciones históricas.
Si algo falla, explícame exactamente qué ocurrió.
```

**Qué salió:** prompt duplicado por error al registrarlo; ejecutado otra vez fue idempotente: todo seguía en su sitio y, tras `make db-down` y `make test`, la base de pruebas volvió vacía, se migró sin conexiones rechazadas y quedó 23/23.

## Prompt 6

**Modelo:** Opus 1M xHigh
**Herramienta:** Claude Code

```
La migración ya está implementada y las pruebas están ejecutándose correctamente.

Ahora quiero hacer una revisión de regresión del cambio.

Analiza qué podría haberse perdido o cambiado como consecuencia de pasar de SQLite a PostgreSQL, aunque la batería de pruebas esté en verde.

Revisa especialmente:

- tipos
- esquema generado
- configuración de pruebas
- comportamiento de consultas
- fechas
- comparaciones
- diferencias entre SQLite y PostgreSQL
- cobertura de las pruebas

No corrijas nada todavía.

Quiero que identifiques posibles diferencias entre el comportamiento anterior y el actual y que proporciones evidencia concreta en el código.
```

**Qué salió:** funcionó a la primera: `schema.ts` con diff vacío porque `schema_rules.ts` fija `due_date` como `string`, y typecheck en verde; con una tarea fechada el 2020-01-01 y mirada el 2026-10-07, el `GET /tasks/:id` devuelve `"2020-01-01T00:00:00.000Z"` e `isOverdue: false` (el PUT da `true`): el comentario de `isOverdueOn` ya no es verdad y ningún test lo cubre.


## Prompt 7

**Modelo:** Opus 1M xHigh
**Herramienta:** Claude Code

```
Profundiza en el cambio que encontraste en `backend/database/schema.ts`.

Compara el estado actual con el estado anterior a la migración utilizando git.

Identifica exactamente:

- qué declaraciones cambiaron
- qué tipo tenían antes
- qué tipo tienen ahora
- por qué se produjo cada cambio
- qué código utiliza esas declaraciones
- si alguno de esos cambios puede modificar el comportamiento de la aplicación

No corrijas los cambios todavía.

Quiero entender primero su impacto.
```

**Qué salió:** funcionó a la primera: `git diff e0f8e50 -- backend/database/schema.ts` está vacío, no cambió ninguna declaración porque `schema_rules.ts` fija `due_date` y `status`, y el resto de columnas mapea igual en los dos motores (`timestamp`→`timestamptz`→`DateTime`); el impacto está en el valor real que llega en ejecución.

## Prompt 8

**Modelo:** Opus 5.5 (1M)
**Herramienta:** Claude Code

```
Ahora quiero seguir el impacto de esos cambios hasta el comportamiento de la aplicación.

Busca dónde se utiliza la información relacionada con las tareas y determina cómo decide actualmente el sistema que una tarea está vencida.

Lee el código completo relacionado con esa decisión y cualquier comentario que explique la comparación.

No modifiques nada.

Explícame:

- qué valores se comparan
- qué tipos tienen
- qué comportamiento esperaba originalmente el código
- si el cambio de SQLite a PostgreSQL afecta esa comparación
- si el comentario existente sigue describiendo correctamente el comportamiento
```

**Qué salió:** funcionó a la primera: `isOverdueOn` compara `this.dueDate`, que en ejecución es un `Date`, con `referenceDay` en texto; `Date < '2026-10-07'` pasa a número contra `NaN` y da siempre `false`, así que el comentario («texto contra texto») ya no es verdad.


## Prompt 9

**Modelo:** Opus 5.5 (1M)
**Herramienta:** Claude Code

```
Las pruebas actuales están pasando.

Quiero determinar si eso significa realmente que el comportamiento importante del sistema está cubierto.

Analiza la batería de pruebas relacionada con la funcionalidad modificada y compárala con los cambios introducidos por la migración.

Identifica comportamientos que podrían haber cambiado pero que la batería actual no comprueba.

Para cada posible caso:

- indica qué comportamiento podría cambiar
- indica qué prueba existente debería detectarlo, si existe
- si no existe, explica por qué la batería actual no podría detectarlo
- proporciona evidencia en el código

No agregues pruebas todavía y no modifiques el código.
```

**Qué salió:** funcionó a la primera: ningún test fija una fecha de vencimiento ni comprueba `isOverdue`, el filtro por estado ni el orden; los de `assignee.spec.ts` leen la tarea con `today` pero con `dueDate` nulo, y esa rama da `false` en los dos motores.

## Prompt 10

**Modelo:** Opus 5.5 (1M)
**Herramienta:** Claude Code

```
Quiero verificar si el cambio realizado modifica datos existentes o solamente la estructura de la base de datos.

Analiza las migraciones involucradas y determina qué datos podrían verse afectados.

Si existe modificación de datos:

- identifica la tabla
- identifica la columna
- construye una consulta que permita medir exactamente cuántas filas cambian
- ejecuta la consulta contra la base correspondiente

Si no existe modificación de datos, demuestra por qué el número es cero.

No quiero estimaciones.

No modifiques los datos para realizar la medición y no modifiques las migraciones.
```

**Qué salió:** funcionó a la primera: 0 filas, demostrado con `count(*)` en SQLite, `db` y `db-test`, un `git diff` de migraciones vacío y ninguna migración con `update`, `insert` o `delete`; las 4 migraciones están aplicadas en el lote 1 en las tres bases.


## Prompt 11

**Modelo:** Opus 5.5 (1M)
**Herramienta:** Claude Code

```
Analiza la reversibilidad del cambio de esquema realizado.

Determina en qué categoría o rama del árbol de reversibilidad debería clasificarse.

Justifica la clasificación basándote en:

- qué cambia en el esquema
- si se modifican datos
- si se pierden datos
- si existe transformación de datos
- si la operación puede revertirse sin pérdida

No cambies el código.
```

**Qué salió:** funcionó a la primera: no cambia el esquema ni toca datos, así que es reversible sin pérdida revirtiendo la conexión; con el matiz de que deja de serlo en cuanto `db` tenga datos que no se copian de vuelta a SQLite.

## Prompt 12

**Modelo:** Opus 5.5 (1M)
**Herramienta:** Claude Code

```
Haz una revisión crítica de todo el cambio realizado.

No quiero una conclusión de que "todo está correcto".

Quiero que identifiques:

- qué supusimos sin poder comprobarlo completamente
- qué comportamiento no pudimos validar
- qué resultado podría tener otra explicación
- qué diferencia entre SQLite y PostgreSQL debería investigarse más
- qué parte de la solución te genera mayor incertidumbre

Para cada punto proporciona la evidencia disponible y qué comprobación adicional sería necesaria para eliminar la duda.

No modifiques código.
```

**Qué salió:** funcionó a la primera: midió con `pg-types` que con `TZ=Europe/Madrid` la fecha llega un día antes; quedan sin comprobar el lado SQLite (no se puede sin `better-sqlite3`), el `RangeError` del `formatDay` del frontend en navegador y `make start` de punta a punta.
