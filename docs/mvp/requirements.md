# Aura v0.1.0 — Requisitos verificables

**Estado:** Especificación inicial del runtime independiente; la integración directa mediante suscripción sigue sujeta a la verificación de factibilidad de la fase 0.  
**Fecha:** 2026-09-25  
**Arquitectura:** [ADR-002](../adr/ADR-002-runtime-independiente-openai.md).

## Aceptación funcional y de seguridad

| ID | Requisito | Evidencia |
| --- | --- | --- |
| R-01 | Iniciar una CLI interactiva en macOS con Node.js 24 o posterior y una raíz de proyecto conocida. | Prueba de instalación e inicio de extremo a extremo en macOS. |
| R-02 | Aura implementa **su propio** ciclo de turnos e interpreta las respuestas del proveedor. Las herramientas solicitadas por el modelo se ejecutan siempre a través del Tool Executor de Aura, nunca por un agente Codex delegado. | Ciclo completo con proveedor mock y una llamada real a herramienta. |
| R-03 | Incluir **solo OpenAI** con un modelo configurado. Sin Anthropic, selección automática entre proveedores ni adaptador de API key de pago en v0.1.0. | Pruebas de selección de proveedor y rechazo de proveedores no admitidos. |
| R-04 | Priorizar inferencia directa y autorizada mediante suscripción ChatGPT/Codex. La fase 0 debe distinguir la identidad OAuth del permiso de inferencia, el protocolo admitido y la cuota de suscripción. | Evaluación documentada de autorización y turno real exitoso del modelo; en caso contrario, bloquear el lanzamiento respaldado por suscripción. |
| R-05 | No copiar tokens ni secretos de cliente de Codex; no usar identidades OAuth prestadas o no autorizadas ni sustituir silenciosamente por facturación API. Manejar explícitamente autenticación no admitida, vencimiento y agotamiento de cuota. | Pruebas de autenticación, vencimiento y rechazo sin API key configurada ni fallback facturado. |
| R-06 | Leer, buscar y editar archivos solo dentro de raíces autorizadas; aplicar rutas protegidas, control de enlaces simbólicos y aprobación explícita para ampliar el alcance. | Pruebas de acceso permitido y denegado al sistema de archivos. |
| R-07 | Ejecutar todos los comandos de shell mediante un Tool Executor controlado por Aura, con aislamiento verificable en macOS y límites de tiempo, salida y árbol de procesos; rechazar la ejecución cuando no sea posible aplicar esas restricciones. | Pruebas de escape del sandbox, cancelación de procesos y timeout. |
| R-08 | Las aprobaciones se limitan a la operación concreta; una acción rechazada nunca se reintenta mediante otra herramienta. Las acciones sensibles de red o publicación requieren autorización aparte. | Pruebas de permitir, denegar, revocar y exfiltración. |
| R-09 | Validar el YAML de usuario y proyecto sin persistir secretos, con precedencia explícita y denegación por defecto ante conflictos de seguridad. | Pruebas de configuración inválida, precedencia y protección de secretos. |
| R-10 | Persistir sesiones JSONL versionadas con identidad estable por repositorio, recuperación tras reiniciar y sin mezclar accidentalmente repositorios. | Prueba de extremo a extremo con reinicio y dos repositorios. |
| R-11 | Aplicar límites configurables de turnos del modelo, tiempo transcurrido, llamadas a herramientas, duración de comandos e iteraciones de corrección en el ciclo propio de Aura. No prometer límites garantizados de cuota, tokens o coste sin soporte del proveedor. | Pruebas de agotamiento, terminación y presentación de uso. |
| R-12 | Explicar un comentario de PR sin builds ni especialistas no solicitados; en una tarea distinta, completar una edición real de código y una prueba proporcional. | Pruebas de explicación de solo lectura y cambio en repositorio. |

## Contrato de configuración propuesto

- Usuario: `~/.aura/config.yaml`; proyecto: `<repo>/.aura/config.yaml`; sesiones: JSONL en `~/.aura/sessions/<repo-id>/`.
- Precedencia candidata para opciones no relacionadas con seguridad: opción explícita de CLI > configuración del proyecto > configuración del usuario > valores por defecto. Los permisos efectivos son la intersección de las restricciones del runtime, usuario y proyecto; el resultado de una herramienta o una skill de proyecto nunca puede elevar permisos.
- Secciones candidatas de primer nivel: `provider`, `session`, `permissions`, `sandbox`, `budgets`, `profiles`. La autenticación del proveedor debe permanecer fuera del YAML y de Git.
- El esquema versionado, la normalización de rutas, los registros persistentes de aprobación y los valores numéricos predeterminados de presupuestos se cerrarán después del spike.

## Plataforma y herramientas de desarrollo

- **Aprobado:** macOS primero; JavaScript ESM + JSDoc; **Node.js 24 como mínimo** y como objetivo inicial de desarrollo y distribución. Las actualizaciones de versión requieren comprobar compatibilidad.
- **Aún propuesto:** npm con lockfile en el repositorio; `node:test` y pruebas de extremo a extremo focalizadas en macOS; licencia MIT.

## Límite del proveedor

El adaptador OpenAI traduce solicitudes y respuestas del modelo y mensajes estructurados de llamadas a herramientas, pero nunca despacha operaciones de shell ni del sistema de archivos. Codex App Server y SDK no son el adaptador de proveedor para v0.1.0. El acceso directo por suscripción debe demostrarse mediante una integración autorizada y suficientemente estable; un adaptador mock permite probar el núcleo independiente mientras esto sigue pendiente. No exponer un fallback a la API de pago.

## Aplazado

Anthropic, uso facturado de la API de OpenAI, cambio o fallback de modelos, capacidades externas, agentes especialistas, workflows avanzados, memoria automática de largo plazo, TUI y distribución multiplataforma requieren especificaciones futuras aprobadas.
