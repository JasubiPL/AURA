# Aura v0.1.0 — Matriz de validación

**Fecha:** 2026-09-29.  
**Estado:** Plan de validación; todos los casos de implementación están pendientes.  
**Entradas:** [Requisitos](requirements.md), [Diseño](design.md), [Tareas](tasks.md) y propuesta de [ADR-003](../adr/ADR-003-contratos-turnos-herramientas-sesiones.md).

Esta matriz planifica la validación del SDD existente antes del código y registra evidencia cuando se ejecute. Define qué demostrar; no afirma que haya pruebas ni capacidades ejecutadas. Según la [guía SDD](../development/sdd.md), cada incremento se contrasta con requisito, diseño, tarea y evidencia antes de cerrarse. Los refinamientos de ADR-003 requieren revisión antes de convertirse en contratos definitivos. Las suites y herramientas de prueba se elegirán en T-04; `node:test` sigue como propuesta.

## Casos por requisito

| ID | Requisito y tareas | Casos mínimos y resultado esperado | Entorno / evidencia | Estado |
| --- | --- | --- | --- | --- |
| V-01 | R-01; T-03/T-10/T-42 | Instalar e iniciar con Node.js 24 en Mac, elegir raíz válida y rechazar raíz/entorno inválidos con error claro. | macOS real; versión, comando, salida y commit. | Pendiente |
| V-02 | R-02; T-00/T-20/T-22 | Mock → llamada a fixture → resultado correlacionado → respuesta. Nombre desconocido, registro obsoleto, ID duplicado, entrada inválida y stream incompleto no ejecutan. Salida inválida tras efecto se registra sin reintento. | Casos deterministas; eventos, contadores de efectos y resultado. | Pendiente |
| V-03 | R-03; T-01/T-20/T-30 | Solo OpenAI/modelo configurado en producción; proveedor o modelo no admitido falla. Mock solo para desarrollo/pruebas, sin fingir conexión real. | Configuración y selección simuladas; catálogo real para integración. | Pendiente |
| V-04 | R-04; T-01/T-30 | Elegibilidad/condiciones de Aura, registro propio y consentimiento. Turno real y tool call local completos por la vía admitida. | Fuentes oficiales y prueba en vivo con cuenta autorizada; credenciales fuera de evidencia. | Pendiente |
| V-05 | R-05; T-01/T-30/T-31 | Callback/state/nonce/PKCE/scopes inválidos, vencimiento, renovación concurrente, logout y cuota agotada se manejan claramente. Ningún caso lee tokens de otro cliente ni activa API key/fallback facturado. | Fixtures de errores y autenticación real; no agotar deliberadamente la cuenta para probar un error. | Pendiente |
| V-06 | R-06; T-02/T-40 | Lectura/búsqueda permitidas y denegadas; traversal, rutas protegidas y symlinks/cambio de objetivo rechazados. Edición obsoleta/ambigua no escribe; creación no sobrescribe; BOM/finales de línea conservados; lote parcial visible. | Workspace desechable y proceso externo que cambia fixture; aislamiento en macOS real. | Pendiente |
| V-07 | R-07; T-02/T-41 | Comando permitido, acceso fuera de raíz, red denegada/permitida, timeout/cancelación con descendientes y salida continua. Falta de mecanismo obligatorio rechaza ejecución; captura permanece acotada. | macOS real; evidencia de enforcement y de ausencia de efectos/descendientes después de cancelar. | Pendiente |
| V-08 | R-08; T-02/T-11/T-22/T-40/T-41 | Aprobación una vez/sesión/regla, caducidad al cerrar, inspección y revocación. Restricción efectiva no se supera con aprobación; cambio mientras se espera se revalida. Rechazo termina sin otra herramienta equivalente. Red/publicación sensible requiere autorización específica. | Fixtures de políticas y E2E de interacción; decisiones y efectos observados. | Pendiente |
| V-09 | R-09; T-03/T-11 | YAML malformado, esquema/versión desconocidos, valores inválidos, precedencia ordinaria y conflicto de seguridad. Fallo seguro sin ejecutar; configuración/reglas no exponen ni persisten secretos. | Fixtures con datos ficticios; configuración efectiva y diagnóstico. | Pendiente |
| V-10 | R-10; T-03/T-12 | Reinicio, clones del mismo remoto, worktrees, no-Git, subdirectorios y alias simbólicos; reemplazar repo en la misma ruta no recupera historial ajeno. Segunda CLI no escribe en sesión activa; lock incierto no se elimina. Registro final truncado conserva prefijo, JSON válido sin salto final se conserva y corrupción intermedia bloquea. Interrumpir tras el efecto y antes del resultado no lo repite. | Al menos dos procesos y fallos inyectados; identidades/eventos y contador de efectos. | Pendiente |
| V-11 | R-11; T-00/T-03/T-21/T-23/T-31 | Agotar por separado cada límite, incluido tiempo esperando aprobación. Captura de streams/argumentos y disco limitada; lectura por fragmentos y artefacto ajeno denegado. Uso reportado/estimado/desconocido y cuota ausente se distinguen; título sin llamadas extra. | Mock, medición de captura y almacenamiento; integración real para campos expuestos. | Pendiente |
| V-12 | R-12; T-21/T-40/T-42 | Explicar comentario con lectura focalizada y cero builds/delegaciones. En otra tarea, editar fixture de código y ejecutar su validación proporcional con permisos; resumen coincide con evidencia. | E2E en repositorio de prueba macOS; herramientas/eventos y resultado de la prueba. | Pendiente |

## Evidencia por etapa

- **Investigación documental:** puede confirmar fuentes y eliminar contradicciones; no marca V-04/V-07 ni T-01/T-02 como completados.
- **Spike con mock:** confirma contratos del núcleo y casos deterministas, sin demostrar transporte real, acceso por suscripción ni aislamiento del sistema operativo.
- **Pruebas del módulo:** verifican sus casos y fallos; actualizar requisito/diseño si el resultado contradice el contrato. No repetir toda la suite por un cambio editorial.
- **Integración y E2E reales:** indispensables para autenticación, instalación y sandbox. Los fallos del proveedor pueden simularse; etiquetar esa evidencia para no confundirla con cuota observada en vivo.

Al ejecutar la fase 0, crear `spike-results.md` con commit, versiones macOS/Node, configuración sin secretos, fuente oficial consultada, comando/pasos, resultado esperado/observado, limitaciones y decisión. Distinguir cada investigación de su prototipo; registrar bloqueos de autenticación, sandbox y estado por separado.

Para aceptación de un módulo o release, enlazar pruebas/comandos y resultados al commit validado desde la PR o esta matriz. Mantener permisos temporales, fixtures y artefactos sensibles fuera de Git. Un mensaje del modelo o un test de OpenCode no constituye evidencia de Aura.

## Puertas de avance

1. **Spike mínimo:** T-00 requiere revisión de su contrato propuesto; puede usar fixture y registro de prueba sin cerrar OAuth ni sandbox de shell.
2. **CLI y núcleo neutral:** avanzar solo con el diseño suficiente para la tarea; contratos de YAML/JSONL dependen de T-03 y decisiones de T-04. Los bloqueos externos no impiden trabajo independiente con mock.
3. **Integración real y shell:** T-30 depende de T-01; T-41 depende de T-02. Una puerta cerrada bloquea la herramienta/adaptador afectado, sin activar alternativas implícitas.
4. **Lanzamiento:** R-01 a R-12 satisfechos con evidencia del commit final y E2E en macOS. Cualquier exclusión de un criterio obligatorio necesita una nueva decisión de alcance; no basta una casilla marcada ni una autenticación aislada.

La validación documental de esta rama se registra en la PR o el resumen del cambio. No debe añadirse a esta matriz como si validara runtime, autenticación o sandbox.
