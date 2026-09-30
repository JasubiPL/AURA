# Aura v0.1.0 — Plan de implementación

**Estado:** Plan para la Opción B, con Agent Runtime independiente de Aura; la viabilidad del acceso por suscripción sigue siendo un bloqueo para el lanzamiento.  
**Fecha:** 2026-09-25  
**Actualizado:** 2026-09-29  
**Rama de integración:** `feature/mvp-v0.1.0`, mediante ramas y PR pequeñas dirigidas a ella, no a `main`.  
**Trazabilidad SDD:** [Brief](brief.md) → [Requisitos](requirements.md) → [Diseño](design.md) → [Tareas](tasks.md) con [validación planificada](validation.md) → implementación → validación ejecutada y reconciliación de specs. Este plan ordena las fases; las tareas contienen casillas, dependencias, referencias al diseño y evidencias. La [guía SDD](../development/sdd.md) define preparación/cierre por incremento. Los refinamientos de [ADR-003](../adr/ADR-003-contratos-turnos-herramientas-sesiones.md) son propuestas para revisión.

## Fase 0 — Spike de factibilidad del runtime independiente y la suscripción

1. Revisar el contrato propuesto de T-00 y construir el ciclo **mínimo y propio de Aura**: streaming con mock, una lectura de fixture y correlación de llamada/resultado. Probar llamadas inválidas, rechazo, cancelación y stream incompleto sin efectos. No implementar shell ni autenticación en este prototipo.
2. OAuth está aprobado como método de inicio de sesión. Evaluar primero la candidata oficial para aplicaciones abiertas/locales referenciada en ADR-002: elegibilidad de Aura, licencia/distribución, registro/consentimiento, scopes, modelos, cuota y restricciones de preview. OpenCode y Codex SDK/App Server son referencias; no copiar identidades ni elegir un backend no documentado por su existencia en otro cliente.
3. Si existe una vía directa autorizada, crear un prototipo de inicio de sesión, renovación de tokens, cierre de sesión, solicitudes estructuradas de herramientas, errores y comportamiento real de la cuota de suscripción **sin** API key. La autenticación de identidad genérica mediante Sign in with ChatGPT no basta por sí sola.
4. Experimentar con un **Tool Executor controlado por Aura** en macOS: rutas normalizadas y enlaces simbólicos, restricciones del sandbox, cancelación del árbol de procesos, límites de salida de comandos, aislamiento de red y aprobaciones por operación. Comprobar su aplicación real con recursos de prueba desechables.
5. Probar esquema YAML, identidades locales de clones/worktrees/no-Git, locks entre procesos, durabilidad/recuperación JSONL y efectos interrumpidos sin duplicación. Definir captura y disco acotados, retención y presupuestos; los campos reales del proveedor dependen de T-01. Validar Node.js 24 en Mac y cerrar decisiones pertinentes mediante T-04, sin convertir propuestas en aprobaciones.

Entregar `docs/mvp/spike-results.md` con versiones de plataforma, URLs de referencia, pasos de reproducción, evidencias, casos fallidos y bloqueos restantes. Nunca guardar tokens ni datos privados de cuenta en commits.

**Condición:** cada módulo exige su contrato suficiente y evidencia pertinente, según las dependencias de `tasks.md`; las investigaciones son independientes. Si no se demuestra inferencia directa autorizada, continuar el núcleo con mock, pero bloquear el lanzamiento respaldado por suscripción hasta aprobar otra estrategia. No delegar en App Server ni pasar a API de pago.

## Fase 1 — CLI, configuración y sesiones

Implementar la CLI interactiva `aura`, detección de la raíz del proyecto, YAML validado por esquema para usuario y proyecto y escritura/recuperación de sesiones JSONL aisladas por repositorio. Añadir configuración explícita del modelo y mensajes claros de diagnóstico.

**Condición:** YAML inválido falla de forma segura; clones/worktrees/directorios sin Git no mezclan historial; escritores concurrentes no corrompen una sesión y reanudar no repite efectos inciertos.

## Fase 2 — Agent Runtime independiente de Aura

Implementar turnos en streaming, contrato del proveedor, registro/esquemas/IDs de herramientas y ejecución secuencial, routers directos separados, contexto, resultados acotados, cancelación y presupuestos observables. Añadir mock determinista y artefactos aislados por sesión con límites durante captura; derivar títulos localmente. No incorporar compactación automática ni extensiones.

**Condición:** un modelo simulado solicita una herramienta; Aura decide por sí misma la autorización y ejecución, devuelve el resultado al modelo, completa una respuesta y se detiene al alcanzar los presupuestos configurados.

## Fase 3 — Adaptador directo a la suscripción OpenAI (si se autoriza)

Implementar OAuth por la vía directa verificada en T-01: registro/consentimiento para Aura, inicio/cierre de sesión y renovación segura coordinada, errores y uso observable. Nunca extraer tokens de Codex ni usar identidades prestadas. No implementar Anthropic ni modo API facturado.

**Condición:** un turno real y autorizado del modelo mediante suscripción funciona con el ciclo y las herramientas propias de Aura en macOS; de lo contrario, el lanzamiento permanece bloqueado con evidencia reproducible.

## Fase 4 — Ejecución segura de herramientas y aceptación de extremo a extremo

Construir lectura por rangos, búsqueda controlada y edición condicionada con diff; ejecutar shell solo con sandbox verificado. Aplicar aprobaciones acotadas/caducidad/revocación, raíces, red y límites de captura/tiempo. Se pueden validar herramientas con mock antes de integrar OAuth, sin relajar T-02. Ejecutar los escenarios y fallos de `validation.md`, incluido un cambio real con prueba proporcional y reanudación sin duplicar efectos.

**Condición:** todos los requisitos aplicables de `requirements.md` pasan en una Mac de prueba real, incluido el adaptador de suscripción en vivo. Solo entonces abrir la PR de `feature/mvp-v0.1.0` a `main` y crear la etiqueta `v0.1.0`.

## Flujo de ramas

Crear ramas acotadas `feat/*`, `fix/*`, `docs/*` y `test/*` desde la versión más reciente de la rama del MVP y dirigir sus PR a `feature/mvp-v0.1.0`. Eliminar cada rama de origen después de comprobar su merge y sus PR dependientes. Eliminar la rama del MVP solo después de integrarla finalmente a la rama estable `main`.

## Versiones posteriores

Las decisiones futuras aprobadas podrán incorporar Anthropic, un adaptador independiente para la API facturada de OpenAI, otros sistemas operativos, más modelos, routers avanzados, workflows y agentes opcionales, capacidades externas, memoria avanzada e inspeccionable y una TUI elaborada. App Server no sustituye automáticamente los requisitos de un runtime independiente.

Según [ADR-004](../adr/ADR-004-sdd-desarrollo-y-flujo-predeterminado.md), Aura completa usará SDD por defecto para cambios de software. Antes de implementar esa capacidad posterior, crear su spec con selección del flujo, continuidad, gestión de artefactos y aceptación; no añadir un Workflow Engine completo a estas fases del MVP.
