# PLAN.md — De la idea al primer código

Este plan responde a "¿qué hay que pensar primero?". El orden importa: **cada paso reduce el riesgo del siguiente**. Programar es el último paso, no el primero.

Cada paso tiene una **puerta**: criterios que deben cumplirse antes de pasar al siguiente. Los pasos 1 y 2 avanzan juntos (las respuestas de negocio alimentan los requisitos). El 3 depende del 2. Del 4 en adelante, en orden.

Estado actual de cada paso: ver `STATUS.md`.

## Mapa

| # | Paso | Pregunta que responde | Estado |
|---|---|---|---|
| 1 | Propuesta comercial y alcance del MVP | ¿Qué vendemos, a quién, a qué precio? | En curso |
| 2 | Requisitos y criterios de calidad | ¿Qué tiene que hacer el sistema y qué importa más? | Pendiente |
| 3 | Opciones de arquitectura y stack, evaluadas | ¿Qué alternativas hay y cuál gana según los criterios? | Pendiente (investigación de herramientas hecha) |
| 4 | Verificaciones técnicas (*spikes*) | ¿Lo que asumimos realmente funciona? | Pendiente |
| 5 | Experiencia de usuario y marca | ¿Cómo se ve y se usa? | Pendiente |
| 6 | Legal, fiscal y cuentas de cobro | ¿Podemos cobrar y operar legalmente? | Pendiente |
| 7 | Entorno y repositorio | ¿Dónde y cómo se trabaja? | Pendiente |
| 8 | Fase 0: cimientos | Esquema, permisos, pruebas | Bloqueado |
| 9 | Fase 1A: producto mínimo vendible | Primera venta real | Bloqueado |

---

## Paso 1 — Propuesta comercial y alcance del MVP

**Por qué primero:** todo lo demás (tecnología, precios, pagos, diseño) depende de qué se vende y a quién. Un error aquí cuesta más que cualquier error técnico.

Qué definir (y escribir en un documento corto de 2–3 páginas):

- **Cliente inicial**: ¿alumnos de danza en Uruguay y Latinoamérica? ¿Adultos, jóvenes, niños con padres? ¿Qué problema concreto resuelve la plataforma para ellos que no resuelve YouTube o una clase presencial?
- **Producto inicial**: ¿cuántos cursos y de qué estilos estarán listos al lanzar? ¿Hay una carrera curricular desde el día uno, o solo cursos sueltos?
- **Modelo de precios inicial**: ¿pago único, suscripción, ambos? Rango de precios y moneda. (Decisiones D1 y D2.)
- **Diferencial**: reproductor pensado para danza (velocidad, espejo, capítulos), avance curricular, certificado de egreso, profesores reconocidos.
- **Cómo llegan los primeros alumnos**: base actual de la academia, redes, referidos. Sin esto el producto está listo pero nadie lo ve.
- **Competencia**: revisar 3–5 plataformas de danza on demand (precios, catálogo, reproductor, qué les falta).
- **Métrica de éxito del MVP**: por ejemplo, "20 ventas pagadas en los primeros 60 días" (el número lo define el usuario).

**Puerta:** documento de propuesta aprobado por el usuario; decisiones D1, D2 y D8 resueltas.

## Paso 2 — Requisitos y criterios de calidad

**Por qué antes que la arquitectura:** la v3.2 y la v3.3 son prototipos que eligieron soluciones sin escribir primero contra qué se medían. Una arquitectura solo es "buena" respecto de requisitos y prioridades concretas. Sin eso no hay forma de saber si es la mejor.

Entregable: `docs/requisitos.md`, con tres partes.

**2.1 Capacidades (qué hace el sistema).** Lista de lo que debe poder hacer, en lenguaje de negocio, separada en *MVP*, *después* y *a futuro*. Sale de las conversaciones con el usuario (resumidas en `STATUS.md` → "Decisiones tomadas") y del Paso 1. Ejemplos: vender un curso con pago único; liberar un módulo al mes y al aprobar un examen; un profesor invitado publica un curso que queda congelado al aprobarse.

**2.2 Atributos de calidad (cómo tiene que comportarse), ordenados por prioridad.** Cada uno con un número medible, aunque sea estimado:

| Atributo | Pregunta a responder |
|---|---|
| Escala | ¿Cuántos alumnos, cursos, horas de video y academias en 1, 3 y 5 años? ¿Cuál es el pico de personas **simultáneas** viendo video bajo demanda y en vivo? (Meta del usuario: 5.000 simultáneos viendo video on demand; las clases en vivo serán de grupos reducidos) |
| Video | ¿Cuántas horas se suben por mes? ¿Cuánto se ve? ¿Desde qué países? ¿Qué nivel de protección contra copia? |
| Costo | ¿Cuánto se puede gastar por mes en infraestructura al inicio y con 1.000 alumnos? |
| Equipo | El equipo son el usuario y un agente de IA. ¿Qué se puede mantener con eso? ¿Se sumará gente después? |
| Seguridad y privacidad | Datos de menores, datos de pago, aislamiento entre academias |
| Disponibilidad | ¿Cuánta caída es tolerable? ¿Hay clases en vivo con hora fija? |
| Dependencia de proveedores | ¿Qué tan grave es depender de un servicio que puede subir precios o cerrar? |
| Velocidad de entrega | ¿Cuánto importa llegar rápido a la primera venta frente a construir todo bien? |
| Internacionalización | Monedas, idiomas, impuestos, países |
| Evolución | Sumar academias, profesores y modalidades sin reescribir |

**2.3 Restricciones.** Lo que no se negocia: país de operación, gateways disponibles, presupuesto, conocimientos del equipo, licencias aceptables (por ejemplo, nada AGPL dentro del producto).

**Puerta:** el usuario aprueba `docs/requisitos.md` y el orden de prioridad de los atributos de calidad.

## Paso 3 — Opciones de arquitectura y stack, evaluadas

**Método:** para cada decisión estructural se listan al menos 2 o 3 alternativas reales, se las puntúa contra los atributos del Paso 2 (con su peso) y se registra la decisión como ADR (registro de decisión de arquitectura) en `docs/decisions/`: contexto, opciones, criterios, elegida, por qué, y qué haría cambiar la decisión. Lo que no se pueda decidir con información, se decide con una prueba del Paso 4.

Decisiones a tomar (las alternativas son el punto de partida, no una lista cerrada):

| Decisión | Alternativas a comparar |
|---|---|
| Forma general del sistema | Monolito modular · servicios separados · plataforma existente extendida |
| Backend | BaaS (Supabase, Firebase, Appwrite) · backend propio (NestJS, Django, Laravel, Rails, .NET) · híbrido |
| Base de datos | PostgreSQL · otras |
| Multi-tenancy | Esquema compartido con RLS · esquema por academia · base por academia |
| Autenticación | La del BaaS · Auth0/Clerk/WorkOS · propia con librería |
| Frontend | Next.js · Remix/React Router · Nuxt/SvelteKit · app móvil desde el inicio o no |
| Video | Bunny.net Stream · Mux · Cloudflare Stream · Vimeo · AWS (S3 + MediaConvert + CloudFront) |
| Pagos | MercadoPago directo · orquestador (dLocal, Rebill, otros) · Stripe primero |
| Hosting | Vercel · Cloudflare · Railway/Render/Fly · VPS · AWS/GCP |
| Lógica de reglas (liberación, egreso) | En la base · en el backend · motor de reglas |
| Organización del código | Repositorio único (monorepo) · varios |

Herramientas ya investigadas que sirven de insumo: tabla siguiente y `STATUS.md` → "Hechos verificados". **El análisis de infraestructura y costos ya hecho (`docs/infraestructura.md`) cubre hosting, base de datos, autenticación, archivos, video y VPS**: las ADR de esas piezas parten de él.

**Principio sobre reutilizar:** reutilizar piezas, no copiar productos enteros. Ningún LMS conocido encaja con el modelo completo, y los bien calificados son AGPL o GPL.

| Pieza | Opción | Decisión propuesta | Notas |
|---|---|---|---|
| Base de la aplicación | Next.js + TypeScript | Usar, desde cero con el starter oficial de Next.js | Sin plantillas ajenas con modelos de datos distintos |
| Autenticación, base de datos, almacenamiento | Supabase | Usar | RLS nativo para multi-tenant |
| Patrón de multi-tenancy con RLS | Basejump Core (`database.dev`) | **Estudiar como referencia**; decidir si se adopta la extensión o se replica el patrón | v2.0.1; el starter de Next.js asociado es legado |
| Componentes de interfaz | shadcn/ui + Tailwind | Usar | Copiables, sin dependencia |
| Reproductor | Vidstack | Usar | Verificar integración con Bunny en el spike |
| Video | Bunny.net Stream | Candidato; confirmar en el spike | Genera previews animados (`preview.webp`) |
| Pagos | MercadoPago (SDK oficial) | Usar; Stripe en una fase posterior | |
| LMS completo de código abierto (EaseLMS, Moodle, otros) | — | **No usar como base** | EaseLMS es AGPL-3.0: obligaría a publicar las modificaciones del servicio. Moodle y similares no encajan con el modelo marketplace. Útil solo como referencia de qué funciones tiene un LMS |
| Kits comerciales de LMS (plantillas pagas) | — | Evaluar solo si ahorran meses de interfaz | No se verificó su calidad ni su estado |
| Referencia de un sistema casi igual | `guillermoscript/lms-front` (MIT) | **Estudiar a fondo antes del Paso 8**: esquema, RLS multi-escuela, monetización con reparto, exámenes y certificados | Ver evaluación en `STATUS.md`. Comparar su modelo con la v3.3 y anotar qué adoptar |

**Cómo se decide:** una tabla de decisión por pieza (opción elegida, por qué, alternativas descartadas) guardada en `docs/decisions/` cuando exista el repositorio.

**Reglas de escala que la v4 debe incluir desde el día uno** (baratas al principio, carísimas de agregar después; ver cálculo en `STATUS.md` → "¿Escala a 5.000 simultáneos?"):

1. El video lo entrega siempre la red de distribución (CDN) del proveedor; nuestros servidores solo autorizan y registran.
2. El permiso de reproducción se verifica una vez por sesión de video (enlace firmado con vencimiento), no en cada fragmento.
3. El progreso de reproducción se envía en lotes (cada 30–60 s y al pausar o terminar) a una tabla de solo agregado, particionada por mes.
4. Toda conexión a la base desde funciones serverless pasa por el *pooler* en modo transacción.
5. Las páginas públicas (inicio, catálogo, ficha de curso) se sirven cacheadas; no consultan la base en cada visita.
6. Reportes pesados sobre réplica de lectura o tablas preagregadas, nunca sobre la base principal en horario pico.
7. Webhooks de pago idempotentes y encolados.
8. Límites de frecuencia y anti-bots (Turnstile) en registro, inicio de sesión y checkout.
9. `organization_id` en todas las tablas: una academia grande se puede mover a su propia base sin rediseñar.
10. Prueba de carga en staging antes de cada lanzamiento grande (spike S8).

**Después de las ADR:** se escribe la arquitectura nueva (v4), desde los requisitos y las decisiones. La v3.3 se usa como lista de ideas: cada tabla o mecanismo se conserva solo si responde a un requisito.

**Puerta:** cada decisión tiene su ADR aprobada por el usuario; la v4 está escrita y revisada por una segunda mirada independiente; no queda nada "a ver cómo sale".

## Paso 4 — Verificaciones técnicas (*spikes*)

Pruebas **descartables** (código tirable, no del producto) para eliminar suposiciones peligrosas. Cada una responde sí/no/depende.

| Spike | Pregunta | Criterio de éxito |
|---|---|---|
| S1 — Video: Mux vs Bunny, con Vidstack | ¿Cuál reproduce mejor video protegido (enlace firmado) con velocidad, espejo y bucle? ¿Cuánto cuesta con la mezcla real de audiencia (Latinoamérica / Europa)? | El mismo video probado en los dos; decisión registrada |
| S2 — Clip al pasar el mouse | ¿El `preview.webp` de Bunny sirve para el efecto, o hay que subir un clip aparte? ¿Se puede lograr 15–60 s? | Decisión documentada |
| S3 — MercadoPago cobro | ¿Pago único, cuotas con tarjeta y **suscripción recurrente** funcionan en Uruguay y con qué requisitos? | Un pago de prueba de cada tipo |
| S4 — MercadoPago reparto | ¿La división automática (marketplace + OAuth del vendedor) está disponible en Uruguay? ¿Qué pasa en un reembolso? | Respuesta documentada; si es "no", se cambia el plan de la Fase 2 |
| S5 — RLS multi-tenant | ¿El patrón elegido aísla organizaciones y aplica permisos por rol con buen rendimiento? | Pruebas automáticas que intentan leer datos de otra organización y fallan |
| S6 — Liberación de contenido | ¿El motor de reglas (tiempo, examen, combinado) se evalúa bien con un caso real de carrera? | Escenario "Julia y Pedro" de la v3.3 (13.1) pasa en pruebas |
| S9 — Hosting: Vercel vs Cloudflare Workers | ¿Cuánta fricción tiene desplegar el mismo esqueleto de Next.js en Vercel y en Cloudflare Workers (OpenNext)? | Los dos despliegues funcionando; fricción y costos medidos; decisión registrada |
| S10 — Video propio en R2 | ¿Los términos vigentes de Cloudflare permiten servir video desde R2 a alto volumen sin contrato Enterprise? ¿Funciona el circuito ffmpeg → HLS cifrado → R2 → CDN → Vidstack con enlaces firmados? | Términos confirmados por escrito; prototipo reproduciendo; costo por 1.000 minutos medido |
| S8 — Prueba de carga | ¿El sistema en staging aguanta el pico objetivo de usuarios simultáneos (reproducción, progreso, navegación, checkout)? | Herramienta de carga (por ejemplo k6) simulando el pico; tiempos de respuesta y errores dentro de lo definido en el Paso 2. Se repite antes de cada lanzamiento grande |
| S7 — Cobro internacional desde Uruguay | ¿Stripe admite una empresa uruguaya? Si no: ¿qué *Merchant of Record* (Lemon Squeezy, Paddle, Hotmart) acepta cursos de danza, cuánto cobra y cómo paga en Uruguay? | Camino definido para vender en España, con costos y requisitos |

**Puerta:** S1, S3, S4 y S5 resueltos. S2 y S6 pueden cerrarse durante la Fase 1A.

## Paso 5 — Experiencia de usuario y marca

- Mapa de pantallas del MVP: catálogo, ficha de curso, checkout, "Mis cursos", reproductor, panel de cursos, panel de alumnos, pagos.
- Wireframes de baja fidelidad de las 8–10 pantallas críticas (primero en papel o en una herramienta simple).
- Nombre de dominio y marca de la plataforma (D9). Colores, tipografía y logo mínimos.
- Textos claves: propuesta de valor en la portada, mensajes de acceso bloqueado ("se desbloquea el 15 de marzo"), correos transaccionales.

**Puerta:** wireframes aprobados por el usuario.

## Paso 6 — Legal, fiscal y cuentas

- Forma jurídica con la que se cobra (D8) y cuenta de MercadoPago a nombre de esa entidad.
- Términos y condiciones, política de privacidad y de reembolsos (D6).
- Tratamiento de datos de menores y consentimientos (D7).
- Contador: facturación electrónica local y, antes de vender en España, IVA de servicios digitales.
- Contrato modelo con profesores invitados (comisión, propiedad intelectual, uso de imagen, qué pasa si se retira un curso).
- Cuentas de proveedores en modo de prueba: Supabase, Vercel, Bunny.net, MercadoPago, Resend, Sentry.

**Puerta:** cuenta de cobro lista; textos legales mínimos redactados por un profesional; cuentas de prueba creadas.

## Paso 7 — Entorno y repositorio

- **Repositorio git propio** para el proyecto (hoy la carpeta cuelga de un repositorio que abarca todo el directorio del usuario; no usarlo).
- Estructura inicial: `/docs` (documentos y decisiones), `/supabase` (migraciones), `/web` (aplicación).
- CI mínimo: lint, tipos y pruebas en cada cambio.
- Entornos separados: desarrollo local, *staging* y producción, cada uno con su proyecto de Supabase.
- Gestión de secretos en variables de entorno de cada plataforma.
- Copiar `AGENTS.md`, `STATUS.md`, `PLAN.md`, `CHANGELOG.md` y las arquitecturas al repositorio.

**Puerta:** el repositorio compila "hola mundo", despliega en staging y corre una prueba en CI.

## Paso 8 — Fase 0: cimientos

Detalle en la arquitectura v4 (a escribir en el Paso 3). El roadmap de la v3.3, sección 11, es solo un borrador.

1. Migraciones de identidad, organizaciones, membresías, roles y permisos.
2. Columnas reservadas en `courses`, `access`, `prices`.
3. Políticas RLS con pruebas automáticas de aislamiento y permisos.
4. Roles de fábrica cargados (Owner, Administrador, Profesor, Profesor invitado, Revisor, Finanzas, Atención, Alumno).
5. Registro e inicio de sesión funcionando contra el esquema real.

**Puerta:** las pruebas de aislamiento y permisos pasan en CI. Nada de interfaz vendible todavía.

## Paso 9 — Fase 1A: producto mínimo vendible

Detalle en la arquitectura v4 (a escribir en el Paso 3). El roadmap de la v3.3, sección 11, es solo un borrador. Termina con una **primera venta real** de un curso de LA TROUPE.

## Después

Fase 1B (evaluación y egreso) → Fase 2 (profesores y reparto) → Fase 3 (Stripe y España) → Fase 4 (licencias para academias) → Fase 5 (retención y app móvil) → Fase 6 (clases en vivo y presenciales) → Fase 7 (SaaS multi-organización).

---

## Cambio de modelo de IA (Sonnet → Opus)

Para no repetir contexto:

1. Abrir la carpeta `La Troupe` con el nuevo modelo.
2. Decirle: "Leé `AGENTS.md`, `STATUS.md` y `PLAN.md`, y seguí desde donde quedamos."
3. Antes de cambiar, pedirle al modelo actual que actualice `STATUS.md` y `CHANGELOG.md`.

## Cómo se mantiene este plan

- Cada paso se marca como Hecho, En curso o Pendiente en la tabla del Mapa.
- Si un spike cambia una decisión, se registra en `CHANGELOG.md` y se actualiza la arquitectura (v3.4).
- Las duraciones no se estiman todavía. Se calcularán después del Paso 4, cuando se sepa cuántas suposiciones eran falsas.
