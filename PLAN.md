# PLAN.md — De la idea al primer código

Este plan responde a "¿qué hay que pensar primero?". El orden importa: **cada paso reduce el riesgo del siguiente**. Programar es el último paso, no el primero.

Cada paso tiene una **puerta**: criterios que deben cumplirse antes de pasar al siguiente. Los pasos 1, 2 y 3 pueden avanzar en paralelo; del 4 en adelante, en orden.

Estado actual de cada paso: ver `STATUS.md`.

## Mapa

| # | Paso | Pregunta que responde | Estado |
|---|---|---|---|
| 1 | Propuesta comercial y alcance del MVP | ¿Qué vendemos, a quién, a qué precio? | En curso |
| 2 | Arquitectura y modelo de datos | ¿Cómo se estructura para no rehacer? | Hecho (v3.3), pendiente de revisión |
| 3 | Qué construir y qué reutilizar | ¿Qué ya existe para no empezar de cero? | En curso |
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

## Paso 2 — Arquitectura y modelo de datos

**Estado:** escrita (`Arquitectura_LA_TROUPE_v3.3.md`).

Pendiente:
- El usuario lee la v3.3 y marca desacuerdos.
- Revisión por una segunda mirada técnica independiente (otra persona o una revisión con un modelo distinto) **antes** de escribir SQL.
- Cerrar el esquema SQL de la Fase 0 (ver Paso 8).

**Puerta:** v3.3 aprobada por el usuario; lista de cambios incorporada en una v3.4 si hace falta.

## Paso 3 — Qué construir y qué reutilizar

**Principio:** reutilizar piezas, no copiar productos enteros. El producto es un *marketplace* de cursos con avance curricular, reparto a profesores y multi-academia. Ningún LMS conocido encaja bien con eso, y los que se evaluaron tienen condiciones que complican reutilizarlos.

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

**Cómo se decide:** una tabla de decisión por pieza (opción elegida, por qué, alternativas descartadas) guardada en `docs/decisions/` cuando exista el repositorio.

**Puerta:** cada pieza tiene una decisión registrada; no queda ninguna "a ver cómo sale".

## Paso 4 — Verificaciones técnicas (*spikes*)

Pruebas **descartables** (código tirable, no del producto) para eliminar suposiciones peligrosas. Cada una responde sí/no/depende.

| Spike | Pregunta | Criterio de éxito |
|---|---|---|
| S1 — Bunny + Vidstack | ¿Se reproduce video protegido (HLS cifrado + token) en el reproductor con velocidad y espejo? | Video reproducible solo con enlace firmado; espejo y velocidad funcionando |
| S2 — Clip al pasar el mouse | ¿El `preview.webp` de Bunny sirve para el efecto, o hay que subir un clip aparte? ¿Se puede lograr 15–60 s? | Decisión documentada |
| S3 — MercadoPago cobro | ¿Pago único, cuotas con tarjeta y **suscripción recurrente** funcionan en Uruguay y con qué requisitos? | Un pago de prueba de cada tipo |
| S4 — MercadoPago reparto | ¿La división automática (marketplace + OAuth del vendedor) está disponible en Uruguay? ¿Qué pasa en un reembolso? | Respuesta documentada; si es "no", se cambia el plan de la Fase 2 |
| S5 — RLS multi-tenant | ¿El patrón elegido aísla organizaciones y aplica permisos por rol con buen rendimiento? | Pruebas automáticas que intentan leer datos de otra organización y fallan |
| S6 — Liberación de contenido | ¿El motor de reglas (tiempo, examen, combinado) se evalúa bien con un caso real de carrera? | Escenario "Julia y Pedro" de la v3.3 (13.1) pasa en pruebas |

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

Detalle en la v3.3, sección 11.

1. Migraciones de identidad, organizaciones, membresías, roles y permisos.
2. Columnas reservadas en `courses`, `access`, `prices`.
3. Políticas RLS con pruebas automáticas de aislamiento y permisos.
4. Roles de fábrica cargados (Owner, Administrador, Profesor, Profesor invitado, Revisor, Finanzas, Atención, Alumno).
5. Registro e inicio de sesión funcionando contra el esquema real.

**Puerta:** las pruebas de aislamiento y permisos pasan en CI. Nada de interfaz vendible todavía.

## Paso 9 — Fase 1A: producto mínimo vendible

Detalle en la v3.3, sección 11. Termina con una **primera venta real** de un curso de LA TROUPE.

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
