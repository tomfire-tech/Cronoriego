# Planes de implementación de Cronoriego-v2

La migración se divide en tres planes ejecutables. Cada uno debe terminar con software funcional y verificable antes de comenzar el siguiente.

## Orden obligatorio

1. [2026-07-31-foundation-auth.md](2026-07-31-foundation-auth.md)
2. [2026-07-31-admin-onboarding.md](2026-07-31-admin-onboarding.md)
3. [2026-07-31-dashboard-cutover.md](2026-07-31-dashboard-cutover.md)

## Proyecto de destino

Los planes se ejecutan en un repositorio nuevo:

```text
X:\Tomy\Cronoriego-v2
```

El repositorio actual `X:\Tomy\Cronoriego` permanece como referencia ejecutable y documental. No se moverán ni modificarán sus páginas durante la migración.

## Puertas previas

Antes del segundo plan:

- Exportar esquema, vistas, funciones, triggers, índices, grants y RLS de Supabase.
- Definir si `gerencia` puede escribir usuarios y módulos o solo consultar.

Antes del tercer plan:

- Definir el alcance de `tecnico`.
- Confirmar si `mantenimiento` es configurable o está siempre incluido en Estándar.
- Recuperar los nombres y esquemas de las tablas de mantenimiento.
- Recuperar `cronoriego_v4.py` y `subir_tablas_csv.py`.

Estas puertas no cambian los identificadores actuales. El rol técnico del productor seguirá siendo `agricultor`.

## Condición global de término

No se cambia el tráfico al proyecto nuevo hasta completar:

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

Además, debe aprobarse la matriz de paridad y una prueba de aislamiento entre dos razones sociales.
