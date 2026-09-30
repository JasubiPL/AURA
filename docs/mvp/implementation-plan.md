# Aura v0.1.0 — Plan de implementación

**Estado:** Plan para la Opción B, con Agent Runtime independiente de Aura; la viabilidad del acceso por suscripción sigue siendo un bloqueo para el lanzamiento.  
**Fecha:** 2026-09-25  
**Rama de integración:** `feature/mvp-v0.1.0`, mediante ramas y PR pequeñas dirigidas a ella, no a `main`.  
**Trazabilidad SDD:** [Brief](brief.md) → [Requisitos](requirements.md) → [Diseño](design.md) → [Tareas](tasks.md) → implementación → validación. Este plan ordena las fases; las tareas contienen las casillas y evidencias.

## Fase 0 — Spike de factibilidad del runtime independiente y la suscripción

1. Construir un ciclo **mínimo y propio de Aura** de turno y herramienta con un proveedor mock de forma similar a OpenAI; establecer el contrato de solicitudes al modelo y a las herramientas sin dependencia de un agente Codex.
2. Investigar el acceso a inferencia mediante suscripción ChatGPT/Codex para clientes de modelos de terceros, con documentación y autenticación oficiales. Contrastar Codex SDK/App Server oficiales (que controlan agentes Codex) con la ruta comunitaria de OAuth directo de OpenCode; identificar condiciones de uso, autorización del cliente, endpoints, modelos disponibles, cuota y estabilidad del protocolo. No copiar credenciales de otro cliente ni suponer que un endpoint no documentado está permitido.
3. Si existe una vía directa autorizada, crear un prototipo de inicio de sesión, renovación de tokens, cierre de sesión, solicitudes estructuradas de herramientas, errores y comportamiento real de la cuota de suscripción **sin** API key. La autenticación de identidad genérica mediante Sign in with ChatGPT no basta por sí sola.
4. Experimentar con un **Tool Executor controlado por Aura** en macOS: rutas normalizadas y enlaces simbólicos, restricciones del sandbox, cancelación del árbol de procesos, límites de salida de comandos, aislamiento de red y aprobaciones por operación. Comprobar su aplicación real con recursos de prueba desechables.
5. Verificar los campos de uso y límites de frecuencia expuestos, el esquema YAML y la recuperación JSONL. Validar dependencias de Node.js 24 e instalación en una Mac de prueba.

Entregar `docs/mvp/spike-results.md` con versiones de plataforma, URLs de referencia, pasos de reproducción, evidencias, casos fallidos y bloqueos restantes. Nunca guardar tokens ni datos privados de cuenta en commits.

**Condición:** si no existe inferencia directa y autorizada mediante suscripción, continuar el runtime independiente y el proveedor mock, pero bloquear el lanzamiento de v0.1.0 respaldado por suscripción hasta que se apruebe explícitamente otra estrategia de acceso. No delegar silenciosamente en App Server ni pasar a la API de pago.

## Fase 1 — CLI, configuración y sesiones

Implementar la CLI interactiva `aura`, detección de la raíz del proyecto, YAML validado por esquema para usuario y proyecto y escritura/recuperación de sesiones JSONL aisladas por repositorio. Añadir configuración explícita del modelo y mensajes claros de diagnóstico.

**Condición:** el YAML inválido falla de forma segura; dos repositorios locales nunca mezclan historiales de sesión.

## Fase 2 — Agent Runtime independiente de Aura

Implementar el ciclo de turnos, contrato tipado del adaptador del proveedor, normalización de mensajes del modelo y herramientas, ruta directa sin delegación, selección de contexto, manejo de prompts y resultados de herramientas, cancelación y presupuestos observables. Añadir un proveedor mock determinista y recursos de prueba focalizados.

**Condición:** un modelo simulado solicita una herramienta; Aura decide por sí misma la autorización y ejecución, devuelve el resultado al modelo, completa una respuesta y se detiene al alcanzar los presupuestos configurados.

## Fase 3 — Adaptador directo a la suscripción OpenAI (si se autoriza)

Implementar la vía de acceso directo verificada, inicio y cierre de sesión visibles para el usuario, ciclo de vida seguro de tokens, errores del proveedor, contadores de uso disponibles e información de la cuota de suscripción. Nunca extraer archivos de tokens de Codex ni codificar una identidad de cliente OAuth comunitaria. No implementar Anthropic ni modo de API facturada.

**Condición:** un turno real y autorizado del modelo mediante suscripción funciona con el ciclo y las herramientas propias de Aura en macOS; de lo contrario, el lanzamiento permanece bloqueado con evidencia reproducible.

## Fase 4 — Ejecución segura de herramientas y aceptación de extremo a extremo

Construir herramientas propias de Aura para leer, buscar y editar archivos y ejecutar shell dentro de un sandbox en macOS; aplicar aprobaciones, raíces, política de red, límites de tiempo y salida, rechazo y revocación. Pruebas de extremo a extremo: solo explicación; cambio de código con validación proporcional; rechazo del sandbox o permisos; reanudación de sesiones aisladas; cuota no disponible; agotamiento manejado con claridad.

**Condición:** todos los requisitos aplicables de `requirements.md` pasan en una Mac de prueba real, incluido el adaptador de suscripción en vivo. Solo entonces abrir la PR de `feature/mvp-v0.1.0` a `main` y crear la etiqueta `v0.1.0`.

## Flujo de ramas

Crear ramas acotadas `feat/*`, `fix/*`, `docs/*` y `test/*` desde la versión más reciente de la rama del MVP y dirigir sus PR a `feature/mvp-v0.1.0`. Eliminar cada rama de origen después de comprobar su merge y sus PR dependientes. Eliminar la rama del MVP solo después de integrarla finalmente a la rama estable `main`.

## Versiones posteriores

Las decisiones futuras aprobadas podrán incorporar Anthropic, un adaptador independiente para la API facturada de OpenAI, otros sistemas operativos, más modelos, routers avanzados, workflows y agentes opcionales, capacidades externas, memoria avanzada e inspeccionable y una TUI elaborada. App Server no sustituye automáticamente los requisitos de un runtime independiente.
