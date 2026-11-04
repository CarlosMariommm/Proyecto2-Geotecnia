# Plataforma JUPACA — backend (Django)

Proyecto Django dueño del **esquema de la base central** PostgreSQL de la empresa. Contiene:

| App | Contenido |
|---|---|
| `nucleo` | Usuarios, departamentos, empleados, clientes, servicios, proyectos (con técnicos y avances) y reportes semanales |
| `rrhh` | Contratos, horas extra, planillas, líneas de planilla y boletas |
| `finanzas` | Proformas, cuentas por cobrar, sus ítems y pagos |
| `migracion` | Script para copiar la base MongoDB del sistema administrativo (ver [`migracion/README.md`](migracion/README.md)) |
| `lectura` | Lectura de las hojas del laboratorio desde una foto, revisión celda por celda y aprendizaje de las correcciones (ver [`lectura/README.md`](lectura/README.md)) |

El backend Node del sistema administrativo usa estas mismas tablas, pero no las crea ni las modifica: cualquier cambio de esquema se hace aquí, con `makemigrations` y `migrate`.

## Puesta en marcha

```bash
cd backend
py -3.12 -m venv .venv
.venv\Scripts\pip install -r requirements.txt
copy .env.example .env      # y completar los valores
.venv\Scripts\python manage.py migrate
```

Configuración (`config/settings/`):

- `base.py`: lo común. Lee `backend/.env` con python-decouple; las variables de entorno tienen prioridad.
- `development.py`: DEBUG activo. Es la que usan `manage.py`, `wsgi.py` y `asgi.py` por defecto.
- `production.py`: HTTPS y cookies seguras. Se activa con `DJANGO_SETTINGS_MODULE=config.settings.production`.

La base es la **PostgreSQL central**, la misma del sistema administrativo. Se indica con `DATABASE_URL` o, como alternativa, con `DB_NAME`, `DB_USER`, `DB_PASSWORD`, `DB_HOST` y `DB_PORT`.

## Convenciones del esquema

- **Llave primaria UUID versión 7.** Empieza con la hora de creación (ver `nucleo/ids.py`).
- **`mongo_id`.** Guarda el `_id` del documento de MongoDB de origen; queda vacío en lo creado después.
- **`datos_extra` (jsonb).** Guarda los campos que el documento traía fuera del esquema.
- **Montos en `NUMERIC(14,4)`.**
- **Restricciones en la propia base.** Estados y tipos son listas cerradas con `CHECK`, hay unicidad sin distinguir mayúsculas en los correos, un solo contrato activo por empleado y fechas en orden.
- **Contraseñas con bcrypt** (`nucleo/hashers.py`). Es el mismo algoritmo que usa el backend Node.
