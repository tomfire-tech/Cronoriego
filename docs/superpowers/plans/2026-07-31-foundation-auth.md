# Cronoriego-v2 Foundation and Authentication Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Crear el proyecto paralelo, su base React/FastAPI y un flujo de autenticación compatible con los roles actuales.

**Architecture:** React usa Supabase Auth y obtiene un perfil normalizado. FastAPI valida el mismo JWT y expone salud y perfil. La autorización se modela en funciones puras y guards testeables; RLS sigue siendo la barrera de datos.

**Tech Stack:** React, TypeScript, Vite, Sass, React Router, TanStack Query, Supabase JS, Vitest, Testing Library, Playwright, Python, FastAPI, Pydantic, PyJWT, pytest, Ruff y mypy.

## Global Constraints

- Crear el repositorio en `X:\Tomy\Cronoriego-v2`.
- No modificar los HTML de `X:\Tomy\Cronoriego`.
- Conservar `super_admin`, `admin`, `gerencia`, `agricultor` y `tecnico`.
- Mostrar “Productor” en la interfaz y conservar `agricultor` en datos.
- No exponer `SUPABASE_SERVICE_ROLE_KEY` al frontend.
- Usar TypeScript estricto y Sass.
- Fijar dependencias mediante `package-lock.json` y `requirements.lock`.
- Aplicar TDD y un commit por tarea.

---

### Task 1: Repositorio y verificación base

**Files:**
- Create: `README.md`
- Create: `.gitignore`
- Create: `frontend/package.json` mediante Vite
- Create: `backend/pyproject.toml`
- Create: `backend/requirements.lock`

**Interfaces:**
- Consumes: documentación de `X:\Tomy\Cronoriego\docs`.
- Produces: comandos reproducibles de frontend y backend.

- [ ] **Step 1: Crear el repositorio y frontend**

```powershell
Set-Location X:\Tomy
npm create vite@latest Cronoriego-v2\frontend -- --template react-ts
Set-Location X:\Tomy\Cronoriego-v2
git init
npm --prefix frontend install
npm --prefix frontend install react-router-dom @tanstack/react-query @supabase/supabase-js sass
npm --prefix frontend install -D vitest jsdom @testing-library/react @testing-library/jest-dom @testing-library/user-event @playwright/test eslint prettier
```

- [ ] **Step 2: Crear entorno Python**

```powershell
New-Item -ItemType Directory -Force backend\app,backend\tests | Out-Null
python -m venv backend\.venv
backend\.venv\Scripts\python -m pip install fastapi "uvicorn[standard]" pydantic-settings pyjwt cryptography httpx pytest pytest-asyncio ruff mypy
backend\.venv\Scripts\python -m pip freeze | Set-Content backend\requirements.lock
```

- [ ] **Step 3: Definir scripts**

Configurar `frontend/package.json`:

```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc -b && vite build",
    "lint": "eslint .",
    "typecheck": "tsc -b --pretty false",
    "test": "vitest",
    "test:run": "vitest run",
    "test:e2e": "playwright test"
  }
}
```

- [ ] **Step 4: Verificar la base**

```powershell
npm --prefix frontend run build
backend\.venv\Scripts\python -m pytest backend\tests -q
```

Expected: build exitoso; pytest informa que no hay pruebas.

- [ ] **Step 5: Commit**

```powershell
git add .
git commit -m "chore: scaffold Cronoriego v2"
```

### Task 2: Configuración tipada por entorno

**Files:**
- Create: `frontend/.env.example`
- Create: `frontend/src/config/env.ts`
- Create: `frontend/src/config/env.test.ts`
- Create: `backend/.env.example`
- Create: `backend/app/config.py`
- Create: `backend/tests/test_config.py`

**Interfaces:**
- Produces: `env.supabaseUrl`, `env.supabaseAnonKey`, `env.apiBaseUrl` y `get_settings() -> Settings`.

- [ ] **Step 1: Escribir pruebas fallidas**

```ts
// frontend/src/config/env.test.ts
import { describe, expect, it } from "vitest";
import { parseEnv } from "./env";

it("rejects missing Supabase configuration", () => {
  expect(() => parseEnv({ VITE_API_BASE_URL: "http://localhost:8000" })).toThrow(
    "VITE_SUPABASE_URL",
  );
});
```

```python
# backend/tests/test_config.py
import pytest
from pydantic import ValidationError
from app.config import Settings

def test_service_role_is_required() -> None:
    with pytest.raises(ValidationError):
        Settings(supabase_url="https://example.supabase.co")
```

- [ ] **Step 2: Confirmar fallos**

```powershell
npm --prefix frontend run test:run -- src/config/env.test.ts
$env:PYTHONPATH="backend"; backend\.venv\Scripts\python -m pytest backend\tests\test_config.py -q
```

Expected: ambos fallan porque los módulos aún no existen.

- [ ] **Step 3: Implementar configuración mínima**

```ts
// frontend/src/config/env.ts
export type ClientEnv = {
  supabaseUrl: string;
  supabaseAnonKey: string;
  apiBaseUrl: string;
};

export function parseEnv(source: Record<string, unknown>): ClientEnv {
  const required = ["VITE_SUPABASE_URL", "VITE_SUPABASE_ANON_KEY", "VITE_API_BASE_URL"] as const;
  for (const key of required) {
    if (typeof source[key] !== "string" || source[key] === "") throw new Error(`Missing ${key}`);
  }
  return {
    supabaseUrl: source.VITE_SUPABASE_URL as string,
    supabaseAnonKey: source.VITE_SUPABASE_ANON_KEY as string,
    apiBaseUrl: source.VITE_API_BASE_URL as string,
  };
}

export const env = parseEnv(import.meta.env);
```

```python
# backend/app/config.py
from functools import lru_cache
from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings):
    supabase_url: str
    supabase_anon_key: str
    supabase_service_role_key: str
    supabase_jwt_issuer: str
    supabase_jwt_audience: str = "authenticated"
    allowed_origins: list[str] = ["http://localhost:5173"]
    model_config = SettingsConfigDict(env_file=".env", extra="ignore")

@lru_cache
def get_settings() -> Settings:
    return Settings()
```

- [ ] **Step 4: Crear ejemplos sin secretos y ejecutar pruebas**

```text
# frontend/.env.example
VITE_SUPABASE_URL=https://project.supabase.co
VITE_SUPABASE_ANON_KEY=replace-with-anon-key
VITE_API_BASE_URL=http://localhost:8000
```

```text
# backend/.env.example
SUPABASE_URL=https://project.supabase.co
SUPABASE_ANON_KEY=replace-with-anon-key
SUPABASE_SERVICE_ROLE_KEY=replace-with-server-secret
SUPABASE_JWT_ISSUER=https://project.supabase.co/auth/v1
SUPABASE_JWT_AUDIENCE=authenticated
ALLOWED_ORIGINS=["http://localhost:5173"]
```

Run both tests from Step 2. Expected: PASS.

- [ ] **Step 5: Commit**

```powershell
git add frontend/.env.example frontend/src/config backend/.env.example backend/app/config.py backend/tests/test_config.py
git commit -m "chore: add typed environment configuration"
```

### Task 3: FastAPI, errores y salud

**Files:**
- Create: `backend/app/main.py`
- Create: `backend/app/api/errors.py`
- Create: `backend/app/api/routes/health.py`
- Create: `backend/tests/test_health.py`

**Interfaces:**
- Produces: `GET /api/v1/health` y `ApiError(code, message, status_code, details)`.

- [ ] **Step 1: Escribir prueba fallida**

```python
from fastapi.testclient import TestClient
from app.main import app

def test_health() -> None:
    response = TestClient(app).get("/api/v1/health")
    assert response.status_code == 200
    assert response.json() == {"status": "ok", "service": "cronoriego-api"}
    assert response.headers["x-request-id"]
```

- [ ] **Step 2: Confirmar fallo**

```powershell
$env:PYTHONPATH="backend"; backend\.venv\Scripts\python -m pytest backend\tests\test_health.py -q
```

- [ ] **Step 3: Implementar aplicación mínima**

```python
# backend/app/main.py
from uuid import uuid4
from fastapi import FastAPI, Request
from fastapi.middleware.cors import CORSMiddleware
from app.api.routes.health import router as health_router

app = FastAPI(title="CronoRiego API", version="1.0.0")
app.add_middleware(CORSMiddleware, allow_origins=["http://localhost:5173"], allow_credentials=True, allow_methods=["*"], allow_headers=["*"])

@app.middleware("http")
async def request_id(request: Request, call_next):
    response = await call_next(request)
    response.headers["X-Request-ID"] = request.headers.get("X-Request-ID", str(uuid4()))
    return response

app.include_router(health_router, prefix="/api/v1")
```

```python
# backend/app/api/routes/health.py
from fastapi import APIRouter
router = APIRouter()

@router.get("/health")
def health() -> dict[str, str]:
    return {"status": "ok", "service": "cronoriego-api"}
```

- [ ] **Step 4: Ejecutar prueba**

Run Step 2. Expected: PASS.

- [ ] **Step 5: Commit**

```powershell
git add backend/app backend/tests/test_health.py
git commit -m "feat: add API health endpoint"
```

### Task 4: Modelo de roles y rutas

**Files:**
- Create: `frontend/src/features/auth/auth.types.ts`
- Create: `frontend/src/features/auth/routePolicy.ts`
- Create: `frontend/src/features/auth/routePolicy.test.ts`
- Create: `backend/app/security/roles.py`
- Create: `backend/tests/security/test_roles.py`

**Interfaces:**
- Produces: `AppRole`, `UserProfile`, `homeForRole(role)` y `require_roles(*roles)`.

- [ ] **Step 1: Escribir pruebas de paridad**

```ts
import { expect, it } from "vitest";
import { homeForRole } from "./routePolicy";

it.each([
  ["super_admin", "/admin"],
  ["admin", "/admin"],
  ["gerencia", "/dashboard"],
  ["agricultor", "/dashboard"],
  ["tecnico", "/dashboard"],
] as const)("routes %s to %s", (role, path) => expect(homeForRole(role)).toBe(path));
```

```python
from app.security.roles import can_open_admin

def test_current_admin_page_roles() -> None:
    assert can_open_admin("super_admin")
    assert can_open_admin("admin")
    assert can_open_admin("gerencia")
    assert not can_open_admin("agricultor")
```

- [ ] **Step 2: Confirmar fallos**

Run the targeted Vitest and pytest files. Expected: missing modules.

- [ ] **Step 3: Implementar política actual explícita**

```ts
export type AppRole = "super_admin" | "admin" | "gerencia" | "agricultor" | "tecnico";
export type UserProfile = {
  userId: string;
  email: string;
  role: AppRole;
  organizationId: string | null;
  active: boolean;
  modules: string[];
};
export const homeForRole = (role: AppRole) =>
  role === "super_admin" || role === "admin" ? "/admin" : "/dashboard";
```

```python
from typing import Literal
AppRole = Literal["super_admin", "admin", "gerencia", "agricultor", "tecnico"]
ADMIN_PAGE_ROLES: frozenset[str] = frozenset({"super_admin", "admin", "gerencia"})

def can_open_admin(role: str) -> bool:
    return role in ADMIN_PAGE_ROLES
```

- [ ] **Step 4: Ejecutar pruebas y documentar la excepción**

Expected: PASS. Añadir al README que la ruta automática de `gerencia` conserva el comportamiento de `login.html`, aunque la página admin acepta acceso directo; cambiarlo requiere aprobación funcional.

- [ ] **Step 5: Commit**

```powershell
git add frontend/src/features/auth backend/app/security backend/tests/security README.md
git commit -m "feat: codify current role routing"
```

### Task 5: Supabase, sesión y perfil

**Files:**
- Create: `frontend/src/services/supabase.ts`
- Create: `frontend/src/features/auth/profileRepository.ts`
- Create: `frontend/src/features/auth/AuthProvider.tsx`
- Create: `frontend/src/features/auth/ProtectedRoute.tsx`
- Create: `frontend/src/features/auth/AuthProvider.test.tsx`
- Create: `backend/app/security/jwt.py`
- Create: `backend/app/api/routes/me.py`
- Create: `backend/tests/api/test_me.py`

**Interfaces:**
- Produces: `useAuth()`, `<ProtectedRoute allow={...}>`, `GET /api/v1/me`.

- [ ] **Step 1: Escribir pruebas**

Frontend: verificar `loading`, perfil activo, perfil ausente y cierre de sesión. Backend: JWT inválido devuelve `401`; rol no autorizado devuelve `403`; perfil válido devuelve el esquema `UserProfile`.

```python
def test_me_rejects_missing_token(client) -> None:
    response = client.get("/api/v1/me")
    assert response.status_code == 401
    assert response.json()["code"] == "AUTH_TOKEN_MISSING"
```

- [ ] **Step 2: Ejecutar pruebas y confirmar fallos**

```powershell
npm --prefix frontend run test:run -- src/features/auth/AuthProvider.test.tsx
$env:PYTHONPATH="backend"; backend\.venv\Scripts\python -m pytest backend\tests\api\test_me.py -q
```

- [ ] **Step 3: Implementar cliente y repositorio**

```ts
import { createClient } from "@supabase/supabase-js";
import { env } from "../config/env";
export const supabase = createClient(env.supabaseUrl, env.supabaseAnonKey);
```

`profileRepository.ts` debe consultar `v_usuarios` por el correo de la sesión y mapear columnas heredadas a `UserProfile`. Si no existe perfil o `active` es falso, debe devolver un error de dominio y cerrar la sesión.

- [ ] **Step 4: Implementar validación backend**

`jwt.py` debe validar algoritmo, firma, issuer, audience y expiración. `GET /me` debe resolver el perfil vigente en Supabase; no debe confiar en rol o razón social contenidos en el request.

- [ ] **Step 5: Ejecutar pruebas**

Run Step 2. Expected: PASS.

- [ ] **Step 6: Commit**

```powershell
git add frontend/src/services frontend/src/features/auth backend/app/security backend/app/api/routes/me.py backend/tests/api
git commit -m "feat: add Supabase session and profile resolution"
```

### Task 6: Login, rutas y shell visual

**Files:**
- Create: `frontend/src/app/router.tsx`
- Create: `frontend/src/app/AppProviders.tsx`
- Create: `frontend/src/features/auth/LoginPage.tsx`
- Create: `frontend/src/features/auth/LoginPage.module.scss`
- Create: `frontend/src/features/auth/LoginPage.test.tsx`
- Create: `frontend/src/features/admin/AdminPlaceholder.tsx`
- Create: `frontend/src/features/dashboard/DashboardPlaceholder.tsx`
- Create: `frontend/e2e/auth.spec.ts`

**Interfaces:**
- Consumes: `useAuth`, `homeForRole`.
- Produces: rutas `/login`, `/admin`, `/dashboard`.

- [ ] **Step 1: Escribir prueba de login**

```ts
it("redirects an agricultor to the dashboard", async () => {
  // mock Supabase signInWithPassword and v_usuarios profile
  // render LoginPage with router
  // submit valid credentials
  expect(screen.getByText("Panel del productor")).toBeInTheDocument();
});
```

- [ ] **Step 2: Confirmar fallo**

```powershell
npm --prefix frontend run test:run -- src/features/auth/LoginPage.test.tsx
```

- [ ] **Step 3: Implementar login y rutas**

Usar labels accesibles, `autocomplete="email"` y `autocomplete="current-password"`. Mostrar errores para credenciales, perfil ausente, inactivo y red. No usar fallback silencioso a `tecnico`.

- [ ] **Step 4: Añadir E2E con Supabase simulado**

```ts
test("authenticated agricultor lands on dashboard", async ({ page }) => {
  await page.goto("/login");
  await page.getByLabel("Correo").fill("productor@example.test");
  await page.getByLabel("Contraseña").fill("Password1!");
  await page.getByRole("button", { name: "Ingresar" }).click();
  await expect(page).toHaveURL(/\/dashboard$/);
});
```

- [ ] **Step 5: Verificar**

```powershell
npm --prefix frontend run lint
npm --prefix frontend run typecheck
npm --prefix frontend run test:run
npm --prefix frontend run build
backend\.venv\Scripts\python -m ruff check backend
backend\.venv\Scripts\python -m mypy backend\app
backend\.venv\Scripts\python -m pytest backend\tests -q
```

- [ ] **Step 6: Commit**

```powershell
git add frontend backend README.md
git commit -m "feat: complete authentication foundation"
```
