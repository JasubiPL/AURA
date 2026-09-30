# Aura v0.1.0 — Diseño técnico inicial

**Estado:** Diseño preliminar sujeto a la evidencia de la fase 0; no describe componentes implementados.  
**Fecha:** 2026-09-25  
**Actualizado:** 2026-09-29  
**Entradas:** [Brief](brief.md), [Requisitos](requirements.md), [ADR-001](../adr/ADR-001-perfil-javascript-inicial-aura.md) y [ADR-002](../adr/ADR-002-runtime-independiente-openai.md).

Este documento es la etapa **Design** del SDD usado para desarrollar Aura, según la [guía compartida](../development/sdd.md). La aceptación y su plan de validación deben existir antes de implementar la tarea. No implica que Aura ya gestione SDD como producto; [ADR-004](../adr/ADR-004-sdd-desarrollo-y-flujo-predeterminado.md) aprueba ese default para Aura completa como capacidad posterior. Los refinamientos siguientes se proponen en [ADR-003](../adr/ADR-003-contratos-turnos-herramientas-sesiones.md); los contratos condicionados se cerrarán con resultados del spike antes de programar cada módulo. La [revisión de integración](../research/opencode-integration-review.md) conserva la procedencia.

## Recorrido de una tarea

1. La CLI determina la raíz autorizada del proyecto, carga y valida la configuración, selecciona una sesión y muestra el modelo configurado.
2. El Agent Runtime prepara contexto acotado y solicita un turno al adaptador del proveedor. El Model Router inicial resuelve el único modelo OpenAI configurado; el Agent Router inicial usa la ruta directa.
3. El adaptador emite texto, llamadas estructuradas, uso y terminación normalizados. Aura acota el stream, reúne argumentos y espera su terminación satisfactoria antes de ejecutar herramientas. El adaptador no toma decisiones de permisos.
4. Ante llamadas completas, el runtime consulta el registro vigente, límites y políticas. El Tool Executor valida argumentos, rutas, aprobación y sandbox, revalida tras esperas y ejecuta secuencialmente o rechaza. El resultado correlacionado vuelve al modelo en el siguiente turno; un rechazo del usuario termina la ejecución actual.
5. Aura registra los eventos necesarios de la sesión sin secretos, informa el uso realmente disponible y termina por respuesta, rechazo, cancelación, error o agotamiento de presupuesto.

## Límites entre componentes

| Componente | Responsabilidad inicial | Fuera de su responsabilidad |
| --- | --- | --- |
| CLI | Entrada/salida, selección de proyecto y sesión, diagnóstico y solicitud de aprobación. | Ejecutar herramientas o contener la política de seguridad. |
| Agent Runtime | Ciclo de turnos, contexto, cancelación, límites, estados y coordinación de solicitudes de herramientas. | Invocar endpoints o ejecutar shell directamente. |
| Adaptador de proveedor | Autenticación permitida y traducción de solicitudes, respuestas, errores y metadatos de uso. | Ejecutar herramientas, conceder permisos o activar facturación alternativa. |
| Agent Router y Model Router | Contratos separados y resolución directa de un agente y un modelo configurado. | Selección automática de especialistas o varios proveedores en v0.1.0. |
| Tool Executor | Validación y ejecución de operaciones aprobadas dentro de restricciones verificadas. | Aceptar instrucciones del modelo como autorización. |
| Registro de herramientas | Definiciones/esquemas vigentes y correlación con las anunciadas en cada turno. | Otorgar permisos por estar registrado. |
| Configuración y sesiones | Validación de YAML, identidad estable de repositorio, persistencia JSONL y recuperación. | Guardar credenciales o mezclar datos entre repositorios. |

Estas fronteras son una propuesta de diseño conforme al alcance aprobado; nombres de módulos, firmas y formato de eventos no están cerrados.

## Contratos mínimos propuestos

### Proveedor y herramientas

- Entrada del turno: mensajes/contexto acotados, modelo, definiciones/esquemas de herramientas, `turnId` y señal de cancelación. Salida normalizada: `text_delta`, `tool_call`, `usage` y `turn_end`. El esquema completo se verifica con T-00 y T-01; no se acopla el núcleo a un formato de OpenCode.
- Cada llamada incluye `callId`, nombre y argumentos completos, ligada a la definición anunciada. Acotar bytes acumulados también al reconstruir argumentos. Un nombre desconocido, definición obsoleta, ID duplicado o argumentos inválidos produce un error tipado sin ejecución. Stream fallido/incompleto o cancelado no habilita llamadas pendientes.
- Orden del Tool Executor: resolver definición → validar entrada → comprobar presupuesto/política y objetivo canónico → solicitar aprobación si procede → revalidar autoridad/objetivo → persistir inicio → ejecutar → validar/acotar salida → persistir resultado. Fallar al guardar el inicio impide efectos; fallar después exige detenerse con incertidumbre visible.
- Resultado: `callId`, estado y contenido acotado; estados candidatos `succeeded`, `denied`, `failed`, `cancelled`, `interrupted_unknown`. Un fallo al validar la salida no revierte ni repite el efecto. Errores de entrada pueden devolverse al modelo dentro del presupuesto; rechazo del usuario, cancelación o agotamiento terminan la ejecución actual.
- Estados terminales de la ejecución: respuesta completada, rechazo, cancelación, error o límite alcanzado. Solo la terminación satisfactoria del proveedor puede aportar una respuesta final exitosa; el texto parcial se identifica como incompleto.

### Archivos, salida y permisos

- `read_file` por rangos y `search_files` por ruta/patrón devuelven resultados acotados. Buscar también aplica rutas protegidas; usar ripgrep no elimina la necesidad de controlar qué puede recorrer el proceso.
- `edit_file` recibe referencia al contenido leído y cambio inequívoco; genera diff, compara contenido antes de escribir y preserva BOM/finales de línea. La creación usa exclusión de sobrescritura. Cambios de contenido o rutas mientras se espera una aprobación obligan a revalidar o rechazar. Los cambios parciales entre archivos se informan explícitamente; no hay promesa de transacción global.
- Cada operación se autoriza bajo la intersección runtime/usuario/proyecto. Una aprobación acotada satisface `ask` dentro de esa intersección; no supera un `deny`. Concesiones de sesión no se recuperan con el historial. El formato de reglas, su almacenamiento confiable y el mecanismo de revocación se cierran en T-03/T-04.
- Acotar lectura, stream y procesos durante captura, no solo al final. Artefactos opcionales bajo `~/.aura/sessions/<repo-id>/<session-id>/` tienen máximo de disco, limpieza/retención por definir y acceso por identificador opaco y fragmentos. No incluir secretos en previews, JSONL ni artefactos. El shell no recibe acceso a credenciales del proveedor ni al estado de otras sesiones.

### Configuración y recuperación

- YAML versionado de usuario/proyecto; precedencia ordinaria candidata CLI > proyecto > usuario > defaults. Seguridad se combina por intersección. Configuración inválida o conflicto no eleva autoridad. Autenticación fuera de YAML/Git; cerrar campos y valores en T-03.
- Propuesta de `repo-id`: hash SHA-256 versionado de raíz canónica del workspace. La raíz Git es la del checkout/worktree, sin usar remoto ni `.git` común como identidad; sin Git requiere raíz explícita. Clones y worktrees distintos se aíslan; un subdirectorio o alias simbólico del mismo checkout conserva identidad. Mover la raíz crea otra identidad, sin migración automática.
- JSONL versionado correlaciona `repoId`, `sessionId`, `runId`, secuencia, turnos y herramientas. Lock exclusivo entre procesos por sesión, liberado al cerrar normalmente. T-03 debe probar durabilidad, errores de disco y recuperación de locks abandonados sin desalojar escritores activos o inciertos.
- Recuperar el prefijo válido si el último registro está truncado y no es JSON válido; guardar diagnóstico. Un registro válido al final se conserva aunque no termine con salto de línea. Corrupción intermedia, versión no soportada o identidad discordante bloquean reanudación. Una llamada iniciada sin resultado queda `interrupted_unknown`; no se ejecuta automáticamente de nuevo. Definir migraciones, retención y detección de otro repositorio que reutiliza la misma ruta antes de producción; el hash de ruta por sí solo no resuelve ese reemplazo.

### Presupuesto y OpenAI

- Contadores locales de turnos, tiempo, herramientas y correcciones, además de timeout y límites de captura/artefactos. Cerrar valores numéricos con T-03; verificar cada límite por separado. Título local derivado del primer mensaje, sin llamadas auxiliares. Compactación automática pospuesta.
- Uso y cuota: reportado, estimado o desconocido según evidencia. No interpretar coste cero como cuota ilimitada. Llamadas auxiliares futuras pertenecen al presupuesto de la tarea.
- OAuth es el método de inicio de sesión aprobado. T-01 investiga la candidata oficial referenciada en ADR-002: registro para Aura, consentimiento/scopes, catálogo admitido, ciclo seguro de tokens y renovación coordinada sin exponerlos al modelo. El diseño del transporte real se cierra con sus restricciones de preview; el mock no acredita esa compatibilidad.

## Primer spike propuesto (T-00)

Un fixture local y una única definición `read_file`; sin shell, publicación ni autenticación. El proveedor mock emite una llamada completa y termina el turno, Aura la valida/autoriza, registra intención y resultado en un registro de prueba, y el siguiente turno finaliza con texto. El registro del spike no sustituye la persistencia duradera de producción de T-03/T-12.

Variantes deterministas: nombre desconocido, definición obsoleta, ID duplicado, argumentos inválidos, ruta fuera del fixture, rechazo del usuario, cancelación y stream interrumpido. Ninguna variante rechazada produce efectos; el caso permitido ejecuta una sola lectura. T-00 registra comandos y resultados reales en `spike-results.md` cuando se ejecute.

## Seguridad y fallos

La raíz autorizada y las rutas protegidas se comprueban antes de leer o editar. La validación debe contemplar enlaces simbólicos y cambios entre comprobación y uso. Todo comando pasa por un sandbox propio comprobado en macOS; si un control obligatorio no puede aplicarse, se rechaza la operación. Cancelación y timeout deben terminar el árbol de procesos y acotar la salida. Red y publicación requieren permisos específicos. Las credenciales se mantienen fuera de YAML, Git, prompts y registros.

Una autenticación inválida, cuota agotada, protocolo no admitido o dato de uso ausente produce un estado claro. Aura no cambia a otro proveedor, a un agente Codex delegado ni a una API facturada por tokens.

## Decisiones condicionadas por el spike

| Tema | Evidencia que falta | Efecto |
| --- | --- | --- |
| Acceso directo por suscripción | Elegibilidad/condiciones de la candidata oficial, protocolo, inferencia, tool calls, renovación y cuota. | Bloquea el adaptador real y el lanzamiento respaldado por suscripción si no se demuestra. |
| Sandbox de macOS | Aislamiento de archivos, procesos y red, symlinks, cancelación y aprobaciones reproducibles. | Bloquea la herramienta afectada o el lanzamiento si el control es obligatorio. |
| Persistencia/configuración | Esquemas, reglas/precedencia, identidad local, locks/durabilidad, recuperación y límites numéricos. | Impide declarar finalizados los contratos correspondientes. |

La evidencia y las decisiones resultantes se registrarán en `spike-results.md`. Cualquier cambio al alcance aprobado requiere actualizar requisitos y ADR; este diseño no autoriza alternativas de autenticación o facturación.
