# Documentación de CronoRiego

## Propósito

Esta carpeta describe el sistema vigente y define la futura migración a un proyecto paralelo llamado `Cronoriego-v2`. El proyecto actual debe permanecer operativo y sin cambios funcionales mientras se construye y valida la nueva versión.

## Estado de las decisiones

- Aprobado: mantener los identificadores de roles actuales, incluido `agricultor`.
- Aprobado: mostrar “Productor” como término de negocio cuando corresponda.
- Aprobado: arquitectura híbrida con React y Supabase en el navegador, más FastAPI para operaciones sensibles.
- Aprobado: construir la nueva versión en `X:\Tomy\Cronoriego-v2`.
- Aprobado: no reemplazar el proyecto vigente hasta alcanzar paridad funcional y validar seguridad, datos e interfaz.

## Índice

| Documento | Contenido |
| --- | --- |
| [estado-actual.md](estado-actual.md) | Inventario técnico y deuda del sistema vigente |
| [roles-y-permisos.md](roles-y-permisos.md) | Roles, alcance y reglas de autorización |
| [especificaciones-funcionales.md](especificaciones-funcionales.md) | Comportamiento requerido por página y módulo |
| [integracion-supabase.md](integracion-supabase.md) | Fuentes de datos y distribución de responsabilidades |
| [arquitectura-objetivo.md](arquitectura-objetivo.md) | Diseño de React, FastAPI y Supabase |
| [estrategia-migracion.md](estrategia-migracion.md) | Fases, paridad y criterios para el cambio |
| [seguridad-y-riesgos.md](seguridad-y-riesgos.md) | Riesgos observados y controles requeridos |
| [recomendaciones.md](recomendaciones.md) | Mejoras priorizadas sin alterar el alcance funcional |
| [superpowers/plans/README.md](superpowers/plans/README.md) | Planes ejecutables para construir `Cronoriego-v2` |

## Convenciones

- **Actual**: comportamiento comprobado en los HTML del repositorio.
- **Objetivo**: decisión aprobada para `Cronoriego-v2`.
- **Requiere verificación**: depende de configuración no presente en el repositorio, como políticas RLS, funciones, triggers o esquemas completos de Supabase.

## Fuera de alcance en esta etapa

Esta etapa no crea `Cronoriego-v2`, no cambia Supabase y no modifica las páginas actuales. La implementación se planificará después de revisar y aprobar estos documentos.
