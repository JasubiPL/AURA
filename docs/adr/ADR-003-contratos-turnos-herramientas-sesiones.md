# ADR-003 · Contratos mínimos de turnos, herramientas y sesiones

**Fecha:** 2026-09-29.  
**Estado:** Propuesta técnica para revisión; no sustituye decisiones aprobadas ni acredita implementación.  
**Complementa:** [ADR-002](ADR-002-runtime-independiente-openai.md).  
**Origen:** [Revisión de OpenCode](../research/opencode-integration-review.md).

## Contexto

Aura ya ha elegido runtime propio, OpenAI, macOS, JavaScript ESM, YAML y JSONL. El análisis de OpenCode aporta patrones de contratos, pero su núcleo privado, dependencias y políticas no constituyen una implementación reutilizable sin un acoplamiento considerable. Falta definir cómo correlacionar turnos y efectos, recuperar una sesión y limitar resultados sin ampliar el MVP.

## Propuesta

1. **Turnos y ejecución secuencial.** Un adaptador recibe contexto acotado, modelo, definiciones de herramientas y señal de cancelación; emite texto, llamadas estructuradas, uso y terminación normalizados. No recibe ejecutores ni concede autoridad. Aura espera la terminación satisfactoria del turno y los argumentos completos antes de producir efectos; ejecuta las herramientas secuencialmente. Un stream incompleto o fallido nunca se convierte en una respuesta final exitosa.
2. **Registro pequeño.** Cada herramienta define nombre, versión de contrato y esquemas de entrada/salida. La definición anunciada en el turno debe seguir vigente al ejecutar. Aura correlaciona `turnId`/`callId`, rechaza IDs duplicados, nombres desconocidos, definiciones obsoletas y datos inválidos. Validar la salida no deshace un efecto ya producido: registrar ese fallo sin repetirlo.
3. **Autoridad central.** Evaluar las restricciones del runtime, usuario y proyecto por intersección. Una denegación efectiva prevalece; las aprobaciones solo satisfacen una solicitud permitida dentro de ese techo. Vincular concesiones a operación, objetivo canónico y alcance, además de comando/argumentos o destino cuando corresponda. Caducar las concesiones de sesión al terminar la ejecución de la CLI, aunque se reabra el historial. Listar y revocar reglas persistentes. Revalidar después de esperar una aprobación y antes del efecto. El rechazo del usuario termina la ejecución actual; el modelo no busca otra herramienta para repetirlo.
4. **Edición condicionada.** La lectura devuelve una referencia al contenido esperado. La edición requiere esa referencia y coincidencia inequívoca, muestra un diff y revalida rutas y contenido al escribir. Crear archivos con exclusión de sobrescritura; conservar BOM y finales de línea existentes. Un archivo cambiado o un matching ambiguo produce rechazo, sin una búsqueda aproximada silenciosa. El lock de Aura coordina solo escritores propios; no se promete una transacción con otros editores ni atomicidad de cambios en varios archivos. Si un lote falla parcialmente, informar qué archivos cambiaron y no continuar automáticamente.
5. **Identidad local.** Proponer `repo-id` versionado derivado de SHA-256 de la raíz canónica del workspace local. En Git, usar la raíz del checkout/worktree; sin Git, una raíz explícita elegida por el usuario. Subdirectorios del mismo checkout comparten identidad; clones y worktrees distintos quedan separados aunque compartan remoto o `.git` común. Una ruta simbólica a la misma raíz resuelve la misma identidad. Mover el workspace crea identidad nueva; no importar sesiones automáticamente. Esta política privilegia aislamiento sobre continuidad tras mover carpetas y se confirma en T-03.
6. **Eventos y escritor exclusivo.** JSONL versionado registra `repoId`, `sessionId`, `runId`, secuencia, tipo de evento y correlación de turnos/llamadas. Un lock exclusivo por sesión debe cubrir procesos distintos de Aura, no solo tareas dentro de un proceso. El segundo escritor falla de forma clara; no elimina automáticamente un lock cuya propiedad sea incierta. Comprobar la política de locks abandonados en T-03. Una última línea incompleta puede descartarse conservando evidencia; corrupción intermedia o versión desconocida bloquean la reanudación segura.
7. **Efectos interrumpidos.** Persistir intención de ejecución antes del efecto y resultado después, con la política de durabilidad validada en T-03. Una llamada iniciada sin resultado queda `interrupted_unknown`: puede haber modificado un archivo o ejecutado un proceso. Reanudar reconstruye contexto y muestra la incertidumbre; no vuelve a ejecutar automáticamente la llamada ni repite aprobaciones de sesión antiguas. Ante error de persistencia, detener nuevos efectos. No prometer ejecución exactamente una vez frente a cualquier fallo del sistema.
8. **Límites desde la captura.** Lectura por rangos, búsqueda por ruta/patrón y captura de procesos limitan bytes, tiempo y tamaño de resultados mientras se producen. Si se guarda salida adicional, queda bajo la sesión y repo correspondientes, con límite de disco y lectura por fragmentos mediante un identificador opaco; el modelo no elige su ruta. Proteger también los artefactos de secretos y acceso entre repositorios. Si no hay capacidad de captura o almacenamiento seguro, detener la operación con estado explícito.
9. **Uso y trabajo auxiliar.** Contadores locales gobiernan turnos, tiempo, herramientas y correcciones. Mostrar uso reportado, estimado o desconocido sin inferir cuota ilimitada de un coste monetario cero. Derivar el título localmente del primer mensaje. Posponer compactación automática; cualquier llamada auxiliar futura se cobra al presupuesto de la misma tarea.
10. **Reutilización selectiva.** Diseñar los contratos en Aura; no depender de los paquetes privados del núcleo de OpenCode. Antes de adaptar código literalmente, registrar archivo/commit, revisar licencia y atribuciones por archivo y conservar los avisos exigidos. La licencia de Aura permanece pendiente. Skills externas, MCP, LSP, especialistas y workflows avanzados siguen pospuestos.

Los nombres de campos y estados orientan el diseño, pero el esquema completo y sus versiones se cierran mediante las specs y T-03. La identidad basada en ruta requiere además detectar su reutilización por otro repositorio; T-03 debe resolverlo antes de permitir continuidad automática de producción. Este ADR no elige mecanismo de sandbox, librería YAML, gestor de paquetes, licencia ni valores numéricos de presupuesto.

## Alternativas y consecuencias

- Importar el núcleo de OpenCode reduciría algunas piezas propias a costa de adoptar interfaces internas, almacenamiento y dependencias fuera del alcance aprobado. No se recomienda para este MVP.
- Ejecutar herramientas en paralelo reduce latencia, pero complica orden de efectos, aprobaciones y recuperación. La propuesta secuencial simplifica el primer contrato y permite reconsiderarlo con evidencia posterior.
- Agrupar clones por remoto facilita compartir conversaciones, pero entra en conflicto con el aislamiento local. Compartir o migrar sesiones necesita una acción explícita futura.

Esperar el cierre del turno añade latencia antes de ejecutar herramientas y evita actuar sobre argumentos parciales de un stream fallido. JSONL mantiene una implementación pequeña, pero exige pruebas de durabilidad, corrupción y exclusión de escritores. La comprobación de bytes detecta cambios observados; la protección de rutas y el aislamiento efectivo requieren validación independiente, especialmente ante carreras con procesos ajenos.

## Criterios para aceptar la propuesta

- Revisar estos contratos y reflejar su aprobación o ajustes en el estado de este ADR antes de tratarlos como decisiones definitivas.
- T-00 demuestra la frontera proveedor/herramientas con fixture y casos de rechazo, argumentos inválidos y cancelación.
- T-03 demuestra identidades de clones/worktrees/no-Git, escritor único entre procesos, recuperación de JSONL y ausencia de duplicación de efectos.
- T-02/T-40 demuestran rutas, edición obsoleta, permisos/revocación y límites; los casos de aislamiento de shell requieren macOS real.
- Mantener trazabilidad en [requisitos](../mvp/requirements.md), [diseño](../mvp/design.md), [tareas](../mvp/tasks.md) y [validación](../mvp/validation.md). El resultado de OpenCode no sustituye ninguna prueba de Aura.
