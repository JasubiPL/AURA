# Aura v0.1.0 — Tareas de SDD

**Estado:** Pendientes; lista inicial para implementar y verificar el [diseño](design.md) y los [requisitos](requirements.md).  
**Fecha:** 2026-09-25  
**Destino de las PR:** `feature/mvp-v0.1.0`. Cada tarea puede dividirse en PR pequeñas sin perder su criterio de aceptación.

Marca una casilla solo cuando exista evidencia en código o documentación y la validación indicada se haya completado. Si cambia el diseño, actualiza este archivo y los requisitos relacionados en la misma PR. El [plan de implementación](implementation-plan.md) define el orden de fases; esta lista registra trabajo verificable.

## Fase 0 — Resolver factibilidad antes de cerrar contratos

- [ ] **T-00 · Contrato de turnos con mock** (R-02, R-11): prototipo propio de Aura que recibe una solicitud estructurada de herramienta, decide su ejecución y devuelve el resultado al proveedor simulado. Evidencia: ejecución reproducible y prueba de rechazo/cancelación.
- [ ] **T-01 · Acceso OpenAI por suscripción** (R-03 a R-05): investigar fuentes oficiales y términos, distinguir identidad OAuth de inferencia, comprobar autorización del cliente, modelos, protocolo, renovación y cuota. Si hay vía permitida, probar un turno real sin API key. Evidencia: fuentes, pasos y resultados en `spike-results.md`; si falla, registrar bloqueo sin adoptar otro acceso.
- [ ] **T-02 · Sandbox y permisos macOS** (R-06 a R-08): prototipo desechable que pruebe raíces, rutas protegidas, symlinks, árbol de procesos, timeout, salida, red y denegaciones. Evidencia: casos permitidos y rechazados reproducibles en una Mac.
- [ ] **T-03 · Contratos de estado y entorno** (R-01, R-09 a R-11): comprobar Node.js 24 en macOS, recuperación JSONL con dos repositorios, validación YAML y campos reales de uso disponibles. Cerrar en el diseño los esquemas, precedencia, identidad del repositorio y presupuestos con la evidencia obtenida.

**Puerta de fase:** publicar `spike-results.md` con fuentes, versión de macOS/Node, pasos, resultados y bloqueos. La ausencia de una vía de inferencia directa autorizada impide declarar lista la integración por suscripción y lanzar v0.1.0 como tal. No introducir una API facturada ni un runtime delegado sin una nueva decisión.

## Fase 1 — CLI y estado local

- [ ] **T-10 · CLI y raíz de proyecto** (R-01): entrada interactiva, diagnóstico de proyecto/modelo y errores claros. Evidencia: inicio desde una instalación en macOS.
- [ ] **T-11 · Configuración validada** (R-09): carga de YAML de usuario/proyecto con esquema versionado, conflictos de seguridad denegados y credenciales fuera de archivos. Evidencia: pruebas de configuración inválida, precedencia y permisos.
- [ ] **T-12 · Sesiones aisladas** (R-10): escritura JSONL y reanudación con identidad estable por repositorio y recuperación de un registro incompleto. Evidencia: reinicio con dos repositorios sin mezcla de historial.

## Fase 2 — Runtime independiente

- [ ] **T-20 · Ciclo de agente y adaptador mock** (R-02, R-03): contrato del proveedor, respuesta y llamada de herramienta normalizadas; routers iniciales separados con ruta directa y modelo configurado. Evidencia: un turno completo determinista sin Codex App Server.
- [ ] **T-21 · Contexto, cancelación y límites** (R-11, R-12): selección acotada de contexto, interrupción y presupuestos de turnos, tiempo, herramientas y correcciones. Evidencia: terminación reproducible al alcanzar cada límite; explicación de comentario sin build ni delegación.

## Fase 3 — OpenAI directo, condicionada por T-01

- [ ] **T-30 · Autenticación y adaptador permitido** (R-03 a R-05): implementar solo el protocolo aprobado tras el spike, con inicio/cierre de sesión y renovación segura. Evidencia: turno real autorizado sin API key ni uso de tokens privados de otro cliente.
- [ ] **T-31 · Errores, cuota y uso** (R-05, R-11): informar vencimiento, agotamiento y acceso no admitido; distinguir valores medidos, estimados y desconocidos. Evidencia: pruebas de error y ausencia de fallback facturado.

## Fase 4 — Herramientas y aceptación

- [ ] **T-40 · Archivos y aprobaciones** (R-06, R-08): lectura, búsqueda y edición dentro de raíces autorizadas, con comprobación de rutas y permisos por operación. Evidencia: acceso permitido y denegado, symlinks y rechazo sin reintento por otra herramienta.
- [ ] **T-41 · Shell aislado en macOS** (R-07, R-08): Tool Executor propio con sandbox comprobado, límites de proceso/salida/tiempo y política de red. Evidencia: pruebas de escape, cancelación de árbol de procesos y denegación.
- [ ] **T-42 · Validación de extremo a extremo** (R-01 a R-12): escenario de explicación de solo lectura, edición con prueba proporcional, reanudación, denegación y turno real de OpenAI. Evidencia: matriz de requisitos aprobada en Mac de prueba; solo entonces proponer merge a `main` y etiqueta `v0.1.0`.

## Seguimiento

En cada PR identifica los IDs T y R afectados, evidencia ejecutada y decisiones pendientes. Las casillas no sustituyen las pruebas ni convierten propuestas en funciones implementadas. Si T-01 o T-02 bloquean el lanzamiento, continuar únicamente las tareas independientes y reflejar el bloqueo en `docs/status.md`.
