# ADR-002 · Runtime de agentes propio de Aura y MVP solo con OpenAI

**Fecha:** 2026-09-25  
**Estado:** Dirección arquitectónica aprobada; la viabilidad de la autenticación mediante suscripción sigue siendo un bloqueo explícito para el lanzamiento.  
**Sustituye:** El plan provisional para v0.1.0 de delegar el ciclo interno del agente y las herramientas de Aura a un runtime externo, descrito en ADR-001 y en el primer borrador del MVP. Los demás principios arquitectónicos compatibles de ADR-001 siguen vigentes.

## Contexto

El propósito central de Aura es convertirse en un agent harness de ingeniería de software independiente y centrado en la CLI. Delegar todo el ciclo del agente en la primera versión a un runtime externo aplazaría la construcción del propio runtime que define a Aura. Jas eligió explícitamente la **Opción B**: Aura implementará su ciclo de agente y usará OpenAI como único proveedor inicial de modelos, con preferencia por el acceso mediante la suscripción ChatGPT/Codex frente al cobro por uso de la API.

Disponer de un inicio de sesión OAuth no implica, por sí solo, autorización para consultar un modelo con una suscripción. Las integraciones comunitarias de OpenCode muestran una vía técnica de OAuth hacia el backend de Codex, pero su existencia no demuestra que haya un endpoint de terceros oficialmente admitido, estable y autorizado de forma general para un runtime independiente.

Referencias para el spike:

- Alcance de identidad de Sign in with ChatGPT de OpenAI: https://help.openai.com/en/articles/20001410-sign-in-with-chatgpt
- Ejemplo de implementación comunitaria de OpenCode: https://github.com/numman-ali/opencode-openai-codex-auth

## Decisión

1. **Alcance del proveedor:** v0.1.0 incluirá únicamente OpenAI. No implementará Anthropic, selección automática entre proveedores ni modo con API key.
2. **Runtime propio:** Aura orquestará directamente los turnos del modelo, seleccionará y ejecutará sus herramientas, y administrará contexto, presupuestos por tarea, sesiones, permisos, errores y cancelación. El proveedor devolverá la salida del modelo y las solicitudes estructuradas de herramientas; nunca ejecutará herramientas por cuenta de Aura.
3. **Objetivo de autenticación:** un adaptador OpenAI respaldado por suscripción que utilice un inicio de sesión y un protocolo de solicitudes ChatGPT/Codex admitidos y expresamente autorizados, **si el spike de la fase 0 demuestra que existen**. Aplicar controles normales de seguridad OAuth y usar el llavero del sistema solo cuando esté permitido. Nunca extraer credenciales, copiar almacenes privados de tokens de Codex, suplantar la identidad OAuth de otro cliente ni considerar una implementación no oficial como prueba de autorización.
4. **Sin runtime delegado:** el Agent Runtime de Aura no se implementará mediante el runtime de otro proveedor. Si el acceso directo por suscripción no está disponible en condiciones aceptables, marcar el adaptador de suscripción como bloqueado, continuar el desarrollo y las pruebas del runtime independiente del proveedor con un mock y no anunciar una versión utilizable con suscripción.
5. **Sin cargos imprevistos:** un adaptador para la API de OpenAI, créditos de pago o el cambio a facturación por API requieren una nueva decisión explícita y una implementación aparte. Los fallos de autenticación, el agotamiento de cuota y el acceso no admitido por la suscripción deben detenerse claramente, sin cambiar de vía de facturación.
6. **Plataforma y entorno:** macOS primero; Node.js 24 como versión mínima y objetivo inicial de desarrollo y distribución; JavaScript ESM con JSDoc.
7. **Estado y seguridad:** YAML validado para la configuración de usuario y proyecto; sesiones JSONL aisladas por repositorio; límites configurables de turnos del modelo, tiempo, llamadas a herramientas, duración de comandos e iteraciones de corrección. Tool Executor propio que aplique aprobaciones explícitas, raíces autorizadas, manejo de secretos y un sandbox de macOS comprobado. Mostrar datos de uso y cuotas de la cuenta solo cuando el proveedor los exponga de forma verificable, e identificar los valores desconocidos o estimados.
8. **Primera entrega pequeña:** una ruta directa de agente y un modelo OpenAI configurado. Mantener interfaces separadas para Agent Router y Model Router sin construir todavía distribución multiagente, importación de capacidades, memoria avanzada, workflows completos ni una TUI elaborada.

## Evidencia necesaria en la fase 0

- Determinar si existe un protocolo directo de suscripción **documentado o expresamente autorizado** para un runtime de agente de terceros. Separar el permiso de identidad OAuth del derecho de inferencia; registrar fuente precisa, modelos admitidos, límites de uso, reglas de tokens y renovación, estabilidad y aplicación de la cuota de suscripción.
- Contrastar la implementación demostrada por OpenCode con la documentación y las condiciones del proveedor. No distribuir una dependencia de endpoints no documentados ni de IDs de cliente prestados basándose solo en código comunitario.
- Demostrar, con un prototipo pequeño cuyo runtime pertenezca a Aura, el ciclo de solicitud y respuesta y las llamadas estructuradas a herramientas; confirmar que el proveedor no ejecuta internamente las herramientas.
- Validar por separado el sandbox de macOS, las restricciones de raíces del sistema de archivos incluidos enlaces simbólicos, la cancelación del árbol de procesos, la política de red, las aprobaciones y los límites mediante recursos de prueba desechables.
- Verificar que no se necesita API key, que no existe un fallback a la API de pago, que los fallos se informan y que los datos de uso se etiquetan correctamente.
- Registrar resultados reproducibles en `docs/mvp/spike-results.md`, sin tokens de autenticación ni datos privados de la cuenta.

## Consecuencias y condición de lanzamiento

La independencia exige construir una mayor parte del runtime en el MVP, pero evita acoplar herramientas, políticas y sesiones a otro harness. También crea un **riesgo real de viabilidad del acceso por suscripción**. Si no existe acceso directo permitido y suficientemente estable, continuar el núcleo independiente con pruebas contra un mock y **bloquear el lanzamiento de v0.1.0 respaldado por suscripción** hasta que Jas apruebe expresamente otra alternativa. No sustituir silenciosamente el runtime ni usar una API de pago.

La aceptación completa de extremo a extremo exige un turno real y autorizado del modelo OpenAI, ejecución de herramientas a cargo de Aura, edición y validación focalizadas de código, rechazo de accesos no autorizados, aislamiento de sesiones y una presentación veraz del uso. Este ADR no afirma que esa implementación o esas pruebas ya existan.
