# 📚 GUÍA TÉCNICA COMPLETA - Portfolio Administrativo

Una guía detallada para entender cada componente, endpoint y funcionalidad del proyecto. Diseñada para responder cualquier pregunta técnica durante la presentación.

---

## 📋 TABLA DE CONTENIDOS

1. [Descripción General](#descripción-general)
2. [Arquitectura del Proyecto](#arquitectura-del-proyecto)
3. [Base de Datos y Entidades](#base-de-datos-y-entidades)
4. [Seguridad y Autenticación](#seguridad-y-autenticación)
5. [Endpoints REST - Backend](#endpoints-rest---backend)
6. [Flujos de Frontend](#flujos-de-frontend)
7. [Guardias y Protección de Rutas](#guardias-y-protección-de-rutas)
8. [Servicios y Lógica de Negocio](#servicios-y-lógica-de-negocio)

---

## Descripción General

### ¿Qué es el proyecto?

Es una **plataforma de portafolio administrativo** que permite:
- **Administradores**: gestionar usuarios, roles, ver reportes
- **Programadores**: crear perfil, agregar proyectos, definir disponibilidad, responder solicitudes de asesoría
- **Usuarios**: ver perfiles de programadores, solicitar asesorías, consultar estado de sus solicitudes

### Stack Tecnológico

| Parte | Tecnología |
|-------|-----------|
| **Backend** | Spring Boot 3.4.2, Java 21, Gradle |
| **Frontend** | Angular 20, TypeScript, Standalone Components |
| **Base de Datos** | PostgreSQL (Spring Data JPA) |
| **Autenticación** | JWT (Bearer Token) + Google OAuth |
| **Estilos** | Tailwind CSS + DaisyUI |
| **Reportes** | OpenPDF (PDF), Apache POI (Excel) |
| **Imágenes** | Cloudinary (cloud storage) |

---

## Arquitectura del Proyecto

### Estructura de Carpetas - Backend

```
icc-portafolio-backend/src/main/java/ec/edu/ups/icc/portafolio_backend/
├── user/                    # Gestión de usuarios y autenticación
│   ├── controller/         # AuthController, UserProfileController
│   ├── service/           # AuthService, UserProfileService
│   ├── entity/           # User.java
│   ├── dto/             # LoginRequest, AuthResponse
│   └── repository/       # UserRepository
│
├── programmer/              # Gestión de perfil, proyectos, asesorías
│   ├── controller/         # ProgrammerController, PublicController
│   ├── service/           # ProgrammerService, ProjectService, AvailabilityService, AdvisoryService
│   ├── entity/           # ProgrammerProfile.java, Project.java, Advisory.java
│   ├── dto/             # ProgrammerProfileResponse, AdvisoryResponse
│   └── repository/       # ProgrammerProfileRepository, ProjectRepository, AdvisoryRepository
│
├── admin/                   # Gestión administrativa
│   ├── controller/         # AdminController, ReportController
│   ├── service/           # AdminService, ReportService
│   └── dto/             # UserResponse, CreateUserRequest
│
└── shared/                  # Código compartido
    ├── config/           # SecurityConfig.java (Spring Security)
    ├── security/         # JwtService.java, JwtAuthFilter.java
    └── util/            # MailTemplates.java, etc.
```

### Estructura de Carpetas - Frontend

```
icc-portafolio-frontend/src/app/
├── core/                   # Servicios y Guards (no reutilizables en componentes)
│   ├── guards/           # auth.guard.ts, role.guards.ts, redirect-by-role.guard.ts
│   ├── services/         # AuthService, ProgrammerService, UserService
│   ├── interceptors/     # auth.interceptor.ts (agrega token a peticiones)
│   └── ui/              # toast.service.ts
│
├── features/               # Módulos específicos por feature
│   ├── auth/            # Páginas de login y registro
│   └── landing/         # Página de inicio pública
│
├── pages/                 # Rutas principales (admin, programmer, user)
│   ├── admin-page/
│   ├── programmer-page/
│   └── user-page/
│
├── shared/               # Componentes reutilizables
│   └── components/      # notifications-bell,  etc.
│
└── app.routes.ts         # Configuración de rutas
```

---

## Base de Datos y Entidades

### 1. Tabla: `users` (Entidad: User.java)

**Archivo**: `icc-portafolio-backend/src/main/java/.../user/entity/User.java`

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | BIGINT (PK) | Identificador único |
| `name` | VARCHAR | Nombre del usuario |
| `email` | VARCHAR (UNIQUE) | Email único, usado para login |
| `password` | VARCHAR | Contraseña hasheada (BCrypt) |
| `role` | ENUM | `ADMIN`, `PROGRAMMER`, `USER` |
| `enabled` | BOOLEAN | Usuario activo/inactivo |
| `photoUrl` | VARCHAR | URL de foto de perfil (Cloudinary) |
| `headline` | VARCHAR | Título profesional |
| `bio` | VARCHAR(1000) | Biografía |
| `createdAt` | TIMESTAMP | Fecha de creación |

**Relaciones**:
- Un User puede tener un ProgrammerProfile (si su rol es PROGRAMMER)

---

### 2. Tabla: `programmer_profile` (Entidad: ProgrammerProfile.java)

**Archivo**: `icc-portafolio-backend/src/main/java/.../programmer/entity/ProgrammerProfile.java`

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | BIGINT (PK) | Identificador único |
| `user_id` | BIGINT (FK) | Relación con User |
| `specialty` | VARCHAR | Especialidad (ej: "Backend", "Frontend") |
| `bio` | VARCHAR | Biografía profesional |
| `photoUrl` | VARCHAR | Foto del programador |
| [otros campos] | | Campos editables por el programador |

**Relaciones**:
- 1 User ↔ 1 ProgrammerProfile
- 1 ProgrammerProfile ↔ N Projects
- 1 ProgrammerProfile ↔ N AvailabilitySlot
- 1 ProgrammerProfile ↔ N Advisory (advisories)

---

### 3. Tabla: `project` (Entidad: Project.java)

**Archivo**: `icc-portafolio-backend/src/main/java/.../programmer/entity/Project.java`

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | BIGINT (PK) | Identificador único |
| `profile_id` | BIGINT (FK) | Pertenece a qué programador |
| `name` | VARCHAR | Nombre del proyecto |
| `description` | VARCHAR(2000) | Descripción detallada |
| `technologies` | ARRAY | Lista de tecnologías usadas |
| `participation` | ENUM | "SOLO", "TEAM", "LEAD" |
| `section` | ENUM | "FEATURED", "PORTFOLIO", "ACADEMIC" |
| `repoUrl` | VARCHAR | URL del repositorio |
| `demoUrl` | VARCHAR | URL de demostración |
| `imageUrl` | VARCHAR | Imagen del proyecto |
| `active` | BOOLEAN | Proyecto visible o no |

**¿Para qué sirve?**
- Permite que programadores exhiban sus trabajos
- Los usuarios ven estos proyectos en el perfil público del programador

---

### 4. Tabla: `availability_slot` (Entidad: AvailabilitySlot.java)

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | BIGINT (PK) | Identificador único |
| `profile_id` | BIGINT (FK) | Programador que define disponibilidad |
| `day` | VARCHAR | Día de la semana (MONDAY, TUESDAY, etc.) |
| `startTime` | TIME | Hora de inicio (ej: "14:00") |
| `endTime` | TIME | Hora de finalización (ej: "18:00") |
| `modality` | VARCHAR | Modalidad (PRESENCIAL, VIRTUAL, HIBRIDO) |

**¿Para qué sirve?**
- Define cuándo el programador está disponible para asesorías
- Se usa para generar slots de tiempo futuro (próximas 4 semanas)

---

### 5. Tabla: `advisory` (Entidad: Advisory.java)

**Archivo**: `icc-portafolio-backend/src/main/java/.../programmer/entity/Advisory.java`

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | BIGINT (PK) | Identificador único |
| `profile_id` | BIGINT (FK) | Programador que recibe la solicitud |
| `requesterName` | VARCHAR | Nombre de quien solicita |
| `requesterEmail` | VARCHAR | Email de quien solicita |
| `scheduledAt` | TIMESTAMP | Fecha y hora de la asesoría |
| `comment` | VARCHAR(2000) | Comentario del solicitante |
| `status` | ENUM | `PENDIENTE`, `APROBADA`, `RECHAZADA` |
| `response` | VARCHAR(2000) | Respuesta del programador |
| `createdAt` | TIMESTAMP | Cuándo se creó la solicitud |
| `reminderSent` | BOOLEAN | ¿Se envió recordatorio? |

**¿Para qué sirve?**
- Registra solicitudes de asesoría
- El programador puede aprobar o rechazar
- El usuario consulta el estado de sus solicitudes

---

## Seguridad y Autenticación

### Flujo de Autenticación

#### 1. **Login con Email/Contraseña**

**Endpoint**: `POST /api/auth/login`

**Solicitud**:
```json
{
  "email": "usuario@example.com",
  "password": "miContraseña123"
}
```

**Respuesta**:
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "name": "Juan Pérez",
  "role": "PROGRAMMER"
}
```

**¿Qué hace?**
1. Autentica con Spring Security
2. Genera un JWT con `JwtService.generateToken(email)`
3. Devuelve token + info del usuario
4. Frontend guarda el token en `localStorage` como `auth_user`

**Archivo**: `icc-portafolio-backend/src/main/java/.../user/service/AuthService.java`

---

#### 2. **Login con Google**

**Endpoint**: `POST /api/auth/google`

**Solicitud**:
```json
{
  "idToken": "eyJhbGciOiJSUzI1NiI..."
}
```

**Respuesta**: Igual que login normal

**¿Qué hace?**
1. Valida el `idToken` de Google
2. Extrae email y nombre
3. Si no existe, crea usuario con rol USER
4. Genera JWT y devuelve

**Archivo**: `icc-portafolio-backend/src/main/java/.../user/service/AuthService.java` (método `loginWithGoogle`)

---

#### 3. **Persistencia del Token**

**Archivo Frontend**: `icc-portafolio-frontend/src/app/core/services/auth.service.ts`

```typescript
// Constructor carga sesión desde localStorage
constructor() {
  const raw = localStorage.getItem('auth_user');
  if (raw) {
    this.currentUserSignal.set(JSON.parse(raw));
  }
}

// saveSession guarda en localStorage
private saveSession(user: AuthUser) {
  localStorage.setItem('auth_user', JSON.stringify(user));
  this.currentUserSignal.set(user);
}
```

**¿Por qué?**
- El token persiste aunque recargues la página
- El usuario permanece logueado

---

#### 4. **Autorización por Rol**

**En Backend** (Spring Security):
```java
@PreAuthorize("hasRole('PROGRAMMER')")
@GetMapping("/me")
public ProgrammerProfileResponse getProfile(Authentication auth) {
  // Solo programadores pueden acceder
}

@PreAuthorize("hasRole('ADMIN')")
@GetMapping("/users")
public List<UserResponse> listUsers() {
  // Solo admins pueden acceder
}
```

**En Frontend** (Guards):
```typescript
// roleGuard valida que el rol sea permitido
export const roleGuard = (allowedRoles: Role[]): CanActivateFn => {
  return async () => {
    const user = auth.currentUser();
    if (!allowedRoles.includes(user.role)) {
      router.navigate(['/user']);
      return false;
    }
    return true;
  };
};
```

**Archivos**:
- Backend: `icc-portafolio-backend/src/main/java/.../shared/config/SecurityConfig.java`
- Frontend: `icc-portafolio-frontend/src/app/core/guards/role.guards.ts`

---

### JWT y Headers

**¿Cómo se envía el token?**

Cada solicitud autenticada incluye:
```
Authorization: Bearer eyJhbGciOiJIUzI1NiI...
```

**¿Dónde se agrega?**

- **Backend**: `JwtAuthFilter` lo valida
- **Frontend**: `authInterceptor` lo agrega automáticamente

```typescript
// icc-portafolio-frontend/src/app/core/interceptors/auth.interceptor.ts
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const authUserJson = localStorage.getItem('auth_user');
  const token = authUserJson ? JSON.parse(authUserJson).token : null;

  if (token) {
    const authReq = req.clone({
      setHeaders: {
        Authorization: `Bearer ${token}`
      }
    });
    return next(authReq);
  }
  return next(req);
};
```

---

## Endpoints REST - Backend

### 📌 AUTENTICACIÓN

#### `POST /api/auth/register`
- **Descripción**: Registrar nuevo usuario con rol USER
- **Body**:
  ```json
  {
    "name": "Juan",
    "email": "juan@example.com",
    "password": "Pass1234"
  }
  ```
- **Respuesta**: AuthResponse (token + role)
- **Protección**: NINGUNA (público)
- **Archivo**: `AuthController.java`

#### `POST /api/auth/login`
- **Descripción**: Login con email/contraseña
- **Body**:
  ```json
  {
    "email": "juan@example.com",
    "password": "Pass1234"
  }
  ```
- **Respuesta**: AuthResponse
- **Protección**: NINGUNA (público)

#### `POST /api/auth/register-admin`
- **Descripción**: Registrar PRIMER admin (solo funciona si no hay admin)
- **Body**: Igual a `/register`
- **Respuesta**: AuthResponse con role = ADMIN
- **Protección**: NINGUNA (público)
- **⚠️ IMPORTANTE**: Solo se puede ejecutar UNA VEZ

#### `POST /api/auth/google`
- **Descripción**: Login con Google OAuth
- **Body**:
  ```json
  {
    "idToken": "eyJhbGciOiJSUzI1NiI..."
  }
  ```
- **Respuesta**: AuthResponse
- **Protección**: NINGUNA (público)

---

### 📌 USUARIOS (Perfil)

#### `GET /api/users/me`
- **Descripción**: Obtener perfil del usuario autenticado
- **Respuesta**:
  ```json
  {
    "id": 1,
    "name": "Juan",
    "email": "juan@example.com",
    "role": "USER",
    "photoUrl": "https://...",
    "headline": "Abogado",
    "bio": "Soy abogado especializado..."
  }
  ```
- **Protección**: ✅ Requiere JWT (cualquier rol)
- **Archivo**: `UserProfileController.java`

#### `PUT /api/users/me`
- **Descripción**: Actualizar perfil del usuario autenticado
- **Body**:
  ```json
  {
    "name": "Juan Updated",
    "photoUrl": "https://...",
    "headline": "Abogado Senior",
    "bio": "..."
  }
  ```
- **Respuesta**: UserProfile actualizado
- **Protección**: ✅ Requiere JWT

---

### 📌 ADMINISTRACIÓN (Solo ADMIN)

#### `POST /api/admin/users`
- **Descripción**: Crear usuario (asignar rol específico)
- **Body**:
  ```json
  {
    "name": "Carlos",
    "email": "carlos@example.com",
    "password": "Pass1234",
    "role": "PROGRAMMER"
  }
  ```
- **Respuesta**: UserResponse
- **Protección**: ✅ Solo ADMIN
- **Archivo**: `AdminController.java`

#### `GET /api/admin/users?page=0&size=50`
- **Descripción**: Listar todos los usuarios
- **Respuesta**: `List<UserResponse>`
- **Protección**: ✅ Solo ADMIN

#### `PUT /api/admin/users/{id}`
- **Descripción**: Editar usuario existente
- **Body**:
  ```json
  {
    "name": "Carlos Updated",
    "role": "USER"
  }
  ```
- **Respuesta**: UserResponse
- **Protección**: ✅ Solo ADMIN

#### `DELETE /api/admin/users/{id}`
- **Descripción**: Eliminar usuario
- **Respuesta**: Sin contenido (204)
- **Protección**: ✅ Solo ADMIN

---

### 📌 PROGRAMADORES (Privado - Solo PROGRAMMER)

#### `GET /api/programmer/me`
- **Descripción**: Obtener perfil del programador autenticado
- **Respuesta**:
  ```json
  {
    "id": 1,
    "name": "Carlos Programador",
    "email": "carlos@example.com",
    "specialty": "Backend Java",
    "bio": "Experto en Spring Boot",
    "photoUrl": "https://..."
  }
  ```
- **Protección**: ✅ Solo PROGRAMMER
- **Archivo**: `ProgrammerController.java`

#### `PUT /api/programmer/profile`
- **Descripción**: Actualizar perfil como programador
- **Body**:
  ```json
  {
    "specialty": "Full Stack",
    "bio": "Experto en Angular y Spring"
  }
  ```
- **Respuesta**: ProgrammerProfileResponse
- **Protección**: ✅ Solo PROGRAMMER

#### `POST /api/programmer/projects`
- **Descripción**: Crear nuevo proyecto
- **Body**:
  ```json
  {
    "name": "Aplicación de Portafolio",
    "description": "Plataforma para mostrar trabajos...",
    "technologies": ["Angular", "Spring Boot", "PostgreSQL"],
    "participation": "LEAD",
    "section": "FEATURED",
    "repoUrl": "https://github.com/...",
    "demoUrl": "https://app.example.com",
    "imageUrl": "https://cloudinary.com/..."
  }
  ```
- **Respuesta**: ProjectResponse
- **Protección**: ✅ Solo PROGRAMMER

#### `GET /api/programmer/projects`
- **Descripción**: Listar proyectos del programador autenticado
- **Respuesta**: `List<ProjectResponse>`
- **Protección**: ✅ Solo PROGRAMMER

#### `PUT /api/programmer/projects/{id}`
- **Descripción**: Editar proyecto
- **Body**: Igual a creación
- **Respuesta**: ProjectResponse
- **Protección**: ✅ Solo PROGRAMMER

#### `DELETE /api/programmer/projects/{id}`
- **Descripción**: Eliminar proyecto
- **Respuesta**: Sin contenido (204)
- **Protección**: ✅ Solo PROGRAMMER

---

#### `POST /api/programmer/availability`
- **Descripción**: Agregar horario de disponibilidad
- **Body**:
  ```json
  {
    "day": "MONDAY",
    "startTime": "14:00",
    "endTime": "18:00",
    "modality": "VIRTUAL"
  }
  ```
- **Respuesta**: AvailabilityResponse
- **Protección**: ✅ Solo PROGRAMMER
- **¿Para qué?**: Define cuándo puede dar asesorías

#### `GET /api/programmer/availability`
- **Descripción**: Listar disponibilidad del programador
- **Respuesta**: `List<AvailabilityResponse>`
- **Protección**: ✅ Solo PROGRAMMER

#### `DELETE /api/programmer/availability/{id}`
- **Descripción**: Eliminar un horario de disponibilidad
- **Respuesta**: Sin contenido (204)
- **Protección**: ✅ Solo PROGRAMMER

---

#### `GET /api/programmer/advisories`
- **Descripción**: Listar asesorías solicitadas al programador
- **Respuesta**:
  ```json
  [
    {
      "id": 1,
      "requesterName": "Juan",
      "requesterEmail": "juan@example.com",
      "scheduledAt": "2026-02-15T15:00:00",
      "status": "PENDIENTE",
      "comment": "Necesito ayuda con..."
    }
  ]
  ```
- **Protección**: ✅ Solo PROGRAMMER
- **¿Para qué?**: Programador ve solicitudes que recibió

#### `PATCH /api/programmer/advisories/{id}`
- **Descripción**: Aprobar o rechazar solicitud de asesoría
- **Body**:
  ```json
  {
    "status": "APROBADA",
    "response": "Me da gusto ayudarte. Nos vemos el..."
  }
  ```
- **Respuesta**: AdvisoryResponse actualizado
- **Protección**: ✅ Solo PROGRAMMER

#### `GET /api/programmer/advisories/requester`
- **Descripción**: Listar asesorías solicitadas por el usuario autenticado
- **Respuesta**: `List<AdvisoryResponse>`
- **Protección**: ✅ Solo USER
- **¿Para qué?**: Usuario ve sus propias solicitudes
- **Archivo**: `ProgrammerController.java`

---

### 📌 PÚBLICOS (Sin autenticación)

#### `GET /api/programmers?page=0&size=50`
- **Descripción**: Listar todos los programadores
- **Respuesta**:
  ```json
  [
    {
      "id": 1,
      "name": "Carlos",
      "specialty": "Backend",
      "photoUrl": "https://...",
      "bio": "Experto en Java"
    }
  ]
  ```
- **Protección**: NINGUNA (público)
- **Archivo**: `PublicController.java`
- **¿Para qué?**: Usuarios ven qué programadores hay

#### `GET /api/programmers/{id}`
- **Descripción**: Obtener detalles de un programador
- **Respuesta**: ProgrammerProfileResponse completo
- **Protección**: NINGUNA (público)

#### `GET /api/programmers/{id}/projects`
- **Descripción**: Listar proyectos de un programador
- **Respuesta**: `List<ProjectResponse>`
- **Protección**: NINGUNA (público)
- **¿Para qué?**: Ver portafolio del programador

#### `GET /api/programmers/{id}/availability`
- **Descripción**: Listar disponibilidad de un programador
- **Respuesta**: `List<AvailabilityResponse>`
- **Protección**: NINGUNA (público)
- **¿Para qué?**: Saber cuándo está disponible para asesoría

#### `POST /api/advisories`
- **Descripción**: Crear solicitud de asesoría
- **Body**:
  ```json
  {
    "programmerProfileId": 1,
    "requesterName": "Juan",
    "requesterEmail": "juan@example.com",
    "scheduledAt": "2026-02-15T15:00:00",
    "comment": "Necesito ayuda con..."
  }
  ```
- **Respuesta**: AdvisoryResponse
- **Protección**: NINGUNA (público)
- **¿Para qué?**: Usuario solicita asesoría

#### `GET /api/advisories?email=juan@example.com`
- **Descripción**: Obtener asesorías de un solicitante (sin login)
- **Respuesta**: `List<AdvisoryResponse>`
- **Protección**: NINGUNA (público, pero requiere email en query)
- **¿Para qué?**: Usuario no logueado puede ver sus solicitudes con su email

---

### 📌 REPORTES (Solo ADMIN)

#### `GET /api/reports/advisories?from=2026-01-01&to=2026-12-31&status=APROBADA`
- **Descripción**: Resumen de asesorías por rango de fechas y estado
- **Respuesta**: `List<AdvisoryReportItem>`
- **Protección**: ✅ Solo ADMIN
- **Archivo**: `ReportController.java`

#### `GET /api/reports/projects`
- **Descripción**: Resumen de proyectos
- **Respuesta**: `List<ProjectReportItem>`
- **Protección**: ✅ Solo ADMIN

#### `GET /api/reports/advisories/pdf`
- **Descripción**: Descargar reporte de asesorías en PDF
- **Respuesta**: PDF file
- **Protección**: ✅ Solo ADMIN

#### `GET /api/reports/projects/excel`
- **Descripción**: Descargar reporte de proyectos en Excel
- **Respuesta**: Excel file
- **Protección**: ✅ Solo ADMIN

---

## Flujos de Frontend

### 1. **Flujo de Registro e Inicio de Sesión**

**Archivo**: `icc-portafolio-frontend/src/app/features/auth/pages/login-page/login-page.ts`

```
Usuario abre /login
    ↓
redirectByRoleGuard verifica si ya está logueado
    - Si SÍ: redirige a /admin, /programmer o /user según rol
    - Si NO: permite ver login
    ↓
Usuario ingresa email y contraseña
    ↓
login() → AuthService.login(email, password)
    ↓
Backend: POST /api/auth/login
    ↓
Backend devuelve: { token, name, role }
    ↓
Frontend: AuthService.saveSession(user)
    - Guarda en localStorage auth_user = { ...user }
    ↓
Frontend: router.navigate(['/'])
    ↓
✅ Usuario logueado, sesión persistida
```

**¿Qué pasa si recarga la página?**
- Constructor de AuthService lee `localStorage.getItem('auth_user')`
- Restaura sesión automáticamente
- `currentUserSignal` tiene los datos del usuario

---

### 2. **Flujo de Usuario Normal (USER)**

**Ruta**: `/user`

**Archivo**: `icc-portafolio-frontend/src/app/pages/user-page/user-page.ts`

```
1. ngOnInit() se ejecuta
    ↓
2. loadProgrammers()
   → GET /api/programmers
   → Obtiene lista de programadores
    ↓
3. loadMyAdvisories()
   → GET /api/programmer/advisories/requester (si está logueado)
   → O: GET /api/advisories?email=... (si NO está logueado)
   → Obtiene solicitudes del usuario
    ↓
4. Usuario ve:
   - Contador de solicitudes (Pendientes, Aprobadas, Rechazadas)
   - Lista de programadores
   - Formulario para solicitar asesoría
```

**Cuando usuario solicita asesoría:**

```
1. Usuario selecciona programador
   ↓
2. selectProgrammer(id)
   → GET /api/programmers/{id}/availability
   → Obtiene horarios disponibles
   → generateFutureSchedules() crea slots para próximas 4 semanas
    ↓
3. Usuario completa formulario y hace submit
    ↓
4. submit()
   → Valida campos (programador, nombre, email, horario)
   → POST /api/advisories { programmerProfileId, requesterName, ... }
    ↓
5. Backend crea Advisory en BD
    ↓
6. Frontend resetea formulario y recarga lista
    ↓
✅ Solicitud creada, se muestra en "Mis solicitudes"
```

---

### 3. **Flujo de Programador (PROGRAMMER)**

**Ruta**: `/programmer`

**Archivo**: `icc-portafolio-frontend/src/app/pages/programmer-page/programmer-page.ts`

```
1. ngOnInit()
    ↓
2. loadProfile()
   → GET /api/programmer/me
   → Obtiene perfil actual del programador
    ↓
3. loadProjects()
   → GET /api/programmer/projects
   → Lista de proyectos creados
    ↓
4. loadAvailability()
   → GET /api/programmer/availability
   → Horarios de disponibilidad
    ↓
5. loadAdvisories()
   → GET /api/programmer/advisories
   → Asesorías solicitadas (pendientes, aprobadas, rechazadas)
```

**Acciones del Programador:**

- **Editar perfil**: PUT /api/programmer/profile
- **Crear proyecto**: POST /api/programmer/projects
- **Editar proyecto**: PUT /api/programmer/projects/{id}
- **Eliminar proyecto**: DELETE /api/programmer/projects/{id}
- **Agregar horario**: POST /api/programmer/availability
- **Eliminar horario**: DELETE /api/programmer/availability/{id}
- **Responder asesoría**: PATCH /api/programmer/advisories/{id}
  - Status: APROBADA o RECHAZADA
  - Response: Mensaje al solicitante

---

### 4. **Flujo de Administrador (ADMIN)**

**Ruta**: `/admin`

**Archivo**: `icc-portafolio-frontend/src/app/pages/admin-page/admin-page.ts`

```
1. ngOnInit()
    ↓
2. loadUsers()
   → GET /api/admin/users?page=0&size=50
   → Lista de usuarios (ADMIN, PROGRAMMER, USER)
    ↓
3. Acciones:
   - Crear usuario: POST /api/admin/users
   - Editar usuario: PUT /api/admin/users/{id}
   - Eliminar usuario: DELETE /api/admin/users/{id}
```

**Reportes**:
- GET /api/reports/advisories → Resumen de asesorías
- GET /api/reports/projects → Resumen de proyectos
- GET /api/reports/advisories/pdf → Descargar PDF
- GET /api/reports/projects/excel → Descargar Excel

---

## Guardias y Protección de Rutas

**Archivo**: `icc-portafolio-frontend/src/app/core/guards/`

### 1. `authGuard` (auth.guard.ts)

```typescript
export const authGuard: CanActivateFn = async () => {
  const router = inject(Router);
  const auth = inject(AuthService);

  await auth.waitForAuth();  // Espera a que se cargue sesión

  if (!auth.currentUser()) {
    router.navigate(['/login']);
    return false;
  }
  return true;
};
```

**¿Cuándo se usa?**
- En rutas privadas: `/admin`, `/programmer`, `/user`, `/profile`, `/notifications`
- Valida que el usuario esté logueado
- Si NO está logueado → redirige a `/login`

---

### 2. `roleGuard` (role.guards.ts)

```typescript
export const roleGuard = (allowedRoles: Role[]): CanActivateFn => {
  return async () => {
    const auth = inject(AuthService);
    await auth.waitForAuth();

    const user = auth.currentUser();
    if (!user || !allowedRoles.includes(user.role)) {
      router.navigate(['/user']);
      return false;
    }
    return true;
  };
};
```

**¿Cuándo se usa?**
- En rutas restringidas por rol
- Ejemplo en app.routes.ts:
  ```typescript
  {
    path: 'admin',
    canActivate: [authGuard, roleGuard(['admin'])],
    // Solo usuarios con role='admin' pueden entrar
  }
  ```

---

### 3. `redirectByRoleGuard` (redirect-by-role.guard.ts)

```typescript
export const redirectByRoleGuard: CanActivateFn = async () => {
  const auth = inject(AuthService);
  await auth.waitForAuth();

  const user = auth.currentUser();
  if (!user) return true;  // No logueado, permite ver login

  // Logueado → redirige a su dashboard
  switch (user.role) {
    case 'admin': router.navigate(['/admin']); break;
    case 'programmer': router.navigate(['/programmer']); break;
    default: router.navigate(['/user']);
  }
  return false;
};
```

**¿Cuándo se usa?**
- En `/login` y `/register`
- Si ya estás logueado, redirige a tu dashboard
- Evita que usuarios logueados vean la página de login

---

### Flujo Completo de Guards

```
Usuario accede a /admin
    ↓
authGuard se ejecuta
    - ¿User está logueado? SÍ → continúa
    - ¿User está logueado? NO → redirige a /login
    ↓
roleGuard se ejecuta (permitido: ['admin'])
    - ¿User.role === 'admin'? SÍ → carga /admin
    - ¿User.role === 'admin'? NO → redirige a /user
```

---

## Servicios y Lógica de Negocio

### Backend Services

#### 1. **AuthService** (user/service/AuthService.java)

**Métodos**:
- `login(LoginRequest)` → Autentica y genera JWT
- `register(RegisterRequest)` → Crea usuario con rol USER
- `registerFirstAdmin(RegisterRequest)` → Crea primer admin (solo una vez)
- `loginWithGoogle(idToken)` → Login con Google OAuth

**¿Cuándo se llama?**
- Desde `AuthController` cuando usuario hace login/registro

**Lógica importante**:
```java
public AuthResponse login(LoginRequest request) {
    // 1. Autentica con email/password
    authenticationManager.authenticate(
        new UsernamePasswordAuthenticationToken(request.email(), request.password())
    );
    
    // 2. Obtiene usuario de BD
    var user = userRepository.findByEmail(request.email()).orElseThrow();
    
    // 3. Genera JWT
    String token = jwtService.generateToken(user.getEmail());
    
    // 4. Devuelve token + datos del usuario
    return new AuthResponse(token, user.getName(), user.getRole().name());
}
```

---

#### 2. **ProgrammerService** (programmer/service/ProgrammerService.java)

**Métodos**:
- `getByUserId(userId)` → Obtiene perfil del programador
- `createOrUpdateProfile(userId, request)` → Crea/actualiza perfil
- `listAll(page, size)` → Lista todos los programadores (público)

**Relación con User**:
- Cuando un usuario con rol PROGRAMMER llama a `/api/programmer/me`
- Se busca su ProgrammerProfile asociado

---

#### 3. **AdvisoryService** (programmer/service/AdvisoryService.java)

**Métodos**:
- `create(AdvisoryRequest)` → Crea solicitud de asesoría (público)
- `listByProfile(profileId, page, size)` → Asesorías recibidas (para programador)
- `listByRequester(email, page, size)` → Asesorías solicitadas (para usuario)
- `updateStatus(id, request)` → Aprobar/rechazar (para programador)

**Lógica importante**:
```java
public AdvisoryResponse create(AdvisoryRequest request) {
    // Valida que haya nombre y email
    if (isBlank(request.requesterName())) throw new RuntimeException("Nombre requerido");
    if (isBlank(request.requesterEmail())) throw new RuntimeException("Email requerido");
    
    // Busca el programador
    Advisory advisory = new Advisory();
    advisory.setProfile(profileRepository.findById(request.programmerProfileId()).orElseThrow());
    advisory.setRequesterName(request.requesterName());
    advisory.setRequesterEmail(request.requesterEmail());
    advisory.setScheduledAt(request.scheduledAt());
    advisory.setStatus(AdvisoryStatus.PENDIENTE);  // Inicia como pendiente
    
    // Guarda en BD
    return toResponse(advisoryRepository.save(advisory));
}
```

---

#### 4. **AdminService** (admin/service/AdminService.java)

**Métodos**:
- `createUser(userId, request)` → Crea usuario (solo admin)
- `listUsers(page, size)` → Lista usuarios
- `updateUser(id, request)` → Edita usuario
- `deleteUser(currentUserId, id)` → Elimina usuario

---

### Frontend Services

#### 1. **AuthService** (core/services/auth.service.ts)

```typescript
export class AuthService {
  private currentUserSignal = signal<AuthUser | null>(null);

  login(email: string, pass: string): Observable<AuthUser> {
    return this.http.post<AuthResponse>(`${API_BASE_URL}/api/auth/login`, { email, password: pass })
      .pipe(
        map(res => {
          const user: AuthUser = {
            name: res.name,
            email,
            role: this.mapRole(res.role),
            token: res.token,
          };
          return user;
        }),
        tap(user => this.saveSession(user))  // Guarda en localStorage
      );
  }

  currentUser(): AuthUser | null {
    return this.currentUserSignal();
  }

  getAuthHeaders(): HttpHeaders {
    const token = this.currentUserSignal()?.token;
    return token ? new HttpHeaders({ Authorization: `Bearer ${token}` }) : new HttpHeaders();
  }
}
```

**¿Para qué?**
- Gestiona sesión del usuario
- Guarda/recupera token de localStorage
- Proporciona headers para peticiones autenticadas

---

#### 2. **ProgrammerService** (core/services/programmer.service.ts)

Métodos privados (requieren autenticación):
- `getMyProfile()` → GET /api/programmer/me
- `updateMyProfile(payload)` → PUT /api/programmer/profile
- `createProject(payload)` → POST /api/programmer/projects
- `listProjects()` → GET /api/programmer/projects
- `addAvailability(payload)` → POST /api/programmer/availability
- `listAdvisories()` → GET /api/programmer/advisories
- `updateAdvisoryStatus(id, status, response)` → PATCH /api/programmer/advisories/{id}

Métodos públicos:
- `listProgrammersPublic()` → GET /api/programmers
- `getProgrammerPublic(id)` → GET /api/programmers/{id}
- `listAvailabilityPublic(id)` → GET /api/programmers/{id}/availability
- `createAdvisory(payload)` → POST /api/advisories
- `listAdvisoriesByRequester(email)` → GET /api/programmer/advisories/requester (si logueado) o GET /api/advisories?email=... (si no)

---

#### 3. **UserService** (core/services/user.service.ts)

- `userProfile` → Signal que guarda perfil actual
- `getMyProfile()` → GET /api/users/me
- `updateMyProfile(req)` → PUT /api/users/me
- `listUsers()` → GET /api/admin/users (solo admin)
- `createUser(req)` → POST /api/admin/users (solo admin)
- `updateUser(id, req)` → PUT /api/admin/users/{id} (solo admin)
- `deleteUser(id)` → DELETE /api/admin/users/{id} (solo admin)

---

## Resumen de Flujos Clave

### 1. **Usuario solicita asesoría**

```
1. GET /api/programmers (lista programadores)
2. GET /api/programmers/{id}/availability (horarios disponibles)
3. POST /api/advisories (crea solicitud)
4. GET /api/advisories?email=... (ve sus solicitudes)
5. Programador: PATCH /api/programmer/advisories/{id} (aprueba/rechaza)
6. Usuario: GET /api/programmer/advisories/requester (ve estado actualizado)
```

---

### 2. **Programador agrega proyecto**

```
1. GET /api/programmer/me (carga perfil)
2. POST /api/programmer/projects (crea proyecto con imagen Cloudinary)
3. GET /api/programmer/projects (ve sus proyectos)
4. Usuario público: GET /api/programmers/{id}/projects (ve portafolio)
```

---

### 3. **Admin gestiona usuarios**

```
1. GET /api/admin/users (lista usuarios)
2. POST /api/admin/users (crea nuevo usuario, asigna rol)
3. PUT /api/admin/users/{id} (edita rol)
4. DELETE /api/admin/users/{id} (elimina usuario)
5. GET /api/reports/advisories (ve estadísticas)
6. GET /api/reports/advisories/pdf (descarga reporte)
```

---

## Preguntas Frecuentes Técnicas

### **¿Cómo se persiste la sesión?**
- El token JWT se guarda en `localStorage` como `auth_user`
- Al recargar, `AuthService.constructor` lo recupera
- No hay necesidad de volver a hacer login

### **¿Cómo se validar el rol?**
- **Backend**: `@PreAuthorize("hasRole('ADMIN')")` en el controlador
- **Frontend**: `roleGuard(['admin'])` en la ruta
- Si falla → redirige a `/user` o `/login`

### **¿Qué es un JWT?**
- Token único que contiene el email del usuario
- Se valida en cada petición en `JwtAuthFilter`
- No se puede falsificar (está firmado con una clave secreta)

### **¿Cómo se genera disponibilidad futura?**
- Programador define: "Lunes 14:00-18:00 VIRTUAL"
- Frontend: `generateFutureSchedules()` crea slots para 4 semanas
- Usuario ve: "Lun, 17 feb 14:00", "Lun, 17 feb 14:30", etc. (cada 30 min)

### **¿Qué pasa si rechazas una asesoría?**
- Programador: PATCH /api/programmer/advisories/{id} con `status=RECHAZADA`
- Advisory en BD se actualiza
- Usuario ve: "Rechazada" con respuesta del programador

### **¿Cómo se envían correos?**
- En `NotificationService.java` se usa Spring Mail
- Se envía cuando se crea/actualiza/aprueba asesoría
- Backend integrado con servidor SMTP

---

## Resumen Visual

```
┌─────────────────────────────────────────────────────────────┐
│                    USUARIOS                                 │
├─────────────────────────────────────────────────────────────┤
│  ROLE: USER        │  ROLE: PROGRAMMER  │  ROLE: ADMIN     │
├──────────────────┼────────────────────┼──────────────────┤
│ • Ver programad. │ • Mi perfil         │ • Gestionar usr  │
│ • Solicitar as.  │ • Mis proyectos     │ • Ver reportes   │
│ • Ver mis solic. │ • Disponibilidad    │ • PDF/Excel      │
│ • Editar perfil  │ • Responder solicitu│                  │
└──────────────────┴────────────────────┴──────────────────┘

┌──────────────────────── BD POSTGRESQL ───────────────────────┐
│   users   │  programmer_profile  │  project  │  advisory     │
│  id (PK)  │  id (PK)             │ id (PK)   │ id (PK)       │
│  email    │  user_id (FK)        │ profile.. │ profile_id    │
│  role     │  specialty           │ name      │ requester..   │
│  password │  bio                 │ tech..    │ status        │
└──────────────────────────────────────────────────────────────┘

┌─────────── SEGURIDAD ──────────┐
│  JWT Token                     │
│  • Generado en login           │
│  • Guardado en localStorage    │
│  • Enviado en Authorization    │
│  • Validado en JwtAuthFilter   │
└─────────────────────────────────┘
```

---

## Glosario

| Término | Significado |
|---------|------------|
| **JWT** | JSON Web Token - Token seguro para autenticación |
| **Bearer Token** | Formato: `Authorization: Bearer {token}` |
| **CanActivateFn** | Guard de Angular que protege rutas |
| **Signal** | Sistema reactivo de Angular para cambios de estado |
| **Observable** | Stream de datos que puede emitir múltiples valores |
| **Interceptor** | Middleware que intercepta peticiones HTTP |
| **Record** | Clase inmutable de Java (DTO) |
| **Entity** | Clase anotada con @Entity que mapea a BD |
| **Repository** | Interfaz para operaciones CRUD en BD |
| **DTO** | Data Transfer Object - Objeto para transferencias |
| **Enum** | Conjunto de constantes (ej: ADMIN, PROGRAMMER, USER) |

---

**Última actualización**: 11 de febrero de 2026
**Autores**: Rafael Prieto, Adrián Lazo
