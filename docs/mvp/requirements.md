# Aura v0.1.0 — Requisitos verificables

**Estado:** Especificación inicial del runtime independiente; la integración directa mediante suscripción sigue sujeta a la verificación de factibilidad de la fase 0.  
**Fecha:** 2026-09-25  
**Actualizado:** 2026-09-29  
**Arquitectura:** [ADR-002](../adr/ADR-002-runtime-independiente-openai.md).

El alcance aprobado se conserva. Los detalles incorporados desde la revisión de OpenCode refinan su aceptación y siguen como propuestas de [ADR-003](../adr/ADR-003-contratos-turnos-herramientas-sesiones.md) hasta su revisión. La [matriz de validación](validation.md) distingue pruebas planificadas de evidencia ejecutada.

## Aceptación funcional y de seguridad

| ID | Requisito | Evidencia |
| --- | --- | --- |
| R-01 | Iniciar una CLI interactiva en macOS con Node.js 24 o posterior y una raíz de proyecto conocida. | Prueba de instalación e inicio de extremo a extremo en macOS. |
| R-02 | Aura implementa **su propio** ciclo de turnos en streaming. Las herramientas pasan por su Tool Executor, con IDs correlacionados, definición vigente y validación de entrada/salida. Propuesta: ejecución secuencial solo tras recibir llamadas completas y terminación satisfactoria del turno. | Ciclo con mock y fixture; llamada desconocida/obsoleta, ID duplicado, argumentos inválidos y stream interrumpido no producen efectos. |
| R-03 | Incluir **solo OpenAI** con un modelo configurado. Sin Anthropic, selección automática entre proveedores ni adaptador de API key de pago en v0.1.0. | Pruebas de selección de proveedor y rechazo de proveedores no admitidos. |
| R-04 | Priorizar inferencia directa y autorizada mediante suscripción ChatGPT/Codex. Evaluar la candidata oficial de ADR-002, distinguiendo identidad OAuth, permisos de inferencia, elegibilidad de Aura, protocolo y cuota. | Evaluación de autorización y distribución, registro para Aura y turno real exitoso; en caso contrario, bloquear el lanzamiento respaldado por suscripción. |
| R-05 | Iniciar sesión con **OAuth para Aura**, con consentimiento, validación de identidad/scopes y ciclo de vida seguro de tokens. No copiar tokens de Codex, usar identidades prestadas ni sustituir por facturación API. Manejar acceso no admitido, vencimiento y agotamiento de cuota. | OAuth, renovación, logout y fallos sin API key ni fallback facturado; identidad e inferencia verificadas por separado. |
| R-06 | Leer por rangos, buscar y editar solo dentro de raíces autorizadas; proteger rutas y enlaces simbólicos. Propuesta: editar con contenido esperado, diff y coincidencia inequívoca; rechazar contenido cambiado y revalidar tras la aprobación. | Acceso permitido/denegado también durante búsqueda; symlinks, archivo cambiado, matching ambiguo y preservación del formato. |
| R-07 | Ejecutar shell mediante el Tool Executor propio, con aislamiento verificable en macOS y límites de tiempo, captura de salida y árbol de procesos; rechazar si no puede aplicar esas restricciones. | Escape, cancelación, timeout de descendientes y salida continua acotada durante captura; red permitida/denegada. |
| R-08 | Limitar aprobaciones a la operación y alcance concretos; revalidarlas antes del efecto. Propuesta: `deny` efectivo prevalece y las concesiones de sesión caducan al cerrar la CLI. Una acción rechazada nunca se reintenta mediante otra herramienta. Red sensible o publicación requieren autorización aparte. | Permitir, denegar, caducar, inspeccionar y revocar; una aprobación no supera restricciones; rechazo sin nueva llamada equivalente. |
| R-09 | Validar el YAML de usuario y proyecto sin persistir secretos, con precedencia explícita y denegación por defecto ante conflictos de seguridad. | Pruebas de configuración inválida, precedencia y protección de secretos. |
| R-10 | Persistir JSONL versionado y aislado por workspace local. Propuesta: separar clones/worktrees/directorios sin Git, admitir un solo escritor por sesión entre procesos y recuperar sin repetir efectos de llamadas interrumpidas. | Reinicio y línea final incompleta; corrupción intermedia rechazada; identidades distintas, escritor exclusivo y efecto incierto sin reejecución automática. |
| R-11 | Limitar turnos, tiempo, herramientas, comandos, resultados y correcciones durante el ciclo. Propuesta: artefactos acotados por sesión y títulos locales sin llamadas extra. No prometer límites garantizados de cuota, tokens o coste sin soporte del proveedor. | Agotamiento y terminación; memoria/captura/disco limitados; uso medido/estimado/desconocido; aislamiento de artefactos. |
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
