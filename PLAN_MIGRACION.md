# Migración MongoDB → PostgreSQL

Actualizado: 3 de noviembre de 2026. Estado: **plan aprobado; migración y backend adaptado probados contra copias locales**. Falta la copia real de Atlas (sección 7).

## 1. Qué se migra y para qué

- **Origen:** la base MongoDB Atlas del sistema administrativo (repositorio `JupacaProyecta-PTC-2026`, versión 4.4.3, Mongoose 9). Son 14 colecciones: clientes, proyectos, servicios, departamentos, administradores, empleados, contratos, planillas, líneas de planilla, boletas, horas extra, reportes semanales, proformas y cuentas por cobrar.
- **Destino:** una base PostgreSQL que será la **base central de la empresa**, compartida por el sistema administrativo, el Proyecto 2 (geotecnia) y los sistemas que vengan. `Proyecto`, `Cliente`, `Empleado` y `Usuario` son las mismas tablas para todos.
- **Se migra una copia:** MongoDB solo se lee. El script se puede correr de nuevo para traer lo nuevo mientras el sistema original siga en uso.
- **Datos de demostración:** los de `isSeedData: true` no se migran. En pruebas se incluyen con `--incluir-demo`.

## 2. Dónde está cada cosa

| Qué | Dónde |
|---|---|
| Esquema de la base (modelos y migraciones de Django) y script de migración | `backend/` de este repositorio (apps `nucleo`, `rrhh`, `finanzas`, `migracion`) |
| Cómo correr la migración, el informe y las correcciones | [`backend/migracion/README.md`](backend/migracion/README.md) |
| Backend Node adaptado a PostgreSQL | Rama `migracion-postgresql` del fork `JupacaProyecta-PTC-2026` (ver `backend/README.md` y `backend/src/db/README.md` de esa rama) |

## 3. Decisiones de diseño

| Decisión | Por qué |
|---|---|
| **Llaves UUID versión 7**; las filas migradas derivan su UUID del ObjectId, que además queda en `mongo_id` | Igual que los ObjectId: no se repiten entre tablas y ordenados como texto quedan en orden de creación. El backend Node depende de las dos cosas. `mongo_id` permite repetir la migración sin duplicar. |
| **Un solo `Usuario`** para administradores y cuentas de empleados | Todos los sistemas inician sesión contra la misma tabla. Las contraseñas se guardan como `bcrypt$<hash>` (formato de Django), así que sirven en Django y en Node. |
| **`generatedBy` (planillas) y `uploadedBy` (avances) apuntan a `Usuario`** | Guardan la cuenta que hizo la acción, que puede ser un administrador. |
| **Montos en `NUMERIC(14,4)`** | El IVA del 13 % produce montos con 3 y 4 decimales; se conservan, y solo se quita el ruido de punto flotante. |
| **Fechas «de día» como `DATE`** (día UTC) | La aplicación las guarda como medianoche UTC. |
| **Listas cerradas con `CHECK`** (estados, tipos) | La base rechaza valores fuera de la lista. |
| **Campos fuera del esquema en `datos_extra` (jsonb)** | Los esquemas usaban `strict: false`; no se pierde nada y la API los sigue devolviendo. |
| **Al borrar un registro, las referencias a él quedan vacías (NULL)**, y las partes de un registro (líneas de planilla, ítems, pagos) se borran con él | En MongoDB los borrados nunca fallaban y dejaban referencias colgando. Así se conserva ese comportamiento sin perder historial (contratos, líneas, cuentas por cobrar) y sin dejar datos rotos. |
| **El backend Node conserva su API**: una capa de modelos con la misma forma de uso que Mongoose, sobre Knex + `pg` | Los controladores, validaciones, cálculos, PDF y correos no cambian; la web y la app móvil tampoco. |

## 4. Datos sucios o incompletos

Nada se corrige en silencio. Cada caso queda en el informe con colección, `_id`, campo, valor y motivo.

| Caso | Resultado |
|---|---|
| Línea de planilla cuya planilla no existe | Rechazo (la línea es parte de la planilla) |
| Otra referencia a un registro que no existe (por ejemplo, un empleado o una proforma que se borró) | Se migra con la referencia vacía y una advertencia; se conserva el resto del registro |
| Número o booleano guardado como texto exacto | Se convierte y se cuenta |
| Texto que no es número, fecha no reconocida, hora que no es HH:MM, estado fuera de la lista, campo obligatorio vacío | Rechazo |
| Más de 4 decimales | Se redondea a 4, con advertencia |
| Fecha «de día» con hora | Se toma el día UTC, con advertencia |
| `""` en un campo único (correo, DUI, RUC, número de factura) | Pasa a NULL y se cuenta. En los demás campos se conserva |
| Duplicado en un campo único, o dos contratos activos del mismo empleado | Se conserva el más antiguo y se rechaza el otro |
| Correo que es a la vez de un administrador y de un empleado | Se rechaza el administrador (en el sistema actual no podía iniciar sesión) |
| Totales que no cuadran con sus partes | Se migran tal cual, con advertencia |
| Correcciones que decida la empresa | Archivo `correcciones.yaml` fuera de git; se aplican en memoria y se listan en el informe |

## 5. Pruebas hechas (copias locales con datos inventados)

- **Migración, con una copia de prueba de 31 casos válidos y sucios.** Los 31 se trataron como se esperaba, con conteos y sumas cuadrados en las 19 tablas. La migración se puede repetir: la segunda corrida no insertó nada, y ids, totales y fechas quedaron idénticos. Las correcciones se aplican y las que no encuentran su documento se informan.
- **Backend adaptado contra el original.** Los dos corren sobre los mismos datos (MongoDB y su migración a PostgreSQL) y se comparan sus respuestas:
  - **Consultas:** 84. 76 son idénticas. En 3 solo cambia el orden dentro de empates de fecha, que MongoDB no define. En 5 cambia solo el `updatedAt` del administrador después de su primer inicio de sesión.
  - **Escrituras:** 75 pasos (crear, editar, pagar, generar planilla y boletas, borrar con dependencias, bloqueo por intentos fallidos). 74 son idénticos. El restante es la decisión de la sección 3: tras borrar un empleado, su contrato muestra la referencia vacía en vez del id inexistente.

## 6. Cómo repetir las pruebas

Ver `backend/migracion/README.md` (copia de prueba y `verificar_casos_prueba`) y los scripts `backend/scripts/compatibilidad/comparar.mjs` y `escrituras.mjs` del fork.

## 7. Pendiente

1. **Copia real de Atlas.** Requiere un usuario de Atlas con rol `read`. Con esa copia se corre `mongodump` y luego la migración en simulación, y se revisa el informe con la empresa (`correcciones.yaml`).
2. **Base PostgreSQL de producción.** Hay que elegir proveedor, correr `migrate` y la migración definitiva, y configurar `DATABASE_URL` en el backend desplegado.
3. **Fecha de corte.** Mientras el sistema original siga escribiendo en MongoDB, la migración se puede repetir. Al pasar a producción, MongoDB queda solo como respaldo.
