# Cronoriego-v2 Administration and Onboarding Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Migrar administración y onboarding a React, protegiendo todas las operaciones sensibles con FastAPI.

**Architecture:** Las lecturas administrativas pueden usar repositorios React/Supabase cuando RLS lo autorice. Creación de cuentas, cambios de acceso, módulos y onboarding pasan por FastAPI; el servidor deriva al actor desde JWT y audita cada operación.

**Tech Stack:** Stack del plan de fundaciones, Supabase Admin en backend y PostgreSQL transaccional para onboarding.

## Global Constraints

- Completar primero `2026-07-31-foundation-auth.md`.
- Mantener `agricultor` como rol técnico y “Productor” como término visible.
- No implementar permisos de `gerencia` hasta aprobar su matriz; por defecto solo `super_admin` y `admin` pueden escribir.
- Mantener `riego`, `ferti`, `estanque` y `balance` configurables.
- Tratar `mantenimiento` como comportamiento heredado hasta resolver su política.
- Toda escritura sensible requiere auditoría.

---

### Task 1: Contratos y autorización administrativa

**Files:**
- Create: `backend/app/schemas/admin.py`
- Create: `backend/app/security/authorization.py`
- Create: `backend/tests/security/test_admin_authorization.py`
- Create: `frontend/src/features/admin/admin.types.ts`

**Interfaces:**
- Produces: `CreateUserRequest`, `UpdateAccessRequest`, `UpdateModulesRequest`, `authorize_admin_write(profile)`.

- [ ] Escribir pruebas que permitan escritura a `super_admin`/`admin` y devuelvan `403` a `gerencia`, `agricultor` y `tecnico`.
- [ ] Ejecutar pytest y confirmar el fallo por módulos ausentes.
- [ ] Implementar modelos Pydantic con `EmailStr`, contraseña mínima de 8 caracteres, una mayúscula y un número, y roles literales.
- [ ] Implementar la función pura de autorización y repetir los tipos en TypeScript.
- [ ] Ejecutar Ruff, mypy y pytest; expected: PASS.
- [ ] Commit:

```powershell
git add backend/app/schemas backend/app/security frontend/src/features/admin
git commit -m "feat: define administrative access contracts"
```

### Task 2: Auditoría

**Files:**
- Create: `backend/app/services/audit.py`
- Create: `backend/app/repositories/audit_repository.py`
- Create: `backend/tests/services/test_audit.py`
- Create: `supabase/migrations/0001_audit_log.sql`

**Interfaces:**
- Produces: `AuditEvent` y `AuditService.record(event)`.

- [ ] Crear una prueba que confirme actor, acción, objetivo, razón social, resultado y `request_id`, y que rechace campos `password`, `token` y `service_role`.
- [ ] Crear la migración `audit_log` con RLS de solo inserción desde servidor y lectura restringida.
- [ ] Implementar servicio y repositorio.
- [ ] Ejecutar prueba y aplicar migración al Supabase de pruebas.
- [ ] Commit:

```powershell
git add backend supabase
git commit -m "feat: add administrative audit trail"
```

### Task 3: Crear usuarios desde FastAPI

**Files:**
- Create: `backend/app/api/routes/admin_users.py`
- Create: `backend/app/services/user_service.py`
- Create: `backend/app/repositories/user_repository.py`
- Create: `backend/tests/api/test_admin_users.py`

**Interfaces:**
- Produces: `POST /api/v1/admin/users`.

- [ ] Escribir casos `201`, correo duplicado `409`, actor sin permiso `403`, razón social inexistente `404` y fallo de perfil posterior a Auth con compensación.
- [ ] Confirmar fallos.
- [ ] Implementar `auth.admin.create_user`, perfil `usuarios` y auditoría; eliminar el usuario Auth si falla la vinculación.
- [ ] Verificar que la respuesta nunca incluya contraseña, token ni clave de servidor.
- [ ] Ejecutar pruebas y commit:

```powershell
git add backend
git commit -m "feat: create producer accounts through API"
```

### Task 4: Roles, estado y módulos

**Files:**
- Create: `backend/app/api/routes/admin_access.py`
- Create: `backend/app/services/access_service.py`
- Create: `backend/tests/api/test_admin_access.py`
- Create: `frontend/src/features/admin/api/adminApi.ts`

**Interfaces:**
- Produces: `PATCH /admin/users/{id}` y `PATCH /admin/organizations/{id}/modules`.

- [ ] Probar cambio de rol/estado/asociación y módulos ordenados según `riego,ferti,estanque,balance`.
- [ ] Probar que un módulo desconocido devuelve `422`.
- [ ] Implementar escrituras, auditoría y cliente frontend con Bearer token.
- [ ] Ejecutar pruebas frontend/backend y commit:

```powershell
git add backend frontend/src/features/admin/api
git commit -m "feat: manage roles and subscribed modules"
```

### Task 5: Dashboard administrativo React

**Files:**
- Create: `frontend/src/features/admin/AdminPage.tsx`
- Create: `frontend/src/features/admin/AdminPage.module.scss`
- Create: `frontend/src/features/admin/components/ClientTable.tsx`
- Create: `frontend/src/features/admin/components/UserDialog.tsx`
- Create: `frontend/src/features/admin/components/ModuleSelector.tsx`
- Create: `frontend/src/features/admin/AdminPage.test.tsx`
- Create: `frontend/e2e/admin.spec.ts`

**Interfaces:**
- Consumes: endpoints anteriores y lecturas `razones_sociales`, `v_predios`, `v_zonas`, `ndvi_registros`, `v_recomendaciones`.
- Produces: paridad con `admin.html`.

- [ ] Escribir pruebas de KPIs, tabla, detalle, creación de cuenta, error de API y cambio de módulos.
- [ ] Implementar estados loading/empty/error y componentes accesibles.
- [ ] Implementar enlace de onboarding con `navigator.clipboard`.
- [ ] Mantener acciones de procesos deshabilitadas con texto “Proceso no configurado” hasta recuperar los scripts; no simular ejecución.
- [ ] Ejecutar unitarias y E2E; commit:

```powershell
git add frontend
git commit -m "feat: migrate administrative dashboard"
```

### Task 6: Onboarding transaccional

**Files:**
- Create: `backend/app/schemas/onboarding.py`
- Create: `backend/app/api/routes/onboarding.py`
- Create: `backend/app/services/onboarding_service.py`
- Create: `backend/tests/api/test_onboarding.py`
- Create: `supabase/migrations/0002_create_onboarding.sql`

**Interfaces:**
- Produces: `POST /api/v1/onboarding` con header `Idempotency-Key`.

- [ ] Definir fixtures con razón social, predio, dos equipos, zonas y perfiles de suelo.
- [ ] Probar `201`, duplicado idempotente, validación `422`, conflicto `409` y rollback total.
- [ ] Implementar una función PostgreSQL transaccional que inserte todas las entidades y devuelva sus IDs.
- [ ] Implementar FastAPI como validador/orquestador y registrar auditoría.
- [ ] Ejecutar integración contra Supabase de pruebas y commit:

```powershell
git add backend supabase
git commit -m "feat: add transactional onboarding API"
```

### Task 7: Formulario onboarding React

**Files:**
- Create: `frontend/src/features/onboarding/OnboardingPage.tsx`
- Create: `frontend/src/features/onboarding/onboarding.schema.ts`
- Create: `frontend/src/features/onboarding/OnboardingPage.module.scss`
- Create: `frontend/src/features/onboarding/steps/OrganizationStep.tsx`
- Create: `frontend/src/features/onboarding/steps/FarmStep.tsx`
- Create: `frontend/src/features/onboarding/steps/EquipmentStep.tsx`
- Create: `frontend/src/features/onboarding/steps/ReviewStep.tsx`
- Create: `frontend/src/features/onboarding/OnboardingPage.test.tsx`
- Create: `frontend/e2e/onboarding.spec.ts`

**Interfaces:**
- Consumes: `ref_tabla_suelo`, `POST /onboarding`.
- Produces: paridad con `formulario_onboarding.html`.

- [ ] Escribir pruebas de navegación, múltiples equipos/zonas, validación, resumen y retry idempotente.
- [ ] Implementar estado de formulario por pasos y mapeo de catálogo.
- [ ] Generar una UUID como `Idempotency-Key` al primer envío y reutilizarla en reintentos.
- [ ] Mostrar éxito solo con respuesta completa; conservar datos ante error.
- [ ] Ejecutar todas las verificaciones del plan 1 más E2E.
- [ ] Commit:

```powershell
git add frontend backend
git commit -m "feat: migrate producer onboarding"
```
