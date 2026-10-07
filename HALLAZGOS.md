# Hallazgos

Aquí van **las tres líneas** del ejercicio, una por cada punto de abajo. Es lo único que hay que
traer hecho: un cambio de motor a medias con estas tres líneas escritas vale más que lo contrario,
porque lo que se discute en el directo es dónde te chocaste.

Escribe **una sola línea por punto**, con tus palabras y con lo que mediste, no con lo que suponías.

## 1. Las filas que cambian y la rama

Cuántas filas cambian de valor en tu cambio de esquema, medido con una consulta, y en qué rama del
árbol de reversibilidad cae. Si tu migración no toca datos, dilo tal cual: también es una respuesta.

- 0 filas: ninguna migración nueva ni tocada (`git diff e0f8e50 -- backend/database/migrations` vacío) y `count(*)` de `users`, `tasks`, `tasks.due_date` y `auth_access_tokens` da 0 en SQLite, `db` y `db-test`; no toca datos, así que es reversible sin pérdida (revertir la conexión), aunque deja de serlo en cuanto PostgreSQL acumule datos que no vuelven a SQLite.

## 2. Lo que la batería de pruebas no podía ver

Una cosa que la batería de pruebas no podía ver. Si no encontraste ninguna, escribe qué buscaste y dónde.

- El vencimiento: `pg` devuelve `due_date` como `Date` aunque `schema_rules.ts` lo declare `string` (por eso `schema.ts` sale sin diff y el typecheck en verde); una tarea del 2020-01-01 mirada el 2026-10-07 da `isOverdue: false` y `dueDate: "2020-01-01T00:00:00.000Z"` en `GET /api/v1/tasks/:id`, y ningún test fija una fecha de vencimiento.

## 3. Tu duda

De qué dudaste, o qué no pudiste comprobar.

- No pude medir el lado SQLite (al quitar `better-sqlite3` ya no corre), y el `RangeError: Invalid time value` que esa fecha provoca en `formatDay` del frontend lo reproduje solo en Node, no en el navegador; con `TZ=Europe/Madrid` la misma fecha sale `2019-12-31T23:00:00.000Z`, un día de menos.
