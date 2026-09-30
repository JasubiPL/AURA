# Auditoría de SDD en Aura

**Fecha:** 2026-09-29 (America/Mexico_City).

**Base auditada:** `docs/opencode-runtime-analysis`, commit `1f7826d`; correcciones documentales descritas abajo en la misma rama.

**Alcance:** revisión del proceso y sus artefactos. No hay código del producto para auditar la ejecución del ciclo completo.

## Resultado

El repositorio está alineado con SDD en su etapa de especificación: AGENTS.md exige trabajar desde specs, el MVP tiene el conjunto completo de artefactos y requisitos/tareas/validación se relacionan por IDs. No se puede acreditar todavía una práctica de implementación guiada por specs ni el SDD automatizado de Aura; no existe runtime ni evidencia de una tarea de código completada.

La referencia conceptual es [SDD en GitHub Spec Kit](https://github.github.com/spec-kit/concepts/sdd.html), con su [revisión de coherencia y convergencia](https://github.github.com/spec-kit/quickstart.html). La organización `docs/mvp/` constituye una adaptación de Aura, no una instalación de esa herramienta ni una certificación externa de metodología.

## Hallazgos y correcciones

| Área | Evidencia en la base auditada | Evaluación y corrección en esta rama |
| --- | --- | --- |
| Principios y fuente de verdad | AGENTS.md exige specs antes del código, mantiene reglas compartidas y remite a ADR; CLAUDE.md reutiliza esa guía. | Alineado. Conservar las mismas fuentes y remitir a la guía SDD sin duplicar principios. |
| Resultado, alcance y aceptación | Brief y R-01 a R-12 distinguen alcance/exclusiones y evidencias; refinamientos de ADR-003 están propuestos. | Alineado para planificación; no declarar cerrados contratos pendientes del spike. |
| Diseño y factibilidad | `design.md` separa componentes, errores, contratos y decisiones condicionadas. | Alineado como diseño inicial. La ausencia de resultados no es una falta de SDD; bloquea únicamente implementación dependiente. |
| Trazabilidad | Tareas con requisitos/dependencias; V-01 a V-12 cubren requisitos y tareas. Referencia al diseño por documento, sin secciones por tarea. | Reforzar `tasks.md` con referencias concretas a secciones del diseño. La evidencia de código se añadirá cuando exista. |
| Momento de la validación | La secuencia de AGENTS.md muestra `implementación → validation.md`; el archivo ya contiene casos planificados. | Ambiguo. Aclarar que la aceptación y el plan de validación se definen antes del código y las comprobaciones se ejecutan durante/después del incremento. |
| Revisión de coherencia y cierre | AGENTS.md pide retroalimentar specs, pero la plantilla de PR no solicita referencias R/T/V ni contraste final de los artefactos. | Hacer explícitas preparación, revisión de coherencia y cierre con evidencia en la guía y plantilla. |
| Cambios pequeños | AGENTS.md admite documentación proporcional para correcciones. | Conservarlo: contrato compacto con resultado, solución, tarea y comprobación; no omitir intención/aceptación ni forzar archivos vacíos. |
| Default del producto | README describe Workflow Engine opcional; ADR-001 no fija SDD como selección predeterminada. | Faltaba esta decisión. ADR-004 registra la petición de Jas y sus criterios futuros; no la presenta como función implementada del MVP. |
| Aplicación del proceso | `.github/` contiene una plantilla de PR, sin un workflow de CI que compruebe SDD. | Los controles actuales son instrucciones, trazabilidad y revisión de PR; no afirmar enforcement automático. Automatizarlo puede evaluarse después sin bloquear el proceso documental. |
| Estado real | Todas las tareas ejecutables y validaciones están pendientes; no hay CLI ni runtime. | Correcto. No marcar el ciclo de código ni el producto SDD como completados con una revisión de Markdown. |

## Preparación resultante

- **Proceso del repositorio:** suficientemente definido para seguir refinando specs y ejecutar incrementos preparados. La guía describe preparación, validación anticipada, cambios de alcance y cierre; no necesita una dependencia nueva.
- **Siguiente trabajo:** T-00 puede investigar su hipótesis con mock cuando su contrato mínimo esté revisado. Las propuestas de ADR-003 y los spikes de autenticación, sandbox y estado conservan sus condiciones.
- **Aura completa:** SDD por defecto queda aprobado como dirección de producto en ADR-004. Su selección, estado, continuidad y verificación requieren una spec futura y pruebas; no se añaden como condición de lanzamiento de v0.1.0.
- **Integración de documentación:** los hallazgos de proceso se atienden en esta rama antes de preparar la PR hacia `feature/mvp-v0.1.0`. Integrar documentos no aprueba automáticamente ADR-003 ni demuestra capacidades del runtime.

La verificación de enlaces, IDs y coherencia documental se registra en el resumen del cambio o la PR. No equivale a una prueba ejecutada del producto.
