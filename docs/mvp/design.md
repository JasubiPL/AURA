# Aura v0.1.0 — Diseño técnico inicial

**Estado:** Diseño preliminar sujeto a la evidencia de la fase 0; no describe componentes implementados.  
**Fecha:** 2026-09-25  
**Entradas:** [Brief](brief.md), [Requisitos](requirements.md), [ADR-001](../adr/ADR-001-perfil-javascript-inicial-aura.md) y [ADR-002](../adr/ADR-002-runtime-independiente-openai.md).

Este documento es la etapa **Design** del SDD usado para desarrollar Aura. No implica que el Workflow Engine de Aura ya implemente SDD. Los contratos concretos se cerrarán con los resultados del spike antes de programar cada módulo.

## Recorrido de una tarea

1. La CLI determina la raíz autorizada del proyecto, carga y valida la configuración, selecciona una sesión y muestra el modelo configurado.
2. El Agent Runtime prepara contexto acotado y solicita un turno al adaptador del proveedor. El Model Router inicial resuelve el único modelo OpenAI configurado; el Agent Router inicial usa la ruta directa.
3. El adaptador devuelve texto, solicitudes estructuradas de herramientas o un error normalizado. No ejecuta herramientas ni toma decisiones de permisos.
4. Ante una solicitud de herramienta, el runtime consulta límites y políticas. El Tool Executor valida argumentos, rutas, aprobación y sandbox; ejecuta la operación o devuelve un rechazo explícito. El runtime entrega el resultado al modelo en el siguiente turno.
5. Aura registra los eventos necesarios de la sesión sin secretos, informa el uso realmente disponible y termina por respuesta, rechazo, cancelación, error o agotamiento de presupuesto.

## Límites entre componentes

| Componente | Responsabilidad inicial | Fuera de su responsabilidad |
| --- | --- | --- |
| CLI | Entrada/salida, selección de proyecto y sesión, diagnóstico y solicitud de aprobación. | Ejecutar herramientas o contener la política de seguridad. |
| Agent Runtime | Ciclo de turnos, contexto, cancelación, límites, estados y coordinación de solicitudes de herramientas. | Invocar endpoints o ejecutar shell directamente. |
| Adaptador de proveedor | Autenticación permitida y traducción de solicitudes, respuestas, errores y metadatos de uso. | Ejecutar herramientas, conceder permisos o activar facturación alternativa. |
| Agent Router y Model Router | Contratos separados y resolución directa de un agente y un modelo configurado. | Selección automática de especialistas o varios proveedores en v0.1.0. |
| Tool Executor | Validación y ejecución de operaciones aprobadas dentro de restricciones verificadas. | Aceptar instrucciones del modelo como autorización. |
| Configuración y sesiones | Validación de YAML, identidad estable de repositorio, persistencia JSONL y recuperación. | Guardar credenciales o mezclar datos entre repositorios. |

Estas fronteras son una propuesta de diseño conforme al alcance aprobado; nombres de módulos, firmas y formato de eventos no están cerrados.

## Contratos y estados por resolver

- **Turno del proveedor:** entrada con mensajes, modelo y señal de cancelación; salida diferenciada entre respuesta, solicitudes estructuradas de herramientas y error. El proveedor mock permitirá fijar casos deterministas. El protocolo concreto de OpenAI queda condicionado por la fase 0.
- **Solicitud de herramienta:** identificador, nombre y argumentos validados; respuesta con resultado acotado o rechazo/error tipado. El Tool Executor comprueba permisos antes de efectos secundarios y el runtime impide reintentos encubiertos tras una denegación.
- **Configuración:** archivos `~/.aura/config.yaml` y `<repo>/.aura/config.yaml` con esquema versionado. La precedencia de opciones ordinarias y las reglas efectivas de seguridad se concretan tras el spike; ninguna configuración del proyecto amplía los permisos permitidos por el usuario o el runtime.
- **Sesión:** eventos JSONL versionados bajo `~/.aura/sessions/<repo-id>/`; recuperación tras reinicio, aislamiento por identidad de repositorio y manejo explícito de líneas incompletas. Definir identidad, migraciones y retención antes de implementar.
- **Presupuestos y observabilidad:** contadores locales de turnos, tiempo, herramientas y correcciones; duración y salida de comandos limitadas. Tokens, coste y cuota se muestran como medidos, estimados o desconocidos según evidencia del proveedor.

## Seguridad y fallos

La raíz autorizada y las rutas protegidas se comprueban antes de leer o editar. La validación debe contemplar enlaces simbólicos y cambios entre comprobación y uso. Todo comando pasa por un sandbox propio comprobado en macOS; si un control obligatorio no puede aplicarse, se rechaza la operación. Cancelación y timeout deben terminar el árbol de procesos y acotar la salida. Red y publicación requieren permisos específicos. Las credenciales se mantienen fuera de YAML, Git, prompts y registros.

Una autenticación inválida, cuota agotada, protocolo no admitido o dato de uso ausente produce un estado claro. Aura no cambia a otro proveedor, a un agente Codex delegado ni a una API facturada por tokens.

## Decisiones condicionadas por el spike

| Tema | Evidencia que falta | Efecto |
| --- | --- | --- |
| Acceso directo por suscripción | Autorización, protocolo estable, inferencia real, renovación y cuota. | Bloquea el adaptador real y el lanzamiento respaldado por suscripción si no se demuestra. |
| Sandbox de macOS | Aislamiento de archivos, procesos y red, symlinks, cancelación y aprobaciones reproducibles. | Bloquea la herramienta afectada o el lanzamiento si el control es obligatorio. |
| Persistencia/configuración | Esquemas, precedencia, identidad del repositorio, recuperación y valores de presupuesto. | Impide declarar finalizados los contratos correspondientes. |

La evidencia y las decisiones resultantes se registrarán en `spike-results.md`. Cualquier cambio al alcance aprobado requiere actualizar requisitos y ADR; este diseño no autoriza alternativas de autenticación o facturación.
