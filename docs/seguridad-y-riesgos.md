# Seguridad y riesgos

## Riesgos críticos

### 1. Operaciones privilegiadas desde el navegador

Creación de cuentas, roles, módulos y onboarding no deberían depender de privilegios del cliente. Se trasladarán a FastAPI con verificación de JWT y autorización.

### 2. RLS no versionada

El repositorio no permite confirmar qué filas puede leer o modificar cada rol. Debe auditarse y versionarse antes de declarar segura la v2.

### 3. Onboarding no atómico

Las inserciones secuenciales pueden dejar información parcial. La v2 exigirá una operación transaccional e idempotente.

### 4. Separación por razón social

Es el límite principal de aislamiento. Deben existir pruebas negativas que intenten acceder a identificadores válidos de otra razón social.

## Riesgos altos

- Creación de usuario con `auth.signUp` desde administración.
- Fallback silencioso a `tecnico` cuando no se encuentra rol.
- Diferencia entre roles admitidos por login y administración.
- Acciones administrativas simuladas que no ejecutan procesos.
- Ausencia de auditoría.
- Dependencias CDN sin bloqueo completo de versiones.
- Lógica extensa y acoplada en `dashboard.html`.
- Falta de validación de esquemas entre cliente y datos.

## Controles obligatorios

- HTTPS en ambientes no locales.
- JWT validado en FastAPI.
- RLS con pruebas automatizadas.
- `service_role` solo en backend.
- CORS restringido.
- Rate limiting para login indirecto, onboarding y endpoints administrativos.
- Límites de tamaño y validación estricta.
- Idempotency key para onboarding y trabajos.
- Auditoría de operaciones sensibles.
- Logs sin contraseñas, JWT, claves ni datos personales innecesarios.
- Rotación y almacenamiento seguro de secretos.
- Dependencias fijadas y análisis de vulnerabilidades.
- Encabezados de seguridad y CSP compatible con mapas y recursos.
- Backups y restauración probada.

## Contraseñas

- FastAPI puede crear una contraseña temporal o preferir invitación por correo.
- Nunca se registrará ni almacenará la contraseña en logs o tablas de negocio.
- La interfaz actual muestra y permite copiar la contraseña generada; la v2 debe reducir su exposición y exigir cambio inicial cuando la política de Auth lo permita.

## Auditoría mínima

```text
request_id
actor_user_id
actor_role
action
target_type
target_id
organization_id
result
created_at
```

No se almacenará el cuerpo completo si contiene secretos o datos personales.

## Riesgo operativo de procesos

Los scripts mencionados por administración no están disponibles. No deben exponerse endpoints ficticios. Cuando se recuperen, se evaluarán:

- Duración.
- Consumo.
- Ejecuciones concurrentes.
- Reintentos.
- Idempotencia.
- Origen de datos.
- Licencias y límites de APIs.
- Alertas y recuperación.
