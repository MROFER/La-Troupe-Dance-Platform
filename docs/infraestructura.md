# Infraestructura y costos — análisis iterado

**Fecha:** 2026-10-28 · **Estado:** propuesta del agente, pendiente de confirmar con los requisitos (`PLAN.md`, Pasos 2 y 3) y con las pruebas técnicas (spikes).

Este documento responde: qué servicios usamos, si hace falta un VPS, cuánto cuesta cada pieza en cada etapa, por qué Supabase Auth y no Cloudflare, cuándo conviene R2, y qué pasa cuando miles de personas ven el mismo video. Se hizo en tres pasadas con dos revisiones críticas en el medio (sección 12).

---

## 0. La respuesta corta

- **Arrancamos sin VPS**, con servicios administrados. Un VPS solo aparece a gran escala, para procesar video propio.
- **Pila final:** Next.js en Vercel · Supabase para base de datos y autenticación · **Cloudflare para DNS, archivos (R2), anti-bots (Turnstile) y CDN** · Mux para video al inicio · MercadoPago para cobrar.
- **Autenticación con Supabase Auth.** Cloudflare no tiene un servicio de inicio de sesión para alumnos: Cloudflare Access es para empleados de una empresa, y cobra por usuario.
- **R2 sí, desde el día uno**, para imágenes, PDF, diapositivas, certificados, entregas de alumnos y **los originales de todos los videos**.
- **Video:** al inicio un servicio administrado, porque es rápido de integrar y sale casi gratis con pocos alumnos. A gran escala, **video propio servido desde R2**, que puede bajar el costo de video más de un 95 %. Guardar los originales en R2 desde el día uno es lo que hace posible ese cambio.
- **Muchos viendo el mismo video no es un problema:** es el mejor caso para una CDN.
- **Costos mensuales estimados:** desarrollo ~USD 1 · lanzamiento ~USD 65–80 · 500 alumnos ~USD 320–500 · 5.000 simultáneos ~USD 5.300–42.400 con video administrado, o ~USD 900–2.200 con video propio en R2.

---

## 1. Miles de personas viendo el mismo video al mismo tiempo

**No hay problema; es el caso más favorable.**

Un video en streaming se corta en fragmentos de unos segundos (HLS). La CDN (red de servidores repartidos por el mundo) guarda una copia de cada fragmento en el servidor cercano la primera vez que alguien de esa zona lo pide. Todos los demás reciben esa copia sin volver al origen. Cuanto más popular es un video, más se aprovecha la copia y más rápido carga.

- **Con un servicio administrado** (Mux, Bunny, Cloudflare Stream) no cambia el precio: se paga por minuto visto o por GB entregado, sea el mismo video o no. Tampoco cuesta más.
- **Con video propio en R2** detrás de la CDN de Cloudflare, el video popular sale **más barato**: los fragmentos servidos desde la copia no generan lecturas en R2.
- **El permiso de cada alumno** va en su enlace firmado. La CDN valida el enlace, pero comparte la copia del fragmento, porque el permiso no forma parte de la clave de la copia.
- **En nuestra base de datos** no hay conflicto: cada alumno guarda su progreso en su propia fila.
- **Lo único a evitar:** un contador de reproducciones en una sola fila (`vistas = vistas + 1`). Con miles a la vez, todos esperan su turno para escribir esa fila. Se resuelve guardando cada evento aparte y sumando cada tanto (regla de diseño, sección 12).

---

## 2. Quién hace cada trabajo

```
Alumno (navegador o celular)
 │
 ├─ Páginas y API ──────────► Vercel (Next.js)  ── región São Paulo
 │                              │
 │                              ├─► Supabase: PostgreSQL + Auth   ── São Paulo
 │                              ├─► MercadoPago (cobros, webhooks)
 │                              ├─► Resend (correos)
 │                              └─► Mux (crea videos, firma enlaces)
 │
 ├─ Video ──────────────────► CDN del proveedor de video (Mux; a escala, Cloudflare + R2)
 ├─ Imágenes, PDF, descargas ► Cloudflare CDN + R2 (enlaces firmados si son privados)
 └─ DNS, anti-bots ─────────► Cloudflare (DNS, Turnstile)
```

Nuestros servidores **nunca transmiten video ni archivos pesados**: autorizan, firman enlaces y guardan datos. Por eso la parte que pagamos por uso de cómputo queda chica aun con mucho tráfico.

---

## 3. ¿Hace falta un VPS?

**No al principio.** Un VPS es un servidor virtual alquilado (Hetzner, DigitalOcean, Vultr): barato y con control total, pero **lo administramos nosotros**. Eso incluye actualizaciones y parches de seguridad, backups, monitoreo, certificados, caídas a la madrugada y escalar a mano. Para un equipo de una persona y un agente de IA, ese trabajo y ese riesgo pesan más que el ahorro.

| | Servicios administrados (propuesta) | VPS propio |
|---|---|---|
| Costo al inicio | ~USD 45/mes (Vercel Pro + Supabase Pro) | ~USD 5–25/mes |
| Quién mantiene seguridad, backups y caídas | El proveedor | Nosotros |
| Escala ante un pico | Sola | A mano |
| Región cercana (Uruguay) | São Paulo en Supabase y Vercel | Hetzner no tiene región en Sudamérica; en São Paulo hay AWS y Vultr, más caros |
| Estabilidad de precios | Planes estables | Hetzner subió precios dos veces en 2026 (abril ~30–37 %; junio, las líneas CPX/CCX hasta +176 %) |

**Cuándo sí un VPS:** en la etapa de escala, si migramos el video a R2 propio, para **convertir videos** (ffmpeg) a calidades adaptativas. Un servidor de 4 núcleos y 8 GB (por ejemplo Hetzner CAX21, unos €8–10,5/mes tras los aumentos de 2026) alcanza para procesar las clases nuevas de cada mes. Otra opción es encenderlo solo cuando hay videos nuevos. No necesita estar cerca de los alumnos, porque no los atiende.

---

## 4. Hosting de la aplicación

| | Vercel (propuesta) | Cloudflare Workers | VPS |
|---|---|---|---|
| Precio base | Hobby gratis **solo sin uso comercial**; Pro USD 20/mes por persona del equipo | USD 5/mes: 10 millones de pedidos y 30 millones de ms de CPU incluidos; después USD 0,30 por millón de pedidos | USD 5–25/mes |
| Next.js | Plataforma de referencia: todo funciona | Con el adaptador OpenNext: soporta Next.js 16, pero el middleware de Node.js todavía no; madurez a confirmar | Funciona (Docker), lo operamos nosotros |
| A gran escala (estimado, no verificado) | ~USD 200–600/mes | ~USD 80/mes | Depende |
| Riesgo | Factura que crece con el uso | Fricción de compatibilidad | Operación |

**Decisión:** Vercel para empezar. **Condición:** no usar funciones exclusivas de Vercel (sus bases KV, Blob o Edge Config), así la app puede mudarse a Cloudflare Workers o a un servidor propio. **Spike S9:** desplegar el esqueleto en los dos lugares durante la Fase 0 y medir la fricción. **Disparador de mudanza:** factura de hosting mayor a ~USD 150/mes de forma sostenida.

Región: funciones en **São Paulo**, junto a la base de datos (confirmar el nombre de la región al crear el proyecto).

---

## 5. Base de datos

**Supabase (PostgreSQL administrado) en la región São Paulo.** PostgreSQL es la elección correcta para dinero, permisos, reportes y varias academias. Cloudflare D1 (SQLite) no sirve para este dominio: tiene transacciones limitadas y bases chicas por diseño.

| Etapa | Plan | Costo |
|---|---|---|
| Desarrollo | Free (se pausa tras 1 semana sin uso; sin garantías) o local con Docker | USD 0 |
| Lanzamiento | Pro, cómputo Micro incluido | USD 25 |
| 500 alumnos | Pro + cómputo Small | ~USD 25–45 |
| 5.000 simultáneos | Pro + cómputo XL (USD 210) + réplica de lectura para reportes + recuperación a un punto en el tiempo | ~USD 450–600 |
| Techo | 16XL: 64 núcleos, 256 GB | USD 3.730 |

- Conexiones siempre por el *pooler* en modo transacción. A gran escala admite de 800 a 12.000 conexiones según el tamaño.
- **Staging** (entorno de pruebas igual al real) = un segundo proyecto: ~USD 10/mes de cómputo.
- **Backups:** los diarios de Pro, más una **copia semanal propia** (`pg_dump`) guardada en R2. Cuesta centavos y nos deja independientes del proveedor.

---

## 6. Autenticación: por qué Supabase Auth y no Cloudflare

**Cloudflare no ofrece inicio de sesión para clientes.** Su producto de identidad, Cloudflare Access (Zero Trust), protege herramientas internas de una empresa y se cobra por usuario. No está pensado para 60.000 alumnos. "Auth con Cloudflare" en la práctica significa programar el inicio de sesión con una librería (Better Auth) dentro de Workers: eso es Better Auth, no Cloudflare.

| Opción | Costo a ~60.000 usuarios activos | Ventajas | Desventajas |
|---|---|---|---|
| **Supabase Auth (propuesta)** | USD 0 (Pro incluye 100.000 usuarios activos/mes; después USD 0,00325 por usuario) | Integrada con la base y sus permisos; servicio administrado y maduro; email y contraseña, Google, Apple, enlace mágico, segundo factor, CAPTCHA con Turnstile, SMTP propio en todos los planes; **los usuarios quedan en nuestra propia base** | Atada a Supabase (con salida, ver abajo) |
| Better Auth (librería abierta) | USD 0 | Todo en nuestras tablas; plugin de organizaciones y roles | La seguridad y las actualizaciones son nuestras |
| Clerk | ~USD 1.000/mes a 100.000 usuarios (estimación de terceros); gratis hasta 50.000 desde febrero de 2026 | Excelente experiencia | Caro al crecer |
| WorkOS AuthKit | USD 0 hasta 1.000.000 de usuarios | Muy generoso | El inicio de sesión único para empresas cuesta USD 125 por conexión por mes; datos fuera de nuestra base |
| Cloudflare Access | — | — | No es para alumnos |

**Salida:** los usuarios de Supabase Auth viven en el esquema `auth` de nuestro propio PostgreSQL, con contraseñas cifradas en un formato estándar. Si algún día conviene, en principio se pueden migrar a Better Auth conservando las contraseñas. Hay que confirmarlo con una prueba antes de contar con ello. Las membresías y roles por academia son tablas nuestras: no dependen del proveedor de autenticación.

**Obligatorio en producción:** SMTP propio (Resend) con el dominio verificado por SPF, DKIM y DMARC. Sin eso, los correos de registro llegan a spam o no salen.

---

## 7. Archivos e imágenes: R2

| | Cloudflare R2 (propuesta) | Supabase Storage |
|---|---|---|
| Almacenamiento | USD 0,015 por GB al mes (10 GB gratis) | 100 GB incluidos en Pro; después ~USD 0,021 por GB |
| Transferencia (lo que bajan los alumnos) | **USD 0, siempre** | ~250 GB incluidos; después ~USD 0,09 por GB |
| Operaciones | Escritura USD 4,50 por millón; lectura USD 0,36 por millón (1 y 10 millones gratis al mes) | Incluidas |
| Compatibilidad | API de S3: cambiar de proveedor es fácil | Propia de Supabase |

Con 60.000 alumnos bajando unos 20 MB por mes de imágenes y PDF (1,2 TB), Supabase cobraría ~USD 85/mes solo de transferencia. En R2 eso es USD 0.

**Qué va a R2:** imágenes de cursos y avatares (en 2 o 3 tamaños generados al subirlos, así no se paga por transformarlas en cada visita), PDF y diapositivas, descargas, certificados, entregas de alumnos (videos de práctica para que califique el docente) y **los originales de cada video**. Los archivos privados se entregan con **enlaces firmados de corta duración**, generados por nuestro servidor después de verificar el permiso.

---

## 8. Video: administrado ahora, propio a escala

### 8.1 Opción administrada (para empezar)

| | Mux | Bunny Stream | Cloudflare Stream |
|---|---|---|---|
| 1.000 min vistos desde Latinoamérica | ~USD 0,8–1,2, **100.000 min/mes gratis** | ~USD 1,1 (USD 0,045/GB) | USD 1,0 |
| 1.000 min vistos desde Europa | ~USD 0,8–1,2 | ~USD 0,25 (USD 0,01/GB) | USD 1,0 |
| 100 h de biblioteca almacenada | ~USD 15–18 | ~USD 4 | USD 30 |
| Herramientas | Las mejores: estadísticas de visualización, clips, firmas | Vista previa animada automática | Simple, todo en Cloudflare |

**Para empezar: Mux.** Sus 100.000 minutos gratis por mes (unas 1.600 horas vistas) cubren el lanzamiento entero. Ojo: el plan gratuito admite solo 10 videos guardados, así que desde el lanzamiento hace falta el plan de pago por uso. Los montos de la tabla ya lo suponen. Sus estadísticas además reemplazan parte del registro de visualizaciones que el prototipo guardaba en la base. Bunny gana si hay mucho público europeo o una biblioteca enorme. Lo confirma el spike S1.

### 8.2 Opción propia en R2 (a escala)

Los videos se convierten nosotros (ffmpeg) a calidades adaptativas (HLS), se guardan en R2 y se sirven por la CDN de Cloudflare.

- **Por qué:** R2 no cobra la transferencia. A escala, el video administrado es más del 90 % del costo total.
- **Términos de uso:** desde 2023 Cloudflare permite servir video por su CDN si está alojado en sus propios servicios (R2, Stream, Images). Una página de su documentación menciona el plan Enterprise para cierto tipo de entrega de video. **Hay que confirmar los términos vigentes antes de planificar** (spike S10). A ese volumen es probable que Cloudflare pida un contrato Enterprise, que igual sería mucho más barato.
- **Lo que hay que construir:** conversión de video, cifrado AES-128 con claves entregadas solo a quien tiene permiso, enlaces firmados (un Worker que los valida), miniaturas y vistas previas con ffmpeg, y estadísticas propias. El reproductor (Vidstack) no cambia. Estimación: varias semanas de trabajo más mantenimiento.
- **Costo a 5.000 simultáneos:** almacenamiento de 2 TB (~USD 30) + lecturas de fragmentos que no estaban en la copia de la CDN (~USD 30–130) + conversión (~USD 20–60) ≈ **USD 100–250/mes**, frente a USD 4.500–40.500 administrado.

### 8.3 La decisión

**Mux al inicio; video propio en R2 cuando la factura de video pase ~USD 1.500–2.000/mes** (unos 1.500–2.000 alumnos activos), si el spike S10 confirma los términos. Lo que se hace desde el día uno para que el cambio sea fácil:

1. Guardar **cada original en R2** (unos USD 15/mes por 1 TB).
2. Tabla de assets independiente del proveedor, ya prevista en el prototipo.
3. El reproductor es nuestro (Vidstack) y no el del proveedor, así las funciones de danza (espejo, velocidad, bucle, capítulos) no dependen de él.

---

## 9. Correo, tareas, monitoreo y dominio

| Pieza | Al inicio | A escala |
|---|---|---|
| Correo | Resend gratis (3.000/mes; 100/día) → Pro USD 20 (50.000/mes) | Amazon SES (USD 0,10 por 1.000: ~USD 40/mes por 360.000 correos), o Resend en un plan mayor (~USD 200–400) |
| Tareas programadas (desbloqueos, recordatorios, cobros) | Cron de Vercel (incluido en Pro) | Inngest (gratis hasta ~50.000 ejecuciones/mes) cuando lleguen los planes de pago propios |
| Errores | Sentry gratis (5.000/mes) | Sentry Team (~USD 26+) |
| Disponibilidad | UptimeRobot gratis | Igual, o Better Stack |
| Anti-bots | Cloudflare Turnstile, gratis | Igual |
| DNS | Cloudflare, gratis | Igual |
| Dominio | ~USD 10–15/año (.com; un .uy se registra aparte) | Igual |

---

## 10. Lo que cuesta más que la infraestructura: cobrar

Las comisiones de MercadoPago se cobran por venta. **No encontré las tarifas vigentes para Uruguay.** Como referencia, en Argentina una opción es 1,79 % + IVA con el dinero a 35 días, y otra sin costo con el dinero a 70 días. Con un 4–6 % de comisión, a 60.000 alumnos serían USD 36.000–108.000 por mes, más que toda la infraestructura. **Es la línea de costo a negociar cuando haya volumen.**

Para vender en España, Stripe probablemente no admite empresas uruguayas (a confirmar, spike S7). Un *Merchant of Record* (Lemon Squeezy, Paddle, Hotmart) vende en nuestro nombre y se encarga del IVA europeo, a cambio de una comisión mayor.

---

## 11. Costos por etapa (USD por mes)

Supuestos: **lanzamiento** ~100 alumnos × 8 h de video/mes, biblioteca de 30 h · **crecimiento** ~500 alumnos activos × 10 h/mes, 150 h · **escala** 5.000 simultáneos ≈ 60.000 alumnos activos × 10 h/mes, 500 h. Audiencia mayoritariamente latinoamericana. Precios de lista verificados en fuentes públicas el 2026-10-28; donde dice "estimado", no está verificado.

| Pieza | Desarrollo | Lanzamiento | Crecimiento | Escala, video administrado | Escala, video propio R2 |
|---|---|---|---|---|---|
| Hosting (Vercel) | 0 | 20 | 20–40 | 200–600 (estimado) | 200–600 (estimado) |
| Base de datos (Supabase) | 0 | 25 | 25–45 | 450–600 | 450–600 |
| Staging | 0 | 10 | 10 | 10–25 | 10–25 |
| Autenticación | 0 | 0 | 0 | 0 | 0 |
| Video | 0 | 5–20 (Mux) | 230–345 | 4.500–40.500 | 100–250 |
| Archivos (R2) | 0 | 2 | 10 | 50 | 50 |
| Correo | 0 | 0 | 20 | 40–400 | 40–400 |
| Tareas, errores, monitoreo | 0 | 0 | 0–26 | 50–225 | 50–225 |
| Dominio | 1 | 1 | 1 | 1 | 1 |
| **Total** | **~1** | **~65–80** | **~320–500** | **~5.300–42.400** | **~900–2.200** |
| Por alumno activo | — | ~0,7 | ~0,6–1,0 | ~0,09–0,70 | ~0,02–0,04 |

Contexto: con cuotas de USD 15–30, la infraestructura queda entre ~0,1 % y ~7 % del ingreso según la etapa y la opción de video. El peor caso es la etapa de crecimiento con cuotas bajas: ahí el costo fijo todavía pesa. **No incluye** comisiones de cobro, impuestos, personal ni producción de contenido.

---

## 12. Iteraciones

### Pasada 1: propuesta inicial

Vercel + Supabase (base, autenticación y archivos) + Mux o Bunny para video + Resend + Sentry.

### Revisión 1: crítica de la pasada 1

1. **Archivos en Supabase Storage:** la transferencia pasa a costar mucho a escala. → Archivos a **R2**, que no cobra transferencia.
2. **El video administrado es más del 90 % del costo a escala**, y nada en la pasada 1 permitía salir de eso. → Camino a **video propio en R2**; **originales en R2 desde el día uno**.
3. **Vercel puede encarecerse a escala** y Cloudflare Workers es más barato. → Prohibido usar funciones exclusivas de Vercel; **spike S9** para medir la mudanza.
4. **"¿Por qué no autenticación con Cloudflare?"** → No existe para alumnos. Better Auth queda como salida y Supabase Auth se mantiene.

### Pasada 2

Vercel + Supabase (base y autenticación) + **Cloudflare (DNS, R2, Turnstile, CDN)** + Mux al inicio → video propio en R2 a escala.

### Revisión 2: crítica de la pasada 2

1. **Un solo proveedor de base de datos:** → copia semanal propia (`pg_dump`) en R2.
2. **Correos de registro en spam:** → SMTP propio con SPF, DKIM y DMARC desde el día uno.
3. **Contadores en una sola fila** (vistas, alumnos inscritos) se traban con miles a la vez. → Eventos aparte y sumas periódicas.
4. **El entorno de staging no estaba en los costos.** → Agregado (~USD 10/mes).
5. **Términos de Cloudflare para video de alto volumen sin confirmar.** → **Spike S10** antes de contar con ese ahorro.
6. **Latencia:** → base y funciones en **São Paulo**. España recibe el video por CDN; para lo demás, ~200 ms extra es aceptable. Con mucho público europeo, una réplica de lectura en Europa.
7. **Protección de datos:** guardar datos de alumnos uruguayos (incluidos menores) en Brasil o Estados Unidos tiene reglas propias (Ley 18.331). → Consultar con asesoría legal.

### Pasada 3: final

Es la sección 0. Ninguna crítica de la revisión 2 cambió la elección de servicios: agregaron reglas de diseño y pruebas.

### Revisión 3: control de cifras

Se recalculó cada número de las secciones 0, 7, 8, 10 y 11. Tres correcciones:
1. La infraestructura llega a ~7 % del ingreso en la etapa de crecimiento con cuotas bajas, no a 5 %.
2. El plan gratuito de Mux admite solo 10 videos: desde el lanzamiento se usa el plan de pago por uso.
3. La migración de contraseñas a Better Auth es posible en principio, no un hecho comprobado.

---

## 13. Verificaciones pendientes (spikes)

| Spike | Qué confirma |
|---|---|
| S1 | Mux frente a Bunny con el mismo video: calidad, protección, costo con la mezcla real de audiencia |
| S3 / S4 | Tarifas y plazos reales de MercadoPago en Uruguay; suscripciones; reparto automático |
| S7 | Stripe y Uruguay; opciones de *Merchant of Record* |
| S8 | Prueba de carga con el pico objetivo |
| **S9 (nuevo)** | Desplegar el esqueleto en Vercel y en Cloudflare Workers (OpenNext) y medir la fricción |
| **S10 (nuevo)** | Términos vigentes de Cloudflare para video en R2 a alto volumen; prototipo de conversión y entrega HLS con enlaces firmados |

---

## 14. Tecnologías

Las versiones se confirman con Context7 al iniciar la Fase 0.

- **Lenguaje y entorno:** TypeScript, Node.js 24 LTS (la versión instalada en la máquina del usuario es la 24.20.0), pnpm.
- **Aplicación:** Next.js (App Router) con React; Tailwind CSS y shadcn/ui; next-intl para idiomas.
- **Datos:** PostgreSQL en Supabase; Drizzle ORM y drizzle-kit para migraciones; Zod para validar datos.
- **Autenticación:** Supabase Auth con `@supabase/ssr`; Turnstile en registro e inicio de sesión.
- **Video:** Mux (SDK oficial) al inicio; Vidstack como reproductor; ffmpeg y hls.js en la etapa de video propio.
- **Archivos:** Cloudflare R2 con la API de S3 y enlaces firmados.
- **Pagos:** SDK oficial de MercadoPago.
- **Pruebas y calidad:** Vitest, Playwright, GitHub Actions (gratis en repositorios públicos), y la acción de revisión de seguridad de Anthropic.
- **Desarrollo local:** Supabase CLI, que necesita Docker.

---

## 15. Fuentes

- Vercel: [costos 2026](https://makerkit.dev/blog/saas/vercel-cost) · [Pro y Hobby](https://livemy.app/blog/vercel-pricing)
- Supabase: [precios oficiales](https://supabase.com/pricing) · [guía 2026](https://www.jetadmin.io/blog/supabase-pricing-2026-guide-to-plans-limits-and-real-world-costs/) · [tamaños de cómputo](https://selfhost.dev/blog/supabase-pricing-explained/) · [transferencia](https://www.activepieces.com/blog/supabase-pricing-free-tier-limits-pro-costs-egress)
- Cloudflare: [R2, precios oficiales](https://developers.cloudflare.com/r2/pricing/index.md) · [calculadora R2](https://r2-calculator.cloudflare.com/) · [Workers, precios oficiales](https://developers.cloudflare.com/workers/platform/pricing/) · [OpenNext para Cloudflare](https://opennext.js.org/cloudflare) · [fin de la sección 2.8](https://blog.cloudflare.com/updated-tos/) · [entrega de video](https://developers.cloudflare.com/fundamentals/reference/policies-compliances/delivering-videos-with-cloudflare/) · [Stream, precios](https://developers.cloudflare.com/stream/pricing)
- Video: [Bunny Stream](https://docs.bunny.net/stream/pricing) · [Mux](https://www.mux.com/docs/pricing/overview)
- Autenticación: [Clerk](https://clerk.com/articles/clerk-pricing-explained) · [WorkOS](https://www.budgetforge.dev/tools/workos-pricing-2026)
- Hetzner: [aumentos de 2026](https://findstack.com/resources/hetzner-price-increase-2026) · [junio de 2026](https://wz-it.com/en/blog/hetzner-price-increase-june-2026-cpx-ccx-alternatives/)
- Pagos: [comisiones de MercadoPago (Argentina, referencia)](https://www.iprofesional.com/tecnologia/386153-mercado-pago-que-debes-saber-sobre-comisiones-plazos-y-montos) · [Stripe y Uruguay](https://payrequest.io/en/guides/accept-paypal-in-uruguay)
- Tareas: [Inngest](https://automationatlas.io/answers/inngest-pricing-explained-2026/)
