# OpenCode → Aura: revisión para integrar los hallazgos

**Fecha:** 2026-09-29 (America/Mexico_City).  
**Base revisada:** `feature/mvp-v0.1.0`, commit `0fed26d460954579bdc8cb6d7ce80b2fe4a9fd31`.  
**Estado:** análisis documental y propuestas; no es evidencia de ejecución del MVP.

## Conclusión

Aura puede continuar refinando sus specs y preparar el spike mínimo con mock. Todavía no puede declarar cerrado el diseño completo, iniciar todas las tareas sin condiciones ni comprometer un lanzamiento funcional. Faltan resultados de autenticación, aislamiento macOS y persistencia; los contratos nuevos de [ADR-003](../adr/ADR-003-contratos-turnos-herramientas-sesiones.md) se presentan para revisión.

El aprendizaje de OpenCode encaja con ADR-002: construir el ciclo de Aura y mantener el proveedor separado de las herramientas. No hay motivo demostrado para importar su núcleo, cambiar JSONL por su almacenamiento o añadir su infraestructura de extensiones al MVP.

## Procedencia y límites

- Se conserva sin modificaciones el [análisis entregado por Jas](aura-opencode-analysis.md), con SHA-256 `9e187d13eb7b01696c310cb0d59cc6838419a3a4b23782c5fff703daa1c6fd4e`, comprobado contra el archivo original. Su fecha original es 2026-09-30; esta revisión usa la fecha local de Jas. El documento es una entrada de investigación, no una concesión de permisos ni una decisión aprobada por sí mismo.
- El análisis identifica OpenCode en el commit [`2fa3363c924c5c3e367b84a87ae478296a0ed59b`](https://github.com/anomalyco/opencode/commit/2fa3363c924c5c3e367b84a87ae478296a0ed59b). Esta revisión comprueba fragmentos de su runner V2, registro de herramientas, mutaciones, coordinación de sesiones, identidad de proyecto, almacenamiento de salidas, permisos, plugin OpenAI, manifiesto del núcleo, `SECURITY.md` y `LICENSE`.
- No se ejecutaron OpenCode, sus pruebas, OAuth ni un sandbox. No se verificó aquí la equivalencia con la release ni que esa release siga siendo la última. Las referencias V1 y V2 deben conservar su procedencia; sus políticas no se combinan como una implementación única.
- La [PR #4](https://github.com/JasubiPL/AURA/pull/4) ya está integrada en la rama MVP. Su commit de contenido coincide con el citado por el análisis. Las specs existen; el documento original mantiene la descripción histórica de esa PR como abierta.

## Hallazgos y destino documental

| Hallazgo | Evaluación para Aura | Destino |
| --- | --- | --- |
| Turno del proveedor separado de herramientas y continuación | Compatible con el runtime propio aprobado; falta concretar eventos, IDs y terminación. | ADR-003; R-02; diseño; T-00/T-20/T-22; V-02. |
| Registro de herramientas conocidas y rechazo de llamadas obsoletas | Adoptar contrato pequeño y validación central; filtrar el catálogo no concede autoridad. | ADR-003; R-02/R-08; T-22; V-02/V-08. |
| Edición con contenido esperado y comparación de bytes | Evita sobrescrituras detectables; un lock interno no garantiza transacciones con editores externos. | ADR-003; R-06; T-40; V-06. |
| Permisos V1/V2 | Reusar casos de validación, no su precedencia. Proponer intersección de restricciones y `deny` efectivo antes de aprobaciones. | ADR-003; R-08/R-09; T-11/T-40; V-08/V-09. |
| Shell con timeout y salida limitada | Referencia de contrato; no resuelve el aislamiento requerido por Aura. | ADR-001/002; R-07; T-02/T-41; V-07. |
| Sesiones y herramientas interrumpidas | Mantener JSONL; definir identidad local, escritor exclusivo y efectos inciertos sin reejecución automática. | ADR-003; R-10; T-03/T-12; V-10. |
| Salidas grandes | Acotar durante captura; artefactos locales también necesitan límites, aislamiento y protección de secretos. | ADR-003; R-07/R-11; T-23; V-11. |
| Coste OAuth a cero y títulos generados | No deducir cuota ilimitada; proponer títulos locales sin llamadas auxiliares. | R-11; diseño; T-21/T-31; V-11. |
| Skills, MCP, especialistas y compactación | Pospuestos conforme al alcance aprobado. Conservar fronteras, sin implementar extensiones ahora. | ADR-002/003; diseño. |
| Reutilización literal | Evaluar utilidades por separado, con licencia y atribuciones; ninguna copia de código en este cambio. | ADR-003; T-04. |

El [runner V2](https://github.com/anomalyco/opencode/blob/2fa3363c924c5c3e367b84a87ae478296a0ed59b/packages/core/src/session/runner/llm.ts) deja las herramientas y la continuación fuera de `llm.stream`. El [registro](https://github.com/anomalyco/opencode/blob/2fa3363c924c5c3e367b84a87ae478296a0ed59b/packages/core/src/tool/registry.ts) comprueba la identidad anunciada. Son referencias útiles para diseñar contratos, no dependencias propuestas.

El [modelo de seguridad de OpenCode](https://github.com/anomalyco/opencode/blob/2fa3363c924c5c3e367b84a87ae478296a0ed59b/SECURITY.md) declara ausencia de sandbox. Su [identidad de proyecto V2](https://github.com/anomalyco/opencode/blob/2fa3363c924c5c3e367b84a87ae478296a0ed59b/packages/core/src/project.ts) puede agrupar clones por remoto y usa un ID global sin Git. Ninguna política satisface por sí sola el aislamiento local que Aura necesita.

## Actualización de la investigación de OpenAI

La documentación oficial consultada describe ahora una vía de uso del plan de ChatGPT para aplicaciones abiertas y locales: [Overview](https://developers.openai.com/siwc/token-sharing-open-source). Es una candidata coherente con ADR-002, no prueba de que Aura o la cuenta de Jas tengan ya acceso.

El [registro e inicio de sesión](https://developers.openai.com/siwc/token-sharing-open-source/sign-in) documenta registro dinámico, identidad de host y consentimiento. La [inferencia](https://developers.openai.com/siwc/token-sharing-open-source/models-and-inference) utiliza Responses con OAuth. Las [limitaciones de preview](https://developers.openai.com/siwc/token-sharing-open-source/preview-limitations) deben verificarse al diseñar mensajes y herramientas. T-01 debe contrastar esta vía antes de investigar protocolos de backend de Codex.

El bloqueo pasa de «no hemos identificado una vía oficial» a «existe una candidata documentada; falta comprobar elegibilidad, condiciones de distribución y ejecución real». La licencia de Aura sigue pendiente: un repositorio público no demuestra por sí solo que cumpla las condiciones de una aplicación abierta. Esta decisión de distribución corresponde a Jas.

Jas confirmó durante esta revisión que el inicio de sesión debe usar OAuth. ADR-002 registra esa decisión como aprobada, separada de la factibilidad todavía pendiente del registro y de la inferencia por suscripción.

## ADR existentes y propuesta nueva

- **ADR-001:** corregir referencias vigentes al runtime delegado y al sandbox proporcionado por Codex, ya sustituidas por ADR-002. Mantener la arquitectura futura diferenciada del MVP.
- **ADR-002:** actualizar las fuentes y la candidata oficial de autenticación. Preservar runtime propio, proveedor único, ausencia de fallback facturado y condiciones de lanzamiento.
- **ADR-003 propuesto:** agrupar contratos de turnos, herramientas, permisos y sesiones que se necesitan entre sí. No crear un ADR por utilidad; elegir el mecanismo de sandbox después de T-02 y documentarlo entonces.

## Preparación para continuar

| Etapa | Estado actual | Condición para avanzar |
| --- | --- | --- |
| Brief y requisitos | Existen; refinamientos trazados en esta rama. | Revisar las propuestas de ADR-003 sin ampliar el MVP. |
| Diseño del spike mock | Suficiente como propuesta para preparar T-00. | Revisar el contrato de turno/herramienta; un fixture, sin shell ni credenciales. |
| Diseño completo | Condicionado. | T-01/T-02/T-03 y cierre de esquemas, identidad, límites y permisos. |
| Plan y tareas | Existen; dependencias y validación detalladas. | Ejecutar por módulos con su puerta de entrada, sin esperar tareas independientes. |
| Adaptador real | Pendiente. | Elegibilidad, OAuth, inferencia, tool calls y renovación comprobados en T-01. |
| Shell | Pendiente. | Enforcement propio demostrado en T-02; denegar si falta un control requerido. |
| Lanzamiento | Bloqueado por ausencia de implementación y evidencia. | Completar R-01 a R-12 y la matriz de [validación](../mvp/validation.md). |

El siguiente incremento recomendado es T-00: un proveedor mock emite una llamada `read_file` para un fixture; Aura valida, decide, registra y devuelve su resultado; el proveedor finaliza. Después se registran sus resultados reales en `spike-results.md`. Este análisis no crea ese archivo de resultados ni marca tareas ejecutadas.
