# STATUS.md — LA TROUPE

**Última actualización:** 2026-10-28
**Fase actual:** Descubrimiento y definición (antes de la Fase 0). **No hay código todavía.**

## Dónde estamos

La arquitectura de datos está definida en la v3.3 (sobre la base de la v3.2). Falta cerrar la parte de negocio, elegir tecnología con verificaciones concretas y preparar el entorno. Los pasos están en `PLAN.md`; el paso en curso es el **Paso 1** (propuesta comercial y alcance del MVP) en paralelo con el Paso 3 (qué reutilizar).

## Hecho

- Conversión de la v3.2 de `.docx` a `.md` y `.pdf`.
- Revisión crítica de la v3.2 contra los objetivos del usuario.
- Arquitectura v3.3 escrita (`Arquitectura_LA_TROUPE_v3.3.md` y `.pdf`).
- `AGENTS.md`, `STATUS.md`, `CHANGELOG.md`, `PLAN.md` y `CLAUDE.md` creados.
- Primera investigación de qué reutilizar y qué verificar (ver "Hechos verificados").
- Repositorio propio creado: https://github.com/MROFER/La-Troupe-Dance-Platform (rama `main`). **Es público** (verificado el 2026-10-28): pendiente que el usuario lo pase a privado. La carpeta local `La Troupe` es la raíz del repositorio. Todavía tiene solo documentos; la estructura `/docs`, `/supabase`, `/web` del Paso 7 está pendiente.

## En curso

- Paso 1 de `PLAN.md`: propuesta comercial y alcance del MVP.
- Paso 3 de `PLAN.md`: decidir qué se construye y qué se reutiliza.

## Siguiente

1. El usuario revisa la v3.3 y marca lo que no coincide.
2. Cerrar la lista de "Decisiones abiertas" que bloquean el MVP (marcadas con ⛔).
3. Hacer los *spikes* técnicos del Paso 4 (Bunny, MercadoPago, RLS).

## Decisiones tomadas (resumen; detalle en la v3.3 sección 15)

- LA TROUPE es la primera academia y no se cobra a sí misma; el SaaS es futuro.
- Primero se construye la venta de cursos on demand; las clases en vivo y presenciales van después.
- Avance: libro abierto o curricular (tiempo, examen o ambos), con reloj individual por alumno.
- Cobros: se modelan las cuatro modalidades (único, cuotas, suscripción, plan de pagos). MercadoPago primero; Stripe después.
- Pago a profesores: automático por división de pago, o manual. Se define por profesor.
- Cada academia futura cobra en su propia cuenta.
- El modelo prevé desde ya licencias de contenido para academias.
- Reparto automático preferido sobre el manual para evitar carga administrativa.

## Decisiones abiertas

| # | Tema | Bloquea | Quién decide |
|---|---|---|---|
| D1 ⛔ | Propuesta comercial: qué se vende primero, a quién y a qué precio inicial | MVP | Usuario |
| D2 ⛔ | Tipos de suscripción y planes iniciales | Fase 1A | Usuario |
| D3 | Si el reloj del alumno sigue corriendo con acceso pausado por impago | Fase 1A | Usuario |
| D4 | Porcentajes de comisión por tipo de profesor | Fase 2 | Usuario |
| D5 | Diseño del certificado y firmantes | Fase 1B | Usuario |
| D6 | Política de reembolsos de cursos on demand | Fase 1A | Usuario + asesoría |
| D7 | Edad mínima para cuenta propia y consentimiento de padres | Fase 1A | Usuario + asesoría legal |
| D8 ⛔ | Forma jurídica y cuenta de MercadoPago con la que se cobrará | Fase 1A | Usuario |
| D9 | Nombre de dominio y marca de la plataforma (el repositorio se llama `La-Troupe-Dance-Platform`; la marca comercial sigue abierta) | Fase 1A | Usuario |
| D10 | Proveedor de video definitivo (Bunny.net propuesto) | Fase 0 | Se decide tras el spike |

## Hechos verificados (2026-10-28)

- **Bunny.net Stream** genera una animación de vista previa por video en WebP/GIF (`preview.webp`) y una versión de mayor calidad en WebM/MP4. Sirve de base para el clip al pasar el mouse. Falta probar calidad y duración (¿se puede configurar a 15–60 s?) → spike.  
  Fuente: https://docs.bunny.net/stream/storage-structure.md
- **MercadoPago** documenta pagos divididos para marketplace: cada vendedor autoriza por OAuth y el pago lleva una comisión del marketplace. La documentación específica de **Uruguay** es escasa (solo describe Checkout API con `application_fee`). Los reembolsos dependen del saldo del vendedor. → spike obligatorio antes de la Fase 2.  
  Fuente: https://www.mercadopago.com.uy/developers/en/docs/split-payments/split-1-1/integration-configuration/integrate-marketplace
- **Basejump Core** (extensión de Supabase para cuentas, equipos y permisos con RLS) existe en `database.dev`, versión 2.0.1. El starter de Next.js asociado es legado. Sirve como **referencia** del patrón de multi-tenancy.  
  Fuente: https://database.dev/basejump/basejump_core
- **EaseLMS** es un LMS de código abierto con Next.js + Supabase, licencia **AGPL-3.0**. No se evaluó a fondo. La AGPL obliga a publicar el código de cualquier versión modificada que se ofrezca como servicio, lo que choca con un SaaS comercial. → solo como referencia de estudio, no como base.  
  Fuente: https://github.com/enyojoo/easelms

## Riesgos abiertos

- Reparto automático por país (MercadoPago Uruguay) sin confirmar.
- Plan de pagos propio: cobros programados, reintentos y conciliación son lo más delicado.
- IVA y fiscalidad de venta digital a España: requiere contador.
- Datos de menores (notas, salud, fotos): requiere asesoría legal.
- Volumen de tablas alto: se crean por fase, con pruebas de permisos desde el inicio.

## Notas para quien retome

- El usuario quiere "hacerlo bien" y sin rehacer: prefiere decidir la arquitectura completa antes de programar.
- El usuario escribe en español rioplatense y mezcla ideas en un mismo mensaje; conviene devolverle un resumen ordenado y preguntas concretas.
- No se estimaron plazos. La secuencia de fases es firme; las duraciones no.
