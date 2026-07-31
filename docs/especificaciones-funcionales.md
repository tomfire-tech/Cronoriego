# Especificaciones funcionales

## 1. Sitio público

### Objetivo

Presentar CronoRiego, explicar el problema, funcionamiento, plataforma, beneficios, precios, clientes y vías de contacto.

### Requisitos de paridad

- Mantener navegación por secciones.
- Mantener acceso al inicio de sesión.
- Mantener demostración de módulos y cálculo comercial.
- Mantener diseño responsivo.
- Corregir el enlace de correo para usar `mailto:`.
- Definir un destino real para el formulario de contacto antes de la migración.

## 2. Inicio de sesión

### Objetivo

Autenticar por correo y contraseña y dirigir al usuario según su rol vigente.

### Flujo

1. El usuario ingresa correo y contraseña.
2. Supabase Auth valida credenciales.
3. Se consulta `v_usuarios` por la identidad autenticada.
4. Se verifica que el usuario exista y esté activo.
5. Se redirige a administración o dashboard.
6. Si la sesión ya existe, se resuelve el destino sin solicitar nuevamente credenciales.

### Errores requeridos

- Credenciales inválidas.
- Usuario sin perfil.
- Usuario inactivo.
- Rol no reconocido.
- Sesión vencida.
- Supabase no disponible.

## 3. Página administrativa

### Objetivo

Mostrar clientes y gestionar acceso, asociación y módulos entregados a los productores.

### Funciones actuales que deben conservarse

- Validación de sesión y rol administrativo.
- Indicadores de clientes, predios, zonas activas, NDVI reciente, recomendaciones y último cálculo.
- Tabla de razones sociales con resumen de predios y zonas.
- Detalle por cliente.
- Creación de cuenta vinculada a una razón social.
- Asignación de rol.
- Gestión de módulos contratados.
- Copia del enlace de onboarding.
- Presentación de acciones de descarga y generación de recomendaciones.

### Cambios de arquitectura sin cambio funcional

- Crear usuarios mediante FastAPI y Supabase Admin, no mediante `signUp` público.
- Modificar roles y módulos mediante endpoints administrativos.
- Registrar auditoría de actor, operación, fecha y resultado.
- Convertir las acciones de procesamiento en trabajos reales o declararlas fuera del producto; no mantener botones simulados como si ejecutaran tareas.
- Conservar los nombres técnicos actuales de roles y módulos.

## 4. Ingreso con rol productor

### Objetivo

Un usuario con rol `agricultor` debe visualizar únicamente la información correspondiente a su razón social.

### Carga de contexto

1. Resolver usuario desde la sesión.
2. Obtener razón social y módulos.
3. Consultar predios de la razón social.
4. Consultar equipos del predio elegido.
5. Consultar zonas activas del equipo elegido.
6. Cargar datos del módulo y semana seleccionados.

### Funciones del dashboard

- Selección de predio, equipo y semana.
- Indicadores y calendario de riego.
- Datos de estación y pronóstico.
- Recomendaciones por zona.
- Información NDVI e imágenes georreferenciadas.
- Confirmación de ejecución de riego y fertirriego.
- Operario y profesional por defecto.
- Fertirriego y cobertura nutricional.
- Dinámica de estanque.
- Balance nutricional.
- Mantenimiento y aforos de fuente, equipo y zona.
- Historial y exportaciones XLSX.
- Cambio de contraseña.
- Experiencia adaptada a escritorio y móvil.
- Bloqueo informativo de módulos no contratados.

### Estados requeridos

- Cargando sesión.
- Sin predios.
- Sin equipos.
- Sin zonas activas.
- Sin información para la semana.
- Módulo no contratado.
- Acceso denegado.
- Error de red o consulta.
- Sesión vencida.

## 5. Onboarding

### Objetivo

Registrar la estructura productiva de un nuevo cliente.

### Entidades

1. Razón social.
2. Predio.
3. Equipos de riego.
4. Zonas.
5. Perfiles y parámetros de suelo asociados a cada zona.

### Requisitos

- Consultar `ref_tabla_suelo` para opciones y parámetros.
- Permitir múltiples equipos y zonas.
- Validar campos requeridos, números, coordenadas y relaciones.
- Mostrar un resumen antes del envío.
- Enviar una sola solicitud idempotente a FastAPI.
- Evitar duplicados.
- Confirmar creación completa o informar que no se creó ninguna entidad.
- No dejar razones sociales, predios o equipos huérfanos tras un error intermedio.
- Registrar auditoría y un identificador de solicitud.

### Aclaración

El onboarding crea la estructura de datos del productor. La cuenta de acceso puede crearse desde administración antes o después, pero ambos flujos deben vincularse mediante la razón social y no duplicar identidades.

## 6. Procesos administrativos

El código actual menciona:

- Descarga de ETo y NDVI mediante `cronoriego_v4.py`.
- Generación o carga de recomendaciones mediante `subir_tablas_csv.py`.

Los scripts no están presentes. La migración requiere recuperar su código y especificar entradas, salidas, frecuencia, credenciales, reintentos y observabilidad antes de ofrecer estas acciones como endpoints o trabajos programados.
