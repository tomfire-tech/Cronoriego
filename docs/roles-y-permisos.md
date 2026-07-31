# Roles, permisos y acceso

## Principio general

El acceso efectivo no depende solamente del rol. Debe evaluarse como:

```text
rol + usuario activo + razón social asociada + módulos contratados + RLS
```

Los identificadores técnicos existentes se conservan para evitar incompatibilidades con Supabase.

## Roles observados

| Rol técnico | Nombre de negocio | Destino actual | Alcance esperado |
| --- | --- | --- | --- |
| `super_admin` | Superadministrador | Administración | Control global y configuración crítica |
| `admin` | Administrador | Administración | Gestión operativa de clientes, usuarios y módulos |
| `gerencia` | Gerencia | Administración | Visibilidad global; las operaciones de escritura deben definirse explícitamente |
| `agricultor` | Productor | Dashboard | Datos de su razón social y módulos contratados |
| `tecnico` | Técnico | Dashboard | Fallback actual; su alcance exacto requiere definición antes de migrar |

Actualmente `login.html` envía a administración solo a `super_admin` y `admin`, mientras `admin.html` acepta también `gerencia`. Esta inconsistencia debe resolverse en la nueva matriz de rutas. Para conservar el comportamiento visible sin bloquear una decisión pendiente, `Cronoriego-v2` deberá tratar esta diferencia como una prueba de paridad explícita.

## Reglas obligatorias

1. React no decidirá permisos basándose solo en valores guardados en memoria o recibidos desde formularios.
2. Supabase RLS debe impedir acceso cruzado entre razones sociales.
3. FastAPI validará el JWT de Supabase y consultará el usuario vigente antes de ejecutar operaciones sensibles.
4. FastAPI derivará correo, rol y razón social desde la identidad validada; no confiará en campos equivalentes enviados por el cliente.
5. Un usuario inactivo no podrá operar aunque conserve una sesión válida.
6. Los módulos contratados controlan funcionalidades, no sustituyen permisos de datos.
7. La interfaz puede ocultar o bloquear opciones, pero esa presentación nunca será el único control de seguridad.

## Módulos comerciales

| Identificador | Nombre | Plan mostrado actualmente |
| --- | --- | --- |
| `riego` | Riego | Estándar |
| `ferti` | Fertirriego | Pro |
| `estanque` | Dinámica de estanque | Pro |
| `balance` | Balance nutricional | Premium |
| `mantenimiento` | Mantenimiento y aforos | Estándar |

El administrador actual solo edita `riego`, `ferti`, `estanque` y `balance`. Antes de implementar la v2 se debe decidir si `mantenimiento` siempre forma parte del plan Estándar o si también debe ser configurable. Hasta esa decisión, la migración debe conservar el comportamiento actual por cliente.

## Matriz objetivo inicial

| Acción | super_admin | admin | gerencia | agricultor | tecnico |
| --- | :---: | :---: | :---: | :---: | :---: |
| Ver todos los clientes | Sí | Sí | Sí | No | No |
| Crear usuarios | Sí | Sí | Requiere definición | No | No |
| Cambiar roles | Sí | Sí | Requiere definición | No | No |
| Cambiar módulos | Sí | Sí | Requiere definición | No | No |
| Ejecutar procesos globales | Sí | Sí | Requiere definición | No | No |
| Ver su razón social | Sí | Sí | Sí | Sí | Según asociación |
| Ver dashboard contratado | Sí | Sí | Sí | Sí | Según asociación |
| Registrar ejecuciones/mantenimiento | Sí | Sí | Sí | Sí | Requiere definición |
| Cambiar su contraseña | Sí | Sí | Sí | Sí | Sí |

Las celdas “Requiere definición” no autorizan la operación. Deben resolverse antes de implementar endpoints.

## Resolución de sesión objetivo

1. Supabase Auth autentica al usuario.
2. React obtiene el JWT.
3. Se consulta un perfil normalizado derivado de `v_usuarios`.
4. Se verifica `activo`.
5. Se determina la ruta permitida.
6. Cada consulta queda limitada por RLS.
7. Cada endpoint sensible repite la validación en FastAPI.
