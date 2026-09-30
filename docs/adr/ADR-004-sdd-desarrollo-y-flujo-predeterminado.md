# ADR-004 · SDD para construir Aura y como flujo predeterminado del producto

**Fecha:** 2026-09-29.

**Estado:** Decisión de metodología y dirección del producto aprobada por Jas; la automatización de SDD en Aura no está implementada.

**Complementa:** [ADR-001](ADR-001-perfil-javascript-inicial-aura.md) y [ADR-002](ADR-002-runtime-independiente-openai.md). Aclara el carácter predeterminado de SDD sin adelantar el Workflow Engine completo al MVP.

## Contexto

El repositorio ya tiene brief, requisitos, diseño, tareas y validación del MVP. Sin embargo, la arquitectura describe SDD como un workflow opcional y no declara que Aura lo seleccione por defecto. Jas confirmó que quiere construir este proyecto con Spec-Driven Development y que Aura, cuando esté completa, use esa metodología por defecto para trabajar en software.

Contar con archivos de especificación no acredita un ciclo de desarrollo completo: deben dirigir el cambio y su aceptación, mantenerse coherentes y recibir evidencia de implementación. El proceso necesita una aplicación proporcional para no convertir cada consulta o ajuste pequeño en una colección de documentos.

## Decisión

1. **Desarrollo del repositorio desde ahora.** SDD es la metodología predeterminada para construir y evolucionar Aura. Antes de modificar comportamiento, identificar el resultado esperado y su aceptación, concretar el diseño necesario y derivar tareas. Implementar contra ese contrato y contrastar código, pruebas y specs antes de cerrar el cambio. La [guía de SDD](../development/sdd.md) fija el procedimiento compartido; AGENTS.md remite a ella.
2. **Fuente de verdad mantenida.** Las specs definen el comportamiento deseado; el código y las pruebas demuestran el comportamiento real. Una diferencia exige corregir la implementación o revisar explícitamente la spec, sin cambiar la aceptación para ocultar un fallo. Los ADR conservan decisiones y motivos, sin duplicar las specs. Cada incremento se relaciona con requisito, sección de diseño, tarea y validación.
3. **Planificar la validación antes del código.** Los criterios de aceptación nacen con los requisitos; las comprobaciones concretas se diseñan antes de ejecutar las tareas. La validación se realiza durante cada incremento y al cerrar su alcance. La secuencia no reserva la definición de pruebas para el final.
4. **SDD como default del producto completo.** Aura deberá seleccionar automáticamente el flujo SDD ante solicitudes de crear o modificar software, incluyendo funcionalidades, refactors y correcciones. Resolverá las specs vigentes del proyecto, aclarará ambigüedades que afecten el resultado, preparará el diseño y las tareas necesarias, implementará y comprobará convergencia contra la aceptación. El usuario no tendrá que activar SDD en cada petición. Una preferencia explícita del usuario puede cambiar el flujo de trabajo, sin cambiar permisos ni límites.
5. **Proporción y contexto existente.** Reutilizar la estructura y convenciones del proyecto atendido; no migrar sus specs ni imponer los nombres de archivos de Aura. En cambios pequeños puede bastar un contrato compacto con comportamiento esperado, solución, tarea y verificación en la PR o spec existente. Explicar, investigar o revisar sin implementar usa las specs pertinentes como contexto; no crea artefactos ni ejecuta pruebas automáticamente. Una revisión puede comprobar conformidad con la spec sin iniciar una implementación.
6. **Autonomía con decisiones explícitas.** SDD no exige una aprobación humana por cada etapa. Avanzar con la autorización ya concedida y comprobar la suficiencia de los artefactos. Pedir información cuando falte una decisión necesaria de alcance/producto, exista una ambigüedad material o la operación requiera permiso. Un documento, checklist o workflow no concede autoridad para ejecutar herramientas.
7. **Entrega gradual.** El MVP se desarrolla siguiendo SDD, pero conserva el alcance acotado de ADR-002: CLI, runtime, proveedor y herramientas. No incorpora por esta decisión un Workflow Engine completo, importadores ni especialistas. La capacidad de Aura de gestionar SDD por defecto requiere una spec posterior con su propio diseño, tareas y validación antes de implementarse. Un prompt que mencione SDD no basta para declarar esa capacidad terminada.

## Consecuencias

El desarrollo debe mantener trazabilidad y resolver contradicciones antes de codificar el módulo afectado. Esto añade trabajo de especificación, pero hace revisables el alcance y la aceptación, permite retomar tareas y evita depender solo del historial de chat. El proceso debe escalar con el cambio, preservando la ruta directa para consultas.

Se conserva la organización documental existente. SDD es una metodología; esta decisión no adopta Spec Kit, Kiro ni otra dependencia. La elección de una herramienta o automatización futura requiere necesidad demostrada y su diseño correspondiente.

## Criterios verificables

- **Repositorio:** un incremento de comportamiento cuenta antes del código con requisito/aceptación, referencia al diseño, tarea y comprobación; su cierre aporta evidencia del commit validado y reconcilia las specs afectadas. Los spikes prueban hipótesis previamente descritas y no se presentan como implementación de producción.
- **Producto completo:** una solicitud normal de funcionalidad inicia SDD sin activación manual, reutiliza specs existentes y vincula tareas, cambios y validación. Ante un requisito material incompleto, resuelve la ambigüedad antes de modificar el comportamiento afectado.
- **Cambios pequeños y consultas:** una corrección usa un contrato compacto y una comprobación proporcional; una explicación no genera specs, builds ni delegación innecesarios.
- **Continuidad y coherencia:** retomar una tarea recupera su spec y estado; un cambio de alcance actualiza requisitos/diseño/tareas y aceptación. Una comprobación fallida deja el trabajo pendiente en lugar de marcarlo completado.
- **Seguridad:** el flujo SDD usa el Tool Executor y presupuesto de la sesión; sus etapas no amplían permisos ni cruzan estado privado entre repositorios.

La [auditoría inicial](../research/sdd-audit.md) verifica la preparación documental. Los criterios del producto completo permanecen pendientes hasta su especificación e implementación posteriores; no se añaden como requisitos de lanzamiento de v0.1.0.
