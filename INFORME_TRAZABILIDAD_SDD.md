# Informe de Trazabilidad y Cumplimiento SDD — Manager Library
**Ficha ADSO 3413974 · Servicio Nacional de Aprendizaje (SENA)**  
**Candidato / Aprendiz:** Joan Vargas (`joanvargas130204`)  
**Metodología:** *Spec-Driven Development* (SDD)  
**Fecha de entrega:** 10 de octubre de 2026 — 15:00 COP  

---

## 1. Resumen Ejecutivo de la Traducción Tecnológica

La especificación técnica original fue provista en dos variantes de referencia (`spec-python/` y `spec-.net/`). Siguiendo los lineamientos de la prueba técnica y la Constitución del proyecto, la totalidad de los requerimientos de negocio, contratos de API, restricciones de concurrencia e invariantes de dominio fueron traducidos e implementados sobre el stack objetivo:

* **Backend:** PHP 8.2+ con **Laravel 11** (API REST pura bajo `/api/v1`).
* **Frontend:** **React 18** + Vite, Tailwind CSS y Lucide Icons (SPA reactiva y tipada).
* **Base de Datos:** **PostgreSQL 16** con esquema relacional `library` (inicialmente vacío, administrado por migraciones).
* **Infraestructura:** Docker Compose para orquestación de contenedores en red aislada.
* **Sitio Público:** HTML5 + CSS3 estático e independiente (sin consumo de la API interna).
* **Herramientas de Soporte:** Script Node.js de población automatizada de datos a través de la API pública.

---

## 2. Matriz de Cumplimiento de Tareas (T-01 a T-23)

| Código | Tarea | Estado | Evidencia y Verificación |
| :--- | :--- | :---: | :--- |
| **T-01** | Esqueleto del proyecto y `GET /health` | ✅ Completado | `GET /api/v1/health` responde `{"status":"ok"}` con código HTTP 200. |
| **T-02** | Migración inicial: 6 tablas, FKs, esquema `library` y categorías sembradas | ✅ Completado | Migración Laravel crea esquema `library`, tablas `categories`, `books`, `members`, `users`, `loans`, `loan_items`. 5 categorías base sembradas sin `DEFAULT` en columnas de auditoría. |
| **T-03** | Dominio: Objetos de valor (ISBN-13, cantidades, validación estricta) | ✅ Completado | FormRequest y validadores de dominio aseguran formato de 13 dígitos y cantidades enteras positivas. |
| **T-04** | Dominio: Modelo `Book` (disponibilidad y baja lógica) | ✅ Completado | Campo `deleted_at` (`SoftDeletes`), `available_copies` decrementable e incrementable sin permitir stock negativo. |
| **T-05** | Dominio: Modelo `Loan` (plazo de 14 días, cálculo dinámico de estado) | ✅ Completado | Cálculo automático de `due_at = loaned_at + 14 días`. Estado dinámico `open`, `overdue`, `closed`. Límite de 1 a 3 ítems por préstamo. |
| **T-06** | Dominio: `Member` y `User` (normalización de cadenas y roles) | ✅ Completado | `mb_strtolower(trim($name))` aplicado. Creación de rol `admin` vía API rechazada con 422 (`ADMIN_ROLE_NOT_ALLOWED`). |
| **T-07** | Autenticación y control de acceso (JWT) | ✅ Completado | `POST /auth/login` emite JWT con expiración de 8 horas. Contraseñas hasheadas con Bcrypt. Claves nunca expuestas en respuestas ni logs. |
| **T-08** | Catálogo de libros (CRUD, búsqueda, baja lógica) | ✅ Completado | Endpoints de libros operativos. Baja lógica preserva integridad histórica de préstamos anteriores. |
| **T-09** | Caso de uso Prestar (`POST /loans`) | ✅ Completado | Valida límite de 3 libros por socio, bloquea si el socio tiene préstamos vencidos (409 `MEMBER_HAS_OVERDUE`), y descuenta stock atómicamente. |
| **T-10** | Caso de uso Devolver (`PATCH /loans/{id}/items/{itemId}`) | ✅ Completado | Registra `returned_at`, restituye `available_copies`. Rechaza devoluciones dobles con 409 `ITEM_ALREADY_RETURNED`. |
| **T-11** | Reporte de libros más prestados (`GET /reports/most-borrowed`) | ✅ Completado | Agregación por `book_id`, preserva el título congelado en `loan_items` según ADR-004. |
| **T-12** | Persistencia en PostgreSQL | ✅ Completado | Modelos Eloquent mapeados al esquema `library` con claves foráneas e integridad referencial estricta. |
| **T-13** | Concurrencia pesimista/optimista | ✅ Completado | `DB::transaction()` con `lockForUpdate()` en selección de copias para evitar sobre-préstamo concurrente. |
| **T-14** | Seguridad y JWT Middleware | ✅ Completado | Middleware intercepta `Authorization: Bearer <token>`, valida firma y expone claims en el contexto del request. |
| **T-15** | Manejo de portadas | ✅ Completado | Carga y almacenamiento seguro de imágenes con validación de peso y tipo MIME. |
| **T-16** | Contrato de errores uniforme de API | ✅ Completado | Todas las respuestas de error siguen estrictamente `{ "error": { "code": "...", "message": "...", "details": [...] } }`. |
| **T-17** | Arranque del sistema y administrador inicial | ✅ Completado | `DatabaseSeeder` crea el usuario administrador predeterminado si no existe (`admin@library.com` / `admin123456`). |
| **T-18** | Invariantes de base de datos | ✅ Completado | Check constraints a nivel de motor y reglas de validación en controladores. |
| **T-19** | Pruebas de integración automatizadas | ✅ Completado | Suite de Feature Tests en PHPUnit con 9 escenarios y 29 aserciones superadas con éxito (0 fallos). |
| **T-20** | Infraestructura Docker Compose | ✅ Completado | `docker-compose.yml` orquesta servicio PostgreSQL 16 con volumen persistente y base de datos `library`. |
| **T-21** | Herramienta de sembrado automatizado (`tool`) | ✅ Completado | Script `seed.js` puebla 40 libros, 10 socios, 15 préstamos vigentes y 3 préstamos vencidos consumiendo la API REST. |
| **T-22** | Aplicación de gestión frontend (`app`) | ✅ Completado | SPA en React con login, catálogo, nuevo préstamo, devolución rápida, monitor de vencidos y reportes. Build limpio de producción. |
| **T-23** | Página de presentación pública (`page`) | ✅ Completado | Landing page moderna, responsiva, sin dependencias externas y sin llamadas a la API privada. |

---

## 3. Matriz de Criterios de Aceptación (Historias de Usuario)

* **HU-01 / HU-02 (Autenticación y Roles):** Solo bibliotecarios y administradores pueden emitir préstamos. No se permite elevar permisos a admin mediante endpoints estándar.
* **HU-03 / HU-04 (Gestión de Libros):** Control de ISBN-13 único, baja lógica sin pérdida de registros en préstamos históricos.
* **HU-05 (Gestión de Socios):** Documento de identidad único, nombres normalizados.
* **HU-06 (Préstamos):**
  * Máximo 3 libros activos por socio.
  * Prohibición de préstamo si existen ítems vencidos pendientes.
  * Restricción de duplicidad del mismo libro en un mismo préstamo.
  * Decremento automático e indivisible de stock disponible.
* **HU-07 (Devoluciones):**
  * Restitución del stock en tiempo real.
  * Idempotencia/control de error ante doble devolución (409).
* **HU-09 (Vencidos):**
  * Listado filtrado donde `returned_at IS NULL` y `due_at < NOW()`.
* **HU-10 (Reportes):**
  * Ranking de popularidad de libros con título histórico persistido.

---

## 4. Evidencias de Ejecución Técnica

### 4.1. Pruebas Unitarias y de Integración (Backend)
```text
   PASS  Tests\Feature\ApiTest
  ✓ health check returns ok
  ✓ login fails with wrong credentials
  ✓ login succeeds with valid credentials
  ✓ protected route denies unauthenticated access
  ✓ categories are seeded
  ✓ book lifecycle and duplicate isbn validation
  ✓ member registration and duplicate document validation
  ✓ loan lifecycle stock decrement double return rejection
  ✓ admin creation via api is rejected

  Tests:    9 passed (29 assertions)
  Duration: 0.82s
```

### 4.2. Compilación del Frontend (`app`)
```text
  vite v5.4.14 building for production...
  ✓ 1969 modules transformed.
  dist/index.html                   0.85 kB │ gzip:  0.44 kB
  dist/assets/index-D8Y2w8f7.css   15.42 kB │ gzip:  3.89 kB
  dist/assets/index-B7qK1nOp.js   248.16 kB │ gzip: 78.91 kB
  ✓ built in 3.56s
```

---

## 5. Repositorios Entregados

| Repositorio | Enlace Oficial Fork |
| :--- | :--- |
| **Documentación y SDD** | [test-manager-library-3413974-docs](https://github.com/joanvargas130204/test-manager-library-3413974-docs) |
| **Infraestructura** | [test-manager-library-3413974-infra](https://github.com/joanvargas130204/test-manager-library-3413974-infra) |
| **Backend API (Laravel)** | [test-manager-library-3413974-api](https://github.com/joanvargas130204/test-manager-library-3413974-api) |
| **Frontend App (React)** | [test-manager-library-3413974-app](https://github.com/joanvargas130204/test-manager-library-3413974-app) |
| **Landing Page Pública** | [test-manager-library-3413974-page](https://github.com/joanvargas130204/test-manager-library-3413974-page) |
| **Herramienta Seeder** | [test-manager-library-3413974-tool](https://github.com/joanvargas130204/test-manager-library-3413974-tool) |
