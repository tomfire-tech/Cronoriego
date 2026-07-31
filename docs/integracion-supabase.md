# Integración con Supabase

## Proyecto actual

Las páginas apuntan a un proyecto remoto de Supabase. La URL y la clave anónima aparecen embebidas en `login.html`, `admin.html`, `formulario_onboarding.html` y `dashboard.html`.

La clave anónima es publicable por diseño, pero solo es segura cuando las políticas RLS están correctamente configuradas. La clave `service_role` nunca debe aparecer en React, Vite, archivos públicos ni registros del navegador.

## Tablas y vistas observadas

### Identidad y estructura

- `usuarios`
- `v_usuarios`
- `razones_sociales`
- `predios`
- `v_predios`
- `equipos`
- `v_equipos`
- `zonas`
- `v_zonas`
- `ref_tabla_suelo`

### Riego y recomendaciones

- `sc_demanda_hidrica_diaria_zona`
- `sc_demanda_hidrica_semanal_equipo`
- `sc_calculos_estaticos_zona`
- `v_recomendaciones`
- `sc_confirmacion_ejecucion`
- `sc_equipo_operarios`

### NDVI

- `ndvi_registros`
- `ndvi_imagenes`

### Fertirriego y nutrición

- `sc_recomendacion_ferti`
- `sc_balance_nutricional`

### Estanque y mantenimiento

- `sc_dinamica_estanque`
- Las tablas de mantenimiento se construyen dinámicamente en `dashboard.html`; sus nombres y esquemas deben inventariarse desde el código y Supabase antes de implementar.

## Acceso directo permitido desde React

Siempre sujeto a RLS:

- Supabase Auth.
- Perfil y contexto del usuario autenticado.
- Lecturas del dashboard limitadas a la razón social.
- Lectura de catálogos públicos necesarios para formularios.
- Escrituras operativas del productor solo cuando RLS pueda expresar y auditar la regla con claridad, como confirmaciones o mantenimiento de su propia razón social.

## Acceso mediante FastAPI

- Creación de cuentas.
- Cambio de rol, estado o razón social de un usuario.
- Gestión de módulos contratados.
- Onboarding compuesto.
- Procesos globales de descarga y cálculo.
- Operaciones que necesiten `service_role`.
- Auditoría administrativa.

## Variables objetivo

### Frontend

```text
VITE_SUPABASE_URL
VITE_SUPABASE_ANON_KEY
VITE_API_BASE_URL
```

### Backend

```text
SUPABASE_URL
SUPABASE_ANON_KEY
SUPABASE_SERVICE_ROLE_KEY
SUPABASE_JWT_ISSUER
SUPABASE_JWT_AUDIENCE
ALLOWED_ORIGINS
```

Los valores reales no deben versionarse. Se entregarán archivos `.env.example` sin secretos.

## Contrato de identidad

FastAPI recibirá el JWT en `Authorization: Bearer <token>`, validará firma, issuer, audience y expiración, y resolverá el perfil vigente. No aceptará como autoridad un `email`, `rol`, `user_id` o `razon_social_id` enviado en el cuerpo.

## Requisito de exportación

Antes de migrar se debe obtener una exportación versionable y revisable de:

- Migraciones SQL.
- Vistas y funciones.
- RLS y grants.
- Triggers.
- Índices y restricciones.
- Storage.
- Datos de referencia no sensibles.

Sin esa exportación, solo puede documentarse el consumo visible; no puede garantizarse la seguridad ni la integridad del backend actual.
