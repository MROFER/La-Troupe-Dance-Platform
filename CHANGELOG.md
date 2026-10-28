# CHANGELOG.md — LA TROUPE

Historial de cambios del proyecto y de sus documentos. Lo más reciente arriba. Formato: fecha, qué cambió, por qué.

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
