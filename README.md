# TaskFlow

> Aplicación web para gestionar proyectos y tareas en equipo, con autenticación, tableros Kanban y notificaciones en tiempo real.

[![CI](https://github.com/tu-usuario/taskflow/actions/workflows/ci.yml/badge.svg)](https://github.com/tu-usuario/taskflow/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Node.js](https://img.shields.io/badge/Node.js-20.x-339933?logo=node.js&logoColor=white)](https://nodejs.org/)
[![React](https://img.shields.io/badge/React-18.x-61DAFB?logo=react&logoColor=black)](https://react.dev/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-4169E1?logo=postgresql&logoColor=white)](https://www.postgresql.org/)

## 📋 Tabla de contenidos

- [Demo](#-demo)
- [Características](#-características)
- [Stack tecnológico](#-stack-tecnológico)
- [Arquitectura](#-arquitectura)
- [Flujo de autenticación](#-flujo-de-autenticación)
- [Modelo de datos](#-modelo-de-datos)
- [Requisitos previos](#-requisitos-previos)
- [Instalación](#-instalación)
- [Variables de entorno](#-variables-de-entorno)
- [Scripts disponibles](#-scripts-disponibles)
- [Estructura del proyecto](#-estructura-del-proyecto)
- [Testing](#-testing)
- [Despliegue](#-despliegue)
- [Contribuir](#-contribuir)
- [Licencia](#-licencia)

## 🚀 Demo

- **Aplicación:** https://taskflow.example.com
- **Documentación API:** https://api.taskflow.example.com/docs
- **Usuario de prueba:** `demo@taskflow.dev` / `demo1234`

> Reemplaza estos enlaces por los de tu proyecto. Si aún no tienes demo, elimina esta sección.

## ✨ Características

- Autenticación con JWT y refresh tokens.
- Gestión de proyectos, tareas y etiquetas.
- Tablero Kanban con drag & drop.
- Comentarios y actividad por tarea.
- Notificaciones en tiempo real mediante WebSockets.
- Búsqueda y filtros por estado, prioridad y responsable.
- Diseño responsive y modo oscuro.
- API REST documentada con OpenAPI/Swagger.
- Tests unitarios, de integración y E2E.

## 🧰 Stack tecnológico

| Capa | Tecnología |
|---|---|
| Frontend | React 18, Vite, TypeScript, Tailwind CSS, React Query |
| Backend | Node.js 20, Express, TypeScript, Zod |
| Base de datos | PostgreSQL 16, Prisma ORM |
| Caché / colas | Redis, BullMQ |
| Autenticación | JWT, bcrypt |
| Testing | Vitest, Testing Library, Playwright, Supertest |
| Infraestructura | Docker, GitHub Actions, Vercel / Railway |
| Calidad | ESLint, Prettier, Husky, lint-staged |

## 🏗️ Arquitectura

```mermaid
flowchart TD
    U[Usuario] -->|HTTPS| CDN[CDN / Edge]
    CDN --> FE[Frontend React + Vite]
    FE -->|REST /api| API[API Express]
    API --> AUTH[Servicio de autenticación]
    API --> DB[(PostgreSQL)]
    API --> CACHE[(Redis Cache)]
    API --> QUEUE[Cola de trabajos]
    QUEUE --> WORKER[Worker de notificaciones]
    WORKER --> DB
    WORKER --> WS[WebSocket Server]
    WS --> FE
```

**Descripción rápida:**

- El frontend se sirve como SPA estática desde un CDN.
- La API expone endpoints REST bajo `/api`.
- PostgreSQL es la fuente de verdad; Redis se usa para caché y colas.
- Los workers procesan notificaciones y eventos asíncronos.
- WebSockets mantienen actualizaciones en tiempo real en el cliente.

## 🔐 Flujo de autenticación

```mermaid
sequenceDiagram
    autonumber
    participant U as Usuario
    participant FE as Frontend
    participant API as API
    participant DB as PostgreSQL

    U->>FE: Introduce email y contraseña
    FE->>API: POST /api/auth/login
    API->>DB: Buscar usuario por email
    DB-->>API: Usuario + password_hash
    API->>API: Verificar contraseña con bcrypt
    API-->>FE: accessToken + refreshToken
    FE-->>U: Redirige al dashboard

    Note over FE,API: El accessToken se envía en Authorization: Bearer <token>
```

## 🗃️ Modelo de datos

```mermaid
erDiagram
    USER ||--o{ PROJECT : owns
    PROJECT ||--o{ TASK : contains
    USER ||--o{ COMMENT : writes
    TASK ||--o{ COMMENT : has

    USER {
        uuid id PK
        string email UK
        string password_hash
        string name
        timestamp created_at
    }

    PROJECT {
        uuid id PK
        uuid owner_id FK
        string name
        string description
        timestamp created_at
    }

    TASK {
        uuid id PK
        uuid project_id FK
        string title
        string status
        string priority
        date due_date
        timestamp created_at
    }

    COMMENT {
        uuid id PK
        uuid task_id FK
        uuid user_id FK
        text body
        timestamp created_at
    }
```

## ✅ Requisitos previos

- Node.js >= 20
- pnpm >= 9 o npm >= 10
- Docker y Docker Compose
- PostgreSQL 16 si no usas Docker
- Redis 7 si no usas Docker

## ⚙️ Instalación

```bash
# 1. Clonar el repositorio
git clone https://github.com/tu-usuario/taskflow.git
cd taskflow

# 2. Instalar dependencias
pnpm install

# 3. Copiar variables de entorno
cp .env.example .env

# 4. Levantar servicios locales
docker compose up -d

# 5. Ejecutar migraciones y seed
pnpm db:migrate
pnpm db:seed

# 6. Iniciar en desarrollo
pnpm dev
```

La aplicación estará disponible en:

- Frontend: http://localhost:5173
- API: http://localhost:3000
- Documentación API: http://localhost:3000/docs

## 🔑 Variables de entorno

Crea un archivo `.env` a partir de `.env.example`. Ejemplo:

| Variable | Descripción | Ejemplo |
|---|---|---|
| `DATABASE_URL` | Cadena de conexión a PostgreSQL | `postgresql://user:pass@localhost:5432/taskflow` |
| `REDIS_URL` | URL de conexión a Redis | `redis://localhost:6379` |
| `JWT_SECRET` | Secreto para firmar access tokens | `cambia-este-valor` |
| `JWT_REFRESH_SECRET` | Secreto para refresh tokens | `cambia-este-valor` |
| `PORT` | Puerto de la API | `3000` |
| `CORS_ORIGIN` | Origen permitido del frontend | `http://localhost:5173` |
| `VITE_API_URL` | URL pública de la API para el frontend | `http://localhost:3000/api` |

> ⚠️ Nunca subas `.env` al repositorio. Usa `.env.example` como plantilla.

## 📜 Scripts disponibles

| Comando | Descripción |
|---|---|
| `pnpm dev` | Inicia frontend y backend en modo desarrollo |
| `pnpm build` | Compila frontend y backend para producción |
| `pnpm start` | Ejecuta la API compilada |
| `pnpm test` | Ejecuta tests unitarios |
| `pnpm test:e2e` | Ejecuta tests end-to-end con Playwright |
| `pnpm lint` | Ejecuta ESLint |
| `pnpm format` | Formatea con Prettier |
| `pnpm db:migrate` | Aplica migraciones de Prisma |
| `pnpm db:seed` | Carga datos de ejemplo |
| `pnpm db:studio` | Abre Prisma Studio |

## 📁 Estructura del proyecto

```text
taskflow/
├── apps/
│   ├── web/                 # Frontend React + Vite
│   │   ├── src/
│   │   │   ├── components/
│   │   │   ├── pages/
│   │   │   ├── hooks/
│   │   │   └── lib/
│   │   └── tests/
│   └── api/                 # Backend Express
│       ├── src/
│       │   ├── controllers/
│       │   ├── services/
│       │   ├── routes/
│       │   ├── middleware/
│       │   └── prisma/
│       └── tests/
├── packages/
│   ├── ui/                  # Componentes compartidos
│   └── config/              # Configuración compartida
├── docker-compose.yml
├── .env.example
├── package.json
└── README.md
```

## 🧪 Testing

```bash
# Tests unitarios y de integración
pnpm test

# Tests E2E
pnpm test:e2e

# Cobertura
pnpm test -- --coverage
```

La integración continua se ejecuta en GitHub Actions en cada push y pull request. El workflow está en `.github/workflows/ci.yml`.

## 🚢 Despliegue

### Frontend

El frontend se despliega en Vercel. Configura:

- Build command: `pnpm build`
- Output directory: `apps/web/dist`
- Variable `VITE_API_URL`

### Backend

La API se despliega en Railway o Render usando Docker. Variables necesarias:

- `DATABASE_URL`
- `REDIS_URL`
- `JWT_SECRET`
- `JWT_REFRESH_SECRET`
- `CORS_ORIGIN`

### Base de datos

Usa PostgreSQL gestionado. Ejecuta migraciones en el proceso de release:

```bash
pnpm db:migrate
```

## 🤝 Contribuir

1. Haz un fork del repositorio.
2. Crea una rama: `git checkout -b feature/nueva-funcionalidad`.
3. Realiza tus cambios y añade tests.
4. Ejecuta `pnpm lint && pnpm test`.
5. Abre un pull request describiendo el cambio.

Consulta `CONTRIBUTING.md` para más detalles.

## 📄 Licencia

Este proyecto está bajo la licencia MIT. Consulta [LICENSE](LICENSE) para más información.

## 🙌 Agradecimientos

- [React](https://react.dev/)
- [Vite](https://vitejs.dev/)
- [Express](https://expressjs.com/)
- [Prisma](https://www.prisma.io/)
- [Mermaid](https://mermaid.js.org/)

---

Hecho con ❤️ por el equipo de TaskFlow.
