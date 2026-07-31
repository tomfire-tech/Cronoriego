# Recomendaciones

## Prioridad 0: antes de implementar

1. Exportar y auditar Supabase, especialmente RLS.
2. Recuperar los scripts `cronoriego_v4.py` y `subir_tablas_csv.py`.
3. Definir permisos de `gerencia` y `tecnico`.
4. Resolver si `mantenimiento` es configurable o siempre pertenece al plan Estándar.
5. Inventariar las tablas de mantenimiento construidas dinámicamente.
6. Definir cómo se procesa el formulario público de contacto.

## Prioridad 1: seguridad e integridad

1. Trasladar cuentas, permisos, módulos y onboarding a FastAPI.
2. Hacer el onboarding transaccional e idempotente.
3. Añadir auditoría.
4. Probar aislamiento entre razones sociales.
5. Eliminar fallbacks silenciosos de autorización: un perfil ausente debe producir un error controlado.

## Prioridad 2: mantenibilidad

1. Usar TypeScript en React.
2. Separar el dashboard por features.
3. Centralizar consultas en hooks/servicios.
4. Encapsular nombres heredados de columnas en repositorios y mapeadores.
5. Definir modelos de dominio con nombres consistentes.
6. Centralizar diseño en Sass: tokens, tipografía, espaciado, colores y breakpoints.
7. Sustituir atributos `onclick` e HTML generado como cadenas por componentes.

## Prioridad 3: calidad y operación

1. CI para lint, tipos, pruebas y build.
2. Entorno Supabase de pruebas.
3. Datos semilla anonimizados.
4. Monitoreo de frontend, API y trabajos.
5. Métricas de errores, tiempos de carga y tareas.
6. Manual operativo y procedimiento de reversión.

## Mejoras que conservan funcionalidad

- Estados de carga y error consistentes.
- Caché y cancelación de consultas al cambiar predio/equipo/semana.
- Accesibilidad de formularios, modales, navegación y gráficos.
- URLs navegables por módulo y filtros relevantes.
- Versiones fijas de dependencias.
- Carga diferida de mapas, gráficos y exportación.
- Sustitución del enlace de correo incorrecto por `mailto:`.
- Mensajes de módulo no contratado centralizados.
- Exportaciones generadas desde servicios testeables.

## Mejoras posteriores a la paridad

Estas ideas no deben incluirse en la primera migración sin una decisión funcional independiente:

- Invitaciones por correo en lugar de compartir contraseñas.
- Permisos granulares más allá del rol.
- Administración de usuarios existentes, suspensión y reactivación.
- Historial de cambios de módulos.
- Notificaciones de riego y fertirriego.
- Ejecución programada y monitoreo de trabajos desde administración.
- Panel de calidad y frescura de datos.
- Modo offline o tolerancia a conectividad rural.

## Recomendación final

La modernización es viable y conveniente, pero React y Python por sí solos no corrigen los riesgos principales. El éxito depende de capturar el comportamiento actual, versionar Supabase, definir autorización y construir pruebas de paridad antes de cambiar el proyecto productivo.
