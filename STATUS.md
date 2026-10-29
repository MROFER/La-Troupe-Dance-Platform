# STATUS.md — LA TROUPE

**Última actualización:** 2026-10-28
**Fase actual:** Descubrimiento y definición (antes de la Fase 0). **No hay código todavía.**

## Dónde estamos

**Decisión del usuario (2026-10-28): la v3.2 y la v3.3 son prototipos.** Ni la arquitectura ni el stack están decididos. Se van a definir desde cero: primero requisitos y criterios de calidad (Paso 2), después alternativas evaluadas con registros de decisión (Paso 3), y recién entonces una arquitectura v4. Los prototipos quedan como insumo. El trabajo en curso son los **Pasos 1 y 2** de `PLAN.md`, juntos: propuesta comercial y requisitos.

## Hecho

- Conversión de la v3.2 de `.docx` a `.md` y `.pdf`.
- Revisión crítica de la v3.2 contra los objetivos del usuario.
- Arquitectura v3.3 escrita (`Arquitectura_LA_TROUPE_v3.3.md` y `.pdf`).
- `AGENTS.md`, `STATUS.md`, `CHANGELOG.md`, `PLAN.md` y `CLAUDE.md` creados.
- Primera investigación de qué reutilizar y qué verificar (ver "Hechos verificados").
- Repositorio propio creado: https://github.com/MROFER/La-Troupe-Dance-Platform (rama `main`). **Es público por decisión del usuario** (2026-10-28), para evitar problemas en el despliegue. Se puede revisar más adelante. La carpeta local `La Troupe` es la raíz del repositorio. Todavía tiene solo documentos; la estructura `/docs`, `/supabase`, `/web` del Paso 7 está pendiente.

## En curso

- Paso 1 de `PLAN.md`: propuesta comercial y alcance del MVP.
- Paso 3 de `PLAN.md`: decidir qué se construye y qué se reutiliza.

## Siguiente

1. El usuario revisa la v3.3 y marca lo que no coincide.
2. Cerrar la lista de "Decisiones abiertas" que bloquean el MVP (marcadas con ⛔).
3. Hacer los *spikes* técnicos del Paso 4 (Bunny, MercadoPago, RLS).

## Decisiones tomadas (requisitos de negocio dichos por el usuario; no son decisiones técnicas)

- LA TROUPE es la primera academia y no se cobra a sí misma; el SaaS es futuro.
- Primero se construye la venta de cursos on demand; las clases en vivo y presenciales van después.
- Avance: libro abierto o curricular (tiempo, examen o ambos), con reloj individual por alumno.
- Cobros: se modelan las cuatro modalidades (único, cuotas, suscripción, plan de pagos). MercadoPago primero; Stripe después.
- Pago a profesores: automático por división de pago, o manual. Se define por profesor.
- Cada academia futura cobra en su propia cuenta.
- El modelo prevé desde ya licencias de contenido para academias.
- Reparto automático preferido sobre el manual para evitar carga administrativa.
- Las clases en vivo serán de grupos reducidos. La meta de escala es **5.000 personas simultáneas viendo video on demand** en la aplicación web.

## Stack candidato (recomendación del agente, 2026-10-28)

**No está decidido.** Es el candidato principal que se confirma o se descarta contra los requisitos (Paso 2) con registros de decisión (Paso 3). Precios verificados ese día en fuentes públicas (ver "Hechos verificados").

**El análisis completo y vigente está en `docs/infraestructura.md`** (VPS, hosting, base de datos, autenticación, R2, video, costos por etapa, tres iteraciones). Donde sus cifras difieran de las de esta sección, mandan las del documento. Cambios principales de ese análisis: archivos en **Cloudflare R2** (no en Supabase Storage); Cloudflare también para DNS, Turnstile y CDN; **sin VPS** al inicio; **Mux al inicio y video propio en R2 a escala**, con los originales en R2 desde el día uno; Supabase Auth porque Cloudflare no ofrece inicio de sesión para clientes.

| Capa | Candidato | Costo mensual | Alternativa que perdió, y por qué |
|---|---|---|---|
| Lenguaje | TypeScript en todo (web, servidor y, más adelante, app móvil) | — | PHP/Python/Java: dos lenguajes y menos herramientas para agentes |
| Aplicación | Next.js como monolito modular: sitio público, área del alumno, panel y API en un solo proyecto. **La lógica de negocio vive en el servidor, en TypeScript, con pruebas** (no repartida en SQL y funciones como en el prototipo) | — | Laravel + Filament: mejor panel de administración listo, pero dos lenguajes y el reproductor igual necesita React |
| Hosting | Vercel. Hobby gratis en desarrollo; **Pro USD 20 desde que se cobra (Hobby no permite uso comercial)** | 0 → 20 | Cloudflare Workers (USD 5): más barato, más fricción con Next.js |
| Base de datos y autenticación | Supabase (región São Paulo): Postgres administrado, Auth, backups. **Free se pausa tras una semana sin actividad: solo para desarrollo. Pro USD 25** | 0 → 25 | Neon + Better Auth: más portable, más piezas que mantener |
| Archivos | Cloudflare R2: imágenes, PDF, certificados, entregas y originales de video. Sin costo de transferencia | ~2 → 50 | Supabase Storage: transferencia cara a escala |
| Acceso a datos | Drizzle ORM (transacciones reales para pagos y repartos) + Zod. RLS como red de seguridad, no como lugar de la lógica | 0 | supabase-js + funciones SQL: sin transacciones desde el código |
| Video | **Mux al inicio** (confirmar contra Bunny en el spike S1). **A escala, video propio en R2** servido por la CDN de Cloudflare (baja el costo de video más de un 95 %; términos a confirmar en el spike S10). Originales en R2 desde el día uno | 5–20 al inicio | Cloudflare Stream: almacenamiento caro (USD 5 por 1.000 min) |
| Reproductor | Vidstack (velocidad, espejo, capítulos, bucle) | 0 | — |
| Pagos | MercadoPago | Comisión por venta | — |
| Cobro internacional | **Stripe probablemente no admite empresas uruguayas** (a confirmar). Evaluar un *Merchant of Record* (Lemon Squeezy, Paddle, Hotmart), que además resuelve el IVA europeo, o una entidad en el exterior. Spike S7 | — | Stripe directo |
| Tareas programadas | Cron de Vercel al inicio; Inngest (gratis hasta ~50.000 ejecuciones/mes) cuando lleguen los planes de pago propios | 0 | Trigger.dev |
| Email | Resend (gratis 3.000/mes) | 0 → 20 | Amazon SES: más barato, más configuración |
| Errores | Sentry (gratis 5.000/mes) | 0 | — |
| Interfaz | Tailwind + shadcn/ui; next-intl para idiomas | 0 | — |
| Pruebas y CI | Vitest, Playwright, GitHub Actions (gratis en repositorios públicos) | 0 | — |
| Dominio, DNS, anti-bots | Cloudflare (DNS gratis, Turnstile) | ~USD 1 (dominio anual) | — |

**Modelo de costo de video** (supuesto: ~1,5 GB por hora vista; ~4 GB almacenados por hora de video en Bunny):

| | Bunny | Cloudflare Stream | Mux |
|---|---|---|---|
| 1.000 minutos vistos desde Latinoamérica | ≈ USD 1,1 (USD 0,045/GB) | USD 1,0 | ≈ USD 0,8–1,2, con 100.000 min/mes gratis |
| 1.000 minutos vistos desde Europa | ≈ USD 0,25 (USD 0,01/GB) | USD 1,0 | ≈ USD 0,8–1,2 |
| 100 horas de biblioteca almacenadas | ≈ USD 4 | USD 30 | ≈ USD 15–18 |

El prototipo decía que Bunny era "5–10 veces más barato". Eso vale para audiencia de Europa y Norteamérica; **para Latinoamérica la diferencia con Mux y Cloudflare es mínima**.

**Costo mensual total estimado** (sin comisiones de cobro; detalle por pieza en `docs/infraestructura.md`, sección 11):
- Desarrollo: ~USD 1.
- Lanzamiento (hasta ~100 alumnos): ~USD 65–80.
- 500 alumnos activos viendo 10 h/mes: ~USD 320–500.
- 5.000 simultáneos (~60.000 alumnos): ~USD 5.300–42.400 con video administrado; ~USD 900–2.200 con video propio en R2.

### ¿Escala a 5.000 simultáneos? (análisis del 2026-10-28)

Respuesta: **sí, con las reglas de escala de `PLAN.md` (Paso 3)**. Estimación de carga para 5.000 personas viendo video bajo demanda al mismo tiempo:

| Componente | Carga estimada | Quién la absorbe | Costo aproximado |
|---|---|---|---|
| Video | ~17 Gbps; ~800 pedidos/s de fragmentos | CDN del proveedor (Mux/Bunny) | ~USD 300 por hora de pico a precio de lista (~USD 1 por 1.000 min) |
| Web y API | ~200–400 pedidos/s (progreso cada 30 s, navegación, inicio de reproducción) | Vercel, escala automáticamente | Decenas a pocos cientos de USD/mes (estimado, no verificado) |
| Base de datos | ~300–500 consultas/s pequeñas | Supabase Large o XL (USD 110–210/mes; 800–3.000 conexiones por pooler). Techo: 16XL, 64 núcleos, 256 GB, USD 3.730/mes | USD 110–210/mes |
| Autenticación | ~1–2 renovaciones/s | Supabase Auth (100.000 usuarios activos/mes incluidos en Pro) | Incluido |

Contexto: 5.000 simultáneos equivale, con un modelo aproximado (10 h de video por alumno por mes, consumo concentrado en unas 6 horas del día), a **~50.000–90.000 alumnos activos**. El video cuesta **~USD 0,15–0,70 por alumno por mes** a precio de lista: con cuotas de USD 15–30, entre 1 % y 5 % del ingreso. Métrica a seguir siempre: costo de infraestructura por alumno activo.

**Costo total mensual a esa escala** (reemplazado por `docs/infraestructura.md`, sección 11): ~USD 5.300–42.400 con video administrado (el video es más del 90 %) o ~USD 900–2.200 con video propio en R2. Por alumno: ~USD 0,09–0,70 administrado o ~USD 0,02–0,04 propio. Fuera de esta cuenta: comisiones de cobro (porcentaje por venta, a verificar con MercadoPago Uruguay), impuestos, personal y producción de contenido. Las comisiones probablemente pesen tanto o más que toda la infraestructura.

**Lo que el prototipo (v3.3) no resolvía para esta escala:**
1. Guardaba cada evento del reproductor (play, pausa, salto) en la base principal: con 5.000 personas es el primer cuello de botella. Solución: lotes, tabla particionada, o las estadísticas del propio proveedor de video (Mux Data).
2. Ponía la lógica de permisos en políticas RLS que cruzan varias tablas: a gran escala son lentas si no se diseñan con índices y funciones específicas. Solución: lógica en el servidor y RLS simple como red de seguridad.
3. No preveía pooler de conexiones, páginas públicas cacheadas ni pruebas de carga.

El resto (video servido por CDN, PostgreSQL, servidor que escala solo) sí estaba bien orientado.

**En vivo** (no es el caso planteado: las clases en vivo serán de grupos reducidos, y para eso Zoom alcanza). Solo para una eventual clase magistral masiva: Zoom no sirve para 5.000. La licencia estándar de webinar es de 500, ampliable con complementos pagos hasta ~10.000 (fuentes institucionales), y las reuniones admiten 100–1.000. Para clases magistrales masivas: transmisión en vivo por HLS (Mux Live o Cloudflare Stream Live), con 10–30 s de demora. Costo estimado de 5.000 personas × 90 min: ~USD 450–500 por evento.

Datos de Supabase, Zoom y Mux en vivo tomados de fuentes de terceros con discrepancias menores; verificar en las páginas oficiales antes de presupuestar.

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

### Investigación de skills y repositorios de referencia (2026-10-28)

Datos de la API pública de GitHub, consultada ese día.

**Skills ya instaladas en este entorno:** `emilkowalski/skills` (51k estrellas, MIT; aparece como `emil-design-eng`, `animate`, etc.) y `addyosmani/agent-skills` (111k, MIT; aparece como `agent-skills:*`). También hay skills de Anthropic (docx, pdf, xlsx, skill-creator).

| Recurso | Estrellas | Licencia | Recomendación |
|---|---|---|---|
| `supabase/agent-skills` | 2.9k | MIT | Instalar al empezar el Paso 4. Oficial de Supabase: Postgres, RLS, esquema |
| `vercel-labs/agent-skills` | 33k | (sin declarar) | Instalar al empezar a codear la interfaz (React/Next.js) |
| MCP oficial de MercadoPago (`https://mcp.mercadopago.com/mcp`) | — | — | Conectar en el spike S3/S4: busca en la documentación oficial |
| `affaan-m/ECC` | 291k | MIT | No instalar entero: 311 skills, hooks automáticos y superposición con `agent-skills`. Tomar piezas puntuales si hace falta |
| `debpalash/VoiceStudio` | 93k | AGPL-3.0 | No aplica ahora. Posible uso futuro: doblar cursos a otros idiomas (Fase 3, internacional) |
| `guillermoscript/lms-front` | 42 | MIT | **Referencia principal.** LMS multi-escuela con Next.js 16 + Supabase RLS + Stripe Connect, exámenes, certificados verificables, suscripciones, español. ~2.900 commits, casi todo de un autor, activo. No es un fork base: estudiar su esquema, RLS y monetización. Raíz desordenada y con un `cookies.txt` commiteado: señales de higiene a tener en cuenta |
| `frappe/lms` / `classroomio/classroomio` | 3.4k / 1.7k | AGPL-3.0 | Solo referencia funcional (la AGPL complica un SaaS comercial) |
| `usebasejump/basejump` | 939 | MIT | Referencia del patrón multi-tenant con RLS |
| `makerkit/nextjs-saas-starter-kit-lite` | 465 | MIT | No sirve como base: no incluye multi-tenancy |
| `vidstack/player` | 3.7k | MIT | Confirmado como reproductor |
| `mercadopago/mercadopago-claude-marketplace` | 32 | Apache-2.0 | **Oficial de MercadoPago** (organización verificada), beta v4.3.2. 4 skills (`mp-integrate`, `mp-webhooks`, `mp-test-setup`, `mp-review`), soporta Uruguay, bloquea filtración de credenciales. Instalar en el spike S3 |
| `Graphify-Labs/graphify` | 138k | Apache-2.0 | Convierte un código en un grafo consultable: ahorra tokens al estudiar código grande. Usarlo para analizar `lms-front`; en nuestro repo, recién cuando haya mucho código |
| `pbakaus/impeccable` | 93k | Apache-2.0 | Diseño y auditoría de interfaz. Instalar en el Paso 5 (junto con las de Emil Kowalski ya instaladas) |
| `cathrynlavery/diagram-design` | 62k | MIT | Diagramas de arquitectura. Opcional; útil para documentar la v3.3 |
| `vercel-labs/agent-browser` | 45k | Apache-2.0 | Automatización de navegador. No hace falta ahora: este entorno ya tiene navegador integrado. Evaluar para pruebas de punta a punta |
| `rohitg00/agentmemory` | 30k | Apache-2.0 | Memoria persistente para agentes. No instalar por ahora: la memoria del proyecto vive en `AGENTS.md`/`STATUS.md`, versionada en git |
| `volcengine/OpenViking` | 41k | AGPL-3.0 | Base de contexto y memoria para agentes. Excesivo para el proyecto |
| `mukul975/Anthropic-Cybersecurity-Skills` | 35k | Apache-2.0 | **No es de Anthropic** pese al nombre. 817 skills de ciberseguridad ofensiva/defensiva. No instalar entero; para revisar seguridad ya están `security-review` y `agent-skills:security-auditor` |
| `ai-boost/awesome-harness-engineering` | 5k | — | Lista de lectura, no se instala |
| `mattpocock/skills` | 306k | MIT | **Recomendada ahora.** Skills chicas y combinables: `grill-me` (interrogatorio para definir ideas, sirve para el Paso 1), `domain-modeling`, `to-spec`, `to-tickets`, `tdd`, `handoff`, `codebase-design`. Se superpone en parte con `addyosmani/agent-skills`: usar una como principal para no duplicar. Instalación oficial: `claude plugin install mattpocock-skills@claude-plugins-official`, luego `/setup-matt-pocock-skills` en el repo |
| `morluto/rea` (REA: Reverse Engineer Anything) | 403k | MIT | MCP de ingeniería inversa de binarios y apps (Frida, Hopper). Su propio descargo limita el uso a software propio o autorizado, y prohíbe copiar código o recursos protegidos. **No instalar por ahora**: para estudiar plataformas web de la competencia alcanza el navegador; usar REA solo sobre software propio o de código abierto |

### Repositorios oficiales de Anthropic y del stack (2026-10-28)

| Repositorio | Estrellas | Licencia | Uso propuesto | Cuándo |
|---|---|---|---|---|
| `anthropics/claude-plugins-official` | 45k | Apache-2.0 | Directorio oficial de plugins revisados por Anthropic. Fuente preferida para instalar plugins | Siempre |
| `anthropics/skills` | 185k | — | Skills oficiales (docx, pdf, xlsx, skill-creator…). Varias ya están instaladas | Ya disponible |
| `anthropics/claude-code-security-review` | 6.6k | MIT | Acción de GitHub que revisa seguridad en cada cambio. Importante porque el repo es público | Paso 7 |
| `anthropics/claude-code-action` | 9.7k | MIT | Acción de GitHub para revisar PRs con Claude | Paso 7 (opcional) |
| `anthropics/claude-cookbooks` | 54k | MIT | Recetas de uso de la API de Claude. Referencia para un futuro tutor con IA | Futuro |
| `upstash/context7` | 64k | MIT | Documentación actualizada de librerías para el agente: evita usar APIs viejas y ahorra tokens | Paso 4 |
| `supabase/mcp` | 3k | Apache-2.0 | Conecta el agente a Supabase. **Solo a proyectos de desarrollo**, nunca a producción | Paso 4 |
| `supabase/supabase` | 114k | Apache-2.0 | Código y ejemplos oficiales (auth, RLS, Edge Functions) | Paso 4 en adelante |
| `vercel/next.js` | 143k | MIT | Framework; su carpeta `examples` tiene ejemplos oficiales con Supabase | Paso 7 |
| `vercel/next-forge` | 7.7k | MIT | Plantilla de producción de Vercel para Next.js en monorepo. Evaluar como estructura del repo | Paso 7 |
| `shadcn-ui/ui` | 127k | MIT | Componentes de interfaz | Paso 5 en adelante |
| `microsoft/playwright` y `playwright-mcp` | 99k / 39k | Apache-2.0 | Pruebas automáticas de punta a punta (compra, acceso, desbloqueos) | Fase 0 en adelante |
| `github/spec-kit` | 144k | MIT | Desarrollo guiado por especificaciones. Se superpone con Matt Pocock y Addy Osmani: elegir uno | Evaluar |
| `github/github-mcp-server` | 34k | MIT | Issues y PRs desde el agente. No hace falta todavía | Opcional |
| `stripe/ai` | 2k | MIT | Herramientas oficiales de Stripe para agentes | Fase 3 |
| `resend/resend-mcp` | 578 | MIT | MCP oficial de Resend (email) | Fase 1A |
| `modelcontextprotocol/servers` | 92k | — | Servidores MCP de referencia del protocolo | Referencia |

No verificado: `stripe/agent-toolkit` (probablemente reemplazado por `stripe/ai`), `getsentry/sentry-mcp` (la consulta no lo encontró con ese nombre).

### Plataformas de cursos de código abierto (2026-10-28)

Ninguna cubre el modelo completo (marketplace de profesores + avance curricular + MercadoPago + multi-academia). Las bien calificadas son casi todas AGPL o GPL: usar su código obliga a publicar el nuestro con la misma licencia y cierra la puerta a un SaaS de código cerrado. **Se usan para estudiar, no para copiar código.**

| Plataforma | Estrellas | Licencia | Tecnología | Qué estudiar |
|---|---|---|---|---|
| `guillermoscript/lms-front` | 42 | MIT | Next.js + Supabase | Lo más parecido: multi-escuela, reparto con Stripe Connect, exámenes, certificados, español. **Única con licencia libre**. No figura MercadoPago |
| `codelitdev/courselit` | 1.3k | AGPL-3.0 | TypeScript | Alternativa abierta a Teachable: venta de cursos y descargas con marca propia. Flujo de venta y página pública |
| `learnhouse/learnhouse` | 2.4k | AGPL-3.0 | Python + Next.js | Editor de lecciones por bloques, experiencia moderna |
| `frappe/lms` | 3.4k | AGPL-3.0 | Python + Vue | Cursos, evaluaciones, certificados, lotes de alumnos |
| `classroomio/classroomio` | 1.7k | AGPL-3.0 | TypeScript | Orientado a capacitación de empresas; gestión de alumnos |
| `moodle/moodle` | 7.6k | GPL-3.0 | PHP | Referencia de evaluaciones, calificaciones y reglas de finalización (lo más completo del mundo en eso) |
| `instructure/canvas-lms` | 6.9k | AGPL-3.0 | Ruby | Referencia académica (libro de calificaciones) |
| `TryGhost/Ghost` | 56k | MIT | TypeScript | No es LMS: membresías y suscripciones pagas. Referencia de flujo de suscripción |
| Open edX (`overhangio/tutor`) | 1.1k | AGPL-3.0 | Python | Pesado, universitario. No encaja |

Alternativa estratégica no técnica: usar una plataforma comercial existente (por ejemplo Hotmart, Teachable o Thinkific) para **validar la demanda** mientras se construye la propia. Condiciones para Uruguay sin verificar.

### Skills y herramientas de seguridad (2026-10-28)

Estrategia por capas propuesta (no instalado nada todavía):

| Capa | Herramienta | Estrellas / licencia | Cuándo |
|---|---|---|---|
| Diaria, en cada cambio | `security-review` (incluida en Claude Code) y `agent-skills:security-auditor` (ya instalada) | — | Desde la Fase 0 |
| Base de datos y RLS (el mayor riesgo: fuga de datos entre academias) | `supabase/agent-skills` | 2.9k, MIT | Paso 4 |
| En cada cambio subido a GitHub | `anthropics/claude-code-security-review` (acción de GitHub) | 6.6k, MIT | Paso 7 |
| GitHub nativo (gratis en repos públicos) | Secret scanning con *push protection*, Dependabot | — | Paso 7 (lo activa el usuario en Settings → Code security) |
| Auditoría profunda en hitos (antes de cobrar, antes de lanzar, antes de abrir a otras academias) | `cloudflare/security-audit-skill`: auditoría en 6 fases con agentes aislados que intentan refutar cada hallazgo | 52k, MIT | Hitos. Consume muchos tokens: no usar en cada cambio |
| Selectivo | `trailofbits/skills`: `audit-context-building`, `agentic-actions-auditor` (audita acciones de GitHub con agentes) | 7.9k, CC-BY-SA-4.0 | Paso 7 en adelante. La mayoría de sus plugins son de contratos inteligentes y C: no instalar todo |
| Revisar skills antes de instalarlas | `NVIDIA/SkillSpector` o `snyk/agent-scan` | 23k / 3.2k, Apache-2.0 | Antes de instalar skills de terceros. Correr en entorno aislado: el escaneo de MCP ejecuta comandos |
| Solo si hay funciones con IA | OWASP Top 10 para aplicaciones con LLM | 1.5k | Futuro (tutor con IA) |

Descartados: `semgrep/mcp` (sin cambios desde 2025-10; mejor Semgrep directo en CI si hace falta), `gadievron/raptor` y `mukul975/Anthropic-Cybersecurity-Skills` (orientados a seguridad ofensiva; el segundo no es de Anthropic).

### Herramientas instaladas (2026-10-28)

- **Skills a nivel del proyecto** (registradas en `.claude/settings.json`, viajan con el repositorio): `mattpocock-skills` (de `claude-plugins-official`), `supabase` y `postgres-best-practices` (de `supabase-agent-skills`). Ninguna trae acciones automáticas (hooks). **Corrección:** las dos de Supabase incluyen además el servidor MCP oficial de Supabase limitado a su documentación (`https://mcp.supabase.com/mcp?features=docs`), sin acceso a bases de datos, que no se conecta hasta que el usuario lo autorice.
- **MCP Context7** a nivel del proyecto (`.mcp.json`): documentación actualizada de librerías. La primera vez, Claude Code pide aprobar el servidor.
- **Lector de voz** (complemento de esta sesión, fuera del repositorio): `C:\Users\Usuario\.claude\dev-mods\62183fbb-850e-49e7-b835-75ee95b80447\lector-de-voz\`. Panel que lee en voz alta cada respuesta con la voz Helena de Windows: leer, pausar, continuar, detener, velocidad, lectura automática y cambio de voz; comando `/lector`. **Estado (2026-10-28): escrito y validado, pero no cargado.** No se aceptó la activación de complementos en la sesión en que se creó. Carga al iniciar la próxima sesión, o se puede volver a pedir la activación.
- La CLI global `claude` instalada por npm es la **2.1.116**; el motor de la app de escritorio es la **2.1.309** (`C:\Users\Usuario\AppData\Roaming\Claude\claude-code\2.1.309\9078b9c1d466\claude.exe`). Para validar complementos hay que usar la del motor.

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
