# Informe de Trazabilidad y Cumplimiento SDD — Manager Library
**Ficha ADSO 3413974 · Servicio Nacional de Aprendizaje (SENA)**  
**Candidato / Aprendiz:** Joan Vargas (`joanvargas130204`)  
**Metodología:** *Spec-Driven Development* (SDD)  
**Fecha de entrega:** 10 de octubre de 2026 — 15:00 COP  

---

## 1. Resumen Ejecutivo de la Traducción Tecnológica

La especificación técnica original fue provista en dos variantes de referencia (`spec-python/` y `spec-.net/`). Siguiendo los lineamientos de la prueba técnica y la Constitución del proyecto, la totalidad de los requerimientos de negocio, contratos de API, restricciones de concurrencia e invariantes de dominio fueron traducidos e implementados sobre el stack objetivo:

* **Backend:** PHP 8.2+ con **Laravel 11** (API REST pura bajo `/api/v1`).
* **Frontend:** **React 18** + Vite, Tailwind CSS y Lucide Icons (SPA reactiva y modular).
* **Base de Datos:** **PostgreSQL 16** con esquema relacional `library` (inicialmente vacío, gobernado por migraciones).
* **Infraestructura:** Docker Compose para orquestación de servicios en red aislada.
* **Sitio Público:** HTML5 + CSS3 estático e independiente (sin consumo de la API interna).
* **Herramientas de Soporte:** Script Node.js de población automatizada de datos a través de la API REST pública.

---

## 2. Matriz de Cumplimiento de Tareas (T-01 a T-23) con Evidencia de Código

| Código | Tarea del Spec | Estado | Archivo Fuente / Componente | Evidencia y Validación Técnica |
| :--- | :--- | :---: | :--- | :--- |
| **T-01** | Esqueleto del proyecto y `GET /health` | ✅ Completado | `routes/api.php`, `HealthController.php` | Test `test_health_check_returns_ok`: responde `200 OK` con `{"status":"ok"}`. |
| **T-02** | Migración inicial: 6 tablas, FKs, esquema `library` y categorías | ✅ Completado | `database/migrations/2026_10_10_000001_create_library_schema.php` | Migración crea esquema `library`, tablas y siembra las 5 categorías oficiales sin `DEFAULT` en auditoría. |
| **T-03** | Dominio: ISBN-13, cantidades enteras y validaciones | ✅ Completado | `BookController.php` | Validación estricta `ctype_digit` y 13 dígitos; rechazo con 422 `VALIDATION_ERROR`. |
| **T-04** | Dominio: Modelo `Book` (disponibilidad y baja lógica) | ✅ Completado | `app/Models/Book.php`, `BookController.php` | `SoftDeletes` implementado (`deleted_at`); control de no stock negativo. |
| **T-05** | Dominio: Modelo `Loan` (14 días, estados dinámicos, máx. 3) | ✅ Completado | `app/Models/Loan.php`, `LoanController.php` | `due_at = loaned_at + 14 días`; validación de 1 a 3 ítems por préstamo. |
| **T-06** | Dominio: `Member` y `User` (normalización y roles) | ✅ Completado | `MemberController.php`, `UserController.php` | Nombres normalizados con `trim` y minúsculas; intento de rol `admin` por API rechazado con 422. |
| **T-07** | Autenticación y JWT (8 horas) con contraseñas Bcrypt | ✅ Completado | `AuthController.php`, `JwtService.php` | `POST /auth/login` emite JWT con expiración de 8h; credenciales erróneas devuelven 401 `INVALID_CREDENTIALS`. |
| **T-08** | Catálogo de libros (CRUD, búsqueda, baja lógica) | ✅ Completado | `BookController.php` | Búsqueda por término o categoría; baja lógica no borra historial en préstamos previos. |
| **T-09** | Caso de uso Prestar (`POST /loans`) | ✅ Completado | `LoanController.php` | Test `test_member_cannot_exceed_3_active_books_limit` (422) y `test_member_with_overdue_loan_is_blocked` (409). |
| **T-10** | Caso de uso Devolver y control de duplicidad | ✅ Completado | `LoanController.php` | Test `test_loan_and_return_cycle`: devolución exitosa restituye stock; doble devolución da 409 `ITEM_ALREADY_RETURNED`. |
| **T-11** | Reporte más prestados con título congelado (ADR-004) | ✅ Completado | `ReportController.php` | Test `test_most_borrowed_report_preserves_frozen_title_adr004`: renombrar libro no altera título congelado en `loan_item`. |
| **T-12** | Persistencia PostgreSQL en esquema `library` | ✅ Completado | `app/Models/*` | Modelos configuran `protected $table = 'library.<tabla>'` sobre PostgreSQL real. |
| **T-13** | Concurrencia pesimista/optimista | ✅ Completado | `LoanController.php` | `DB::transaction()` con `lockForUpdate()` en stock de libros para evitar sobre-préstamo concurrente. |
| **T-14** | Seguridad y Middleware JWT en rutas protegidas | ✅ Completado | `app/Http/Middleware/JwtAuthMiddleware.php` | Intercepta token Bearer; rutas sin token devuelven 401 `INVALID_CREDENTIALS`. |
| **T-15** | Manejo de portadas con control MIME y tamaño | ✅ Completado | `BookController::uploadCover` | Valida tipo MIME (`image/jpeg`, `image/png`) y límite de 2 MB; guarda en storage público. |
| **T-16** | Contrato de error estricto uniforme de la API | ✅ Completado | `app/Http/Responses/ApiResponse.php` | Todas las respuestas de error siguen `{ "code": "...", "message": "...", "details": [...] }` según `api-contract.md` §1. |
| **T-17** | Composición y arranque con admin inicial | ✅ Completado | `database/seeders/DatabaseSeeder.php` | Siembra inicial de admin `admin@library.com` con clave encriptada si la tabla `user` está vacía. |
| **T-18** | Invariantes de base de datos y check constraints | ✅ Completado | Migración esquema `library` | Checks `total_copies >= 1`, `available_copies >= 0`, `role IN ('admin','librarian')`. |
| **T-19** | Pruebas de integración automatizadas (PHPUnit) | ✅ Completado | `tests/Feature/ApiTest.php` | Suite de 14 Feature Tests con 47 aserciones aprobadas al 100% (0 errores, 0 fallos). |
| **T-20** | Infraestructura Docker Compose | ✅ Completado | `infra/docker-compose.yml` | Orquestación de servicio PostgreSQL 16 con motor inicialmente vacío según Constitución Art. V. |
| **T-21** | Herramienta de sembrado automatizado (`tool`) | ✅ Completado | `tool/seed.js` | Script en Node.js que consume la API REST para poblar 40 libros, 10 socios, préstamos activos y 3 vencidos. |
| **T-22** | Frontend App React SPA (`app`) | ✅ Completado | `src/App.jsx`, `src/components/*` | SPA modular con Catálogo, Préstamos, Devoluciones, Socios y Reportes. Compilación limpia en Vite. |
| **T-23** | Sitio público estático de presentación (`page`) | ✅ Completado | `page/index.html`, `page/assets/*` | Landing page mobile-first, accesible, con reglas visibles de préstamo y cero llamadas a la API interna. |

---

## 3. Decisiones, Omisiones y Desviaciones Justificadas

En concordancia con los principios de desarrollo guiado por especificación (SDD) y la Constitución del proyecto, se documentan las siguientes decisiones arquitectónicas:

1. **Adopción de Laravel 11 como motor backend:**
   * *Razón:* Provee una arquitectura robusta de controladores y migraciones que facilita el aislamiento del dominio y el manejo de transacciones con bloqueo (`lockForUpdate`), cumpliendo el RNF-2 de prevención de sobre-asignación de recursos concurrentes.
2. **Formato plano de errores HTTP:**
   * *Razón:* La sección 1 de `api-contract.md` establece textualmente la estructura `{ "code": "...", "message": "...", "details": [...] }`. No se envolvió en una clave `"error"` para respetar el contrato literal definido en la especificación técnica.
3. **Parámetro opcional controlado `loaned_at` en `POST /loans`:**
   * *Razón:* Para cumplir el requisito de sembrar préstamos vencidos exclusivamente a través de la API REST (sin tocar SQL directo), se habilitó el parámetro `loaned_at` protegido con validación de fechas (rechazo de fechas futuras y formato ISO estricto). Esto garantiza que el seeder funcione como un cliente HTTP estándar.
4. **Desarrollo en React 18 con JSX y Vite en el Frontend:**
   * *Razón:* Se priorizó velocidad de compilación, estabilidad y modularidad de componentes frente al límite de tiempo de la prueba. Los componentes se estructuraron de manera atómica con validación de estado local y tipado defensivo.
5. **Cero acoplamiento en la Landing Page (`page`):**
   * *Razón:* La especificación estipula que el sitio público no debe depender de la API interna para garantizar que la presentación institucional esté disponible de manera autónoma incluso con los servicios internos fuera de línea.

---

## 4. Evidencias de Ejecución Técnica Real

### 4.1. Suite de Pruebas Automatizadas (PHPUnit Feature Tests)
```text
PHPUnit 11.5.57 by Sebastian Bergmann and contributors.
Runtime:       PHP 8.2.12
Configuration: phpunit.xml

..............                                                    14 / 14 (100%)

Time: 00:05.773, Memory: 42.00 MB

OK (14 tests, 47 assertions)
```

**Escenarios verificados:**
1. `test_health_check_returns_ok` (200 OK con payload `{"status":"ok"}`)
2. `test_login_validation_and_failures` (401 INVALID_CREDENTIALS)
3. `test_login_succeeds_with_valid_credentials` (Generación de token JWT)
4. `test_protected_routes_require_token` (Rechazo sin token con 401)
5. `test_categories_are_seeded` (5 categorías institucionales)
6. `test_create_and_duplicate_isbn_book` (Alta de libro y 409 ISBN_TAKEN)
7. `test_member_registration_and_duplicate_document` (Alta de socio y 409 DOCUMENT_TAKEN)
8. `test_loan_and_return_cycle_and_business_rules` (Disminución de stock, 409 BOOK_NOT_AVAILABLE, 200 devolución, 409 ITEM_ALREADY_RETURNED)
9. `test_admin_cannot_be_granted_via_api` (422 al intentar crear rol admin)
10. `test_member_cannot_exceed_3_active_books_limit` (422 MEMBER_LIMIT_EXCEEDED al intentar superar 3 libros activos)
11. `test_member_with_overdue_loan_is_blocked_from_new_loans` (409 MEMBER_HAS_OVERDUE si hay préstamos vencidos)
12. `test_cannot_delete_book_with_open_loans` (409 BOOK_HAS_OPEN_LOANS si el libro tiene préstamos sin devolver)
13. `test_most_borrowed_report_preserves_frozen_title_adr004` (ADR-004: renombrar libro en catálogo conserva el título histórico en el reporte)
14. `test_loaned_at_validation_rejects_invalid_or_future_dates` (422 ante fechas futuras o formato no ISO)

### 4.2. Compilación de Producción del Frontend (`app`)
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
