# Spec — Vuelta de tuerca a la web de Como La Seda

**Fecha:** 2026-06-12 · **Estado:** pendiente de aprobación de Alex
**Contexto completo:** decisiones y evidencia en el vault → `Projects/Como La Seda.md` (Key Decisions 2026-06-12)

## Objetivo

Transformar la landing actual (un público, formulario→email) en una **web todo-en-uno,
WhatsApp-first y con ambiente personal** que sirva de aterrizaje a **tres tipos de anuncio**
distintos, y que convierta la duda en conversación de WhatsApp en vez de en formulario.

**Por qué (evidencia):** los leads B2B no responden emails pero sí WhatsApp ("se nota más
personal"); piden catálogo en vez de imaginar productos; el "conseguimos cualquier cosa"
queda desesperado en frío; las empresas pequeñas se autodescartan por la duda de cantidades.
Caso real: conversación WhatsApp con Jorge Alcaraz (Jamones Suprem), 2026-06-10.

## Decisiones de diseño (cerradas)

1. **Estrategia:** regalos como puerta, recurrente detrás.
2. **Tres públicos / tres anuncios → una sola página** con secciones ancladas
   (`#regalos`, `#suministros`, `#revende`). Cada campaña de Meta enlaza a su ancla.
   *(Sustituye a la decisión previa de página aparte `suministros.html`.)*
3. **WhatsApp protagonista:** todos los CTA → `wa.me/34680211326` con mensaje prellenado
   distinto por sección. Formulario Web3Forms queda secundario/discreto.
4. **Catálogo:** teaser en la web (líneas de producto con ejemplos concretos en texto;
   fotos Unsplash de placeholder hasta tener fotografía propia) + CTA estrella
   "Pide el catálogo completo por WhatsApp".
5. **Cantidades:** mensaje "cuéntanos tu cantidad y te decimos si compensa" (sin números).
6. **Ambiente personal:** bloque "Quién está detrás" (Alejandro, nombre y cara), tono cercano,
   contactar = empezar una conversación, no un trámite.
7. **Estructura:** multi-archivo con `styles.css` + `app.js` compartidos. `privacidad.html` nueva.
8. **Teléfono visible y WhatsApp:** +34 680 211 326 (sustituye al 622 200 594 en toda la web).

## Archivos

```
index.html        ← reformada: todo-en-uno con secciones por público
privacidad.html   ← NUEVA: política de privacidad (requisito Meta lead forms)
styles.css        ← NUEVO: marca + componentes compartidos (extraído del inline actual)
app.js            ← NUEVO: i18n + helpers WhatsApp + tracking compartidos
favicon.png       ← sin cambios
```

`suministros.html` **no se crea** (decisión 2 la sustituye).

## Estructura de `index.html`

| # | Sección | Contenido | CTA WhatsApp (texto prellenado) |
|---|---------|-----------|--------------------------------|
| 1 | Header | Logo, nav a secciones, ES\|EN, botón WhatsApp | genérico |
| 2 | Hero | Marca actual + propuesta válida para los 3 públicos: precio de origen sin intermediarios, trato directo por WhatsApp | "Hola, vengo de la web y quiero el catálogo" |
| 3 | `#regalos` | Evolución de lo actual: 3 ventajas (fábrica directa, personalización, muestras) + teaser de catálogo por líneas de producto con ejemplos en texto | catálogo de regalos |
| 4 | `#suministros` | "Los consumibles que ya compras, a precio de origen". 3 pasos: foto/factura → cotización 48 h → muestra → suministro recurrente. Nota: "verificamos certificaciones UE (p. ej. contacto alimentario) antes de cotizar" | cotizar consumibles |
| 5 | `#revende` | "El margen del distribuidor, para ti": compra directo de fábrica en vez de a almacenistas locales. Muestras antes de pedir | importar para revender |
| 6 | Quién está detrás | Alejandro con foto, primera persona, "te atiendo yo por WhatsApp" | genérico |
| 7 | Contacto | Bloque WhatsApp grande + formulario actual plegado bajo "¿Prefieres email?" | genérico |
| 8 | Footer | + enlace a `privacidad.html` |  |

Transversal: **botón flotante de WhatsApp** al hacer scroll; la línea de cantidades
("cuéntanos tu cantidad…") aparece junto al CTA de cada sección de público.

## Tracking (Meta Pixel)

- Clic en cualquier enlace de WhatsApp → evento estándar **`Contact`** con
  `content_category` = sección de origen (regalos / suministros / revende / general).
- Formulario → mantiene **`Lead`** (no contaminar la señal de optimización de campañas).
- `PageView` sin cambios. CSP de nginx sin cambios (wa.me es navegación, no recurso).

## i18n

ES/EN se mantiene en `index.html` (diccionario ampliado, movido a `app.js`).
`privacidad.html` solo en español (suficiente legalmente para el mercado objetivo).

## Manejo de errores

- Enlaces `wa.me` son `<a>` normales: funcionan sin JavaScript (el tracking es best-effort).
- Formulario: mismo comportamiento actual (validación, estados, fallback de error).

## Verificación

1. Servidor local (`python -m http.server`) → revisión visual móvil + escritorio, ES y EN.
2. Anclas: probar aterrizaje directo en `#regalos`, `#suministros`, `#revende`.
3. Pixel: verificar `Contact`/`Lead` con Meta Pixel Helper.
4. Deploy: push a GitHub → la Orange Pi lo coge por cron en ≤5 min → revisar en producción.

## Pendiente de Alex (no bloquea empezar, sí bloquea publicar)

- [ ] Foto para el bloque "Quién está detrás" (si no hay, arrancamos sin foto: nombre + texto)
- [ ] Datos del responsable de datos para `privacidad.html` (nombre/NIF/dirección) — placeholders mientras
- [ ] Validar los 4 textos prellenados de WhatsApp cuando estén redactados

## Fuera de alcance (futuro)

- Fotografía de producto propia (tras el viaje a China) y catálogo navegable completo.
- Landing pages dedicadas por campaña (reevaluar si las secciones ancladas rinden mal).
- Catálogos PDF a medida por cliente (pipeline Jamón Suprem, proceso aparte).
