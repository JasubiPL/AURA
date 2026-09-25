# Estado del proyecto

Actualizado: 2026-09-25. Documentación propuesta en PR; el código y las pruebas deben verificar cualquier afirmación de implementación.

## Implementado

- Repositorio público, README, retrato, AGENTS.md, CLAUDE.md y flujo de ramas/PR.
- ADR-001 y ADR-002 con dirección arquitectónica; documentación SDD inicial del MVP (brief, requisitos, diseño preliminar y tareas pendientes).
- **No existe todavía una CLI funcional, Agent Runtime independiente, integración OAuth ni sandbox validado.**

## Proceso de desarrollo

- SDD desde el MVP: brief → requisitos → diseño → tareas → implementación → validación. El diseño sigue sujeto al spike y las tareas no representan código terminado. Para cambios pequeños, documentación proporcional sin flujos paralelos.

## Aprobado para v0.1.0

- **Opción B:** Aura construye y controla el Agent Runtime completo, Tool Executor, contexto, sesiones, presupuestos y permisos. No delegar su ciclo a Codex App Server.
- **Solo OpenAI:** prioridad a la suscripción ChatGPT/Codex, sujeta a verificar autorización y estabilidad del acceso OAuth/modelo directo. No incluir Anthropic ni modo API pagado en el MVP; no hacer fallback silencioso.
- macOS primero, JavaScript ESM con JSDoc, **Node.js 24 mínimo y target de desarrollo/distribución**.
- YAML validado para usuario/proyecto, sesiones JSONL aisladas por repositorio y presupuestos configurables.
- Mantener interfaces simples de Agent Router y Model Router; capacidades avanzadas, memoria automática y TUI se posponen.

## Bloqueo de factibilidad

La autenticación de identidad mediante ChatGPT no demuestra por sí sola derecho a invocaciones directas al modelo con cuota de suscripción. Existen implementaciones comunitarias como OpenCode, pero antes de adoptar un protocolo necesitamos verificar términos, estabilidad y uso autorizado. Si no existe una vía aceptable, continuar el runtime propio con un mock y bloquear el lanzamiento que prometa acceso por suscripción. No sustituirlo por App Server ni facturación API sin nueva decisión.

## Propuestas técnicas sin aprobar

- npm con lockfile, `node:test` y E2E de macOS.
- Licencia MIT.
- Formatos finales y precedencia de permisos, defaults numéricos de presupuesto y compatibilidad de modelos.

## Próximos pasos

1. Spike: acceso OpenAI directo permitido + sandbox propio macOS + prueba de tool-calling mediante mock.
2. Cerrar el diseño técnico, los esquemas YAML y la matriz de validación con evidencia.
3. Implementar por PR pequeñas desde `feature/mvp-v0.1.0`; validar de forma proporcional.
4. Integrar el MVP a `main` y etiquetar `v0.1.0` solo si se supera la aceptación real.

## Decisiones y especificaciones

La documentación y las especificaciones se redactan en español. Los nombres de ramas y los mensajes de commit permanecen en inglés.

- [ADR-001](adr/ADR-001-perfil-javascript-inicial-aura.md)
- [ADR-002: runtime propio y MVP solo con OpenAI](adr/ADR-002-runtime-independiente-openai.md)
- [Brief del MVP](mvp/brief.md)
- [Requisitos](mvp/requirements.md)
- [Diseño inicial](mvp/design.md)
- [Tareas](mvp/tasks.md)
- [Plan de implementación](mvp/implementation-plan.md)

Eliminar la rama de origen de cada PR tras un merge verificado, siempre que no exista trabajo o PR dependiente.
