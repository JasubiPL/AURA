# Estado del proyecto

Actualizado: 2026-09-25. Este archivo resume el estado verificable; el código y los ADR son la referencia detallada.

## Implementado

- Repositorio público con README, retrato e instrucciones de trabajo para agentes.
- ADR-001 con dirección de arquitectura, límites y criterios de aceptación del MVP.
- Flujo de ramas y PR documentado. No hay todavía runtime ni CLI funcional.

## Decidido para el MVP

- Runtime inicial en JavaScript ESM con JSDoc; CLI con un ciclo básico de agente, herramientas de archivos y terminal con permisos y aislamiento verificables.
- OpenAI y Anthropic como proveedores iniciales objetivo mediante adaptadores separados. Autenticación y orden de integración por validar.
- Contratos separados para Agent Router y Model Router; selección simple al principio.
- Presupuestos por tarea, contexto selectivo y distinción entre capacidades propias, del usuario y externas.
- Memoria de sesión, personal y de proyecto separadas; no guardar secretos. Los detalles se concretarán en especificaciones.

## Pendiente

1. Elaborar brief y requisitos mínimos verificables del MVP, sin expandir la arquitectura objetivo a obligaciones de la primera entrega.
2. Spike acotado: probar Bubblewrap (Linux), Seatbelt (macOS) y WSL2 si corresponde; comprobar restricciones de rutas, red, procesos y timeout; validar mecanismos de autenticación de OpenAI y Anthropic, almacenamiento de aprobaciones y catálogo mínimo.
3. Con los resultados, definir `design`, `tasks` y `validation` del MVP; concretar configuración, manifiestos, permisos y presupuestos.
4. Implementar por ramas pequeñas desde `feature/mvp-v0.1.0`: CLI/runtime básico, adaptadores de proveedores, herramientas controladas, configuración/sesiones y validación integral.
5. Integrar cada PR en la rama MVP y eliminar su rama de origen; tras superar los criterios de aceptación, integrar MVP en `main`, etiquetar `v0.1.0` y eliminar la rama temporal.

## Decisiones

- [ADR-001: perfil JavaScript inicial y arquitectura transversal](adr/ADR-001-perfil-javascript-inicial-aura.md).

La documentación pública contiene solo el contexto de Aura. La memoria privada de cada herramienta o usuario se mantiene fuera del repositorio.
