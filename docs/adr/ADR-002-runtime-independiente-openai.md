# ADR-002 · Runtime de agentes propio de Aura y MVP solo con OpenAI

**Fecha:** 2026-09-25  
**Actualizado:** 2026-09-29  
**Estado:** Dirección arquitectónica aprobada; la viabilidad de la autenticación mediante suscripción sigue siendo un bloqueo explícito para el lanzamiento.  
**Sustituye:** El plan provisional para v0.1.0 de delegar el ciclo interno del agente y las herramientas de Aura a un runtime externo, descrito en ADR-001 y en el primer borrador del MVP. Los demás principios arquitectónicos compatibles de ADR-001 siguen vigentes.

**Decisión confirmada por Jas (2026-09-29):** el inicio de sesión con OpenAI será OAuth. La selección de OAuth queda aprobada; el registro concreto, los scopes de inferencia, la elegibilidad y el acceso real mediante suscripción siguen sujetos al spike.

## Contexto

El propósito central de Aura es convertirse en un agent harness de ingeniería de software independiente y centrado en la CLI. Delegar todo el ciclo del agente en la primera versión a un runtime externo aplazaría la construcción del propio runtime que define a Aura. Jas eligió explícitamente la **Opción B**: Aura implementará su ciclo de agente y usará OpenAI como único proveedor inicial de modelos, con preferencia por el acceso mediante la suscripción ChatGPT/Codex frente al cobro por uso de la API.

Disponer de un inicio de sesión OAuth no implica, por sí solo, autorización para consultar un modelo con una suscripción. OpenCode muestra una implementación técnica hacia el backend de Codex, pero su existencia no acredita autorización para Aura. La investigación documental del 2026-09-29 identifica además una candidata oficial de uso del plan de ChatGPT en aplicaciones abiertas y locales. Falta verificar su elegibilidad para Aura, condiciones de distribución, estabilidad y ejecución real; el bloqueo de lanzamiento continúa.

Referencias para el spike:

- [Uso del plan en aplicaciones abiertas y locales](https://developers.openai.com/siwc/token-sharing-open-source): distinguir la capacidad de inferencia de los permisos de identidad y comprobar condiciones aplicables.
- [Registro e inicio de sesión](https://developers.openai.com/siwc/token-sharing-open-source/sign-in): candidato con registro dinámico propio, host persistente, consentimiento, PKCE, `state`, `nonce` y validación de identidad/scopes. No necesita una identidad OAuth prestada.
- [Modelos e inferencia](https://developers.openai.com/siwc/token-sharing-open-source/models-and-inference): candidato directo a Responses con OAuth, `store: false`, `stream: true` y confirmación de terminación; no usar endpoints `backend-api` para esta vía.
- [Limitaciones de preview](https://developers.openai.com/siwc/token-sharing-open-source/preview-limitations): revisar formato de instrucciones, historial enviado por el cliente, herramientas admitidas y campos no soportados antes de fijar el adaptador.
- [Plugin OpenAI del repositorio oficial de OpenCode](https://github.com/anomalyco/opencode/blob/2fa3363c924c5c3e367b84a87ae478296a0ed59b/packages/opencode/src/plugin/openai/codex.ts): referencia de implementación, no fuente de autorización ni protocolo elegido para Aura.

Estas fuentes actualizan la investigación, no aprueban una integración implementada. La licencia de Aura continúa pendiente y debe resolverse si afecta su elegibilidad. El informe y sus límites están en la [revisión de integración](../research/opencode-integration-review.md).

## Decisión

1. **Alcance del proveedor:** v0.1.0 incluirá únicamente OpenAI. No implementará Anthropic, selección automática entre proveedores ni modo con API key.
2. **Runtime propio:** Aura orquestará directamente los turnos del modelo, seleccionará y ejecutará sus herramientas, y administrará contexto, presupuestos por tarea, sesiones, permisos, errores y cancelación. El proveedor devolverá la salida del modelo y las solicitudes estructuradas de herramientas; nunca ejecutará herramientas por cuenta de Aura.
3. **Autenticación OAuth aprobada:** el inicio de sesión del adaptador OpenAI será OAuth, conforme a la confirmación de Jas del 2026-09-29. Usar registro y consentimiento para Aura por una vía admitida; el protocolo concreto y la inferencia respaldada por suscripción se habilitan **solo si el spike de fase 0 verifica autorización, elegibilidad y acceso real**. Aplicar controles de seguridad OAuth y usar el llavero del sistema solo cuando esté permitido. Nunca extraer credenciales, copiar almacenes privados de tokens de Codex, suplantar la identidad OAuth de otro cliente ni considerar una implementación ajena como prueba de autorización.
4. **Sin runtime delegado:** el Agent Runtime de Aura no se implementará mediante el runtime de otro proveedor. Si el acceso directo por suscripción no está disponible en condiciones aceptables, marcar el adaptador de suscripción como bloqueado, continuar el desarrollo y las pruebas del runtime independiente del proveedor con un mock y no anunciar una versión utilizable con suscripción.
5. **Sin cargos imprevistos:** un adaptador para la API de OpenAI, créditos de pago o el cambio a facturación por API requieren una nueva decisión explícita y una implementación aparte. Los fallos de autenticación, el agotamiento de cuota y el acceso no admitido por la suscripción deben detenerse claramente, sin cambiar de vía de facturación.
6. **Plataforma y entorno:** macOS primero; Node.js 24 como versión mínima y objetivo inicial de desarrollo y distribución; JavaScript ESM con JSDoc.
7. **Estado y seguridad:** YAML validado para la configuración de usuario y proyecto; sesiones JSONL aisladas por repositorio; límites configurables de turnos del modelo, tiempo, llamadas a herramientas, duración de comandos e iteraciones de corrección. Tool Executor propio que aplique aprobaciones explícitas, raíces autorizadas, manejo de secretos y un sandbox de macOS comprobado. Mostrar datos de uso y cuotas de la cuenta solo cuando el proveedor los exponga de forma verificable, e identificar los valores desconocidos o estimados.
8. **Primera entrega pequeña:** una ruta directa de agente y un modelo OpenAI configurado. Mantener interfaces separadas para Agent Router y Model Router sin construir todavía distribución multiagente, importación de capacidades, memoria avanzada, workflows completos ni una TUI elaborada.

## Evidencia necesaria en la fase 0

- Determinar si existe un protocolo directo de suscripción **documentado o expresamente autorizado** para un runtime de agente de terceros. Separar el permiso de identidad OAuth del derecho de inferencia; registrar fuente precisa, modelos admitidos, límites de uso, reglas de tokens y renovación, estabilidad y aplicación de la cuota de suscripción.
- Evaluar primero la candidata oficial documentada, incluyendo elegibilidad, condiciones de distribución, registro para Aura y limitaciones de preview. Contrastar después los patrones de OpenCode; no distribuir una dependencia de endpoints no documentados ni de IDs de cliente prestados basándose solo en su código.
- Demostrar, con un prototipo pequeño cuyo runtime pertenezca a Aura, el ciclo de solicitud y respuesta y las llamadas estructuradas a herramientas; confirmar que el proveedor no ejecuta internamente las herramientas.
- Validar por separado el sandbox de macOS, las restricciones de raíces del sistema de archivos incluidos enlaces simbólicos, la cancelación del árbol de procesos, la política de red, las aprobaciones y los límites mediante recursos de prueba desechables.
- Verificar que no se necesita API key, que no existe un fallback a la API de pago, que los fallos se informan y que los datos de uso se etiquetan correctamente.
- Registrar resultados reproducibles en `docs/mvp/spike-results.md`, sin tokens de autenticación ni datos privados de la cuenta. Una revisión documental no completa ese spike. [ADR-003](ADR-003-contratos-turnos-herramientas-sesiones.md) propone sus contratos mínimos y [validation.md](../mvp/validation.md) relaciona pruebas y requisitos.

## Consecuencias y condición de lanzamiento

La independencia exige construir una mayor parte del runtime en el MVP, pero evita acoplar herramientas, políticas y sesiones a otro harness. También crea un **riesgo real de viabilidad del acceso por suscripción**. Si no existe acceso directo permitido y suficientemente estable, continuar el núcleo independiente con pruebas contra un mock y **bloquear el lanzamiento de v0.1.0 respaldado por suscripción** hasta que Jas apruebe expresamente otra alternativa. No sustituir silenciosamente el runtime ni usar una API de pago.

La aceptación completa de extremo a extremo exige un turno real y autorizado del modelo OpenAI, ejecución de herramientas a cargo de Aura, edición y validación focalizadas de código, rechazo de accesos no autorizados, aislamiento de sesiones y una presentación veraz del uso. Este ADR no afirma que esa implementación o esas pruebas ya existan.
