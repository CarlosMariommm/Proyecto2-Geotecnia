# Contexto del Proyecto 2 (para continuar en otro chat)

Resumen de lo decidido hasta el 3 de noviembre de 2026. Las referencias completas están en `referencias/contexto.md` (carpeta ignorada por git).

## Qué es el sistema

- JUPACA PROYECTA (consultoría geotécnica). El laboratorio llena a mano tres hojas: SPT de campo, clasificación SUCS (M23-REG 02) y límites de Atterberg (M04-REG01).
- **Núcleo del sistema:** leer esas hojas desde una foto, revisar lo leído con la foto al lado (por teclado), validar, clasificar (SUCS y AASHTO), generar el Anexo 02 (Excel y PDF) y exportar a GEO5 y Civil 3D, que hoy se llenan a mano.
- Fase 0: macro de Excel (VBA) con SUCS, AASHTO y hoja de casos de prueba.

## Decisiones tomadas

| Tema | Decisión |
|---|---|
| Backend | Django + **PostgreSQL** (se descartó MongoDB y Node/Express) |
| Frontend | React |
| Motor de reglas | Python puro dentro del backend, sin depender de Django; comparte con la macro un solo archivo de casos de prueba |
| Lectura de hojas | Servicio externo de lectura de imágenes (autorizado), con revisión humana obligatoria; exactitud se mide con los 36 folios de referencia antes de prometer un porcentaje |
| Coordenadas | Este, Norte y cota se capturan a mano desde el plano (a veces solo hay una imagen); cotas locales, p. ej. 100.00 |
| Nombres de suelo | Según USCS (ASTM D2487) |
| Cronograma | `Cronograma_Proyecto2.xlsx`: 2 personas, tope 15-ene-2027 |
| Confidencialidad | Datos de clientes solo en `referencias/`; nada en el repositorio ni en sitios públicos; demos con datos inventados |
| Entregables | No mencionan herramientas de IA |

## Por qué PostgreSQL y no MongoDB

Los datos son muy relacionales (proyecto → sondeo → muestra → ensayo) y tienen reglas entre filas: los estratos de un sondeo no pueden traslaparse ni dejar huecos. En PostgreSQL eso se garantiza en la propia base (`ExclusionConstraint` sobre rangos de profundidad, llaves foráneas, `CHECK`); en MongoDB habría que programarlo a mano en la aplicación.

## Modelo de datos (borrador)

| Entidad | Campos clave |
|---|---|
| Proyecto | nombre, cliente, ubicación, estructura |
| Sondeo | número, fecha inicial, Este, Norte, cota, origen de coordenadas (plano CAD / imagen), aproximadas (sí/no), herramientas de avance y muestreo, operador, peso del golpeador, profundidad final |
| Muestra (cada 0.50 m) | profundidad, golpes de los tres tramos, N, humedad, recuperación, descripción, símbolo, % grava/arena/finos, plasticidad (PLAS/NP), orgánico |
| Ensayo de Atterberg | referencia (L1, L2), LL, LP, IP con decimales, determinaciones; se liga a **varias** muestras |
| Estrato agrupado | sondeo, desde, hasta, símbolo, clave de patrón; sin traslapes (para el perfil y las exportaciones) |
| Clasificación | entradas, símbolo y nombre, ruta de decisión, advertencias, versión de reglas, quién confirmó o corrigió y por qué |
| Hoja escaneada | imagen, tipo de hoja, folio, campos leídos con nivel de confianza, estado de revisión |
| Patrón de estrato | clave (símbolo + PLAS/NP + ORG), imagen |

## Migración desde MongoDB

La base MongoDB es la del **sistema administrativo** (MongoDB Atlas, repositorio `JupacaProyecta-PTC-2026`): 14 colecciones de clientes, proyectos, empleados, planillas, proformas y cuentas por cobrar. No tiene datos geotécnicos. Se decidió migrarla completa a una PostgreSQL **central de la empresa**, que también usará el Proyecto 2, y adaptar el backend Node del fork para que use PostgreSQL.

Estado al 3-nov-2026, con plan aprobado: ver [`PLAN_MIGRACION.md`](PLAN_MIGRACION.md).

- **Proyecto Django.** El esquema y el script de migración están en `backend/` (apps `nucleo`, `rrhh`, `finanzas`, `migracion`).
- **Backend Node.** La versión adaptada está en la rama `migracion-postgresql` del fork.
- **Pruebas.** Todo se probó contra copias locales con datos inventados, incluida la comparación de respuestas del backend original contra el adaptado.
- **Pendiente.** Copia real de Atlas (usuario de solo lectura), revisión del informe con la empresa y base de producción.

Reglas de la migración: MongoDB solo en modo lectura, probar primero contra una copia, script repetible y validación con conteos y sumas por tabla.

## Lectura de hojas (app `backend/lectura`)

Desde el 4-nov-2026 está la base del lector. Detalle en [`backend/lectura/README.md`](backend/lectura/README.md).

- **Flujo.** Foto → servicio de lectura → propuesta por celda → revisión con la foto al lado → cierre → aprendizaje.
- **Aprendizaje.** Se aprende de las correcciones: sinónimos y abreviaturas, confusiones de caracteres por técnico, y reglas de la hoja que detectan lecturas imposibles. No se entrena un reconocedor de letra propio, porque no hay datos suficientes.
- **Proveedores.** Por ahora solo `manual` y `simulado`. Los servicios reales se agregan después de la prueba comparativa con los folios de referencia, usando `medir_exactitud_lectura`.
- **Pendiente.**
  - Confirmar las plantillas contra las hojas impresas.
  - Crear las tablas de sondeos y muestras.
  - Hacer la pantalla de revisión en React.

## Pendiente de confirmar con la empresa

Precisión de LL/LP (entero o decimales), quién aprueba el símbolo final, criterio SPT (tramo de 15 o 20 cm y rechazo), datos de tamices No. 10 y No. 40, Excel original del Anexo 02, proyecto de ejemplo de GEO5 y versión de Civil 3D, servicio de lectura y presupuesto.
