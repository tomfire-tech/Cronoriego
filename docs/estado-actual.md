# Estado actual del proyecto

## Resumen

CronoRiego es actualmente una aplicación web estática. Cada página contiene su HTML, CSS y JavaScript en un solo archivo y consume Supabase directamente desde el navegador. No existe un backend Python dentro del repositorio; el servidor Python usado en desarrollo solamente publica archivos estáticos.

## Inventario

| Archivo | Función principal | Líneas aproximadas |
| --- | --- | ---: |
| `index.html` | Sitio público, presentación comercial, precios, contacto y demostración | 1.771 |
| `login.html` | Inicio de sesión y redirección según rol | 129 |
| `admin.html` | Administración, cuentas, clientes y módulos contratados | 506 |
| `formulario_onboarding.html` | Alta de razón social, predio, equipos y zonas | 547 |
| `dashboard.html` | Aplicación del productor, escritorio y móvil | 3.048 |

El repositorio también contiene logo, favicon, `CNAME` y un `README.md` mínimo.

## Tecnología actual

- HTML5, CSS y JavaScript sin proceso de compilación.
- Supabase JS v2 para autenticación y acceso a PostgreSQL.
- Leaflet para mapas.
- Chart.js para gráficos.
- SheetJS/XLSX para exportaciones.
- Tabler Icons y Google Fonts.
- Recursos externos cargados desde CDN.

## Flujo de ejecución

Las páginas pueden publicarse en cualquier hosting estático. En desarrollo se sirven con:

```powershell
python -m http.server 8000 --bind 127.0.0.1
```

Ese proceso no ejecuta lógica de negocio ni protege datos. La aplicación se comunica directamente con el proyecto remoto de Supabase.

## Distribución actual de responsabilidades

| Responsabilidad | Ubicación actual |
| --- | --- |
| Inicio y persistencia de sesión | Supabase Auth desde el navegador |
| Resolución de rol | Consulta de `v_usuarios` por correo |
| Filtrado de datos del productor | Consultas del navegador y políticas RLS no incluidas en el repositorio |
| Creación de cuentas | `admin.html` mediante `auth.signUp` |
| Alta de estructura productiva | Inserciones secuenciales desde `formulario_onboarding.html` |
| Gestión de módulos | Actualización directa de `razones_sociales.modulos` |
| Cálculos y descargas semanales | El panel solo muestra instrucciones para ejecutar scripts externos que no están en este repositorio |
| Presentación y cálculos de interfaz | JavaScript embebido en cada HTML |

## Fortalezas

- El producto ya cubre flujos reales de riego y fertirriego.
- Existe una separación visible entre sitio público, autenticación, administración, onboarding y dashboard.
- Supabase centraliza autenticación y datos.
- El dashboard contempla escritorio y móvil.
- El sistema maneja módulos comerciales y segmentación por razón social.

## Limitaciones observadas

1. `dashboard.html` concentra presentación, consultas, estado, reglas y exportaciones.
2. No hay tipos, módulos de JavaScript, pruebas automatizadas ni contratos de API.
3. URL y clave anónima de Supabase están repetidas en cuatro archivos.
4. La autorización de navegación ocurre en el cliente; la protección real depende de RLS, cuya configuración no está versionada aquí.
5. Onboarding ejecuta inserciones encadenadas sin una transacción visible.
6. Crear un usuario con `auth.signUp` desde la sesión administrativa puede producir efectos de sesión no deseados y depende de la configuración de Supabase Auth.
7. Las acciones “Descarga semanal” y “Recomendaciones” no ejecutan procesos: indican comandos de scripts ausentes.
8. El formulario público de contacto no muestra una integración de envío.
9. Hay dependencias con versión `latest`, lo que reduce reproducibilidad.
10. `mantenimiento` está disponible en el dashboard, pero no figura en la lista editable de módulos del administrador.

## Información que falta versionar

Para asegurar una migración fiel deben incorporarse o exportarse:

- Esquema PostgreSQL.
- Vistas y funciones.
- Políticas RLS.
- Triggers.
- Buckets y políticas de Storage.
- Configuración relevante de Supabase Auth.
- Scripts de descarga, cálculo y carga mencionados por el panel.
- Variables por entorno y procedimiento de despliegue.
