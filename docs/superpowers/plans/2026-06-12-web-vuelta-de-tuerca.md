# Web Vuelta de Tuerca — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Transformar la landing actual en una web todo-en-uno WhatsApp-first con tres secciones ancladas para tres públicos distintos (#regalos, #suministros, #revende), ambiente personal y Pixel correctamente segmentado.

**Architecture:** Un único `index.html` con secciones ancladas (sin build system). CSS extraído a `styles.css` y lógica a `app.js` compartidos. `privacidad.html` nueva. Deploy automático vía git pull cron en la Orange Pi.

**Tech Stack:** HTML5, CSS3 vanilla, JS vanilla ES2020, Web3Forms (formulario), Meta Pixel, WhatsApp `wa.me` links. Servidor local: `python -m http.server 8000`.

**Spec:** `docs/superpowers/specs/2026-06-12-web-vuelta-de-tuerca-design.md`

---

## Ficheros

| Acción | Ruta | Responsabilidad |
|--------|------|-----------------|
| Crear | `styles.css` | Todos los estilos (extraído del `<style>` inline de index.html + nuevas secciones) |
| Crear | `app.js` | i18n dict + setLang() + WhatsApp helpers + Pixel tracking + scroll/observer/form |
| Modificar | `index.html` | Esqueleto HTML únicamente: secciones, textos, `<link>`/`<script>` externos |
| Crear | `privacidad.html` | Política de privacidad (solo HTML + `<link rel="stylesheet" href="styles.css">`) |

---

## Task 0: Proteger /docs de acceso público

El directorio `docs/` se sirve junto al repo vía nginx en la Orange Pi. Los specs/planes no son sensibles pero tampoco deben ser indexables. Hay dos opciones; elige una:

**Opción A — deny en nginx (recomendada):** SSH a la Orange Pi y añadir en el bloque `server` del site de como-la-seda:
```nginx
location /docs {
    deny all;
    return 404;
}
```
Luego: `sudo nginx -t && sudo systemctl reload nginx`

**Opción B — ignorar por ahora:** El contenido no es sensible. Seguir; revisitar antes de añadir specs con datos de clientes.

- [ ] **Decidir opción y ejecutarla (o dejar constancia de que se ignora conscientemente)**

---

## Task 1: Extraer CSS a styles.css

**Files:**
- Crear: `styles.css`
- Modificar: `index.html` (bloque `<style>` → `<link>`)

- [ ] **Copiar el contenido íntegro del bloque `<style>` de index.html** (líneas 33–294) a `styles.css`. Sin cambios de contenido — copia literal.

- [ ] **Sustituir el bloque `<style>` en index.html** por:
```html
<link rel="stylesheet" href="styles.css">
```
Colocar justo antes de `</head>`, después de las líneas de Google Fonts.

- [ ] **Verificar visualmente**
```bash
python -m http.server 8000
# Abrir http://localhost:8000 en navegador
```
La página debe verse idéntica a antes. Comprobar en móvil (Firefox DevTools → modo responsive).

- [ ] **Commit**
```bash
git add styles.css index.html
git commit -m "refactor: extraer CSS a styles.css"
```

---

## Task 2: Extraer JS a app.js

**Files:**
- Crear: `app.js`
- Modificar: `index.html` (bloque `<script>` → `<script src="app.js">`)

- [ ] **Mover el contenido del bloque `<script>` de index.html** (desde `const T = {` hasta el final antes de `</script>`) a `app.js`. Copia literal, sin cambios.

- [ ] **Sustituir el bloque `<script>` en index.html** por:
```html
<script src="app.js"></script>
```
Justo antes de `</body>`.

- [ ] **Verificar: cambio de idioma funciona**
```
http://localhost:8000 → clic en EN → textos cambian al inglés
```

- [ ] **Verificar: formulario funciona**
```
Rellenar los campos requeridos → clic "Enviar solicitud" → aparece el mensaje de éxito
```
(Web3Forms acepta envíos en local pero puede pedir CORS — si falla, es normal; lo importante es que el JS no lanza errores en consola antes del submit.)

- [ ] **Commit**
```bash
git add app.js index.html
git commit -m "refactor: extraer JS a app.js"
```

---

## Task 3: Actualizar datos de contacto en index.html y app.js

**Files:**
- Modificar: `index.html` (teléfono visible)
- Modificar: `app.js` (strings i18n con teléfono)

El teléfono comercial ha cambiado de **622 200 594** a **680 211 326**.

- [ ] **En index.html**, reemplazar TODAS las apariciones de `622 200 594` y `+34622200594` por `680 211 326` / `+34680211326` (usar búsqueda global en el editor).

- [ ] **En app.js**, reemplazar en el diccionario `T` la cadena de teléfono en ambos idiomas (es/en). Buscar `622` — debería aparecer en el placeholder del campo `f.tel`.

- [ ] **Verificar:** el número visible en la sección de contacto y en el footer es el nuevo.

- [ ] **Commit**
```bash
git add index.html app.js
git commit -m "fix: actualizar teléfono comercial a 680 211 326"
```

---

## Task 4: Header actualizado — nav de secciones + botón WhatsApp

**Files:**
- Modificar: `index.html` (bloque `<header id="hdr">`)
- Modificar: `styles.css` (estilos de nav y botón WhatsApp en header)
- Modificar: `app.js` (claves i18n nuevas)

- [ ] **Añadir en app.js, dentro de `T.es` y `T.en`**, las claves de nav:
```js
// ES
'nav.regalos': 'Regalos',
'nav.suministros': 'Suministros',
'nav.revende': 'Revende',
'nav.cta': 'Escríbenos por WhatsApp',

// EN
'nav.regalos': 'Gifts',
'nav.suministros': 'Supplies',
'nav.revende': 'Resell',
'nav.cta': 'WhatsApp us',
```

- [ ] **Sustituir el bloque `<header id="hdr">` en index.html** por:
```html
<header id="hdr">
  <div class="wrap">
    <a href="#" class="hdr-logo logo-wrap" aria-label="Como la Seda">
      como la <span class="seda-inner">seda<svg class="seda-wave" viewBox="0 0 56 8" fill="none" preserveAspectRatio="none"><path d="M0,4 Q7,0 14,4 Q21,8 28,4 Q35,0 42,4 Q49,8 56,4" stroke="#C2A557" stroke-width="1.9" fill="none"/></svg></span>
    </a>
    <nav class="hdr-nav" aria-label="Secciones">
      <a href="#regalos" class="hdr-nav-link" data-i18n="nav.regalos">Regalos</a>
      <a href="#suministros" class="hdr-nav-link" data-i18n="nav.suministros">Suministros</a>
      <a href="#revende" class="hdr-nav-link" data-i18n="nav.revende">Revende</a>
    </nav>
    <div class="hdr-right">
      <div class="lang" role="group" aria-label="Idioma / Language">
        <button class="lang-btn on" data-lang="es" onclick="setLang('es')">ES</button>
        <span class="lang-sep">|</span>
        <button class="lang-btn" data-lang="en" onclick="setLang('en')">EN</button>
      </div>
      <a href="https://wa.me/34680211326?text=Hola%2C+vengo+de+la+web+y+quiero+información"
         class="btn btn-gold hdr-cta"
         data-i18n="nav.cta"
         onclick="trackWA('general')">Escríbenos por WhatsApp</a>
    </div>
  </div>
</header>
```

- [ ] **Añadir en styles.css**, al final de la sección `/* ─── Header ─── */`:
```css
.hdr-nav{display:flex;align-items:center;gap:24px}
.hdr-nav-link{
  font-size:.78rem;font-weight:500;letter-spacing:.08em;
  text-transform:uppercase;color:rgba(242,232,220,.6);
  transition:color var(--ease);
}
.hdr-nav-link:hover{color:var(--cream)}
@media(max-width:900px){.hdr-nav{display:none}}
```

- [ ] **Añadir en app.js** la función `trackWA` (antes del código de formulario):
```js
/* ─── WhatsApp tracking ─── */
function trackWA(category) {
  if (window.fbq) fbq('track', 'Contact', { content_category: category });
}
```

- [ ] **Verificar:** los tres enlaces de nav aparecen en escritorio, desaparecen en móvil (<900px), el botón lleva a wa.me con el mensaje correcto (probar con Ctrl+clic en escritorio).

- [ ] **Commit**
```bash
git add index.html styles.css app.js
git commit -m "feat: header con nav de secciones y CTA WhatsApp"
```

---

## Task 5: Hero — propuesta para los 3 públicos

**Files:**
- Modificar: `index.html` (bloque `<section id="hero">`)
- Modificar: `app.js` (claves i18n del hero)

- [ ] **Actualizar claves del hero en app.js** (`T.es` y `T.en`):
```js
// ES
'hero.tag': 'Regalos · Suministros · Importación directa',
'hero.h1': 'Precio de fábrica,<br>sin intermediarios',
'hero.sub': 'Importamos directo desde China. Con tu logo, para tus empleados, o para vender. Te atiendo yo, por WhatsApp.',
'hero.cta': 'Pide el catálogo por WhatsApp',

// EN
'hero.tag': 'Gifts · Supplies · Direct import',
'hero.h1': 'Factory price,<br>no middlemen',
'hero.sub': 'Direct imports from China. Branded, for your team, or to resell. I\'ll handle it personally, via WhatsApp.',
'hero.cta': 'Get the catalogue on WhatsApp',
```

- [ ] **Actualizar el CTA del hero en index.html** — localizar el `<a href="#cta" class="btn btn-gold"` del hero y sustituir por:
```html
<a href="https://wa.me/34680211326?text=Hola%2C+quiero+ver+el+catálogo"
   class="btn btn-gold"
   data-i18n="hero.cta"
   onclick="trackWA('general')">Pide el catálogo por WhatsApp</a>
```

- [ ] **Verificar:** el hero muestra el nuevo copy en ES y EN, el botón abre WhatsApp con el mensaje correcto.

- [ ] **Commit**
```bash
git add index.html app.js
git commit -m "feat: hero actualizado para 3 públicos"
```

---

## Task 6: Sección #regalos (evolución de la sección actual)

**Files:**
- Modificar: `index.html` (sección `<section id="porq">` → renombrar a `#regalos`)
- Modificar: `app.js` (claves i18n)
- Modificar: `styles.css` (ajuste menor del catálogo teaser)

- [ ] **Renombrar la sección en index.html:** cambiar `<section id="porq">` por `<section id="regalos">`. Actualizar también el `href="#porq"` en cualquier lugar que exista (buscar globalmente).

- [ ] **Actualizar claves en app.js** (`T.es` y `T.en`):
```js
// ES
'regalos.label': 'Regalos de empresa',
'regalos.title': 'Tu marca en cada detalle',
'porq.3.p': 'Antes de fabricar, puedes tocar y validar el producto. ¿Pedido pequeño? Cuéntanos tu cantidad y te decimos sin compromiso si te compensa.',
'cat.cta': 'Pide el catálogo completo por WhatsApp',

// EN
'regalos.label': 'Corporate gifts',
'regalos.title': 'Your brand on every detail',
'porq.3.p': 'Before manufacturing, you can touch and validate the product. Small order? Tell us your quantity and we\'ll let you know if it works for you.',
'cat.cta': 'Get the full catalogue on WhatsApp',
```

- [ ] **En index.html**, actualizar el encabezado de la sección para usar las nuevas claves:
```html
<span class="label" data-i18n="regalos.label">Regalos de empresa</span>
<h2 class="title" data-i18n="regalos.title">Tu marca en cada detalle</h2>
```

- [ ] **Actualizar el CTA del catálogo** en index.html (el `<a class="btn btn-outline"` al final de `#cat`):
```html
<a href="https://wa.me/34680211326?text=Hola%2C+quiero+el+catálogo+de+regalos+de+empresa"
   class="btn btn-outline"
   data-i18n="cat.cta"
   onclick="trackWA('regalos')">Pide el catálogo completo por WhatsApp</a>
```

- [ ] **Añadir i18n a los textos de productos en el catálogo** (si no tienen `data-i18n`, añadirlo a cada `.cat-name`): ya tienen `data-i18n="cat.1"` etc — mantener.

- [ ] **Verificar:** sección se ve correcta, la objeción de cantidades aparece en la tarjeta de muestras, el CTA del catálogo abre WhatsApp con mensaje de regalos.

- [ ] **Commit**
```bash
git add index.html app.js
git commit -m "feat: sección #regalos con objeción de cantidades y CTA WhatsApp"
```

---

## Task 7: Sección #suministros (NUEVA)

**Files:**
- Modificar: `index.html` (añadir sección después de `#regalos`)
- Modificar: `app.js` (claves i18n)
- Modificar: `styles.css` (estilos de la sección de pasos)

- [ ] **Añadir claves en app.js** (`T.es` y `T.en`):
```js
// ES
'sumi.label': 'Suministros recurrentes',
'sumi.title': 'Los consumibles que ya compras,<br>a precio de origen',
'sumi.sub': 'Si ya tienes proveedores de materiales, envases o consumibles, conseguimos lo mismo directo de fábrica. El ahorro se acumula cada pedido.',
'sumi.step1.h': 'Mándanos una foto o factura',
'sumi.step1.p': 'Del producto que ya compras. Con eso buscamos el equivalente en fábrica.',
'sumi.step2.h': 'Cotización en 48 h',
'sumi.step2.p': 'Con precio de fábrica, plazo de entrega y muestra disponible si la quieres.',
'sumi.step3.h': 'Suministro recurrente',
'sumi.step3.p': 'Una vez validado, hacemos los pedidos cuando los necesites. Sin sorpresas.',
'sumi.note': '⚠️ Verificamos certificaciones UE (p. ej. contacto alimentario) antes de cotizar.',
'sumi.cta': 'Cotiza tus consumibles por WhatsApp',

// EN
'sumi.label': 'Recurring supplies',
'sumi.title': 'The supplies you already buy,<br>at factory price',
'sumi.sub': 'Already buying materials, packaging or consumables? We source the same direct from factory. Savings compound with every order.',
'sumi.step1.h': 'Send us a photo or invoice',
'sumi.step1.p': 'Of the product you already buy. We\'ll find the factory equivalent.',
'sumi.step2.h': 'Quote in 48 h',
'sumi.step2.p': 'Factory price, lead time and a sample if you want one.',
'sumi.step3.h': 'Recurring supply',
'sumi.step3.p': 'Once validated, we handle orders whenever you need them. No surprises.',
'sumi.note': '⚠️ We verify EU certifications (e.g. food contact) before quoting.',
'sumi.cta': 'Quote your supplies on WhatsApp',
```

- [ ] **Añadir en index.html**, justo después del cierre `</section>` de `#regalos` (y antes de `<section id="cat">`), la nueva sección:
```html
<!-- ═══════════════════════════ SUMINISTROS ═══════════════════════════ -->
<section id="suministros">
  <div class="wrap">
    <div class="sumi-head">
      <span class="label" data-i18n="sumi.label">Suministros recurrentes</span>
      <h2 class="title" data-i18n-h="sumi.title">Los consumibles que ya compras,<br>a precio de origen</h2>
      <div class="divider"><div class="divider-dot"></div></div>
      <p class="sumi-sub" data-i18n="sumi.sub">Si ya tienes proveedores de materiales, envases o consumibles, conseguimos lo mismo directo de fábrica.</p>
    </div>
    <div class="sumi-steps">
      <div class="sumi-step fu">
        <div class="sumi-num">1</div>
        <h3 data-i18n="sumi.step1.h">Mándanos una foto o factura</h3>
        <p data-i18n="sumi.step1.p">Del producto que ya compras. Con eso buscamos el equivalente en fábrica.</p>
      </div>
      <div class="sumi-step fu">
        <div class="sumi-num">2</div>
        <h3 data-i18n="sumi.step2.h">Cotización en 48 h</h3>
        <p data-i18n="sumi.step2.p">Con precio de fábrica, plazo de entrega y muestra disponible si la quieres.</p>
      </div>
      <div class="sumi-step fu">
        <div class="sumi-num">3</div>
        <h3 data-i18n="sumi.step3.h">Suministro recurrente</h3>
        <p data-i18n="sumi.step3.p">Una vez validado, hacemos los pedidos cuando los necesites. Sin sorpresas.</p>
      </div>
    </div>
    <p class="sumi-note" data-i18n="sumi.note">⚠️ Verificamos certificaciones UE (p. ej. contacto alimentario) antes de cotizar.</p>
    <div class="sumi-cta">
      <a href="https://wa.me/34680211326?text=Hola%2C+quiero+cotizar+mis+consumibles"
         class="btn btn-gold"
         data-i18n="sumi.cta"
         onclick="trackWA('suministros')">Cotiza tus consumibles por WhatsApp</a>
    </div>
  </div>
</section>
```

- [ ] **Añadir en styles.css** los estilos de la sección:
```css
/* ─── Suministros ─── */
#suministros{background:var(--cream);padding:100px 0}
.sumi-head{text-align:center;margin-bottom:60px}
.sumi-sub{max-width:580px;margin:20px auto 0;color:rgba(27,27,27,.65);font-size:.97rem;line-height:1.75}
.sumi-steps{display:grid;grid-template-columns:repeat(3,1fr);gap:32px;margin-bottom:40px}
.sumi-step{background:#fff;padding:40px 28px;text-align:center;border-bottom:3px solid transparent;transition:transform var(--ease),box-shadow var(--ease)}
.sumi-step:hover{transform:translateY(-5px);box-shadow:0 20px 48px rgba(0,0,0,.08)}
.sumi-num{width:52px;height:52px;border-radius:50%;background:rgba(168,50,44,.08);border:2px solid rgba(168,50,44,.2);display:flex;align-items:center;justify-content:center;margin:0 auto 20px;font-family:var(--serif);font-size:1.4rem;font-weight:600;color:var(--red)}
.sumi-step h3{font-family:var(--serif);font-size:1.2rem;font-weight:600;color:var(--dark);margin-bottom:10px}
.sumi-step p{font-size:.9rem;color:rgba(27,27,27,.65);line-height:1.75}
.sumi-note{text-align:center;font-size:.82rem;color:rgba(27,27,27,.5);margin-bottom:36px}
.sumi-cta{text-align:center}
@media(max-width:768px){.sumi-steps{grid-template-columns:1fr}}
```

- [ ] **Verificar:** sección visible, los 3 pasos con números, nota de normativa, CTA de WhatsApp abre con mensaje de consumibles.

- [ ] **Commit**
```bash
git add index.html app.js styles.css
git commit -m "feat: sección #suministros (consumibles recurrentes)"
```

---

## Task 8: Sección #revende (NUEVA)

**Files:**
- Modificar: `index.html` (añadir sección después de `#suministros`)
- Modificar: `app.js` (claves i18n)
- Modificar: `styles.css` (estilos)

- [ ] **Añadir claves en app.js** (`T.es` y `T.en`):
```js
// ES
'rev.label': 'Importa para revender',
'rev.title': 'El margen del distribuidor,<br>para ti',
'rev.sub': 'Hoy le compras a un almacenista local que tiene el stock, te fija el precio y se queda su margen. Nosotros te lo traemos de fábrica: tú decides el precio y te quedas la diferencia.',
'rev.v1.h': 'Sin depender de stock ajeno',
'rev.v1.p': 'Pides lo que necesitas cuando lo necesitas, sin mínimos imposibles de un distribuidor.',
'rev.v2.h': 'Muestras antes de producir',
'rev.v2.p': 'Prueba el producto antes de comprometerte. Sin sorpresas en calidad ni acabados.',
'rev.v3.h': 'Precio de fábrica real',
'rev.v3.p': 'Acceso directo a los mismos precios que tienen los distribuidores. El margen es tuyo.',
'rev.cta': 'Hablemos de lo que quieres importar',

// EN
'rev.label': 'Import to resell',
'rev.title': 'The distributor\'s margin,<br>yours to keep',
'rev.sub': 'Right now you\'re buying from a local stockist who sets the price and pockets the margin. We bring it from the factory: you set the price, you keep the difference.',
'rev.v1.h': 'No dependence on external stock',
'rev.v1.p': 'Order what you need when you need it, without impossible distributor minimums.',
'rev.v2.h': 'Samples before production',
'rev.v2.p': 'Try before you commit. No surprises on quality or finish.',
'rev.v3.h': 'Real factory price',
'rev.v3.p': 'Same prices the distributors pay. The margin is yours.',
'rev.cta': 'Let\'s talk about what you want to import',
```

- [ ] **Añadir en index.html**, justo después del cierre `</section>` de `#suministros`:
```html
<!-- ═══════════════════════════ REVENDE ═══════════════════════════ -->
<section id="revende">
  <div class="wrap">
    <div class="rev-head">
      <span class="label" data-i18n="rev.label">Importa para revender</span>
      <h2 class="title" data-i18n-h="rev.title">El margen del distribuidor,<br>para ti</h2>
      <div class="divider"><div class="divider-dot"></div></div>
      <p class="rev-sub" data-i18n="rev.sub">Hoy le compras a un almacenista local que tiene el stock, te fija el precio y se queda su margen.</p>
    </div>
    <div class="rev-grid">
      <div class="porq-card fu">
        <div class="porq-icon">
          <svg viewBox="0 0 24 24"><path d="M20 7H4a2 2 0 00-2 2v6a2 2 0 002 2h16a2 2 0 002-2V9a2 2 0 00-2-2z"/><path d="M16 3v4M8 3v4M3 11h18"/></svg>
        </div>
        <h3 data-i18n="rev.v1.h">Sin depender de stock ajeno</h3>
        <p data-i18n="rev.v1.p">Pides lo que necesitas cuando lo necesitas, sin mínimos imposibles de un distribuidor.</p>
      </div>
      <div class="porq-card fu">
        <div class="porq-icon">
          <svg viewBox="0 0 24 24"><path d="M9 12l2 2 4-4"/><path d="M21 12a9 9 0 11-18 0 9 9 0 0118 0z"/></svg>
        </div>
        <h3 data-i18n="rev.v2.h">Muestras antes de producir</h3>
        <p data-i18n="rev.v2.p">Prueba el producto antes de comprometerte. Sin sorpresas en calidad ni acabados.</p>
      </div>
      <div class="porq-card fu">
        <div class="porq-icon">
          <svg viewBox="0 0 24 24"><path d="M12 2v20M17 5H9.5a3.5 3.5 0 000 7h5a3.5 3.5 0 010 7H6"/></svg>
        </div>
        <h3 data-i18n="rev.v3.h">Precio de fábrica real</h3>
        <p data-i18n="rev.v3.p">Acceso directo a los mismos precios que tienen los distribuidores. El margen es tuyo.</p>
      </div>
    </div>
    <div class="rev-cta">
      <a href="https://wa.me/34680211326?text=Hola%2C+quiero+información+para+importar+y+revender"
         class="btn btn-gold"
         data-i18n="rev.cta"
         onclick="trackWA('revende')">Hablemos de lo que quieres importar</a>
    </div>
  </div>
</section>
```

- [ ] **Añadir en styles.css**:
```css
/* ─── Revende ─── */
#revende{background:#f7f1e9;padding:100px 0}
.rev-head{text-align:center;margin-bottom:60px}
.rev-sub{max-width:600px;margin:20px auto 0;color:rgba(27,27,27,.65);font-size:.97rem;line-height:1.75}
.rev-grid{display:grid;grid-template-columns:repeat(3,1fr);gap:32px;margin-bottom:52px}
.rev-cta{text-align:center}
@media(max-width:768px){.rev-grid{grid-template-columns:1fr;max-width:460px;margin-left:auto;margin-right:auto}}
```

- [ ] **Verificar:** sección visible con las 3 tarjetas, CTA abre WhatsApp con mensaje de revendedor.

- [ ] **Commit**
```bash
git add index.html app.js styles.css
git commit -m "feat: sección #revende (importar para revender)"
```

---

## Task 9: Bloque "Quién está detrás"

**Files:**
- Modificar: `index.html` (añadir sección antes de `#cta`)
- Modificar: `app.js` (claves i18n)
- Modificar: `styles.css` (estilos)

- [ ] **Añadir claves en app.js**:
```js
// ES
'who.label': 'Quién está detrás',
'who.title': 'Hola, soy Alejandro',
'who.p1': 'Te atiendo yo directamente, sin comerciales ni call centers. Cuando me escribas por WhatsApp, vas a hablar conmigo.',
'who.p2': 'Me encargo de buscar el proveedor, negociar el precio, supervisar la producción y coordinar el envío. Tú te despreocupas.',
'who.cta': 'Escríbeme directamente',

// EN
'who.label': 'Who\'s behind this',
'who.title': 'Hi, I\'m Alejandro',
'who.p1': 'You deal directly with me — no sales reps, no call centres. When you message on WhatsApp, you\'re talking to me.',
'who.p2': 'I handle finding the supplier, negotiating the price, supervising production and coordinating shipping. You don\'t have to worry about any of it.',
'who.cta': 'Message me directly',
```

- [ ] **Añadir en index.html** justo antes del `<section id="cta">`:
```html
<!-- ═══════════════════════════ QUIÉN ═══════════════════════════ -->
<section id="quien">
  <div class="wrap">
    <div class="who-inner">
      <div class="who-photo">
        <!-- Foto de Alejandro — sustituir src cuando esté disponible -->
        <div class="who-photo-placeholder" aria-hidden="true">A</div>
      </div>
      <div class="who-text">
        <span class="label" data-i18n="who.label">Quién está detrás</span>
        <h2 class="title" data-i18n="who.title">Hola, soy Alejandro</h2>
        <p class="who-p" data-i18n="who.p1">Te atiendo yo directamente, sin comerciales ni call centers. Cuando me escribas por WhatsApp, vas a hablar conmigo.</p>
        <p class="who-p" data-i18n="who.p2">Me encargo de buscar el proveedor, negociar el precio, supervisar la producción y coordinar el envío. Tú te despreocupas.</p>
        <a href="https://wa.me/34680211326?text=Hola+Alejandro%2C+vengo+de+tu+web"
           class="btn btn-gold"
           data-i18n="who.cta"
           onclick="trackWA('general')">Escríbeme directamente</a>
      </div>
    </div>
  </div>
</section>
```

- [ ] **Añadir en styles.css**:
```css
/* ─── Quién está detrás ─── */
#quien{background:var(--dark);padding:100px 0}
.who-inner{display:grid;grid-template-columns:280px 1fr;gap:72px;align-items:center}
.who-photo{display:flex;justify-content:center}
.who-photo-placeholder{
  width:220px;height:220px;border-radius:50%;
  background:rgba(194,165,87,.12);border:3px solid var(--gold);
  display:flex;align-items:center;justify-content:center;
  font-family:var(--serif);font-size:4rem;font-weight:600;color:var(--gold);
  flex-shrink:0;
}
.who-text .title{color:var(--cream);margin-bottom:20px}
.who-text .label{margin-bottom:12px}
.who-p{color:rgba(242,232,220,.68);font-size:.97rem;line-height:1.8;margin-bottom:16px}
.who-text .btn{margin-top:12px}
@media(max-width:768px){
  .who-inner{grid-template-columns:1fr;gap:36px;text-align:center}
  .who-photo{justify-content:center}
  .who-photo-placeholder{width:140px;height:140px;font-size:2.5rem}
  .who-text .btn{width:100%;justify-content:center}
}
```

- [ ] **Verificar:** bloque visible en fondo oscuro, placeholder circular con "A", CTA abre WhatsApp con mensaje personal.

> Cuando Alex tenga foto: sustituir `<div class="who-photo-placeholder">` por `<img src="alejandro.jpg" alt="Alejandro" class="who-photo-img">` y añadir `.who-photo-img{width:220px;height:220px;border-radius:50%;object-fit:cover;border:3px solid var(--gold)}` en styles.css.

- [ ] **Commit**
```bash
git add index.html app.js styles.css
git commit -m "feat: bloque quién está detrás con trato personal"
```

---

## Task 10: Sección de contacto — WhatsApp protagonista, formulario secundario

**Files:**
- Modificar: `index.html` (sección `#cta`)
- Modificar: `app.js` (claves i18n, toggle del formulario)
- Modificar: `styles.css` (estilos del toggle)

- [ ] **Actualizar claves en app.js**:
```js
// ES (sustituir las existentes de cta.*)
'cta.label': 'Contacto',
'cta.title': 'Escríbeme por WhatsApp',
'cta.sub': 'Es la forma más rápida. Te respondo yo mismo, normalmente en pocas horas.',
'cta.wa.btn': 'Abrir WhatsApp ahora',
'cta.email.toggle': '¿Prefieres el formulario?',
'cta.location': 'Acceso presencial a fábricas en<br>Guangzhou, Yiwu y Shenzhen',

// EN
'cta.title': 'Message me on WhatsApp',
'cta.sub': 'It\'s the fastest way. I reply personally, usually within a few hours.',
'cta.wa.btn': 'Open WhatsApp now',
'cta.email.toggle': 'Prefer the form?',
'cta.location': 'On-site factory access in<br>Guangzhou, Yiwu and Shenzhen',
```

- [ ] **Sustituir el bloque `<section id="cta">` completo** en index.html:
```html
<!-- ═══════════════════════════ CONTACTO ═══════════════════════════ -->
<section id="cta">
  <div class="wrap">
    <div class="cta-inner">

      <div class="cta-info">
        <span class="label" data-i18n="cta.label">Contacto</span>
        <h2 class="title" data-i18n="cta.title">Escríbeme por WhatsApp</h2>
        <p class="cta-sub" data-i18n="cta.sub">Es la forma más rápida. Te respondo yo mismo, normalmente en pocas horas.</p>
        <div class="cta-detail">
          <svg viewBox="0 0 24 24"><path d="M4 4h16c1.1 0 2 .9 2 2v12c0 1.1-.9 2-2 2H4c-1.1 0-2-.9-2-2V6c0-1.1.9-2 2-2z"/><polyline points="22,6 12,13 2,6"/></svg>
          <span>comolaseda@wisdowl.com</span>
        </div>
        <div class="cta-detail">
          <svg viewBox="0 0 24 24"><path d="M22 16.92v3a2 2 0 01-2.18 2 19.79 19.79 0 01-8.63-3.07A19.5 19.5 0 013.07 11.5a19.79 19.79 0 01-3.07-8.67A2 2 0 011.72 1h3a2 2 0 012 1.72c.127.96.361 1.903.7 2.81a2 2 0 01-.45 2.11L5.91 8.09a16 16 0 006 6l1.27-1.27a2 2 0 012.11-.45c.907.339 1.85.573 2.81.7A2 2 0 0122 16.92z"/></svg>
          <a href="tel:+34680211326"><span>+34 680 211 326</span></a>
        </div>
        <div class="cta-detail">
          <svg viewBox="0 0 24 24"><path d="M12 22s-8-4.5-8-11.8A8 8 0 0112 2a8 8 0 018 8.2c0 7.3-8 11.8-8 11.8z"/><circle cx="12" cy="10" r="3"/></svg>
          <span data-i18n-h="cta.location">Acceso presencial a fábricas en<br>Guangzhou, Yiwu y Shenzhen</span>
        </div>
      </div>

      <div class="cta-right">
        <a href="https://wa.me/34680211326?text=Hola%2C+vengo+de+tu+web+y+quiero+información"
           class="btn-wa-big"
           data-i18n="cta.wa.btn"
           onclick="trackWA('general')">
          <svg viewBox="0 0 24 24" width="28" height="28" fill="currentColor"><path d="M17.472 14.382c-.297-.149-1.758-.867-2.03-.967-.273-.099-.471-.148-.67.15-.197.297-.767.966-.94 1.164-.173.199-.347.223-.644.075-.297-.15-1.255-.463-2.39-1.475-.883-.788-1.48-1.761-1.653-2.059-.173-.297-.018-.458.13-.606.134-.133.298-.347.446-.52.149-.174.198-.298.298-.497.099-.198.05-.371-.025-.52-.075-.149-.669-1.612-.916-2.207-.242-.579-.487-.5-.669-.51-.173-.008-.371-.01-.57-.01-.198 0-.52.074-.792.372-.272.297-1.04 1.016-1.04 2.479 0 1.462 1.065 2.875 1.213 3.074.149.198 2.096 3.2 5.077 4.487.709.306 1.262.489 1.694.625.712.227 1.36.195 1.871.118.571-.085 1.758-.719 2.006-1.413.248-.694.248-1.289.173-1.413-.074-.124-.272-.198-.57-.347z"/><path d="M12 0C5.373 0 0 5.373 0 12c0 2.127.558 4.126 1.532 5.856L.057 23.625a.5.5 0 00.624.606l5.865-1.538A11.95 11.95 0 0012 24c6.627 0 12-5.373 12-12S18.627 0 12 0zm0 21.75a9.713 9.713 0 01-4.953-1.352l-.355-.211-3.68.965.984-3.595-.232-.37A9.712 9.712 0 012.25 12C2.25 6.615 6.615 2.25 12 2.25S21.75 6.615 21.75 12 17.385 21.75 12 21.75z"/></svg>
          <span>Abrir WhatsApp ahora</span>
        </a>

        <button class="cta-email-toggle" onclick="toggleForm()" data-i18n="cta.email.toggle">
          ¿Prefieres el formulario?
        </button>

        <div id="emailForm" class="email-form-wrap" hidden>
          <form class="cform" id="cForm" novalidate>
            <div class="frow">
              <div class="fg">
                <label for="f-nombre" data-i18n="f.nombre">Nombre</label>
                <input type="text" id="f-nombre" name="nombre" required placeholder="Tu nombre">
              </div>
              <div class="fg">
                <label for="f-empresa" data-i18n="f.empresa">Empresa</label>
                <input type="text" id="f-empresa" name="empresa" required placeholder="Nombre de tu empresa">
              </div>
            </div>
            <div class="frow">
              <div class="fg">
                <label for="f-email" data-i18n="f.email">Email</label>
                <input type="email" id="f-email" name="email" required placeholder="tu@empresa.com">
              </div>
              <div class="fg">
                <label for="f-tel" data-i18n="f.tel">Teléfono</label>
                <input type="tel" id="f-tel" name="tel" placeholder="+34 680 211 326">
              </div>
            </div>
            <div class="fg">
              <label for="f-msg" data-i18n="f.msg">Qué necesitas</label>
              <textarea id="f-msg" name="msg" required placeholder="Cuéntanos qué producto te interesa, para qué ocasión, si necesitas personalización..."></textarea>
            </div>
            <div class="fg">
              <label for="f-qty" data-i18n="f.qty">Cantidad aproximada</label>
              <select id="f-qty" name="qty">
                <option value="" disabled selected data-i18n="f.qty.ph">Selecciona una opción</option>
                <option value="<50"     data-i18n="f.qty.1">Menos de 50 unidades</option>
                <option value="50-200"  data-i18n="f.qty.2">50 – 200 unidades</option>
                <option value="200-500" data-i18n="f.qty.3">200 – 500 unidades</option>
                <option value="500-1k"  data-i18n="f.qty.4">500 – 1.000 unidades</option>
                <option value=">1k"     data-i18n="f.qty.5">Más de 1.000 unidades</option>
              </select>
            </div>
            <input type="hidden" name="access_key" value="2daea4a6-3360-4954-84e9-eb9ee65320e5">
            <input type="hidden" name="replyto" id="f-replyto">
            <input type="hidden" name="subject" id="f-subject">
            <input type="hidden" name="from_name" value="Como la Seda — Web">
            <button type="submit" class="btn btn-gold fsub" data-i18n="f.send">Enviar solicitud</button>
            <p class="fnote" data-i18n="f.note">Sin compromiso · Respuesta en menos de 24 h</p>
          </form>
          <div class="fsuccess" id="fOk">
            <div class="fsuccess-ico">
              <svg viewBox="0 0 24 24"><polyline points="20 6 9 17 4 12"/></svg>
            </div>
            <h3 data-i18n="f.ok.h">¡Solicitud recibida!</h3>
            <p data-i18n="f.ok.p">Te contactaremos en menos de 24 horas con una propuesta personalizada.</p>
          </div>
        </div>
      </div>

    </div>
  </div>
</section>
```

- [ ] **Añadir en app.js** la función `toggleForm` (junto a las demás funciones):
```js
/* ─── Toggle formulario ─── */
function toggleForm() {
  const wrap = document.getElementById('emailForm');
  const btn = document.querySelector('.cta-email-toggle');
  const hidden = wrap.hasAttribute('hidden');
  if (hidden) { wrap.removeAttribute('hidden'); btn.textContent = '✕ Cerrar formulario'; }
  else { wrap.setAttribute('hidden', ''); btn.textContent = document.documentElement.lang === 'en' ? 'Prefer the form?' : '¿Prefieres el formulario?'; }
}
```

- [ ] **Añadir en styles.css**:
```css
/* ─── CTA WhatsApp grande ─── */
.cta-right{display:flex;flex-direction:column;align-items:flex-start;gap:20px}
.btn-wa-big{
  display:inline-flex;align-items:center;gap:14px;
  padding:20px 36px;background:#25D366;color:#fff;
  font-family:var(--sans);font-size:1rem;font-weight:600;
  letter-spacing:.04em;transition:all var(--ease);width:100%;justify-content:center;
}
.btn-wa-big:hover{background:#1ebe5d;transform:translateY(-2px);box-shadow:0 8px 28px rgba(37,211,102,.35)}
.cta-email-toggle{
  font-size:.82rem;color:rgba(242,232,220,.4);text-decoration:underline;
  background:none;border:none;cursor:pointer;padding:0;
  transition:color var(--ease);font-family:var(--sans);
}
.cta-email-toggle:hover{color:rgba(242,232,220,.75)}
.email-form-wrap{width:100%}
```

- [ ] **Verificar:** botón de WhatsApp verde y grande en primer plano. El formulario está oculto. Al clicar "¿Prefieres el formulario?" aparece el form; al clicar de nuevo se oculta. Teléfono visible correcto (680 211 326).

- [ ] **Commit**
```bash
git add index.html app.js styles.css
git commit -m "feat: contacto WhatsApp-first con formulario plegable"
```

---

## Task 11: Botón flotante de WhatsApp

**Files:**
- Modificar: `index.html` (añadir elemento flotante antes de `</body>`)
- Modificar: `styles.css` (estilos)
- Modificar: `app.js` (lógica de aparición + Pixel)

- [ ] **Añadir en index.html**, justo antes de `<script src="app.js"></script>`:
```html
<!-- Botón flotante WhatsApp -->
<a id="waFloat"
   href="https://wa.me/34680211326?text=Hola%2C+vengo+de+tu+web"
   class="wa-float"
   aria-label="Contactar por WhatsApp"
   onclick="trackWA('float')">
  <svg viewBox="0 0 24 24" width="28" height="28" fill="currentColor"><path d="M17.472 14.382c-.297-.149-1.758-.867-2.03-.967-.273-.099-.471-.148-.67.15-.197.297-.767.966-.94 1.164-.173.199-.347.223-.644.075-.297-.15-1.255-.463-2.39-1.475-.883-.788-1.48-1.761-1.653-2.059-.173-.297-.018-.458.13-.606.134-.133.298-.347.446-.52.149-.174.198-.298.298-.497.099-.198.05-.371-.025-.52-.075-.149-.669-1.612-.916-2.207-.242-.579-.487-.5-.669-.51-.173-.008-.371-.01-.57-.01-.198 0-.52.074-.792.372-.272.297-1.04 1.016-1.04 2.479 0 1.462 1.065 2.875 1.213 3.074.149.198 2.096 3.2 5.077 4.487.709.306 1.262.489 1.694.625.712.227 1.36.195 1.871.118.571-.085 1.758-.719 2.006-1.413.248-.694.248-1.289.173-1.413-.074-.124-.272-.198-.57-.347z"/><path d="M12 0C5.373 0 0 5.373 0 12c0 2.127.558 4.126 1.532 5.856L.057 23.625a.5.5 0 00.624.606l5.865-1.538A11.95 11.95 0 0012 24c6.627 0 12-5.373 12-12S18.627 0 12 0zm0 21.75a9.713 9.713 0 01-4.953-1.352l-.355-.211-3.68.965.984-3.595-.232-.37A9.712 9.712 0 012.25 12C2.25 6.615 6.615 2.25 12 2.25S21.75 6.615 21.75 12 17.385 21.75 12 21.75z"/></svg>
</a>
```

- [ ] **Añadir en styles.css**:
```css
/* ─── WhatsApp flotante ─── */
.wa-float{
  position:fixed;bottom:28px;right:28px;z-index:200;
  width:58px;height:58px;border-radius:50%;background:#25D366;
  display:flex;align-items:center;justify-content:center;color:#fff;
  box-shadow:0 4px 20px rgba(37,211,102,.45);
  opacity:0;transform:scale(.7);pointer-events:none;
  transition:opacity .3s ease,transform .3s ease,background .2s ease;
}
.wa-float.show{opacity:1;transform:scale(1);pointer-events:auto}
.wa-float:hover{background:#1ebe5d;box-shadow:0 6px 28px rgba(37,211,102,.55)}
@media(max-width:480px){.wa-float{bottom:16px;right:16px;width:52px;height:52px}}
```

- [ ] **Añadir en app.js** (dentro o junto al listener de scroll del header):
```js
/* ─── Botón flotante WhatsApp ─── */
const waFloat = document.getElementById('waFloat');
window.addEventListener('scroll', () => {
  waFloat.classList.toggle('show', scrollY > 300);
}, {passive: true});
```

- [ ] **Verificar:** el botón no aparece en el hero. Aparece al bajar ~300px. Desaparece al volver arriba. En móvil está en la esquina sin tapar el contenido.

- [ ] **Commit**
```bash
git add index.html app.js styles.css
git commit -m "feat: botón flotante de WhatsApp"
```

---

## Task 12: Footer actualizado + privacidad.html

**Files:**
- Modificar: `index.html` (footer)
- Crear: `privacidad.html`
- Modificar: `app.js` (clave i18n footer)

- [ ] **Actualizar el footer en index.html** — añadir enlaces de secciones y privacidad:
```html
<footer>
  <div class="wrap">
    <div class="ft-bot">
      <a href="#" class="ft-logo logo-wrap" aria-label="Como la Seda">
        como la <span class="seda-inner">seda<svg class="seda-wave" viewBox="0 0 56 8" fill="none" preserveAspectRatio="none"><path d="M0,4 Q7,0 14,4 Q21,8 28,4 Q35,0 42,4 Q49,8 56,4" stroke="#C2A557" stroke-width="1.9" fill="none"/></svg></span>
      </a>
      <p class="ft-copy" data-i18n="ft.rights">© 2026 Como la Seda. Todos los derechos reservados.</p>
      <div class="ft-links">
        <a href="#regalos" data-i18n="nav.regalos">Regalos</a>
        <a href="#suministros" data-i18n="nav.suministros">Suministros</a>
        <a href="#revende" data-i18n="nav.revende">Revende</a>
        <a href="mailto:comolaseda@wisdowl.com">comolaseda@wisdowl.com</a>
        <a href="privacidad.html" data-i18n="ft.privacy">Política de privacidad</a>
      </div>
    </div>
  </div>
</footer>
```

- [ ] **Añadir en app.js** (`T.es` y `T.en`):
```js
// ES
'ft.privacy': 'Política de privacidad',
// EN
'ft.privacy': 'Privacy policy',
```

- [ ] **Crear `privacidad.html`** con el siguiente contenido (los `[PLACEHOLDER]` los rellena Alex antes de publicar):
```html
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Política de privacidad — Como la Seda</title>
<link rel="icon" type="image/png" href="favicon.png">
<link rel="stylesheet" href="styles.css">
<style>
.priv-wrap{max-width:780px;margin:0 auto;padding:calc(var(--hh) + 60px) 24px 80px}
.priv-wrap h1{font-family:var(--serif);font-size:2.2rem;margin-bottom:32px}
.priv-wrap h2{font-family:var(--serif);font-size:1.3rem;margin:32px 0 10px}
.priv-wrap p,.priv-wrap li{font-size:.95rem;color:rgba(27,27,27,.75);line-height:1.8;margin-bottom:10px}
.priv-wrap ul{padding-left:20px;list-style:disc}
.priv-back{display:inline-flex;align-items:center;gap:8px;margin-bottom:40px;font-size:.85rem;color:var(--gold)}
</style>
</head>
<body>

<header id="hdr">
  <div class="wrap">
    <a href="index.html" class="hdr-logo logo-wrap" aria-label="Como la Seda — volver a inicio">
      como la <span class="seda-inner">seda<svg class="seda-wave" viewBox="0 0 56 8" fill="none" preserveAspectRatio="none"><path d="M0,4 Q7,0 14,4 Q21,8 28,4 Q35,0 42,4 Q49,8 56,4" stroke="#C2A557" stroke-width="1.9" fill="none"/></svg></span>
    </a>
  </div>
</header>

<div class="priv-wrap">
  <a href="index.html" class="priv-back">← Volver a inicio</a>
  <h1>Política de privacidad</h1>

  <p><strong>Responsable del tratamiento:</strong> [NOMBRE COMPLETO], con NIF [NIF], con domicilio en [DIRECCIÓN], correo electrónico: comolaseda@wisdowl.com.</p>

  <h2>¿Qué datos recogemos?</h2>
  <p>A través del formulario de contacto de esta web recogemos: nombre, nombre de empresa, dirección de correo electrónico, teléfono (opcional) y el mensaje que nos envías.</p>

  <h2>¿Para qué los usamos?</h2>
  <ul>
    <li>Responder a tu solicitud de información o presupuesto.</li>
    <li>Gestionar la relación comercial si decides contratarnos.</li>
  </ul>

  <h2>¿Cuánto tiempo los guardamos?</h2>
  <p>Conservamos tus datos durante el tiempo necesario para atender tu solicitud y, en caso de relación comercial, durante el plazo legalmente exigido (mínimo 5 años para documentación mercantil).</p>

  <h2>¿Los cedemos a terceros?</h2>
  <p>Los datos del formulario se transmiten a <strong>Web3Forms</strong> (api.web3forms.com), servicio de envío de formularios, exclusivamente para hacernos llegar tu solicitud. No los vendemos ni cedemos a terceros para fines publicitarios.</p>
  <p>Esta web usa el <strong>Meta Pixel</strong> de Meta Platforms Ireland Ltd. para medir la efectividad de nuestros anuncios en Facebook e Instagram. Meta puede usar esta información conforme a su <a href="https://www.facebook.com/privacy/policy/" target="_blank" rel="noopener">política de privacidad</a>. Puedes gestionar tus preferencias publicitarias en <a href="https://www.facebook.com/ads/preferences" target="_blank" rel="noopener">facebook.com/ads/preferences</a>.</p>

  <h2>Tus derechos</h2>
  <p>Puedes ejercer tus derechos de acceso, rectificación, supresión, oposición y portabilidad escribiéndonos a comolaseda@wisdowl.com. Tienes también derecho a reclamar ante la <a href="https://www.aepd.es" target="_blank" rel="noopener">Agencia Española de Protección de Datos</a>.</p>

  <p style="margin-top:48px;font-size:.8rem;color:rgba(27,27,27,.4)">Última actualización: junio 2026</p>
</div>

<script>
const hdr = document.getElementById('hdr');
window.addEventListener('scroll', () => hdr.classList.toggle('on', scrollY > 50), {passive:true});
</script>
</body>
</html>
```

- [ ] **Verificar:** el enlace "Política de privacidad" del footer abre `privacidad.html`, el header de privacidad tiene el scroll oscuro, el "← Volver a inicio" funciona.

- [ ] **Commit**
```bash
git add index.html privacidad.html app.js
git commit -m "feat: footer actualizado y privacidad.html"
```

---

## Task 13: Ampliar i18n — revisar textos EN

**Files:**
- Modificar: `app.js` (completar `T.en` con todas las claves nuevas)

- [ ] **Abrir app.js y asegurarse de que `T.en` tiene TODAS las claves** que `T.es` tiene. Buscar cualquier clave nueva añadida en los tasks anteriores que solo esté en `T.es`:
  - `regalos.label`, `regalos.title`
  - `sumi.*` (8 claves)
  - `rev.*` (7 claves)
  - `who.*` (5 claves)
  - `cta.wa.btn`, `cta.email.toggle`
  - `nav.regalos`, `nav.suministros`, `nav.revende`
  - `ft.privacy`

- [ ] **Cambiar idioma a EN en el navegador** y revisar visualmente que no queda ningún texto en español (si algo no traduce, la clave falta en `T.en`).

- [ ] **Commit**
```bash
git add app.js
git commit -m "i18n: completar traducciones EN para todas las secciones nuevas"
```

---

## Task 14: Revisión responsive completa

No hay CSS nuevo que añadir — este task es solo verificación. Si algo falla visualmente, el fix va en `styles.css`.

- [ ] **Abrir DevTools (F12) → modo responsive** y probar en:
  - 375px (iPhone SE) — el más estrecho
  - 768px (iPad)
  - 1280px (escritorio estándar)

- [ ] **Checklist por sección:**
  - Header: en <900px desaparece el nav y queda solo logo + idioma + botón (el botón WhatsApp del header puede ocultarse en móvil como el antiguo `.hdr-cta`)
  - Hero: texto legible, botón no desborda
  - `#regalos`: tarjetas apiladas en columna en móvil ✓ (hereda el responsive actual)
  - `#suministros`: los 3 pasos apilados en móvil ✓ (ya tiene `@media`)
  - `#revende`: las 3 tarjetas apiladas ✓ (ya tiene `@media`)
  - `#quien`: foto y texto en columna centrada ✓ (ya tiene `@media`)
  - `#cta`: columna en móvil, botón WhatsApp ocupa el ancho ✓
  - Botón flotante: no tapa el botón WhatsApp del formulario en móvil
  - Footer: enlaces en columna

- [ ] Si hay fixes de CSS, aplicarlos en `styles.css` y **commit**:
```bash
git add styles.css
git commit -m "fix: ajustes responsive secciones nuevas"
```

---

## Task 15: Verificación Pixel + deploy

**Files:** ninguno (solo verificación y push)

- [ ] **Instalar Meta Pixel Helper** (extensión Chrome/Firefox) si no está instalada.

- [ ] **Con el servidor local activo (`python -m http.server 8000`)**, abrir Pixel Helper y verificar:
  - `PageView` se dispara al cargar
  - Clicar el botón WhatsApp del hero → `Contact` con `content_category: general`
  - Clicar CTA de `#regalos` → `Contact` con `content_category: regalos`
  - Clicar CTA de `#suministros` → `Contact` con `content_category: suministros`
  - Clicar CTA de `#revende` → `Contact` con `content_category: revende`
  - Enviar formulario → `Lead`

- [ ] **Revisar que `docs/` está tratada** (Task 0): si se eligió Opción B (ignorar), verificar que no hay datos sensibles en los archivos antes de pushear.

- [ ] **Push a GitHub:**
```bash
git push origin master
```

- [ ] **Esperar ≤5 min** (el cron de la Orange Pi hace `git pull` cada 5 min) y abrir `https://comolaseda.wisdowl.com` para verificar:
  - La web carga correctamente (CSS y JS externos se sirven bien)
  - Las tres secciones son visibles y los CTAs funcionan
  - `privacidad.html` accesible desde el footer

---

## Self-review

**Spec coverage:**
- ✅ WhatsApp protagonista — Tasks 4, 10, 11
- ✅ Catálogo teaser (texto) en #regalos — Task 6 (se hereda la grid de imágenes con texto concreto en i18n)
- ✅ CTA catálogo por WhatsApp — Task 6
- ✅ Mensaje de cantidades — Task 6 (en la tarjeta de muestras)
- ✅ Sección #suministros — Task 7
- ✅ Sección #revende con gancho "el margen del distribuidor" — Task 8
- ✅ Bloque "Quién está detrás" — Task 9
- ✅ Formulario secundario/plegable — Task 10
- ✅ Botón flotante — Task 11
- ✅ Footer + privacidad.html — Task 12
- ✅ i18n ES/EN — Task 13
- ✅ Pixel Contact por sección + Lead en form — Tasks 4 y 15
- ✅ Teléfono 680 211 326 — Task 3
- ✅ styles.css + app.js compartidos — Tasks 1 y 2

**Inconsistencias resueltas:**
- `trackWA` se define en Task 4 (app.js) y se usa desde Task 4 en adelante — consistente.
- `toggleForm` se define en Task 10 — consistente con el `onclick` del mismo task.
- `data-i18n-h` se usa en `sumi.title` y `rev.title` (contienen `<br>`) — consistente con el uso existente en `hero.h1`.
