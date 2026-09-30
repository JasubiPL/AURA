# Aura v0.1.0 — Brief del MVP

**Estado:** Alcance del runtime independiente aprobado; viabilidad del acceso por suscripción pendiente.  
**Fecha:** 2026-09-25  
**Arquitectura:** [ADR-001](../adr/ADR-001-perfil-javascript-inicial-aura.md) y [ADR-002](../adr/ADR-002-runtime-independiente-openai.md).

## Problema y resultado esperado

Construir un agent harness de ingeniería de software independiente, centrado en la CLI y con macOS como primera plataforma. Aura, no Codex, controla su ciclo de conversación, contexto, despacho de herramientas, permisos, presupuestos y sesiones por proyecto. OpenAI será el **único proveedor inicial de modelos**. Se prefiere aprovechar la suscripción ChatGPT/Codex de Jas sin recurrir a la API de OpenAI facturada por separado, siempre que se confirme una integración directa permitida y estable.

Una v0.1.0 satisfactoria abre un repositorio local, conversa con un modelo OpenAI autorizado, lee archivos y propone cambios, ejecuta herramientas aprobadas dentro de los límites comprobados del sandbox, realiza validaciones focalizadas, persiste una sesión aislada e informa con veracidad el uso observado.

## Alcance aprobado

- Runtime: JavaScript ESM + JSDoc, **Node.js 24 como mínimo**, perfil inicial para proyectos JavaScript.
- Plataforma: macOS primero; otros sistemas operativos vendrán después.
- Arquitectura: **Agent Runtime y Tool Executor propios de Aura**. Los primeros Agent Router y Model Router serán deliberadamente simples: un agente directo y un modelo OpenAI configurado.
- Objetivo de autenticación: acceso directo mediante suscripción ChatGPT/Codex **solo si** se confirma que su vía OAuth y de solicitudes al modelo para terceros está autorizada, tiene soporte suficiente para distribuirla y puede probarse. Un inicio de sesión de identidad exitoso no demuestra por sí solo el derecho a usar el modelo.
- Configuración: YAML validado para usuario y proyecto; historial de sesiones JSONL separado por repositorio.
- Seguridad: aprobaciones explícitas por operación, límites de procesos y sistema de archivos impuestos por el sandbox, controles de cancelación y tiempo, protección de secretos y presupuestos de tarea configurables.
- Uso: mostrar consumo medido y límites de uso cuando el proveedor los exponga; identificar correctamente estimaciones y valores desconocidos. Nunca sustituir silenciosamente la suscripción por créditos de API de pago.

## Fuera del alcance de esta entrega

Codex App Server y Codex SDK **no** serán el runtime inicial de Aura: ambos ejecutan o controlan el agente de Codex. Permanecen como referencias de investigación, sin ser rutas automáticas de respaldo. Este hito tampoco incluirá Anthropic, facturación mediante API key, importación de skills externas, ejecución multiagente, workflows avanzados, memoria automática de largo plazo, TUI elaborada ni soporte para varios sistemas operativos.

## Dependencia y riesgo central

Las integraciones comunitarias de OpenCode demuestran una ruta técnica basada en OAuth hacia el backend de suscripción de Codex. Sirven como evidencia de factibilidad técnica, **pero no confirman** que OpenAI ofrezca una API de modelos sin procesar, documentada y estable, para un harness ajeno. La fase 0 debe verificar alcance permitido, soporte del protocolo, acceso a modelos, comportamiento de cuotas, almacenamiento de tokens y manejo seguro de fallos. No copiar credenciales privadas de otra aplicación ni interpretar la autenticación de identidad genérica de «Sign in with ChatGPT» como permiso para hacer inferencia.

**Si no existe inferencia directa y autorizada mediante suscripción:** desarrollar el runtime independiente y sus pruebas con un proveedor mock, documentar el bloqueo y no anunciar una versión funcional respaldada por suscripción. La delegación en App Server y la facturación por API tampoco son sustitutos implícitos.

## Criterios de aceptación para el lanzamiento

Un ciclo real y autorizado del modelo mediante el loop propio de Aura; ejecución de herramientas controlada íntegramente por Aura; rechazos verificados del sandbox y las aprobaciones; límites de tiempo y cancelación; validación de YAML; recuperación de sesiones JSONL aisladas; y un escenario de edición y prueba focalizada en macOS. La integración no se considera implementada hasta superar estas comprobaciones.
