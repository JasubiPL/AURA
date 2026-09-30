# Aura v0.1.0 — Tareas de SDD

**Estado:** Pendientes; lista inicial para implementar y verificar el [diseño](design.md) y los [requisitos](requirements.md).  
**Fecha:** 2026-09-25  
**Actualizado:** 2026-09-29  
**Destino de las PR:** `feature/mvp-v0.1.0`. Cada tarea puede dividirse en PR pequeñas sin perder su criterio de aceptación.

Marca una casilla solo cuando exista evidencia en código o documentación y la validación indicada se haya completado. Si cambia el diseño, actualiza este archivo y los requisitos relacionados en la misma PR. El [plan de implementación](implementation-plan.md) define el orden de fases; la [matriz de validación](validation.md) fija los casos V-01 a V-12. Los contratos de ADR-003 siguen propuestos; ninguna tarea ejecutable se marca completada con el análisis de OpenCode.

## Fase 0 — Resolver factibilidad antes de cerrar contratos

- [ ] **T-00 · Contrato de turnos con mock** (R-02, R-11; V-02/V-11): tras revisar su diseño propuesto, un mock en streaming pide leer un fixture; Aura valida/autoriza, registra inicio y resultado y completa la respuesta. Sin shell ni credenciales. Evidencia: comando reproducible y variantes de llamada desconocida/obsoleta, ID duplicado, argumentos inválidos, ruta denegada, rechazo, cancelación y stream incompleto sin efectos.
- [ ] **T-01 · OAuth e inferencia OpenAI por suscripción** (R-03 a R-05; V-03 a V-05): OAuth está elegido. Evaluar primero la candidata oficial de ADR-002: elegibilidad y distribución de Aura, registro/consentimiento, identidad/scopes de inferencia, modelos, protocolo/preview, almacenamiento/renovación/logout y cuota. Probar turno real y tool call por el loop propio sin API key cuando la vía sea permitida. Evidencia: fuentes, pasos y resultados en `spike-results.md`; registrar bloqueo sin adoptar otro acceso si falla.
- [ ] **T-02 · Sandbox y permisos macOS** (R-06 a R-08): prototipo desechable que pruebe raíces, rutas protegidas, symlinks, árbol de procesos, timeout, salida, red y denegaciones. Evidencia: casos permitidos y rechazados reproducibles en una Mac.
- [ ] **T-03 · Contratos de estado y entorno** (R-01, R-09 a R-11; V-01/V-09 a V-11): comprobar Node.js 24, YAML, identidad local con clones/worktrees/no-Git, lock de sesión entre procesos, JSONL/durabilidad, línea final incompleta/corrupción y efecto interrumpido sin duplicación. Probar captura/artefactos acotados y errores de persistencia. Cerrar esquemas, retención, límites numéricos y recuperación de locks con evidencia; campos del proveedor dependen de T-01 y se mantienen desconocidos si no están disponibles.
- [ ] **T-04 · Revisión y cierre de contratos** (R-01 a R-11): revisar ADR-003 y registrar aprobación o ajustes; cerrar en las specs esquemas, reglas de permisos y decisiones de desarrollo con T-00/T-02/T-03. npm y `node:test` siguen como propuestas. Resolver con Jas la licencia/distribución si condiciona T-01. Evidencia: ADR/specs alineados, dependencias y casos de aceptación suficientes para cada módulo; ninguna adopción literal sin revisar licencia/atribuciones.

**Puerta de fase:** crear `spike-results.md` al ejecutar los prototipos, con fuentes, versiones, pasos, resultados y bloqueos separados por investigación. T-00/T-01/T-02 pueden investigarse de forma independiente; cerrar cada contrato antes de su módulo. Una vía de inferencia no comprobada impide declarar lista la integración y lanzar v0.1.0 como tal. No introducir API facturada ni runtime delegado sin nueva decisión.

## Fase 1 — CLI y estado local

- [ ] **T-10 · CLI y raíz de proyecto** (R-01): entrada interactiva, diagnóstico de proyecto/modelo y errores claros. Evidencia: inicio desde una instalación en macOS.
- [ ] **T-11 · Configuración validada** (R-09): carga de YAML de usuario/proyecto con esquema versionado, conflictos de seguridad denegados y credenciales fuera de archivos. Evidencia: pruebas de configuración inválida, precedencia y permisos.
- [ ] **T-12 · Sesiones aisladas** (R-10; V-10): implementar los contratos cerrados en T-03/T-04: identidad local, escritor único entre procesos, JSONL/durabilidad y reanudación segura. Evidencia: clones/worktrees/no-Git aislados, reinicio, corrupción, lock y herramienta interrumpida sin duplicación de efectos.

## Fase 2 — Runtime independiente

- [ ] **T-20 · Ciclo de agente y adaptador mock** (R-02, R-03): contrato del proveedor, respuesta y llamada de herramienta normalizadas; routers iniciales separados con ruta directa y modelo configurado. Evidencia: un turno completo determinista sin Codex App Server.
- [ ] **T-21 · Contexto, cancelación y límites** (R-11, R-12): selección acotada de contexto, interrupción y presupuestos de turnos, tiempo, herramientas y correcciones. Evidencia: terminación reproducible al alcanzar cada límite; explicación de comentario sin build ni delegación.
- [ ] **T-22 · Registro y despacho de herramientas** (R-02, R-08; V-02/V-08): registro pequeño, esquemas de entrada/salida, IDs y definiciones vigentes; llamadas secuenciales tras cerrar el turno; políticas centrales, revalidación y estados claros. Evidencia: llamadas inválidas sin efectos y rechazo sin alternativa encubierta.
- [ ] **T-23 · Resultados y artefactos acotados** (R-07, R-10, R-11; V-07/V-10/V-11): límites durante captura, previews y artefactos locales con lectura por fragmentos, protección de secretos y aislamiento por sesión/repo. Evidencia del módulo con streams simulados: salida continua, límite de disco, acceso ajeno denegado y fallo de almacenamiento seguro. T-41 valida después la integración de esa captura con shell en vivo; no es una dependencia para cerrar T-23.

## Fase 3 — OpenAI directo, condicionada por T-01

- [ ] **T-30 · OAuth y adaptador permitido** (R-03 a R-05; V-03 a V-05): implementar el registro/protocolo cerrado tras T-01, con consentimiento propio, inicio/cierre de sesión y renovación coordinada segura. Evidencia: turno real y tool calls por el runtime de Aura sin API key ni tokens privados de otro cliente.
- [ ] **T-31 · Errores, cuota y uso** (R-05, R-11): informar vencimiento, agotamiento y acceso no admitido; distinguir valores medidos, estimados y desconocidos. Evidencia: pruebas de error y ausencia de fallback facturado.

## Fase 4 — Herramientas y aceptación

- [ ] **T-40 · Archivos y aprobaciones** (R-06, R-08; V-06/V-08): lectura por rangos, búsqueda controlada y edición con contenido esperado/diff; permisos acotados, caducidad y revocación, revalidación de rutas. Evidencia: symlinks, búsqueda denegada, edición obsoleta/ambigua, formato conservado y rechazo sin reintento.
- [ ] **T-41 · Shell aislado en macOS** (R-07, R-08): Tool Executor propio con sandbox comprobado, límites de proceso/salida/tiempo y política de red. Evidencia: pruebas de escape, cancelación de árbol de procesos y denegación.
- [ ] **T-42 · Validación de extremo a extremo** (R-01 a R-12; V-01 a V-12): explicación de solo lectura, edición con prueba proporcional, reanudación, denegación y turno real de OpenAI. Evidencia: matriz ejecutada sobre el commit de aceptación en macOS real; solo entonces proponer merge a `main` y etiqueta `v0.1.0`.

## Dependencias para empezar cada módulo

| Trabajo | Entrada necesaria |
| --- | --- |
| T-00 | Diseño mínimo revisado; fixture y mock, sin esperar OAuth o shell. |
| T-01 / T-02 | Investigación independiente; fuentes y entorno de prueba apropiados. |
| T-10 / T-11 / T-12 | Parte pertinente de T-03/T-04 cerrada: entorno, YAML o persistencia respectivamente. |
| T-20 / T-21 / T-22 | T-00 y contratos pertinentes de T-04; mock suficiente. |
| T-23 | Contrato de límites de T-03/T-04 y almacenamiento T-12; módulo validado con streams simulados, integración de shell en T-41. |
| T-30 / T-31 | T-01 y contrato de proveedor revisado; T-20/T-22 para ciclo completo. |
| T-40 | T-22 y políticas/rutas revisadas con T-02; puede probarse con mock antes de T-30. |
| T-41 | T-22, T-02 y captura de T-23 validada con simulación; valida su integración real en macOS sin depender de T-30. |
| T-42 | Módulos de R-01 a R-12 implementados y validados, incluyendo T-30 y T-41 reales. |

## Seguimiento

En cada PR identifica los IDs T y R afectados, evidencia ejecutada y decisiones pendientes. Las casillas no sustituyen las pruebas ni convierten propuestas en funciones implementadas. Si T-01 o T-02 bloquean el lanzamiento, continuar únicamente las tareas independientes y reflejar el bloqueo en `docs/status.md`.
