# AGENTS.md — LA TROUPE

Guía para cualquier agente de IA (o persona) que trabaje en este proyecto. **Leer primero este archivo, después `STATUS.md` y después `PLAN.md`.** Con esos tres basta para retomar sin que el usuario repita nada.

## 1. Qué es el proyecto

Plataforma web para una academia de danza (LA TROUPE) que vende **cursos on demand** (video, texto, diapositivas) y, a futuro, funcionará como:

- **Marketplace** donde profesores invitados venden sus propios cursos bajo la marca de la academia (con comisión).
- **Proveedor de contenido** para otros públicos: profesores que se especializan y academias que usan el material en clases presenciales (licencias).
- **SaaS multi-academia**: otras academias usan la plataforma, cobran en sus propias cuentas y pagan una licencia a los desarrolladores.

LA TROUPE es la primera academia y **no se cobra a sí misma**. Lo de SaaS es a futuro, pero la arquitectura no debe obligar a rehacer nada cuando llegue.

## 2. Personas y forma de trabajo

- Trabajan el **usuario** (dueño del producto, habla español rioplatense) y **un agente de IA**. No hay equipo externo. No hay plazo fijo: prima hacerlo bien.
- Comunicación y documentos: **español**. Código, nombres de tablas y variables: **inglés**.
- Hay un cambio de modelo previsto (de Sonnet a Opus). Por eso **todo el contexto vive en archivos**, no en la conversación.

## 3. Fuentes de verdad (en orden de autoridad)

| Archivo | Para qué |
|---|---|
| `Arquitectura_LA_TROUPE_v3.3.md` | Arquitectura vigente. Manda sobre la v3.2 donde se contradicen |
| `Arquitectura_LA_TROUPE_v3.2.md` | Base: tablas y secciones que la v3.3 no cambia |
| `PLAN.md` | Pasos en orden, con criterios para pasar al siguiente |
| `STATUS.md` | Dónde estamos hoy, qué sigue, qué está abierto |
| `CHANGELOG.md` | Historial de cambios del proyecto y de los documentos |

El original de la v3.2 es `Arquitectura_LA_TROUPE_v3.2.docx` (en Descargas). El `.md` es su conversión y es el que se trabaja. El `.pdf` es solo para lectura humana.

## 4. Reglas para agentes

1. **No escribir código de producto hasta que `PLAN.md` marque la puerta de la Fase 0 como abierta.** Hasta entonces se trabaja en documentos y verificaciones técnicas descartables.
2. **No inventar decisiones de negocio.** Precios, comisiones, políticas de reembolso, edades, textos legales: si no está decidido, se anota en `STATUS.md` → "Decisiones abiertas" y se pregunta.
3. **Verificar antes de afirmar** cómo funcionan servicios externos (MercadoPago, Stripe, Bunny, Supabase). Citar la fuente y marcar lo no confirmado como "pendiente de verificar".
4. **Al terminar cada sesión de trabajo**: actualizar `STATUS.md` (qué quedó hecho, qué sigue) y agregar una entrada a `CHANGELOG.md`. Es obligatorio.
5. **No guardar secretos** (claves, tokens, contraseñas) en archivos del repo, en documentos ni en la conversación. Usar variables de entorno y los gestores de secretos del proveedor.
6. **No hacer commits, pushes ni acciones externas** (crear cuentas, publicar, cobrar) sin pedido explícito del usuario.
7. Ante una contradicción entre documentos, avisar al usuario y proponer cuál corregir. No elegir en silencio.
8. Las respuestas al usuario son claras y sin jerga innecesaria. Explicar el *por qué* de cada recomendación y dar una recomendación concreta, no un menú.

## 5. Decisiones de arquitectura que no se discuten sin motivo fuerte

Resumen de la v3.3. El detalle y las tablas están en el documento.

- **Multi-tenant desde el día uno**: `organization_id` en las tablas relevantes y Row-Level Security en PostgreSQL. Hoy hay una sola organización.
- **Identidad global**: un email único en toda la plataforma; las relaciones con cada academia se modelan con `memberships`.
- **Cuenta ≠ alumno**: `users` y `student_profiles` son entidades distintas (una cuenta puede administrar varios alumnos, p. ej. padres).
- **Roles y permisos editables** (`roles`, `permissions`, `role_permissions`, `membership_roles`). No hay roles fijos en enums.
- **Jerarquía de contenido**: carrera → curso → lección → bloques (video, texto, diapositivas, descarga).
- **Se vende un producto**, no un curso directamente. El producto tiene precios, modalidad de cobro y política de acceso.
- **Reglas como datos**: liberación de contenido (`release_rules`), evaluación y egreso (`completion_rules`) se configuran por curso desde el panel; no se programan por curso.
- **Modalidades de avance**: libro abierto o curricular (por tiempo, por examen, o ambos), con reloj individual por alumno.
- **Cuatro modalidades de cobro** modeladas desde el inicio: pago único, cuotas con tarjeta (las maneja el gateway), suscripción y plan de pagos propio.
- **Acceso**: el titular puede ser un alumno **o una organización** (licencias).
- **Dinero inmutable**: un pago no se edita; reembolsos y ajustes son registros nuevos. Cada pago registra su reparto (`payment_splits`).
- **Reparto a profesores**: automático (división en el gateway) o manual; conviven y se definen por profesor.
- **Contenido aprobado = congelado**: los cambios se piden como solicitud y se aprueban.
- **Captura de datos desde el día uno** (progreso, visualizaciones, eventos): lo que no se captura hoy no se recupera mañana.
- **Lo caro de cambiar se reserva ahora; lo independiente se crea cuando toca** (sección 1.2 de la v3.3).

## 6. Stack propuesto (sujeto a confirmación en `PLAN.md`, paso 3)

Next.js + TypeScript sobre Vercel; Supabase (PostgreSQL con RLS, Auth, Storage, Edge Functions, `pg_cron`); Bunny.net Stream para video con Vidstack como reproductor; MercadoPago primero y Stripe en una fase posterior; Resend para email; Sentry para errores. **No está decidido definitivamente hasta que `STATUS.md` lo registre como decidido.**

## 7. Convenciones (a completar en la Fase 0)

- Nombres de tablas y columnas en `snake_case`, en inglés, en plural para tablas.
- Todas las tablas: `id` (UUID), `created_at`, `updated_at`; las del dominio, `organization_id`.
- Toda tabla con datos de organización tiene RLS activo **y una prueba automática** que demuestra el aislamiento.
- Cambios de base de datos solo mediante migraciones versionadas.
- Cada decisión técnica importante se registra como una nota breve (qué, por qué, alternativas) en `docs/decisions/` cuando exista el repositorio.

## 8. Cómo retomar una sesión

1. Leer `AGENTS.md`, `STATUS.md`, `PLAN.md`.
2. Leer `CHANGELOG.md` solo si hace falta el historial.
3. Abrir la v3.3 solo en las secciones relevantes a la tarea (es larga).
4. Confirmar con el usuario en qué paso del plan se trabaja hoy.
5. Al cerrar: actualizar `STATUS.md` y `CHANGELOG.md`.
