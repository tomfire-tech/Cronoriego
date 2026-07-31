# Cronoriego-v2 Dashboard, Parity and Cutover Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Migrar el dashboard del productor por módulos, demostrar paridad funcional y preparar un cambio reversible.

**Architecture:** Cada módulo del dashboard es una feature independiente con repositorio, hook, componentes y pruebas. El contexto compartido resuelve razón social, predio, equipo, zonas y semana; Supabase aplica RLS. La versión vigente se usa como oráculo de comparación.

**Tech Stack:** Stack anterior más Chart.js, Leaflet y SheetJS instalados como dependencias versionadas.

## Global Constraints

- Completar los planes 1 y 2.
- No consultar datos antes de resolver el perfil activo y su razón social.
- No permitir identificadores de otra razón social aunque sean válidos.
- Conservar módulos, cálculos, exportaciones y experiencia móvil.
- No habilitar procesos globales sin recuperar sus scripts.
- Cada feature exige prueba unitaria, integración de datos y E2E de paridad.

---

### Task 1: Contexto productivo y repositorios

**Files:**
- Create: `frontend/src/features/dashboard/context/DashboardContext.tsx`
- Create: `frontend/src/features/dashboard/api/dashboardRepository.ts`
- Create: `frontend/src/features/dashboard/api/dashboard.mappers.ts`
- Create: `frontend/src/features/dashboard/context/DashboardContext.test.tsx`
- Create: `frontend/src/features/dashboard/dashboard.types.ts`

**Interfaces:**
- Produces: `useDashboardContext()` con `organization`, `farms`, `farm`, `equipment`, `zones`, `weekOffset`, setters y estados.

- [ ] Escribir pruebas de cascada razón social → predios → equipos → zonas y cancelación de requests obsoletos.
- [ ] Implementar mapeadores para `v_predios`, `v_equipos`, `v_zonas` y `razones_sociales.modulos`.
- [ ] Implementar TanStack Query con keys que incluyan organización y selección.
- [ ] Probar estados sin predios/equipos/zonas y errores RLS.
- [ ] Commit:

```powershell
git add frontend/src/features/dashboard
git commit -m "feat: add producer dashboard context"
```

### Task 2: Riego y meteorología

**Files:**
- Create: `frontend/src/features/dashboard/irrigation/irrigationRepository.ts`
- Create: `frontend/src/features/dashboard/irrigation/IrrigationView.tsx`
- Create: `frontend/src/features/dashboard/irrigation/IrrigationCalendar.tsx`
- Create: `frontend/src/features/dashboard/irrigation/IrrigationView.test.tsx`

**Interfaces:**
- Consumes: `sc_demanda_hidrica_diaria_zona`, `sc_demanda_hidrica_semanal_equipo`, `sc_calculos_estaticos_zona`.
- Produces: KPIs, calendario semanal y detalle por día/zona.

- [ ] Crear fixtures doradas extraídas y anonimizadas del sistema actual.
- [ ] Probar ISO week, lunes-domingo, horas/minutos, estación/pronóstico y lluvia.
- [ ] Implementar funciones puras para fechas y presentación antes de componentes.
- [ ] Comparar resultados numéricos con el HTML actual usando tolerancia explícita de `0.01`.
- [ ] Commit:

```powershell
git add frontend/src/features/dashboard/irrigation frontend/test-data
git commit -m "feat: migrate irrigation recommendations"
```

### Task 3: NDVI, mapas y exportación de estación

**Files:**
- Create: `frontend/src/features/dashboard/ndvi/NdviMap.tsx`
- Create: `frontend/src/features/dashboard/ndvi/ndviRepository.ts`
- Create: `frontend/src/features/dashboard/exports/stationExport.ts`
- Create: `frontend/src/features/dashboard/ndvi/NdviMap.test.tsx`
- Create: `frontend/src/features/dashboard/exports/stationExport.test.ts`

**Interfaces:**
- Consumes: `ndvi_registros`, `ndvi_imagenes`.
- Produces: capas NDVI/RGB, opacidad, marcador y XLSX.

- [ ] Instalar `leaflet`, `chart.js`, `xlsx` y sus tipos necesarios.
- [ ] Probar bounds, selección de capas, ausencia de imagen y columnas exportadas.
- [ ] Implementar carga diferida de mapa y exportación.
- [ ] Añadir prueba visual Playwright del drawer NDVI.
- [ ] Commit.

### Task 4: Confirmaciones de ejecución

**Files:**
- Create: `frontend/src/features/dashboard/execution/executionRepository.ts`
- Create: `frontend/src/features/dashboard/execution/ExecutionForm.tsx`
- Create: `frontend/src/features/dashboard/execution/ExecutionForm.test.tsx`

**Interfaces:**
- Consumes: `sc_confirmacion_ejecucion`, `sc_equipo_operarios`.
- Produces: defaults y upsert por `id_zona,tipo,fecha_recomendada`.

- [ ] Probar riego/fertirriego, defaults, edición, timestamp y validaciones.
- [ ] Implementar upserts con invalidación de queries.
- [ ] Probar que RLS rechace una zona de otra razón social.
- [ ] Commit.

### Task 5: Fertirriego y balance nutricional

**Files:**
- Create: `frontend/src/features/dashboard/fertigation/FertigationView.tsx`
- Create: `frontend/src/features/dashboard/fertigation/fertigation.calculations.ts`
- Create: `frontend/src/features/dashboard/fertigation/fertigation.calculations.test.ts`
- Create: `frontend/src/features/dashboard/nutrition/NutritionView.tsx`

**Interfaces:**
- Consumes: `sc_recomendacion_ferti`, `sc_balance_nutricional`.
- Produces: eventos, dosis, coberturas macro/micro y gráficos.

- [ ] Portar primero cálculos puros y cubrir límites de cobertura: crítico, ajustado, adecuado, holgado y excedente.
- [ ] Comparar fixtures con resultados actuales.
- [ ] Implementar vistas y detalle de evento.
- [ ] Aplicar bloqueo por módulo `ferti`/`balance` y probarlo.
- [ ] Commit.

### Task 6: Dinámica de estanque

**Files:**
- Create: `frontend/src/features/dashboard/reservoir/ReservoirView.tsx`
- Create: `frontend/src/features/dashboard/reservoir/reservoirRepository.ts`
- Create: `frontend/src/features/dashboard/reservoir/ReservoirView.test.tsx`

**Interfaces:**
- Consumes: `sc_dinamica_estanque`.
- Produces: KPIs, gráfico, resumen, estadísticas y tabla.

- [ ] Probar fila actual, series, valores faltantes y orden semanal.
- [ ] Implementar componentes y gráficos.
- [ ] Comparar una temporada anonimizadamente con la vista actual.
- [ ] Commit.

### Task 7: Mantenimiento y aforos

**Files:**
- Create: `frontend/src/features/dashboard/maintenance/maintenance.types.ts`
- Create: `frontend/src/features/dashboard/maintenance/maintenanceRepository.ts`
- Create: `frontend/src/features/dashboard/maintenance/MaintenanceView.tsx`
- Create: `frontend/src/features/dashboard/maintenance/maintenanceExport.ts`
- Create: `frontend/src/features/dashboard/maintenance/MaintenanceView.test.tsx`

**Interfaces:**
- Consumes: esquemas recuperados para fuente, equipo y zona.
- Produces: crear, listar, detalle y exportar registros.

- [ ] Incorporar nombres reales de tablas como constantes tipadas después de aprobar la puerta del plan.
- [ ] Escribir tests para Pozo/Canal, equipo, zona, delta de presión, historial y XLSX.
- [ ] Implementar formularios con validación y mensajes normalizados.
- [ ] Probar aislamiento por razón social y módulo contratado.
- [ ] Commit.

### Task 8: Shell responsive y accesibilidad

**Files:**
- Create: `frontend/src/features/dashboard/DashboardPage.tsx`
- Create: `frontend/src/features/dashboard/DashboardPage.module.scss`
- Create: `frontend/src/styles/_tokens.scss`
- Create: `frontend/src/styles/_mixins.scss`
- Create: `frontend/e2e/dashboard-desktop.spec.ts`
- Create: `frontend/e2e/dashboard-mobile.spec.ts`

**Interfaces:**
- Consumes: todas las features anteriores.
- Produces: navegación desktop/móvil y bloqueos comerciales.

- [ ] Definir tokens a partir de colores, tipografías y breakpoints actuales.
- [ ] Implementar navegación, selectores y menú de usuario.
- [ ] Probar teclado, foco de modales/drawers, labels y contraste.
- [ ] Ejecutar E2E a 1440×900, 820×1180 y 390×844.
- [ ] Comparar capturas de estados críticos con el sistema actual.
- [ ] Commit.

### Task 9: Sitio público y formulario de contacto

**Files:**
- Create: `frontend/src/features/public-site/PublicHomePage.tsx`
- Create: `frontend/src/features/public-site/PublicHomePage.module.scss`
- Create: `frontend/src/features/public-site/PublicHomePage.test.tsx`

**Interfaces:**
- Produces: paridad de `index.html`.

- [ ] Migrar secciones, calculadora y demo sin cambiar textos aprobados.
- [ ] Corregir correo a `mailto:contacto@cronoriego.com`.
- [ ] Mantener el formulario sin envío hasta aprobar un destino; mostrar un enlace de contacto en lugar de simular éxito.
- [ ] Probar navegación, calculadora y responsive.
- [ ] Commit.

### Task 10: Trabajos administrativos recuperados

**Files:**
- Create: `backend/app/jobs/data_refresh.py`
- Create: `backend/app/jobs/recommendations.py`
- Create: `backend/app/api/routes/admin_jobs.py`
- Create: `backend/tests/jobs/test_jobs.py`

**Interfaces:**
- Consumes: lógica recuperada de `cronoriego_v4.py` y `subir_tablas_csv.py`.
- Produces: iniciar trabajo, consultar estado, exclusión mutua y auditoría.

- [ ] Caracterizar los scripts con fixtures y pruebas antes de mover código.
- [ ] Extraer funciones puras conservando entradas/salidas.
- [ ] Implementar ejecución en background durable; no usar `BackgroundTasks` para trabajos críticos.
- [ ] Probar idempotencia, reintento, concurrencia y error parcial.
- [ ] Habilitar botones administrativos solo después de aprobar integración.
- [ ] Commit.

### Task 11: Seguridad y paridad

**Files:**
- Create: `tests/parity/parity-matrix.md`
- Create: `tests/security/organization-isolation.spec.ts`
- Create: `tests/parity/data-comparison.spec.ts`
- Create: `docs/runbooks/rollback.md`

**Interfaces:**
- Produces: evidencia firmable de paridad y reversión.

- [ ] Completar cada fila de la matriz con caso, fixture, resultado legacy, resultado v2 y evidencia.
- [ ] Probar Productor A contra IDs válidos de Productor B en frontend, Supabase y FastAPI.
- [ ] Analizar bundle y confirmar ausencia de `service_role`.
- [ ] Ejecutar suite global:

```powershell
npm --prefix frontend run lint
npm --prefix frontend run typecheck
npm --prefix frontend run test:run
npm --prefix frontend run build
backend\.venv\Scripts\python -m ruff check backend
backend\.venv\Scripts\python -m mypy backend\app
backend\.venv\Scripts\python -m pytest backend\tests -q
npm --prefix frontend run test:e2e
```

- [ ] Documentar reversión al hosting actual y probarla en staging.
- [ ] Commit:

```powershell
git add tests docs
git commit -m "test: certify v2 functional parity"
```

### Task 12: Cambio controlado

**Files:**
- Modify: configuración de hosting/DNS del proyecto nuevo.
- Create: `docs/runbooks/cutover.md`

**Interfaces:**
- Consumes: aprobación de matriz, seguridad, RLS, backups y rollback.
- Produces: v2 activa y legacy disponible para reversión.

- [ ] Crear backup verificable y registrar punto de restauración.
- [ ] Reducir TTL de DNS con anticipación acordada.
- [ ] Desplegar frontend y backend v2 sin cambiar tráfico.
- [ ] Ejecutar smoke tests con cuentas de cada rol.
- [ ] Cambiar tráfico en una ventana aprobada.
- [ ] Monitorear autenticación, errores, latencia y trabajos.
- [ ] Revertir inmediatamente si falla aislamiento, login, onboarding o integridad.
- [ ] Restaurar TTL normal después del periodo de observación.
