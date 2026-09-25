# Estado del proyecto

Actualizado: 2026-09-24. Este archivo resume el estado verificable; el código y los ADR son la referencia detallada.

## Implementado

- Repositorio público con README, retrato e instrucciones de trabajo para agentes.
- ADR-001 con dirección de arquitectura, límites y criterios de aceptación del MVP.
- Flujo de ramas y PR documentado. No hay todavía runtime ni CLI funcional.

## Decidido para el MVP

- CLI con un ciclo básico de agente, una integración real de modelo, herramientas de archivos y terminal con permisos y aislamiento verificables.
- Contratos separados para Agent Router y Model Router; selección simple al principio.
- Presupuestos por tarea, contexto selectivo y distinción entre capacidades propias, del usuario y externas.
- Memoria de sesión, personal y de proyecto separadas; no guardar secretos. Los detalles se concretarán en especificaciones.

## Pendiente

1. Spike acotado: comprobar sandbox, permisos, almacenamiento de aprobaciones, proveedor inicial y catálogo de capacidades.
2. Especificaciones SDD del MVP (`requirements`, `design`, `tasks`, `validation`) basadas en el resultado del spike.
3. Elegir lenguaje del runtime, proveedor inicial y mecanismos concretos. JavaScript ESM con JSDoc es una propuesta, no una implementación aprobada.
4. Implementar y validar la CLI; registrar evidencia y actualizar este estado conforme avance.

## Decisiones

- [ADR-001: perfil JavaScript inicial y arquitectura transversal](adr/ADR-001-perfil-javascript-inicial-aura.md).

La documentación pública contiene solo el contexto de Aura. La memoria privada de cada herramienta o usuario se mantiene fuera del repositorio.
