# ADR-001 · Perfil JavaScript inicial de Aura

**Fecha:** 2026-09-24  
**Actualizado:** 2026-09-25  
**Estado:** Decisión de arquitectura aprobada como dirección de Aura; contratos y criterios de MVP definidos, implementación pendiente. Los parámetros expresamente señalados como pendientes no quedan aprobados por este ADR.  

## Contexto

El desarrollo inicial se centrará en proyectos JavaScript, pero Aura debe servir también para otros lenguajes. El runtime será independiente del editor y del stack. El diseño favorece operaciones proporcionales a la tarea, carga selectiva de contexto y permisos aplicados por el ejecutor.

## Decisión

1. Distribuir un perfil inicial `javascript`, habilitado por defecto en la instalación de Aura, configurable por usuario y proyecto. «Habilitado» significa disponible; no insertar todas sus instrucciones en cada prompt. Mostrar el perfil activo y las capacidades invocadas al inicio o bajo demanda sin panel permanente.
2. Crear capacidades nativas de Aura a partir de contratos de ingeniería generales, evaluados individualmente. Documentar su propósito, alcance y evolución dentro de este proyecto, sin importar configuraciones o credenciales de otros proyectos.
3. El núcleo de Aura permanece neutral respecto al lenguaje. El perfil JavaScript aporta reglas y especialistas bajo demanda. Se podrán añadir, desactivar o retirar capacidades sin editar el runtime. Una capacidad instalada por defecto también podrá deshabilitarse en un proyecto.
4. Distinguir skill (conocimiento/instrucciones), agent (rol y permisos), workflow (secuencia opcional con condiciones) y herramienta (operación ejecutable). Un agente usa skills sin requerir workflow; un workflow puede usar el mismo agente o especialistas si aportan evidencia independiente.
5. Clasificar intención antes de cargar capacidades: explicar, investigar, revisar, modificar, verificar. Las preguntas y explicaciones de comentarios de PR no ejecutan pruebas ni delegan por defecto. Seleccionar lectura y validaciones por alcance/riesgo, reutilizar evidencia vigente, detener ciclos repetidos y reportar límites reales.
6. Ofrecer un arnés de ingeniería independiente del lenguaje y framework: los perfiles JavaScript, Python, Go, C#, React Native u otros se conectan a los mismos principios y flujos mediante capacidades optativas. Los flujos transversales no contienen comandos ni convenciones exclusivos de un stack.
7. Definir desde la primera versión las fronteras de **Agent Router** y **Model Router** y un presupuesto común, aunque el MVP ejecute un agente y un modelo configurado. La selección automática entre especialistas, proveedores o modelos requiere adaptadores y evaluaciones posteriores.
8. El runtime, no los prompts ni las skills, decide la autorización de herramientas. El MVP ya debe imponer aislamiento por espacio de trabajo, permisos graduados de archivos, comandos y red, con concesiones limitadas por operación, sesión o configuración persistente revocable.

## Decisiones complementarias aprobadas (2026-09-25)

- **Runtime inicial:** JavaScript ESM con JSDoc. Mantener interfaces del núcleo independientes del perfil tecnológico; evaluar un cambio de lenguaje solo si aparecen necesidades verificables.
- **Proveedores iniciales objetivo:** OpenAI y Anthropic, con adaptadores separados y configuración explícita. El soporte de ambos forma parte de la dirección aprobada; el orden de implementación y los métodos de autenticación se validarán en el spike. Una suscripción de producto no implica acceso API ni autoriza reutilizar credenciales.
- **Sandbox:** evaluar Bubblewrap en Linux y Seatbelt en macOS mediante un `SandboxAdapter` con contratos independientes de plataforma. WSL2 es candidato para Windows, sujeto a pruebas. Ninguna tecnología está aprobada como implementación definitiva hasta verificar aislamiento de archivos, procesos, tiempo y red; ante ausencia de aislamiento verificable, solicitar aprobación explícita o denegar la operación.
- **Pendiente:** esquema de manifiestos/configuración/sesiones, reglas de precedencia, política concreta de permisos, presupuestos y formato de memoria.

## Principios de diseño y límites

Aura adopta un runtime independiente del editor, un catálogo privado de capacidades propias y externas, descubrimiento sin activación automática, routers separados, contexto selectivo, presupuestos por tarea y una interfaz terminal sobria. La compatibilidad con servicios, modelos y formatos de otros harnesses es una hipótesis que se verificará por adaptador; no se considera implementada por este ADR.

**No se fija un número de skills, agentes o workflows.** El criterio para incorporar una capacidad nativa es que resuelva una tarea comprobable que el núcleo no cubre de forma más sencilla. La primera distribución será pequeña; los totales que vea el usuario se derivan del registro efectivo.

## Base transversal de ingeniería

La base de Aura contiene skills pequeñas de **calidad de ingeniería**, **diseño/arquitectura**, **pruebas**, **revisión fundamentada** y **seguridad de cambios**, sin reglas de sintaxis ni comandos de un stack. Son orientaciones aplicadas según la intención y el riesgo, no cinco documentos insertados siempre en el prompt. Incluyen cohesión, contratos, límites entre módulos, estado y dependencias explícitos, errores observables, pruebas útiles y hallazgos con evidencia.

Los workflows iniciales son **implement** y **review**, además de la ruta directa de **explain/investigate**. `implement` selecciona: comprender petición y contrato → localizar contexto → plan proporcional → cambiar → validar el impacto → resumir evidencia y pendientes. `review` selecciona: intención y alcance → diff/comentarios y contratos → comprobación focalizada si aporta evidencia → hallazgos ordenados por impacto. Cada paso admite saltos y condiciones de salida; no presupone que haya tests, subagentes o gates formales en todas las tareas.

Un perfil tecnológico aporta skills de lenguaje/framework, detectores y comandos **declarados por el proyecto**. Puede aportar pasos adicionales o workflows propios como alternativas o extensiones explícitas. Para evitar que el orden de instalación cambie el comportamiento sin aviso, el runtime resolverá una configuración efectiva e informará qué regla o extensión eligió. Las reglas del proyecto y del usuario pueden especializar prácticas de ingeniería; ninguna capacidad importada puede ampliar permisos, eliminar límites de seguridad o ejecutar comandos por instalarse. El contrato exacto de prioridad y composición se especificará cuando exista el cargador de capacidades.

Un workflow de un perfil no convierte al perfil en dependencia del núcleo. Por ejemplo, la revisión de una API pública sigue el flujo `review`; una skill de Go aporta convenciones y checks del repositorio; una skill de React Native añade comprobaciones de interfaz únicamente si el cambio las afecta.

## Contexto y memoria: separación aprobada

| Capa | Ubicación prevista | Contenido y alcance |
| --- | --- | --- |
| Contexto de la tarea | Memoria temporal del Agent Runtime | Petición actual, archivos pertinentes, decisiones de trabajo y evidencia vigente; se descarta o resume al terminar. No equivale a memoria permanente. |
| Sesión | `~/.aura/sessions/`, asociada a un proyecto cuando aplique | Historial y estado necesarios para continuar una conversación; no se introduce completo en cada petición ni se comparte entre proyectos por accidente. |
| Memoria personal | `~/.aura/memory/` | Preferencias y decisiones generales del usuario aplicables a sus proyectos. No debe absorber datos privados de un repositorio ni instrucciones específicas de otro proyecto. |
| Memoria del proyecto | `<repo>/.aura/memory/` | Contexto vigente **solo** para ese proyecto, con referencias a archivos y decisiones. No pasa a la memoria global ni a otro repositorio automáticamente. Su política exacta de versionado se decidirá por proyecto; no se hará commit automático. |
| Fuente de verdad compartible | `<repo>/.aura/` o `docs/`, según el repositorio | Configuración, ADR, specs y contratos versionables. Una decisión arquitectónica confirmada se mantiene en su ADR/spec y la memoria solo la referencia; no se mantienen copias divergentes. |

El **Context Manager** busca primero el repositorio y los archivos cercanos a la tarea, recupera fragmentos pertinentes de memoria y registra sus fuentes. La **memoria no es una instrucción superior**: ante conflicto mandan la petición actual, las instrucciones aplicables y la evidencia vigente del proyecto; Aura señala contradicciones en vez de repetir una nota obsoleta. Una referencia sin contenido verificado no cuenta como conocimiento.

El **Memory Manager** distingue observación, propuesta y decisión aprobada; conserva procedencia/fecha y permite inspeccionar, corregir y borrar entradas. La escritura persistente será selectiva: una consulta o una explicación de PR no crea memoria por sí sola. Una decisión confirmada o un dato recurrente útil puede registrarse en su capa adecuada, evitando duplicar specs y ADR. Nunca se guardan secretos, tokens ni credenciales. La captura automática, deduplicación avanzada, búsqueda semántica y migración entre proyectos quedan fuera del MVP hasta contar con reglas y pruebas de aislamiento.

## Capacidades candidatas para Aura

| Capacidad | Propósito | Activación prevista |
| --- | --- | --- |
| `engineering-quality` | Cohesión, contratos, dependencias y severidad basada en evidencia. | Cambios o revisiones donde aporte contexto. |
| `javascript-review` | Correctitud, asincronía, entradas y seguridad del código JavaScript. | Revisión de archivos pertinentes en ese lenguaje. |
| `test-quality` | Pruebas observables y regresiones pertinentes. | Cambios cuyo riesgo justifique pruebas. |
| `implementation-guidance` | Cambio acotado, cumplimiento del contrato y comprobación focalizada. | Tareas de implementación. |
| Revisor, implementador y verificador opcionales | Responsabilidades distinguibles con acceso mínimo. | Solo cuando un especialista aporte valor comprobable. |
| Revisor de arquitectura opcional | Límites entre módulos, estado, APIs y dependencias de impacto transversal. | Decisiones de arquitectura. |
| `implement` y `review` | Flujos transversales ligeros. | Según intención y alcance; admiten ejecución directa. |

Estas son candidatas, no un inventario instalado ni un compromiso de crear una skill por cada fila. La detección de código no utilizado requiere evidencia sobre consumidores reales; los patrones de estilo se evalúan contra los contratos del repositorio, sin convertirlos en infracciones universales.

## Arquitectura prevista

- **CLI** recibe la intención; **Agent Runtime** mantiene una sesión, presupuesto y permisos.
- **Context Manager** lee instrucciones del repositorio y contexto cercano, y recupera memoria pertinente con sus fuentes. No exige crear `.aura/` para responder una pregunta.
- **Memory Manager** separa sesión, memoria personal y memoria de proyecto; su primera entrega prioriza persistencia explícita e inspección, y deja la captura automática avanzada para después.
- **Capability Manager** resuelve manifiestos y configuración efectiva desde capacidades empaquetadas, `~/.aura/` y `<repo>/.aura/`, carga solo las seleccionadas y deja inspeccionar o desactivar cada una. Los perfiles añaden o especializan habilidades y pasos sin alterar el núcleo. La prioridad y la confianza de las fuentes deben definirse antes de aceptar capacidades externas.
- **Agent Router** selecciona ejecución directa o un agente especializado; **Model Router** resuelve proveedor/modelo después, sujeto a permisos, presupuesto y capacidad técnica. Ninguno concede herramientas.
- **Provider adapter** realiza la llamada al modelo; **Tool executor** aplica límites de rutas, tiempo, salida y aprobación pertinente antes de ejecutar operaciones.
- **Workflow runner** opcional ejecuta pasos con condiciones y finalización explícita. Un flujo puede revisar o implementar sin delegación. Agentes especialistas reciben el contexto mínimo y evidencia reutilizable.
- **Sesiones** guardan lo necesario para continuidad y se aíslan por proyecto; no almacenan secretos. Se implementan desde el MVP los contratos mínimos de ambos routers, presupuestos y autorización; se posponen la selección automática compleja, la memoria avanzada, la TUI y los adaptadores adicionales.

## Catálogo y resolución de capacidades

| Origen | Ubicación lógica | Descubrimiento y activación |
| --- | --- | --- |
| Nativas de Aura (core y perfil inicial) | Distribución de Aura y estado privado en `~/.aura/{agents,skills,workflows}/` | Disponibles tras instalar, con listado inspeccionable; deshabilitables por configuración, sujetas a selección por tarea. No se exportan a directorios globales de otros harnesses. |
| Añadidas por el usuario para uso personal | `~/.aura/{agents,skills,workflows}/` y fuentes externas explícitamente configuradas | Se registran con fuente, propietario y confianza; su presencia no las habilita en todos los proyectos. |
| Capacidad del proyecto | `<repo>/.aura/{agents,skills,workflows}/` y fuentes externas configuradas del repositorio | Alcance de ese repositorio; habilitación explícita según configuración efectiva. |
| Capacidades de otros harnesses o instalaciones | Rutas globales y del repositorio **admitidas por un adaptador comprobado** | Se descubren como externas, se muestran por origen y se asignan por proyecto antes de usarse; jamás se copian, borran ni interpretan ciegamente como instrucciones de Aura. |

Estados separados: **descubierta → instalada o accesible → habilitada/asignada al proyecto → seleccionada/cargada en la tarea**. «Interna siempre disponible» significa que puede seleccionarse conforme al perfil; no que su texto completo entre en cada solicitud. En la primera apertura de un proyecto con capacidades externas detectadas, la CLI presenta un resumen y permite escoger cuáles asignar; continuar sin asignarlas es válido. Esa elección se guarda en `<repo>/.aura/config` (formato definitivo por especificar). También se puede cambiar después. La detección de `.agents`, `.skills`, `.github`, `.claude` u otras carpetas es específica de adaptador; no se promete leer cualquier formato ni se inventa compatibilidad con un host no verificado.

`/agents`, `/skills` y `/workflows` muestran, para cada tipo, total descubierto y un desglose excluyente por **propiedad: Aura / usuario / otro harness**, más una segunda dimensión de **alcance: global / proyecto**; indican por separado los subconjuntos habilitados y cargados en la tarea. Una capacidad externa de proyecto se cuenta una sola vez dentro de «otro harness» y también figura en la vista por alcance «proyecto». La inspección individual indica origen, versión si existe, propósito, alcance, permisos solicitados, dependencias y motivo de selección. Un mismo identificador en dos fuentes no se fusiona silenciosamente: se usa identidad calificada por origen y se exige una selección explícita cuando haya colisión. Listar el catálogo carga metadatos breves, no el cuerpo de todas las skills. El recuento es dinámico, nunca codificado en el prompt ni confundido con consumo de tokens.

Un manifiesto mínimo declarará tipo, id, versión/formato, procedencia, compatibilidad, dependencias, capacidades solicitadas y condiciones de activación. El runtime valida y limita lo solicitado a los permisos efectivos. Deshabilitar o retirar una capacidad exige informar qué agentes/workflows la requieren; no borra recursos ajenos. La precedencia y la composición de instrucciones se fijarán mediante ejemplos y pruebas antes de activar extensiones externas; ante conflicto de reglas o un formato desconocido, Aura lo muestra y no ejecuta esa extensión. Los archivos del repositorio y las salidas de herramientas son datos no confiables y no pueden modificar la política de permisos.

## Dos routers, una sola política de ejecución

El **Agent Router** recibe intención (`explain`, `investigate`, `review`, `modify`, `verify`), alcance, riesgo y capacidades habilitadas; devuelve `direct` o un agente elegible junto con el motivo. Un workflow puede llamar agentes si aporta una función distinta, pero no inicia un conjunto fijo de especialistas ni crea delegaciones para explicar una PR. El **Model Router** recibe esa decisión, la configuración del usuario, los modelos realmente disponibles, restricciones del proveedor, capacidades necesarias, presupuesto, coste y latencia; devuelve adaptador y modelo o un error explicable. La interfaz entre ambos evita que una skill elija credenciales o eleve privilegios.

La ruta inicial usa reglas locales y modelo configurado, sin invocación adicional para clasificar cada solicitud. Permite selección manual o fijada por sesión y, cuando existan varios modelos conectados, selección automática optativa con fallback explícito y supervisado por el mismo presupuesto. Una suscripción de ChatGPT, Codex, Copilot, Anthropic o Google **no equivale** a una clave API, acceso autorizado, cuota o precio conocido; cada adaptador verificará el mecanismo disponible sin reutilizar credenciales de otra aplicación por suposición. Si falta un modelo elegible o excede el presupuesto, Aura lo indica antes de gastar o cambiar de proveedor.

Cada tarea tiene presupuesto configurable de llamadas al modelo, tokens **cuando el proveedor los reporte**, coste estimado con fuente/tarifa configurada, tiempo, ejecuciones de herramientas e iteraciones de corrección. El runtime acumula consumo de agente principal y especialistas en la misma tarea; delegar no reinicia la cuenta. Define alertas y un límite duro operativo para tiempo/iteraciones/herramientas; un límite exacto de tokens o coste solo es garantizable cuando el adaptador dispone de medición y control suficientes. Si faltan, muestra «desconocido/estimado» y usa límites conservadores observables, sin inventar cifras. Tras dos intentos de corrección equivalentes sin avance, detiene el bucle y expone el bloqueo; se permite un nuevo intento cuando cambie la hipótesis, el contexto o la autorización y el usuario lo pida.

Para reducir contexto, el router transmite intención, índice breve de capacidades y fragmentos pertinentes; lee una skill completa únicamente al seleccionarla. El Context Manager recupera archivos y memoria con referencias, acota salidas largas y resume sesiones sin promover esos resúmenes a instrucciones. Una prueba, build o auditoría se ejecuta según riesgo y alcance; la evidencia vigente se reutiliza si no cambiaron sus entradas pertinentes. `/usage` muestra contadores reales de la sesión/tarea y, únicamente cuando el proveedor los exponga, cuotas o ventanas de cuenta; rotula por separado las estimaciones. No promete barras semanales o tokens empresariales si la integración no ofrece esos datos.

## Permisos, herramientas y aislamiento desde el MVP

Aura es una herramienta de ingeniería de software: puede investigar en internet, leer y editar código, crear espacios de trabajo aislados, ejecutar comandos y pruebas, administrar archivos de un proyecto y consultar servicios de desarrollo habilitados. **No registra herramientas de generación de imagen, video o audio** en el producto inicial; escribir código que manipule estos formatos sigue siendo una tarea de programación. Un modelo que redacte un comando no puede ejecutarlo sin pasar por el Tool Executor.

| Operación | Política inicial y condición de aprobación |
| --- | --- |
| Lectura/listado/búsqueda de archivos del proyecto abierto | Permitidos dentro de las raíces concedidas, salvo carpetas sensibles protegidas. El primer acceso a una carpeta sensible o a una raíz fuera del proyecto solicita un permiso específico para esa ruta. No recorrer el home entero por defecto. |
| Edición/creación dentro del proyecto | Permitidas según el modo de sesión del usuario, con protección de rutas sensibles; cambios fuera de raíces concedidas y sobrescrituras de archivos ajenos requieren autorización explícita. |
| Terminal y pruebas | Ejecución en sandbox con directorio de trabajo y rutas autorizadas, timeout, límites de salida/recursos y cancelación del árbol de procesos. Solicitar permiso para escape del sandbox, ejecución con efectos externos o comandos que accedan a rutas adicionales. La clasificación de comandos no se basa solo en su nombre o en la afirmación del modelo de que son seguros. |
| Internet | Consulta/búsqueda y descargas necesarias para programar a través de herramientas habilitadas, con política de red del entorno. Envíos de datos, publicación, instalación remota ejecutable, acceso a dominios no permitidos y acciones autenticadas con efectos externos requieren permisos separados. Una URL o contenido remoto nunca amplía autoridad. |
| Operaciones sensibles | Borrado masivo, cambios de permisos, secretos, credenciales, procesos privilegiados, push/publicación y acceso externo se someten a aprobación específica o se bloquean si el sandbox no permite imponer la restricción. |

La UI de aprobación ofrece **una vez**, **durante esta sesión** o **guardar regla revocable**. Cada regla persistente debe acotarse a raíz/camino, operación y, cuando corresponda, comando/argumentos o destino de red; no significa «todos los comandos para siempre». La concesión de sesión caduca al terminar; la persistente se puede listar y revocar. Los actos destructivos de alto impacto no reciben un permiso ilimitado por herencia: se vuelven a evaluar para su objetivo concreto. Si la operación es rechazada, Aura no la reintenta mediante otra herramienta o un subagente. Un permiso para *leer* una carpeta no autoriza *modificarla* ni *exfiltrarla*.

El sandbox y las comprobaciones de rutas se aplican por el ejecutor antes de invocar el proceso y durante los accesos que pueda controlar; validar ruta canónica, ancestros y symlinks evita escapes triviales. Para comandos arbitrarios, un filtrado textual o una lista de rutas en el prompt no constituye aislamiento: si la plataforma no ofrece aislamiento verificable para el alcance solicitado, Aura debe pedir autorización para ejecución sin ese aislamiento o rechazarla. Los secretos se obtienen mediante mecanismos seguros del sistema/proveedor, se ocultan de logs y memorias, y el usuario puede inspeccionar por qué una herramienta fue aprobada, denegada o ejecutada. Las skills y agentes externos nunca reciben más autoridad que la sesión.

## Primeras entregas y criterios

1. **MVP ejecutable:** CLI conversacional + runtime + integración real con un modelo autorizado + lectura/búsqueda/edición + terminal en sandbox verificable + controles de rutas, red, timeout y aprobaciones graduadas + configuración y sesiones aisladas por proyecto. Una consulta sencilla no ejecuta checks ni carga skills irrelevantes. La implementación inicial del router es directa y usa un modelo configurado.
2. **Capacidades y uso:** catálogo pequeño propio, manifiesto mínimo, perfil JavaScript y carga selectiva; comandos para listar, inspeccionar, habilitar, deshabilitar y ver uso real/estimado. Mostrar conteos por origen e impedir que una capacidad externa descubierta se active sola. Probar permisos una vez/sesión/persistentes, revocación, rechazo, aislamiento de proyecto y acceso a carpetas externas.
3. **Ingeniería transversal:** flujos `implement`/`review` proporcionales y ruta directa de explicación, probados primero con JavaScript. El contrato del flujo no contiene comandos ni reglas sintácticas de JavaScript. Dos intentos equivalentes sin avance se detienen.
4. **Evolución:** comprobar portabilidad con un segundo stack; después sumar modelos/proveedores, fallback, selección automática y especialistas según casos medidos. TUI, SDD más completo, memoria automática y coordinación paralela se incorporan en fases posteriores con criterios propios.

**Aceptación mínima del MVP:** (a) completar una tarea de edición y validación en un proyecto de prueba, (b) denegar una lectura fuera del proyecto hasta concederla y distinguir lectura de escritura, (c) pedir aprobación del alcance preciso de un comando y ejecutar/denegar según elección, (d) impedir que un proceso exceda los límites de sandbox/tiempo y mostrar la causa, (e) descubrir una skill externa sin cargarla, asignarla solo a un proyecto y comprobar los conteos, (f) mantener sesiones de dos repositorios separadas, (g) informar uso real y etiquetar lo desconocido sin inventar coste, (h) explicar comentarios de PR sin suite ni delegación automática, (i) verificar que no existe herramienta de generación multimedia. Estas pruebas requieren un ambiente controlado y evidencias reales; este ADR no afirma haberlas ejecutado.

## Pendientes de implementación

- El repositorio público ya está creado y contiene documentación inicial; todavía no existe un runtime ni una CLI funcional.
- Convertir los contratos de este ADR en especificaciones verificables antes de codear: `requirements`, `design`, `tasks` y validación. Abrir con un spike técnico acotado para probar el sandbox, los límites efectivos del proveedor, el almacenamiento de aprobaciones y la resolución de capacidades. El spike investiga factibilidad dentro del proceso SDD (Spec-Driven Development).
- Concretar esquema de manifiesto/configuración/sesión, precedencia, formato de rutas, permisos, plataforma de sandbox y matriz de pruebas. El contrato de protección y la UX de aprobación son obligatorios en el MVP aunque su implementación concreta se decida en el spike.
- Verificar los mecanismos de autenticación y configurar credenciales locales para OpenAI y Anthropic, los dos proveedores objetivo iniciales; una suscripción no implica acceso API.
- Implementar el runtime en JavaScript ESM con JSDoc, conforme a la decisión aprobada. La elección del lenguaje del runtime es independiente de que el perfil inicial sea JavaScript.
- Probar, antes de prometerlo, el descubrimiento e interpretación de formatos externos de cada harness; acordar instalación/actualización segura y desinstalación con dependencias. Las reglas propias de organizaciones y productos ajenos no forman parte del perfil inicial de Aura.
- Definir valores configurables iniciales y métricas para tiempo, presupuesto, coste e iteraciones; no inventar cuotas o cifras universales ni convertir un cálculo estimado en un límite garantizado.
- Definir formato y reglas de edición/eliminación de entradas de memoria al implementar su primera versión; comprobar aislamiento entre dos repositorios y evitar que una nota obsoleta suplante el código o un ADR vigente.

## Riesgos y límites

- Importar literalmente skills de otro sistema puede arrastrar dependencias y reglas no portables.
- Declarar capacidad instalada no equivale a cargarla; un prompt que siempre enumere todo aumentaría coste y ruido.
- Este ADR define una dirección y criterios; no demuestra que sus componentes estén implementados ni que funcionen en una plataforma concreta.
- Un agente revisor separado ayuda con juicios independientes en cambios sensibles, pero no es garantía automática de corrección ni requisito para cada edición.
- Declarar un flujo «agnóstico» tras probarlo solo en JavaScript sería prematuro: se verificará con otro stack y se corregirán las suposiciones ocultas que aparezcan.
- Resumir una sesión no equivale a autorizar su contenido como memoria; cruzar recuerdos entre proyectos o copiar secretos a notas persistentes sería una falla de aislamiento.
- Las instrucciones de un archivo, skill externa o salida web son entradas no confiables: no pueden crear permisos persistentes ni activar herramientas excluidas.
- Un permiso de lectura de internet no autoriza publicaciones ni envío de código privado; el enforcement debe comprobarse en herramientas reales y en el sandbox, no solo en texto de prompts.
