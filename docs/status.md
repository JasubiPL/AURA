# Estado del proyecto

Actualizado: 2026-09-29 (America/Mexico_City). La documentación inicial está integrada en `feature/mvp-v0.1.0` mediante PR #4; esta rama propone refinamientos desde el análisis de OpenCode. El código y las pruebas deben verificar cualquier afirmación de implementación.

## Implementado

- Repositorio público, README, retrato, AGENTS.md, CLAUDE.md y flujo de ramas/PR.
- ADR-001 y ADR-002 con dirección arquitectónica; documentación SDD inicial del MVP (brief, requisitos, diseño preliminar y tareas pendientes).
- **No existe todavía una CLI funcional, Agent Runtime independiente, integración OAuth ni sandbox validado.**

## Revisión documental de esta rama

- Análisis original de OpenCode conservado y revisión con procedencia, límites y evaluación de preparación para continuar.
- Referencias obsoletas de runtime delegado en ADR-001 alineadas con la decisión aprobada de ADR-002.
- ADR-002 actualizado con OAuth confirmado por Jas y una candidata oficial documentada para investigar el acceso por suscripción.
- ADR-003 **propuesto** para contratos de streaming, herramientas, permisos, edición y recuperación. Specs existentes refinadas, tareas/dependencias y matriz de validación planificada; ninguna prueba de implementación ejecutada.

## Proceso de desarrollo

- SDD desde el MVP: brief → requisitos → diseño → tareas → implementación → validación. El diseño sigue sujeto al spike y las tareas no representan código terminado. Para cambios pequeños, documentación proporcional sin flujos paralelos.

## Aprobado para v0.1.0

- **Opción B:** Aura construye y controla el Agent Runtime completo, Tool Executor, contexto, sesiones, presupuestos y permisos. No delegar su ciclo a Codex App Server.
- **Solo OpenAI y login OAuth:** OAuth confirmado por Jas el 2026-09-29. Prioridad a suscripción ChatGPT/Codex, sujeta a comprobar elegibilidad de Aura, autorización y estabilidad de la inferencia directa. No incluir Anthropic ni modo API pagado; no hacer fallback silencioso.
- macOS primero, JavaScript ESM con JSDoc, **Node.js 24 mínimo y target de desarrollo/distribución**.
- YAML validado para usuario/proyecto, sesiones JSONL aisladas por repositorio y presupuestos configurables.
- Mantener interfaces simples de Agent Router y Model Router; capacidades avanzadas, memoria automática y TUI se posponen.

## Bloqueo de factibilidad

La revisión identifica una candidata oficial de uso del plan de ChatGPT en aplicaciones abiertas/locales; fuentes en ADR-002. OAuth está elegido, pero falta demostrar registro para Aura, elegibilidad/condiciones de distribución, scopes de inferencia, turno real, herramientas, renovación y errores. Un login exitoso no basta. Si no existe una vía aceptable para Aura, continuar con mock y bloquear el lanzamiento respaldado por suscripción; no sustituirlo por App Server ni API facturada.

El sandbox propio de macOS también carece de prototipo y evidencia. OpenCode no aporta aislamiento de seguridad; T-02 debe comprobarlo por separado. Los contratos de YAML/JSONL, locks, límites y recuperación requieren T-03 antes de sus módulos de producción.

## Propuestas técnicas sin aprobar

- npm con lockfile, `node:test` y E2E de macOS.
- Licencia MIT.
- Formatos finales y precedencia de permisos, defaults numéricos de presupuesto y compatibilidad de modelos.
- Contratos de ADR-003: ejecución secuencial tras terminar el turno, registro validado, edición condicionada, identidad local por workspace, escritor único y efectos interrumpidos sin reejecución automática.

## Preparación para continuar

**Sí:** refinar las specs existentes y revisar el contrato mínimo para T-00 con mock. El diseño propuesto incluye fixture, resultados esperados y variantes de fallo; no requiere OAuth ni shell.

**Todavía no:** declarar cerrado el diseño completo o lista la implementación/lanzamiento del MVP. Cada módulo debe cumplir sus dependencias en `tasks.md`; integración real y shell requieren T-01/T-02, y estado local requiere T-03. ADR-003 sigue para revisión, sin marcarlo aprobado por la existencia del informe.

## Próximos pasos

1. Revisar propuestas de ADR-003 y preparar/ejecutar T-00, primer spike mínimo de turno/herramienta con mock; registrar resultados reales cuando existan.
2. Investigar T-01 (OAuth/inferencia) y T-02 (sandbox) por separado; ejecutar T-03 y cerrar contratos mediante T-04. Actualizar diseño y matriz de validación con evidencia, no con supuestos.
3. Implementar por PR pequeñas desde `feature/mvp-v0.1.0`; validar de forma proporcional.
4. Integrar el MVP a `main` y etiquetar `v0.1.0` solo si se supera la aceptación real.

## Decisiones y especificaciones

La documentación y las especificaciones se redactan en español. Los nombres de ramas y los mensajes de commit permanecen en inglés.

- [ADR-001](adr/ADR-001-perfil-javascript-inicial-aura.md)
- [ADR-002: runtime propio y MVP solo con OpenAI](adr/ADR-002-runtime-independiente-openai.md)
- [ADR-003: contratos mínimos propuestos](adr/ADR-003-contratos-turnos-herramientas-sesiones.md)
- [Revisión de integración de OpenCode](research/opencode-integration-review.md)
- [Brief del MVP](mvp/brief.md)
- [Requisitos](mvp/requirements.md)
- [Diseño inicial](mvp/design.md)
- [Tareas](mvp/tasks.md)
- [Plan de implementación](mvp/implementation-plan.md)
- [Matriz de validación pendiente](mvp/validation.md)

Eliminar la rama de origen de cada PR tras un merge verificado, siempre que no exista trabajo o PR dependiente.
