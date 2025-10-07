# Via Alta — Sistema de Inscripciones (Instituto Vía Diseño)

Proyecto de gestión de horarios e inscripciones para estudiantes del Instituto Vía Diseño. Esta aplicación
permite consultar horarios generales por semestre/carrera, obtener horarios individuales desde un sistema
central y confirmar/guardar horarios de estudiantes en una base de datos PostgreSQL.

## Contenido

- Qué hace
- Funcionalidades principales
- Arquitectura y stack
- Cómo ejecutar localmente
- Variables de entorno necesarias
- Scripts de base de datos
- Endpoints importantes
- Tests y comandos útiles
- Notas de despliegue

## Qué hace

- Consulta horarios de estudiantes (individuales y generales).
- Guarda y confirma horarios elegidos por estudiantes.
- Integra con un sistema central para obtener datos de estudiantes (semestre, carrera, correo).
- Envía correos de confirmación utilizando Resend (configurable).

## Funcionalidades principales

- Frontend con Next.js (App Router) y componentes React para login y visualización de horarios.
- API server con rutas Next.js que:
  - GET /api/student-schedule — devuelve un horario individual o general según disponibilidad.
  - POST /api/student-schedule — persiste un horario elegido y opcionalmente lo confirma (y envía correo).
- Persistencia en PostgreSQL. Scripts para inicializar, poblar y limpiar la base de datos.

## Arquitectura & stack

- Next.js 15 (App Router)
- React + TypeScript
- PostgreSQL (pg)
- Resend para envío de correos (resend package)
- TailwindCSS para estilos
- Jest para pruebas

## Estructura relevante del proyecto

- `src/app` — páginas y rutas API (Next.js)
- `src/components` — UI y componentes principales
- `src/lib` — modelos y helpers de BD (Schedule, Student, etc.)
- `src/config` — configuración (base de datos, mail)

## Cómo ejecutar localmente

### Requisitos

- Node.js 18.x
- npm 9.x
- PostgreSQL (local o remoto)

### Pasos rápidos

1. Instala dependencias

```bash
npm install
```

2. Crea un archivo `.env` con las variables necesarias (ver sección abajo).

3. Inicializa la base de datos (scripts incluidos)

```bash
npm run db:init
# o para cargar grupos/datos de ejemplo
npm run db:init-groups
```

4. Ejecuta en modo desarrollo

```bash
npm run dev
```

Abre http://localhost:3000

## Variables de entorno (resumen)

Coloca estas claves en `.env` antes de ejecutar la app.

- DATABASE_URL — (opcional) URL de conexión completa a Postgres (Heroku style)
- PGUSER, PGHOST, PGDATABASE, PGPASSWORD, PGPORT — detalles de Postgres si no usas DATABASE_URL
- NODE_ENV — development|production
- NEXT_PUBLIC_APP_URL — URL pública (ej. http://localhost:3000)

Integración con sistema central (opcional pero usada por APIs)
- API_BASE_URL
- CLIENT_ID
- CLIENT_SECRET

Email (Resend)
- RESEND_API_KEY
- RESEND_FROM_EMAIL (opcional)

## Scripts de base de datos

- `npm run db:init` — crea tablas y datos iniciales (`src/lib/db-init.js`).
- `npm run db:init-groups` — carga grupos y datos de ejemplo.
- `npm run db:drop` — elimina tablas.
- `npm run db:delete` — elimina datos.

## Endpoints importantes (resumen)

### Consulta y confirmación de horarios

- GET /api/student-schedule?studentId=<ivd_id>&semester=<opt>&degree=<opt>
  - Intenta obtener detalles del estudiante desde el sistema central (semester, degree, email).
  - Si existe horario individual en la DB lo devuelve; si no, devuelve el horario general filtrado por semestre/carrera.
  - Implementación: `src/app/api/student-schedule/route.ts`

- POST /api/student-schedule
  - Payload: { studentId, schedule: [...], confirm: boolean }
  - Flujo: crea registro de estudiante si hace falta, borra horarios previos, inserta nuevas filas para los IdGrupo y, si `confirm` es true, marca confirmado y envía correo.
  - Implementación: `src/app/api/student-schedule/route.ts`

## Flujo de confirmación (simplificado)

1. El cliente solicita GET /api/student-schedule con el ivd_id del estudiante.
2. La API enriquece la información consultando el sistema central si está disponible.
3. Si el estudiante tiene horario individual guardado, se devuelve; si no, se busca el horario general por semestre/carrera.
4. Cuando el cliente publica (POST) el horario elegido, la API guarda los grupos, opcionalmente confirma y dispara un correo de confirmación.

## Tests y comandos útiles

- `npm test` — ejecuta Jest
- `npm run test:watch` — modo watch
- Hay comandos en package.json para tests de integración específicos, p. ej. `test:schedule` o `test:confirmacion`.

## Notas de despliegue

- Hay archivos y notas para desplegar en Heroku (Procfile, heroku_deployment_commands.txt) y consideraciones para Vercel.
- Asegúrate de configurar las variables de entorno en el proveedor de hosting.

## Sugerencias rápidas (mejoras)

- Añadir `.env.example` con valores de ejemplo para onboarding más rápido.
- Incluir un screenshot de la UI en `public/` y añadirlo al README.

## Contacto

Este repositorio contiene el proyecto de curso. Para incluirlo en tu portfolio o para consultas, abre un issue o contacta a los autores.

---
README generado y actualizado para presentar el proyecto en GitHub (setup, endpoints y notas rápidas).
