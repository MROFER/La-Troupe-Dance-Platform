# CHANGELOG.md — LA TROUPE

Historial de cambios del proyecto y de sus documentos. Lo más reciente arriba. Formato: fecha, qué cambió, por qué.

## 2026-10-28 (infraestructura, skills y lector de voz)

### Agregado
- `docs/infraestructura.md`: análisis iterado (tres pasadas y tres revisiones) de VPS, hosting, base de datos, autenticación, archivos, video, correo y costos por etapa.
- Skills a nivel del proyecto: `mattpocock-skills`, `supabase`, `postgres-best-practices` (`.claude/settings.json`), y MCP Context7 (`.mcp.json`).
- Complemento "Lector de voz" (fuera del repositorio, en la carpeta de complementos de la sesión): lee las respuestas en voz alta con la voz Helena de Windows. Escrito y validado; no cargado todavía.

### Corregido
- `STATUS.md`: las skills de Supabase incluyen también el servidor MCP de Supabase limitado a documentación (inactivo hasta autorizarlo); antes se dijo que eran solo instrucciones.
- `PLAN.md`: spikes S9 (Vercel frente a Cloudflare Workers) y S10 (video propio en R2).

### Cambiado
- Stack candidato: archivos en Cloudflare R2; Cloudflare también para DNS, Turnstile y CDN; Mux al inicio y video propio en R2 a escala; sin VPS al inicio.
- Costos estimados actualizados: lanzamiento ~USD 65–80/mes (incluye staging); 500 alumnos ~USD 320–500/mes; 5.000 simultáneos ~USD 5.300–42.400 con video administrado o ~USD 900–2.200 con video propio.

## 2026-10-28 (cambio de enfoque)

### Cambiado
- La v3.2 y la v3.3 pasan a ser **prototipos**, por decisión del usuario. Ni la arquitectura ni el stack están decididos.
- `PLAN.md`: el Paso 2 pasa a ser "Requisitos y criterios de calidad" y el Paso 3 "Opciones de arquitectura y stack, evaluadas", con registros de decisión (ADR) y una arquitectura v4 nueva al final.
- `AGENTS.md`: las decisiones de arquitectura pasan a "hipótesis a validar"; el stack queda sin decidir.

### Agregado
- `STATUS.md`: investigación de skills, herramientas de seguridad, repositorios oficiales y plataformas de cursos de código abierto.
- `STATUS.md`: stack candidato con precios verificados y modelo de costo de video. Hallazgos que corrigen el prototipo: Vercel Hobby no permite uso comercial; Supabase Free se pausa; para audiencia latinoamericana Bunny no es mucho más barato que Mux o Cloudflare; Stripe probablemente no admite empresas uruguayas.
- `PLAN.md`: el spike S1 compara Mux y Bunny; nuevo spike S7 (cobro internacional desde Uruguay).
- `PLAN.md`: 10 reglas de escala obligatorias para la v4 y spike S8 (prueba de carga). El atributo "Escala" incluye el pico de usuarios simultáneos (meta: 5.000).
- `STATUS.md`: análisis de carga y costo para 5.000 usuarios simultáneos, incluido el caso de clases en vivo.

## 2026-10-28 (repositorio)

### Agregado
- Repositorio git propio en la carpeta `La Troupe`, publicado en https://github.com/MROFER/La-Troupe-Dance-Platform. Primer commit con todos los documentos.
- `README.md` con el índice de documentos y `.gitignore` base (dependencias, secretos `.env`, archivos de sistema).

### Cambiado
- La carpeta deja de depender del repositorio git del directorio del usuario.

## 2026-10-28

### Agregado
- `AGENTS.md`: guía para agentes y personas (reglas, decisiones de arquitectura, fuentes de verdad, cómo retomar).
- `CLAUDE.md`: apunta a `AGENTS.md` para que Claude Code lo cargue automáticamente.
- `STATUS.md`: estado actual, decisiones abiertas y hechos verificados.
- `PLAN.md`: pasos ordenados desde la propuesta comercial hasta el primer código, con criterios de avance.
- `Arquitectura_LA_TROUPE_v3.3.md` y `.pdf`: arquitectura de marketplace.
  - Roles y permisos editables (reemplazan los tres enums de `memberships`).
  - Carreras, cursos, lecciones y bloques de contenido.
  - Reglas de liberación de contenido (tiempo, examen o ambos) con reloj individual.
  - Evaluaciones automáticas, manuales y mixtas; reglas de egreso; certificados verificables.
  - Productos, precios y las cuatro modalidades de cobro.
  - `access` con titular persona u organización (licencias).
  - Cuentas de cobro por organización y por autor; acuerdos de reparto; registro de reparto por pago.
  - Flujo de revisión, aprobación y solicitudes de cambio para profesores invitados.
  - Roadmap rehecho: primero ventas on demand; clases en vivo pasan a la Fase 6.

### Cambiado
- Se unifica la contradicción de la v3.2: `staff_profile` **sí existe** (sección 4.10 manda sobre 4.5).
- Se reemplaza `course_lessons.is_locked_until` por `release_rules`.

### Documentado
- Primera investigación de referencias y verificaciones técnicas (Bunny, MercadoPago, Basejump, EaseLMS) en `STATUS.md`.

## 2026-10-28 (antes en el día)

### Agregado
- `Arquitectura_LA_TROUPE_v3.2.md` y `.pdf`: conversión del `.docx` original (que no abría por falta de un programa compatible; el archivo no estaba dañado).

### Revisado
- Evaluación de la v3.2 contra los objetivos: bien encaminada; huecos en permisos, autoría de contenido, pagos SaaS, espacio público por academia y orden del roadmap.

## 2026-04 (documento original)

- `Arquitectura_LA_TROUPE_v3.2.docx` recibido (generado en abril de 2026). Base de todo lo anterior.
