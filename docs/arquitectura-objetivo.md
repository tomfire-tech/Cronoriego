# Arquitectura objetivo de Cronoriego-v2

## Decisión

`Cronoriego-v2` será un proyecto paralelo en `X:\Tomy\Cronoriego-v2`. El sistema actual permanecerá sin modificaciones funcionales hasta completar la validación de paridad.

## Componentes

```text
Cronoriego-v2/
├── frontend/
│   ├── src/
│   │   ├── app/
│   │   ├── features/
│   │   ├── components/
│   │   ├── services/
│   │   ├── styles/
│   │   └── types/
│   └── vite.config.*
├── backend/
│   └── app/
│       ├── api/
│       ├── services/
│       ├── repositories/
│       ├── schemas/
│       ├── security/
│       └── main.py
├── tests/
└── README.md
```

## Frontend

- React con TypeScript.
- Vite para desarrollo y compilación.
- Sass con variables, mixins, componentes y estilos por feature.
- React Router para rutas públicas y protegidas.
- Cliente oficial de Supabase para Auth y acceso directo permitido.
- Capa separada para la API FastAPI.
- Gestión de consultas con una biblioteca de caché y estados asíncronos.
- Componentes de gráficos, mapas y exportación encapsulados.
- Diseño responsivo conservando escritorio y móvil.

### Features propuestas

```text
features/
├── public-site/
├── auth/
├── admin/
├── onboarding/
└── dashboard/
    ├── irrigation/
    ├── fertigation/
    ├── reservoir/
    ├── nutrition/
    ├── maintenance/
    └── ndvi/
```

## Backend

- Python con FastAPI.
- Pydantic para contratos y validación.
- Cliente Supabase de servidor.
- Validación de JWT de Supabase.
- API versionada bajo `/api/v1`.
- Servicios independientes de HTTP.
- Repositorios que encapsulan nombres heredados de tablas y columnas.
- Identificadores de solicitud, logs estructurados y auditoría.

### Endpoints iniciales

```text
GET    /api/v1/health
GET    /api/v1/me
POST   /api/v1/admin/users
PATCH  /api/v1/admin/users/{user_id}
PATCH  /api/v1/admin/organizations/{id}/modules
POST   /api/v1/onboarding
POST   /api/v1/admin/jobs/data-refresh
POST   /api/v1/admin/jobs/recommendations
GET    /api/v1/admin/jobs/{job_id}
```

Los endpoints de trabajos solo se implementarán cuando se recuperen y especifiquen los scripts actuales.

## Flujo híbrido

```text
Lecturas autorizadas
React → Supabase Auth → JWT
React → Supabase/PostgreSQL con RLS → datos de la razón social

Operaciones sensibles
React → FastAPI + JWT
FastAPI → valida JWT y perfil vigente
FastAPI → Supabase con privilegio de servidor
FastAPI → respuesta normalizada + auditoría
```

## Contrato de errores

```json
{
  "code": "ONBOARDING_VALIDATION_ERROR",
  "message": "No fue posible registrar el productor.",
  "details": {},
  "request_id": "identificador-correlacionable"
}
```

| Estado | Uso |
| --- | --- |
| `400` | Solicitud mal formada |
| `401` | JWT ausente, inválido o vencido |
| `403` | Rol o alcance insuficiente |
| `404` | Recurso inexistente dentro del alcance |
| `409` | Duplicado o conflicto de estado |
| `422` | Validación de datos |
| `500` | Error interno no atribuible al usuario |

## Onboarding transaccional

La operación debe ser atómica e idempotente. La opción preferida es una función PostgreSQL transaccional invocada por FastAPI. Si esa opción no es viable, el servicio deberá implementar transacción mediante una conexión PostgreSQL segura. No se recomienda conservar inserciones independientes con compensación como mecanismo principal.

## Principios de diseño

- Paridad antes que rediseño funcional.
- Seguridad en servidor y RLS, no solo en la interfaz.
- Features pequeñas con contratos claros.
- Configuración por entorno.
- Nombres heredados encapsulados en repositorios.
- Pruebas automatizadas como condición de migración.
