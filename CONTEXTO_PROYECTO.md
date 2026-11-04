# Proyecto 2 — Contexto completo

Actualizado: 4 de noviembre de 2026. Este archivo basta para retomar el proyecto en otro chat o explicárselo a alguien nuevo. Los detalles técnicos están en los README enlazados.

---

## 1. En pocas palabras

- **Empresa.** JUPACA PROYECTA, S.A. de C.V. (El Salvador): consultoría geotécnica y control de calidad.
- **Problema.** El laboratorio llena a mano tres hojas: SPT de campo, clasificación SUCS y límites de Atterberg. Pasarlas a reportes es lento porque la letra cuesta leerla y todo se vuelve a teclear. Después, los mismos datos se teclean otra vez en Civil 3D y GEO5.
- **Solución.** Un sistema web que **lee las hojas desde una foto** y luego:
  - deja revisar lo leído con la foto al lado;
  - valida y clasifica el suelo (SUCS y AASHTO);
  - genera el Anexo 02 (Excel y PDF);
  - exporta a GEO5 y Civil 3D.
- **Aprende de cada corrección.** El sistema aprende sinónimos, letras que se confunden por técnico y reglas que detectan lecturas imposibles.
- **Base de datos.** Una **PostgreSQL central de la empresa**, compartida con el sistema administrativo, que hoy está en MongoDB y se está migrando.
- **Equipo y plazo.** Dos personas, con fecha máxima el 15 de enero de 2027.

---

## 2. El problema real

1. **Hojas escritas a mano.** En el laboratorio se escriben a mano los golpes del SPT, las humedades, los porcentajes de grava, arena y finos, y los límites líquido y plástico. La letra es difícil y se usan **sinónimos y abreviaturas propias**. Por eso el escáner que ya tienen no sirve: el texto sale ilegible.
2. **Doble y triple captura.** Con esas hojas se arma el Anexo 02 (hoja de registro por sondeo). Luego los mismos datos se pasan **a mano** a Civil 3D y GEO5, porque hoy no hay ningún archivo de intercambio.
3. **Errores que nadie detecta.** En los documentos de referencia aparecieron:
   - porcentajes que no suman 100;
   - porcentajes invertidos;
   - errores de dedo («HINO» por «LIMO»);
   - nombres de suelo que no corresponden a los porcentajes;
   - símbolos que no siguen la regla de la línea A;
   - notas pegadas que contradicen la hoja.

   Nadie los detectó porque nada se valida automáticamente.
4. **Precisión perdida.** Las hojas anotan valores ya redondeados (humedades enteras, sin masas de tara). Por eso no siempre se puede recalcular un resultado desde la hoja.

---

## 3. Cómo se piensa solucionar

### Flujo

```
Foto de la hoja (teléfono)
  → servicio de lectura (texto + confianza por celda)
  → propuesta por celda (sinónimos, confusiones de letras, reglas de la hoja)
  → pantalla de revisión con la foto al lado (también sirve para capturar a mano)
  → validaciones (detectan errores; no corrigen solas)
  → clasificación SUCS y AASHTO con la ruta de decisión
  → Anexo 02 en Excel y PDF
  → exportación a GEO5 y Civil 3D
```

### Principios

- **Revisión humana obligatoria.** Nada se corrige en silencio. Cada cambio que propone el sistema queda marcado, y una persona lo confirma o lo corrige. Para valores fuera de la lista se pide el motivo.
- **No se inventan valores, reglas ni formatos.** Si algo no se sabe, se pregunta a la empresa o queda marcado como **SUPUESTO** en el código y en la documentación.
- **Primero medir, luego prometer.** La exactitud se mide con los 36 folios de referencia antes de dar cualquier porcentaje.
- **No se reimplementa Civil 3D ni GEO5.** Solo se les entregan los datos en un formato que importen.
- **Rediseñar las hojas impresas** es la mejora más fuerte para la lectura: casillas para marcar, un recuadro por dígito y un código de folio o QR. Se propone a la empresa como entregable.
- **El reconocedor de letra se compra hecho.** Entrenar uno propio desde cero exige miles de ejemplos, y hay 36 folios. Lo que el servicio no sabe del laboratorio lo aprende nuestro sistema con cada revisión, y cada hoja revisada queda como ejemplo para entrenar un modelo personalizado más adelante.

---

## 4. Funcionalidades

| Funcionalidad | Estado |
|---|---|
| Lectura de hojas: subir foto, leer, proponer, revisar, cerrar, aprender (`backend/lectura`) | **Hecha la base**, probada con datos inventados; falta un servicio de lectura real |
| Medir la exactitud de la lectura por campo (`medir_exactitud_lectura`) | Hecha |
| Cargar hojas ya digitalizadas para enseñar o medir (`cargar_hoja_transcrita`) | Hecha |
| Base central PostgreSQL: usuarios, empleados, clientes, proyectos, RR. HH. y finanzas | **Hecha**; esquema en Django |
| Migración MongoDB → PostgreSQL del sistema administrativo | **Hecha y probada contra copias**; falta la copia real de Atlas |
| Backend Node del sistema administrativo adaptado a PostgreSQL | Hecho, en la rama `migracion-postgresql` del fork |
| Macro de Excel (fase 0): SUCS, AASHTO con índice de grupo y hoja de casos de prueba | Pendiente (9–20 nov) |
| Motor de reglas en Python (SUCS, AASHTO, validaciones), con los mismos casos de prueba que la macro | Pendiente |
| Tablas de sondeos, muestras, ensayos y estratos | Pendiente (modelo en borrador, sección 7) |
| Pantalla de revisión en React con la foto al lado | Pendiente |
| Anexo 02 en Excel y PDF | Pendiente |
| Agrupar muestras en estratos con su patrón de relleno (23 imágenes de la biblioteca) | Pendiente |
| Exportación a GEO5 y Civil 3D (CSV o AGS) | Pendiente (4–8 ene) |
| Piloto: comparar el tiempo por sondeo contra el método actual | Pendiente (11–15 ene) |

---

## 5. Estructura del repositorio

```
Proyecto2-Geotecnia/
├── CONTEXTO_PROYECTO.md       ← este archivo
├── CONTEXTO_SESION.md         resumen anterior (3-4 nov), más corto
├── PLAN_MIGRACION.md          plan y resultados de la migración MongoDB → PostgreSQL
├── Cronograma_Proyecto2.xlsx  cronograma (una hoja, 2 personas, hasta el 15-ene-2027)
├── .gitignore                 excluye referencias/, .env, media/, informes de migración
├── referencias/               CONFIDENCIAL (ignorada por git)
│   ├── contexto.md            descripción original del proyecto
│   ├── <folios de referencia>.pdf   36 hojas escaneadas de un proyecto real
│   ├── Anexo 02 - Hojas de Registro pantilla v7.pdf
│   ├── 09 - ESTRATOS GEOTECNIA/     23 patrones de relleno (PNG)
│   └── _extraccion/           CSV y scripts de la extracción preliminar (ver LEEME.md)
├── Proyecto2-Geotecnia/       clon anterior del repositorio de GitHub; ya está integrado en la raíz y sobra
└── backend/                   proyecto Django, dueño del esquema de la base central
    ├── README.md
    ├── requirements.txt       Django 6.1, psycopg 3, python-decouple, dj-database-url, pymongo, bcrypt, PyYAML
    ├── .env / .env.example    secretos y conexiones (.env fuera de git)
    ├── config/
    │   ├── settings/          base.py (común), development.py (por defecto), production.py
    │   └── urls.py            /admin/ y /api/lectura/
    ├── nucleo/                usuarios, departamentos, empleados, clientes, servicios, proyectos
    ├── rrhh/                  contratos, horas extra, planillas, boletas
    ├── finanzas/              proformas, cuentas por cobrar, ítems, pagos
    ├── migracion/             comando migrar_mongo, conversiones, informes (ver su README)
    └── lectura/               lector de hojas (ver lectura/README.md)
        ├── motor/             Python puro, sin Django
        │   ├── texto.py         normalización y alineación de textos
        │   ├── confusiones.py   letras que se confunden (aprendidas e iniciales)
        │   ├── vocabularios.py  símbolos SUCS, PLAS/NP, LL/LP
        │   ├── plantillas.py    campos de cada hoja (BORRADOR)
        │   ├── validaciones.py  N = tramo 2 + tramo 3, IP = LL − LP, suma 100 %, profundidad creciente
        │   ├── propuestas.py    propuesta por celda y por hoja
        │   └── proveedores.py   interfaz con servicios de lectura (manual, simulado)
        ├── models.py          HojaEscaneada, CampoLeido, Sinonimo, ConfusionCaracter
        ├── servicios.py       flujo: registrar, leer, revisar, cerrar, reabrir
        ├── aprendizaje.py     aprender de una hoja; reconstruir todo
        ├── views.py, urls.py  API JSON
        ├── management/commands/  cargar_hoja_transcrita, medir_exactitud_lectura, reentrenar_lectura
        └── tests/             test_motor.py (28), test_flujo.py (13)
```

Fuera de este repositorio:

- **Fork del sistema administrativo:** `C:\Users\spide\Documents\JupacaProyecta-PTC-2026`, con la rama `migracion-postgresql`.
- **Bases locales de prueba:** `C:\Users\spide\herramientas-bd` (sección 13).

**Aún no hay frontend.** Será React, en su propia carpeta, cuando se empiece.

---

## 6. Stack y decisiones

| Tema | Decisión | Por qué |
|---|---|---|
| Backend | Django 6.1 + PostgreSQL, configuración en `config/settings/` (base, development, production) con python-decouple | Los datos son muy relacionales (proyecto → sondeo → muestra → ensayo) y tienen reglas entre filas, como estratos sin traslapes. PostgreSQL las garantiza en la base con `CHECK`, `ExclusionConstraint` y llaves foráneas. |
| Frontend | React | El equipo ya lo conoce |
| Fase 0 | Macro VBA en Excel (.xlsm), con el código guardado como `.bas` en git | Da algo útil al laboratorio en dos semanas, sin servidor |
| Motor de reglas | Python puro, sin depender de Django | Se prueba solo y comparte el archivo de casos de prueba con la macro. React pide la clasificación a la API y no duplica reglas. |
| Lectura | Servicio externo, autorizado por la empresa, más revisión humana | Ver sección 8 |
| Base | Una sola PostgreSQL central para todos los sistemas de la empresa | `Proyecto`, `Cliente`, `Empleado` y `Usuario` son comunes |
| Idioma del código | Identificadores y comentarios en **español**, como el resto del backend; interfaz en español | Consistencia con lo ya construido |
| Convenciones de la base | UUID v7, `mongo_id` en lo migrado, `datos_extra` (jsonb), montos `NUMERIC(14,4)`, listas cerradas con `CHECK`, bcrypt compatible con Node | Ver `backend/README.md` |
| API | Vistas JSON de Django con sesión y CSRF, sin dependencias nuevas | Se puede pasar a Django REST Framework si crece |

---

## 7. Modelo de datos

**Ya existe:**

| App | Tablas |
|---|---|
| `nucleo` | Usuario, Departamento, Empleado, Cliente, Servicio, Proyecto, ProyectoTecnico, AvanceProyecto, ReporteSemanal |
| `rrhh` | Contrato, HoraExtra, Planilla, líneas de planilla, boletas |
| `finanzas` | Proforma, CuentaPorCobrar, ítems, pagos |
| `lectura` | HojaEscaneada, CampoLeido, Sinonimo, ConfusionCaracter |

**Borrador de las tablas geotécnicas (pendiente):**

| Entidad | Campos clave |
|---|---|
| Sondeo | Proyecto, número, fecha, Este, Norte, cota, origen de coordenadas (plano CAD o imagen), si son aproximadas, herramientas, operador, peso del martillo, profundidad final |
| Muestra (cada 0.50 m) | Profundidad, golpes de los tres tramos, N, humedad, recuperación, descripción, símbolo, % grava/arena/finos, plasticidad (PLAS/NP), orgánico |
| Ensayo de Atterberg | Referencia, LL, LP, IP con decimales, determinaciones. Se liga a **varias** muestras. |
| Estrato agrupado | Sondeo, desde, hasta, símbolo, patrón de relleno. Sin traslapes (`ExclusionConstraint`). |
| Clasificación | Entradas, símbolo y nombre, ruta de decisión, advertencias, versión de reglas, quién confirmó o corrigió y por qué |
| Patrón de estrato | Clave (símbolo + PLAS/NP + ORG) e imagen |

Las hojas revisadas en `lectura` alimentarán estas tablas.

---

## 8. Lector de hojas (`backend/lectura`)

Detalle completo en [`backend/lectura/README.md`](backend/lectura/README.md).

**Flujo de la API:**

1. `POST /api/lectura/hojas/` sube la foto. Acepta JPEG, PNG, HEIC o PDF, detectado por los primeros bytes, y rechaza duplicados por huella SHA-256.
2. El proveedor de lectura devuelve texto y confianza por celda.
3. El motor propone un valor por celda. Origen, de más a menos seguro: `lectura`, `formato`, `sinonimo`, `parecido`, `confusion`, `validacion`, `sin_propuesta`. Todo lo que no sea `lectura` con confianza ≥ 0.90 queda marcado para revisión.
4. `PATCH .../campos/` guarda la revisión de cada celda: confirmada, corregida o ilegible. «Confirmar sin observaciones» acepta de una vez lo no marcado, y solo cuando lo pide una persona.
5. `POST .../cerrar/` cierra la hoja. Exige que no haya celdas pendientes y que las advertencias se acepten de forma explícita (quedan guardadas). Al cerrar, el sistema aprende.

**Qué aprende:**

- Sinónimos con confirmaciones y rechazos.
- Confusiones de caracteres, en general y por técnico que llenó la hoja.
- Las hojas del conjunto `prueba` nunca enseñan; sirven para medir sin trampa.

**Proveedores:**

- Hoy existen `manual` (captura con la foto al lado) y `simulado` (JSON, para pruebas).
- Los servicios reales se eligen en la prueba comparativa (25 nov – 2 dic) por la cifra de **errores sin marcar**.
- Candidatos: Google Cloud Document AI, Amazon Textract y modelos de lenguaje con visión por API.
- Por verificar: Transkribus (entrena modelos propios con transcripciones), Pen to Print, Parseur y Extend.
- Google Lens y Keep no sirven para integrarlas: son apps para personas.

**Supuestos:**

- Las plantillas de campos están en borrador.
- Umbral de confianza: 0.90.
- Confusiones iniciales letra→número (O→0, S→5...).
- Con 50 golpes hay rechazo y no se exige N = tramo 2 + tramo 3.
- IP = LL − LP se exige exacto.

---

## 9. Migración MongoDB → PostgreSQL

Detalle en [`PLAN_MIGRACION.md`](PLAN_MIGRACION.md) y [`backend/migracion/README.md`](backend/migracion/README.md).

- **Origen.** MongoDB Atlas del sistema administrativo (repositorio `JupacaProyecta-PTC-2026`). Son 14 colecciones: clientes, proyectos, empleados, planillas, proformas, cuentas por cobrar, etc. **No tiene datos geotécnicos.**
- **Destino.** La PostgreSQL central.
- **Comando.** `python manage.py migrar_mongo [--simular] --destino <base>`.
- **Reglas.** MongoDB solo se lee y se prueba primero contra una copia. El script se puede repetir sin duplicar (`mongo_id`). Nada se corrige en silencio: todo queda en un informe con rechazos y advertencias, y las correcciones que decida la empresa van en `correcciones.yaml`, fuera de git.
- **Probado con copias locales y datos inventados:**
  - 31 casos sucios tratados como se esperaba.
  - Al comparar el backend original con el adaptado: 84 consultas y 75 escrituras con diferencias explicadas.
- **Pendiente:**
  - Copia real de Atlas con un usuario de solo lectura.
  - Revisión del informe con la empresa.
  - Base de producción y fecha de corte.

---

## 10. Reglas del dominio (glosario)

| Término | Qué es |
|---|---|
| **Sondeo** | Perforación en un punto del terreno; se toma una muestra cada 0.50 m |
| **SPT** (ASTM D1586) | Ensayo de penetración: golpes para hundir el muestreador en 3 tramos. **N = tramo 2 + tramo 3.** 50 golpes = rechazo (el criterio exacto de la empresa está pendiente). |
| **Límites de Atterberg** (ASTM D4318) | **LL** (límite líquido): humedad a 25 golpes en la curva de flujo (ajuste log-lineal). **LP** (límite plástico): promedio de las determinaciones. **IP = LL − LP.** Un suelo no plástico se anota NP (LL = LP = 0). |
| **Carta de plasticidad** | Línea A: IP = 0.73 (LL − 20). Línea U: IP = 0.9 (LL − 8). |
| **SUCS / USCS** (ASTM D2487) | Clasificación de suelos. Para los finos: CL si IP > 7 y está sobre la línea A; ML si IP < 4 o está bajo la línea A; CL-ML en la zona intermedia; con LL ≥ 50 la letra es H (CH, MH). Con 5–12 % de finos el símbolo es doble (p. ej. SP-SM). W o P según Cu y Cc. |
| **Nombre del suelo** | Según el porcentaje de material grueso: menos de 15 % sin modificador; de 15 a 29 % «con arena/grava»; 30 % o más «arenoso/gravoso» |
| **AASHTO M145** | Otra clasificación, con índice de grupo (IG). Está pendiente cómo redondear en el hueco entre 40 y 41 %. |
| **Anexo 02** | Hoja de registro por sondeo: profundidades, golpes, N, humedad, descripción, símbolo, porcentajes, LL, LP, IP y patrón de relleno |
| **Patrones de relleno** | 23 imágenes, una por clave símbolo + PLAS/NP + ORG (p. ej. «CL PLAS», «SM NP», «ROCA»). No tienen imagen: GM, GC, CL-ML, SC-SM, GP-GC, SW-SC, SP-SC, PT. |
| **Civil 3D** | Su Geotechnical Modeler importa CSV o AGS 4.0. LandXML no lleva sondeos. |
| **GEO5** | Importa tablas o AGS. Es probable que el perfil estratigráfico que usa la empresa salga de GEO5 Estratigrafía (sin confirmar). |
| **Coordenadas** | Este, Norte y cota se toman del plano. A veces solo hay una imagen del plano, así que se guarda el origen y si son aproximadas. Se usan cotas locales (p. ej. 100.00). |

---

## 11. Hallazgos en los documentos de referencia

Son datos de un cliente real: aquí solo se resumen, sin valores.

- **Regla de la línea A.** Se reprodujo en 45 estratos con LL, LP, IP y finos: 38 coinciden con el símbolo del Anexo y 7 no. Los 7 están a menos de 0.8 puntos de la línea A. Recalcular sin redondear muestra que las hojas no traen la precisión suficiente.
- **Patrones de relleno.** El patrón dibujado coincide con el símbolo y la plasticidad en 181 de 182 filas. La excepción es un patrón tapado por otro.
- **Errores de datos encontrados:**
  - porcentajes que no suman 100 y porcentajes invertidos;
  - un error de dedo («HINO»);
  - «arenoso» usado con menos de 30 % de arena;
  - una hoja SUCS con un símbolo distinto al del Anexo;
  - notas pegadas y anotaciones al margen que se contradicen.
- **Qué son estos documentos.** Sirven como **referencia de formato** (qué se llena y cómo), **no** como una pareja verificada de entrada y salida. El símbolo correcto lo decide el laboratorio.

---

## 12. Cronograma (`Cronograma_Proyecto2.xlsx`)

Dos personas (A y B) y 484 h en total. Fecha máxima: **viernes 15 de enero de 2027**.

| Fechas | Entregable |
|---|---|
| 3–6 nov | Plan aprobado y preguntas enviadas a la empresa |
| 9–13 nov | Macro v0.1: SUCS, AASHTO con IG, casos de prueba |
| 16–20 nov | Macro v1.0: validación con muestras y captura por teclado |
| 23–24 nov | Diseño conjunto del sistema web |
| 25 nov – 2 dic | Transcripción de los 36 folios, prueba de lectura con dos servicios, propuesta de rediseño de hojas → **exactitud medida y servicio elegido** |
| 30 nov – 4 dic | Proyecto en marcha (login, subida de fotos) y motor de reglas en Python |
| 7–11 dic | Lectura automática de la hoja SPT y pantalla de revisión |
| 14–18 dic | Lectura de las hojas de clasificación y Atterberg, Anexo 02 en Excel y PDF |
| 21 dic – 1 ene | Pausa de fin de año |
| 4–8 ene | Exportación a GEO5 y Civil 3D; agrupar estratos, servidor y usuarios |
| 11–15 ene | Piloto: comparar el tiempo por sondeo → **sistema en uso piloto** |

La migración de MongoDB va aparte del cronograma.

---

## 13. Entorno local

- **Python 3.12** con el entorno `backend/.venv` (Django 6.1.3).
  - Pruebas: `backend/.venv/Scripts/python.exe manage.py test` (55 pasan: 41 de lectura y 14 de migración).
- **PostgreSQL 18** portátil en `localhost:5433`, en `C:\Users\spide\herramientas-bd`.
  - Bases de prueba: `jupaca_demo`, `jupaca_sin_demo`, `jupaca_contrato`.
  - Iniciar con `iniciar-bd.ps1`. Si deja de aceptar conexiones con el error 487 de Windows, reiniciarla con `pg_ctl ... stop -m fast` y luego volver a iniciarla.
- **MongoDB 8** portátil en `127.0.0.1:27018`, con copias de prueba. **Nunca se apunta a Atlas** mientras se prueba.
- **Migración pendiente en `jupaca_demo`.** La de `lectura` todavía no se ha aplicado ahí. Se aplica con:

  ```bash
  python manage.py migrate lectura
  ```
- **No tocar** los datos de la instalación anterior de MongoDB en `C:\Program Files\MongoDB\Server\8.3\data`.

---

## 14. Confidencialidad y reglas de trabajo

- **Datos de clientes.** Viven solo en `referencias/` (ignorada por git). No se suben a repositorios ni a servicios externos, y no aparecen en ejemplos públicos. Las demostraciones usan **datos inventados**.
- **Usos autorizados por el usuario:**
  - usar los folios de referencia en pruebas internas;
  - enviar fotos de hojas a un servicio externo de lectura.
- **Imágenes subidas.** Se guardan en `LECTURA_MEDIA_ROOT`, fuera del repositorio, y solo se ven con sesión iniciada.
- **Secretos.** Solo en `backend/.env`, nunca en el código ni en el chat.
- **No reimplementar** Civil 3D ni GEO5. **No inventar** valores, reglas ni formatos de archivo. **No corregir en silencio** ni afirmar una exactitud que no se haya medido.
- **Entregables a la empresa** (cronograma, documentos): no mencionan herramientas de IA.
- **Git:**
  - Repositorio: `CarlosMariommm/Proyecto2-Geotecnia`. Se trabaja **en la raíz** de esta carpeta y siempre contra la misma base PostgreSQL central.
  - **Los commits y el push los hace el usuario a mano.** Aquí solo se preparan los cambios y se le da el comando.
  - En GitHub están «Inicio del proyecto» y «crear estructura inicial del backend con Django y PostgreSQL». El resto de `backend/`, los `.md` y el cronograma están **sin commitear**.
  - El backend Node adaptado tampoco está commiteado ni subido.

---

## 15. Pendiente de confirmar con la empresa

**Laboratorio y reglas:**

- Precisión de LL y LP (entero o con decimales).
- Quién aprueba el símbolo final.
- Criterio del SPT: tramo de 15 o 20 cm, y cuándo hay rechazo.
- Datos de los tamices No. 10 y No. 40.
- Cómo se decide qué ensayo de Atterberg corresponde a qué muestra.
- Patrones de relleno que faltan.
- Glosario de abreviaturas del laboratorio (se puede cargar como sinónimos con origen «cargado»).

**Documentos:**

- Excel original del Anexo 02.
- Hojas impresas en blanco, para confirmar las plantillas de campos.
- Muestras de referencia con su resultado verificado.

**Software:**

- Programa que genera el perfil estratigráfico.
- Un proyecto de ejemplo de GEO5.
- Versión de Civil 3D y si tienen el Geotechnical Modeler.
- Sistema de coordenadas de los planos.
- Versión de Excel y si se permiten macros.

**Operación:**

- Alojamiento del sistema web.
- Tiempo actual por sondeo, como línea base del piloto.
- Volumen mensual de hojas y presupuesto del servicio de lectura.
- Quién rediseña las hojas.
- Qué usaba la persona anterior para estos reportes.

**Migración:**

- Usuario de Atlas con rol `read`.
- Revisión del informe de la migración.
- Base de producción.

---

## 16. Historial

| Fecha | Qué se hizo |
|---|---|
| 3 nov 2026 | **Referencias y plan.** Se revisaron las referencias y se extrajo una muestra de datos del Anexo 02 (182 filas y 112 estratos, en `referencias/_extraccion/`). Se armó la tabla de símbolos → patrones de relleno y se reprodujo la regla de la línea A. Se hizo el plan por fases y se redactaron las preguntas para el encargado del área. |
| 3 nov 2026 | **Decisiones y cronograma.** Se eligió el stack: Django + PostgreSQL, React y macro VBA. Se aclaró que el núcleo es leer las hojas. Se hizo el cronograma en Excel. |
| 3 nov 2026 | **Migración.** En una sesión aparte: esquema de la base central en Django, script de migración MongoDB → PostgreSQL, backend Node adaptado y pruebas contra copias locales. |
| 4 nov 2026 | **Lector de hojas.** App `backend/lectura`: motor de propuestas, aprendizaje de sinónimos y confusiones, reglas de la hoja, API de revisión, comandos para cargar y medir, y 41 pruebas. |

## Siguientes pasos sugeridos

1. Confirmar las plantillas de campos contra las hojas impresas o los folios de referencia.
2. Empezar la macro de la fase 0 (9 de noviembre).
3. Crear las tablas geotécnicas (sondeo, muestra, ensayo, estrato) y el paso de hoja revisada a muestras.
4. Hacer la prueba comparativa de servicios de lectura con los 36 folios transcritos.
5. Hacer la pantalla de revisión en React.
