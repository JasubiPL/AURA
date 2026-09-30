# SDD en el desarrollo de Aura

**Fecha:** 2026-09-29.

**Decisión:** [ADR-004](../adr/ADR-004-sdd-desarrollo-y-flujo-predeterminado.md). Esta guía define el proceso del repositorio; el SDD automatizado de Aura como producto sigue pendiente.

## Principio de trabajo

Spec-Driven Development empieza por el resultado esperado y convierte requisitos y diseño en tareas y evidencia de aceptación. La spec mantiene el comportamiento deseado; código y pruebas muestran el estado real. No basta con redactar documentación después del código ni con conservar una lista de tareas desligada del contrato.

Los principios compartidos de ingeniería, seguridad e idioma viven en [AGENTS.md](../../AGENTS.md), y las decisiones arquitectónicas en los ADR. No crear una segunda copia de esas reglas. La documentación de [SDD de GitHub Spec Kit](https://github.github.com/spec-kit/concepts/sdd.html) y su [guía de contratos](https://github.github.com/spec-kit/guides/contract-driven-development.html) sirven de referencia conceptual; este repositorio aplica el procedimiento siguiente con sus archivos existentes.

## Ciclo de un incremento

| Etapa | Trabajo y criterio de salida | Artefacto del MVP |
| --- | --- | --- |
| Comprender y especificar | Problema, resultado, alcance/exclusiones y restricciones. Requisitos identificables con aceptación observable; explicitar supuestos y resolver ambigüedades materiales. | [Brief](../mvp/brief.md) y [requirements.md](../mvp/requirements.md). |
| Diseñar e investigar | Solución suficiente para el incremento, entradas/salidas, fallos y dependencias. Decisiones compatibles con ADR; hipótesis de factibilidad separadas de garantías. | [design.md](../mvp/design.md); ADR cuando cambia arquitectura; resultados del spike cuando existan. |
| Planificar y preparar validación | Tareas acotadas vinculadas a requisitos y diseño, dependencias y resultados de comprobación esperados. Revisar coherencia antes del código. | [tasks.md](../mvp/tasks.md), [implementation-plan.md](../mvp/implementation-plan.md) y plan de [validation.md](../mvp/validation.md). |
| Implementar | Ejecutar una tarea preparada dentro de su alcance y permisos; actualizar la spec si aparece una diferencia que deba decidirse. | Código y pruebas focalizadas; tarea en curso hasta aportar evidencia. |
| Validar y reconciliar | Contrastar resultado contra aceptación y diseño; registrar comando/pasos, resultado y commit. Corregir desvíos o documentar bloqueo. Cerrar solo lo demostrado. | Evidencia en PR y referencias desde tareas/validación; [estado](../status.md) actualizado cuando cambie una capacidad. |

Este es un ciclo por incremento, no una cascada que obliga a terminar todo el diseño del producto antes del primer spike. Definir aceptación antes de implementar; ejecutar validación durante el trabajo y al cerrar. La evidencia puede llevar a revisar el diseño y repetir las etapas afectadas.

## Preparación de una tarea

Antes de implementar comportamiento, comprobar:

- Resultado y alcance claros; requisito/aceptación vigente y diferencia entre aprobado, propuesto y pendiente.
- Sección de diseño suficiente para esa tarea, contratos y errores relevantes, dependencias resueltas o aisladas mediante un mock explícito.
- Tarea acotada con referencias a requisito, diseño y comprobación; sin decisiones materiales sin resolver para el comportamiento que se va a implementar.
- Validación proporcional planificada con resultado esperado y entorno necesario. Una prueba simulada no prueba una integración o sandbox real.

La revisión de suficiencia puede realizarla el implementador con la autorización existente; no exige una confirmación humana por cada documento. Pedir a Jas decisiones de producto/alcance que realmente falten, sin detener trabajo independiente autorizado. Una decisión marcada como propuesta no se vuelve aprobada por generar código o una checklist.

Un spike puede explorar un contrato propuesto si ya tiene hipótesis, límites, fixture y criterio de evaluación. Registrar observaciones antes de adoptar su resultado en producción. `docs/mvp/spike-results.md` se crea cuando se ejecuten prototipos; no generar una plantilla vacía para aparentar evidencia.

## Cierre y cambios de alcance

Antes de integrar el incremento, revisar requisito → diseño → tarea → código → validación. La PR identifica las referencias afectadas, evidencia ejecutada y limitaciones. Las casillas se marcan con la evidencia correspondiente; completar un análisis documental no completa tareas ejecutables.

Si cambia el alcance, actualizar el requisito y revisar diseño, tareas, aceptación y ADR pertinente en el mismo cambio. Mantener IDs existentes; añadir los nuevos cuando corresponda sin reutilizarlos para otro requisito. Una comprobación fallida produce corrección o bloqueo documentado, no una relajación silenciosa de la aceptación. No declarar lista una release si falta un criterio obligatorio.

Una diferencia de implementación no siempre requiere cambiar la spec: primero determinar si es un bug o un cambio deseado. Las specs se mantienen como documentos vivos en Git; sus commits conservan versiones previas sin crear copias editables paralelas.

## Proporción y organización

- **MVP actual:** continuar en `docs/mvp/`; no crear un segundo conjunto con otros nombres.
- **Funcionalidad posterior:** cuando se vaya a especificar, usar `docs/specs/<feature>/` para brief, requisitos, diseño, tareas y validación suficientes. Crear solo los artefactos que contengan información útil y referenciar los ADR compartidos.
- **Corrección o refactor pequeño:** expresar resultado esperado, solución acotada, tarea y validación en la PR o spec existente; actualizar los documentos que cambian. Este contrato compacto sigue SDD sin exigir cinco archivos por ajuste.
- **Consulta, investigación o revisión sin cambios:** recuperar specs pertinentes y aportar evidencia proporcional; no iniciar implementación ni persistir documentos solo por responder.

No se requieren generación automática, un lenguaje concreto, especialistas, instalación de Spec Kit ni un Workflow Engine para seguir este proceso en el repositorio. La capacidad futura de Aura de seleccionar y mantener SDD por defecto es otro entregable y se valida según ADR-004.
