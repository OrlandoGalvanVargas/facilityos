<div align="center">

<img src="./docs/assets/logo.png" alt="FacilityOS Logo" width="140" />

# FacilityOS

**Plataforma empresarial para la gestión integral de instalaciones educativas**

![.NET](https://img.shields.io/badge/.NET-10.0-512BD4?style=flat-square)
![React](https://img.shields.io/badge/React-19-61DAFB?style=flat-square&logo=react&logoColor=black)
![Vite](https://img.shields.io/badge/Vite-7-646CFF?style=flat-square&logo=vite&logoColor=white)
![SQL Server](https://img.shields.io/badge/SQL_Server-2022-CC2927?style=flat-square)
![License](https://img.shields.io/badge/License-Proprietary-red?style=flat-square)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=flat-square)

[🔗 Sitio en vivo (Mock)](https://da802b4d.facilityos-9er.pages.dev) · [Demo](#demo) · [Arquitectura](#arquitectura-general) · [Backend](./server/README.md) · [Frontend](./client/README.md)

</div>

---

## Tabla de Contenidos

- [Descripción General](#descripción-general)
- [Demo](#demo)
- [Arquitectura General](#arquitectura-general)
- [Estructura del Monorepo](#estructura-del-monorepo)
- [Stack Tecnológico](#stack-tecnológico)
- [Módulos de Negocio](#módulos-de-negocio)
- [Roles y Permisos](#roles-y-permisos)
- [Primeros Pasos](#primeros-pasos)
- [Modo Mock vs. Modo API](#modo-mock-vs-modo-api)
- [Estado de Despliegue](#estado-de-despliegue)
- [Documentación por Servicio](#documentación-por-servicio)
- [Licencia](#licencia)

---

## Descripción General

**FacilityOS** es una plataforma SaaS multi-tenant para la administración de distritos escolares, escuelas, personal académico (faculty) y dispositivos de seguridad tipo *beacon* utilizados en protocolos de emergencia.

El proyecto está organizado como un **monorepo** con dos servicios independientes y desacoplados entre sí:

| Servicio | Carpeta | Descripción |
|---|---|---|
| **Backend** | [`/server`](./server) | API RESTful en .NET 10 bajo Vertical Slice Architecture + CQRS |
| **Frontend** | [`/client`](./client) | SPA en React 19 con soporte offline y modo mock integrado |

Cada carpeta cuenta con su propio README detallado con instrucciones específicas de instalación, arquitectura interna y convenciones de código.

---

## Demo

![Demostración de la aplicación](./docs/assets/demo.gif)

🔗 **Pruébalo en vivo:** [https://da802b4d.facilityos-9er.pages.dev](https://da802b4d.facilityos-9er.pages.dev)

El frontend puede probarse completamente **sin backend**, gracias a su modo mock (ver [Modo Mock vs. Modo API](#modo-mock-vs-modo-api)).

**Credenciales de acceso (modo mock):**

```
Usuario:     admin@facilityos.com
Contraseña:  admin123
```

Este usuario inicia sesión con rol **Admin**, con acceso completo a todos los módulos y acciones del sistema (Distritos, Escuelas, Usuarios, Beacons y Facultades).

---

## Arquitectura General

```
┌───────────────────────────┐         ┌───────────────────────────┐
│      FacilityOS Client    │  HTTPS  │      FacilityOS API       │
│   React 19 + Vite (SPA)   │ ──────► │   .NET 10 (REST + JWT)    │
│   /client                 │ ◄────── │   /server                 │
└───────────────────────────┘         └──────────────┬────────────┘
         │                                            │
         │ modo mock (MSW + IndexedDB)                │ EF Core
         ▼                                            ▼
   Sin backend requerido                        SQL Server 2022
```

- El **frontend** consume la API vía REST/JSON con autenticación JWT, o funciona de forma completamente local usando **Mock Service Worker (MSW)** cuando no hay backend disponible.
- El **backend** sigue Vertical Slice Architecture con tres capas físicas (`Domain` → `Application` → `API`) y aplica sus propias migraciones de base de datos automáticamente al arrancar.
- Ambos servicios son independientes en despliegue: el frontend no requiere que el backend esté en línea para funcionar en modo demo/mock.

---

## Estructura del Monorepo

```text
facilityos/
├── client/                 # Frontend — React 19 + Vite
│   ├── src/
│   ├── Dockerfile
│   ├── docker-compose.production.yml
│   └── README.md
├── server/                  # Backend — .NET 10 API
│   ├── FacilityOS.API/
│   ├── FacilityOS.Application/
│   ├── FacilityOS.Domain/
│   ├── FacilityOS.Tests/
│   ├── Dockerfile
│   ├── docker-compose.local.yml
│   └── README.md
├── docs/                    # Logo, capturas
└── README.md                # Este archivo
```

---

## Stack Tecnológico

| Capa | Tecnologías |
|---|---|
| **Frontend** | React 19, Vite 7, Ant Design 6, TanStack Query 5, Zustand 5, React Router 7, Axios, MSW, idb-keyval |
| **Backend** | .NET 10, C# 14, ASP.NET Core, EF Core 10, MediatR (CQRS), FluentValidation, JWT Bearer, Serilog |
| **Base de Datos** | SQL Server 2022 |
| **Infraestructura** | Docker (multi-stage), Nginx (frontend), despliegue independiente por servicio |
| **Testing** | xUnit (backend) · Vitest + Testing Library (frontend) |

---

## Módulos de Negocio

| Módulo | Descripción |
|---|---|
| **Auth** | Login, refresh de sesión, logout, perfil autenticado |
| **Distritos** | Gestión de distritos escolares (jerarquía raíz) |
| **Escuelas** | CRUD con asignación a distrito, filtros por nivel/tipo |
| **Usuarios** | Gestión de cuentas con roles y alcance jerárquico |
| **Beacons** | Dispositivos de emergencia (pendant, wristband, fixed, mobile) |
| **Facultades** | Personal académico, asignación a distrito/escuela y beacon opcional |

---

## Roles y Permisos

| Capacidad | Admin | DistrictAdmin | SchoolAdmin |
|---|---|---|---|
| Distritos | CRUD completo | Solo lectura | Solo lectura |
| Escuelas | CRUD completo | Su distrito | Su escuela |
| Usuarios | CRUD completo | Su distrito | Su escuela |
| Beacons | CRUD completo | Solo lectura | Solo lectura |
| Facultades | CRUD completo | Su distrito | Su escuela |

La jerarquía de roles y el aislamiento multi-tenant se validan tanto en el backend (autoridad final) como en el frontend (a nivel de UX).

---

## Primeros Pasos

### Clonar el repositorio

```bash
git clone https://github.com/OrlandoGalvanVargas/architecture-districts.git
cd facilityos
```

### Frontend (modo mock, sin backend)

```bash
cd client
npm install
cp .env.example .env
# Asegúrate de que VITE_ENABLE_MOCK=true en tu .env
npm run dev
```

Disponible en `http://localhost:3000`. Inicia sesión con las [credenciales de demo](#demo).

### Backend (opcional, para desarrollo full-stack)

```bash
cd server
dotnet restore
dotnet ef database update --project FacilityOS.API
dotnet run --project FacilityOS.API
```

Disponible en `http://localhost:5000/swagger`.

> Instrucciones detalladas, variables de entorno y ejecución vía Docker en el README de cada servicio: [`/client/README.md`](./client/README.md) · [`/server/README.md`](./server/README.md)

---

## Modo Mock vs. Modo API

El frontend soporta dos modos de operación, controlados por la variable de entorno `VITE_ENABLE_MOCK`:

| Modo | Variable | Comportamiento |
|---|---|---|
| **Mock (local)** | `VITE_ENABLE_MOCK=true` | Toda la aplicación funciona sin backend, usando Mock Service Worker (MSW) y datos persistidos en IndexedDB. Ideal para demos y evaluación del proyecto. |
| **API (real)** | `VITE_ENABLE_MOCK=false` | El frontend consume la API real de `/server` mediante `VITE_API_URL`. |

Esto permite que el frontend se pueda desplegar y usar de forma completamente autónoma, sin depender de que el backend esté en línea.

---

## Estado de Despliegue

| Servicio | Estado | Notas |
|---|---|---|
| **Frontend** | 🟢 Desplegado en **Cloudflare Pages**: [https://da802b4d.facilityos-9er.pages.dev](https://da802b4d.facilityos-9er.pages.dev) | Funciona en **modo mock** (`VITE_ENABLE_MOCK=true`). No consume la API real, ya que el backend no se mantendrá en línea de forma indefinida. |
| **Backend** | ⚪ No desplegado | Listo para desplegarse: incluye Dockerfile multi-stage, migraciones automáticas (Database-as-Code) y configuración por variables de entorno. Compatible con **Render**, **Railway**, **Azure App Service / Container Apps**, o **AWS ECS / Elastic Beanstalk**, entre otros proveedores con soporte para contenedores .NET y SQL Server. |

Si en algún momento se despliega el backend, basta con configurar `VITE_API_URL` y `VITE_ENABLE_MOCK=false` en el frontend para conectarlo a la API real.

---

## Documentación por Servicio

Para detalles de arquitectura interna, endpoints, seguridad, convenciones de código y configuración específica de cada servicio:

- 📘 **Backend:** [`/server/README.md`](./server/README.md)
- 📗 **Frontend:** [`/client/README.md`](./client/README.md)

---

## Licencia

**Copyright © 2026 FacilityOS. Todos los derechos reservados.**

Este software, incluyendo su código fuente, arquitectura y documentación, es propiedad exclusiva de FacilityOS y se distribuye bajo licencia **propietaria**. Queda prohibida su reproducción, distribución, modificación, ingeniería inversa o uso total o parcial fuera del alcance autorizado, sin el consentimiento previo y por escrito del propietario.

Para consultas sobre licenciamiento, colaboración o uso autorizado, contactar al equipo responsable del proyecto.
