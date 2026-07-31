# Estrategia de migración

## Enfoque

Se realizará una reescritura completa en paralelo. El repositorio actual será la referencia ejecutable y no se moverá a una carpeta `legacy`. La nueva implementación vivirá en `X:\Tomy\Cronoriego-v2`.

## Fase 0: completar el conocimiento

- Aprobar esta documentación.
- Exportar esquema, vistas, funciones y RLS de Supabase.
- Recuperar scripts de descarga y recomendaciones.
- Definir permisos exactos de `gerencia` y `tecnico`.
- Resolver el tratamiento comercial de `mantenimiento`.
- Inventariar tablas dinámicas de mantenimiento.
- Registrar casos reales representativos sin exponer datos personales.

## Fase 1: fundaciones

- Crear repositorio `Cronoriego-v2`.
- Configurar frontend React + TypeScript + Vite + Sass.
- Configurar backend FastAPI.
- Añadir ambientes local, pruebas y producción.
- Implementar validación de JWT, `/health`, `/me`, logs y errores.
- Preparar CI con lint, tipos, pruebas y compilación.

## Fase 2: autenticación y sitio público

- Migrar sitio público.
- Implementar login y cierre de sesión.
- Resolver rutas por rol.
- Añadir guards de rutas y estados de sesión.
- Verificar paridad visual y responsiva.

## Fase 3: administración

- Migrar indicadores, clientes y detalle.
- Crear usuarios mediante FastAPI.
- Gestionar roles, asociación y estado.
- Gestionar módulos contratados.
- Añadir auditoría.

## Fase 4: onboarding

- Migrar el formulario y catálogo de suelos.
- Implementar endpoint transaccional e idempotente.
- Probar éxito, duplicados, validaciones y rollback.

## Fase 5: dashboard del productor

Orden recomendado:

1. Contexto de razón social, predio, equipo, zona y semana.
2. Riego y datos meteorológicos.
3. NDVI y mapas.
4. Confirmaciones de ejecución.
5. Fertirriego.
6. Estanque.
7. Balance nutricional.
8. Mantenimiento y exportaciones.
9. Experiencia móvil.

## Fase 6: procesos y operación

- Integrar los scripts recuperados como trabajos.
- Añadir estados, reintentos, exclusión mutua y trazabilidad.
- Definir programación automática.
- Documentar operación y recuperación.

## Matriz de paridad mínima

| Área | Caso crítico | Evidencia requerida |
| --- | --- | --- |
| Auth | Login y redirección por cada rol | E2E |
| Seguridad | Productor A no accede a razón social B | Prueba RLS e integración |
| Admin | Crear usuario y vincularlo | E2E + auditoría |
| Admin | Cambiar módulos | E2E + persistencia |
| Onboarding | Crear estructura completa | Integración |
| Onboarding | Fallo intermedio no deja huérfanos | Integración |
| Riego | Semana, zonas y recomendaciones coinciden | Comparación de datos |
| NDVI | Capas y detalle coinciden | E2E visual |
| Fertirriego | Eventos y balances coinciden | Comparación de datos |
| Estanque | KPIs y gráficos coinciden | Comparación de datos |
| Mantenimiento | Crear, listar y exportar | E2E |
| Responsive | Flujos principales en móvil | E2E visual |

## Criterios para cambiar de proyecto

- Matriz funcional aprobada.
- Pruebas críticas exitosas.
- RLS validada para todos los roles.
- Onboarding completo probado.
- Datos comparados sobre una muestra acordada.
- Sin secretos de servidor en el bundle.
- Monitoreo, respaldos y procedimiento de reversión disponibles.
- Aprobación de responsables funcionales.

## Reversión

El cambio inicial debe permitir volver al hosting actual sin transformar irreversiblemente los datos. Las migraciones de base de datos deben ser compatibles durante el periodo de convivencia o contar con scripts de reversión verificados.
